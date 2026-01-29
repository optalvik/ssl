# Osa 9 - Automatiseerimise tööriistad

## Käsitsi haldamine ei skaleeru

Kui sul on üks veebileht, saad sertifikaati käsitsi uuendada. Kalendrisse reminder, kord aastas lood uue CSR-i, saad uue sertifikaadi, laed serverisse üles. Tüütu, aga hallatav.

Kui sul on kümme serverit, hakkab asi keerukaks minema. Iga serveri jaoks eraldi reminder, eraldi protseduur, eraldi dokumentatsioon. Keegi unustab ühe.

Kui sul on sada serverit, on käsitsi haldamine tee hukatusse. Mitte küsimus, kas keegi unustab, vaid millal ja kui palju.

Lahendus: automatiseeri. Las masinad teevad seda, milles nad head on — korduvaid ülesandeid täpselt ja väsimatult.

## Certbot ja ACME protokoll

ACME (Automatic Certificate Management Environment) on protokoll, mis võimaldab automaatset sertifikaatide taotlemist ja uuendamist. Let's Encrypt tegi selle kuulsaks, aga nüüd toetavad seda ka teised CA-d.

Certbot on kõige levinum ACME klient. Ta räägib Let's Encrypt'iga (või teise ACME-toega CA-ga), tõestab domeeni omandiõigust, saab sertifikaadi ja installib selle.

Domeeni omandiõigust saab tõestada mitut moodi. HTTP-01 challenge paneb faili veebiserverisse ja CA kontrollib, kas see on seal. DNS-01 challenge lisab DNS TXT kirje ja CA kontrollib seda. DNS-01 on vajalik wildcard sertifikaatide jaoks.

```bash
# Installeeri certbot
apt install certbot

# Hangi sertifikaat (nginx plugin)
certbot --nginx -d minusait.ee -d www.minusait.ee

# Hangi sertifikaat (standalone, kui veebiserverit pole)
certbot certonly --standalone -d minusait.ee

# Uuenda kõiki sertifikaate
certbot renew

# Testi uuendamist ilma tegelikult uuendamata
certbot renew --dry-run
```

Pärast esmast seadistust lisab certbot automaatselt croni või systemd timeri, mis käivitab uuendamist kaks korda päevas. Uuendamine toimub ainult siis, kui sertifikaat aegub varsti.

## HashiCorp Vault

Vault on salahalduse tööriist, mis oskab ka sertifikaate väljastada. See on ideaalne sisemiste sertifikaatide jaoks.

Vault PKI secrets engine töötab nii: sa seadistad Vaulti kui CA (või impordid olemasoleva CA), lood rolle, mis määravad, milliste omadustega sertifikaate saab väljastada, ja siis rakendused küsivad Vaultilt sertifikaate API kaudu.

```bash
# Luba PKI secrets engine
vault secrets enable pki

# Genereeri juur-CA
vault write pki/root/generate/internal \
    common_name="Minu CA" \
    ttl=87600h

# Loo roll serverite sertifikaatide jaoks
vault write pki/roles/serverid \
    allowed_domains="sisemine.ee" \
    allow_subdomains=true \
    max_ttl=720h

# Küsi sertifikaati
vault write pki/issue/serverid \
    common_name="server1.sisemine.ee" \
    ttl=24h
```

Vaulti võlu on selles, et sertifikaadid võivad olla lühiajalised — tunde, mitte päevi. Rakendus küsib iga päev (või iga tund) uue sertifikaadi. Kui võti lekib, on see mõne tunni pärast nagunii kasutu.

## Cert-manager Kubernetesis

Kui jooksutad rakendusi Kubernetesis, on cert-manager standard sertifikaatide haldamiseks. Ta jälgib Certificate ressursse ja hoolitseb selle eest, et vastavad sertifikaadid oleksid olemas ja kehtivad.

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: minusait-tls
spec:
  secretName: minusait-tls-secret
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
  - minusait.ee
  - www.minusait.ee
```

Cert-manager toetab mitmeid CA-sid: Let's Encrypt (ACME), HashiCorp Vault, sisemised CA-d, ise-allkirjastatud sertifikaate. Sa määrad Issueri või ClusterIssueri, mis ütleb, kust sertifikaate saada, ja cert-manager teeb ülejäänu.

Sertifikaadid salvestatakse Kubernetese Secretitesse. Podid saavad neid mountida. Kui sertifikaat uueneb, uueneb Secret automaatselt.

## Smallstep

Smallstep on modernne lahendus sisemise CA jaoks. See on lihtsam kui täieõiguslik PKI, aga võimekam kui lihtsalt OpenSSL skriptid.

Step CA on server, mis väljastab sertifikaate. Step CLI on klient, millega sertifikaate küsida. Toetab ACME protokolli, nii et saad kasutada ka Certboti.

```bash
# Initsialiseeri CA
step ca init

# Käivita CA server
step-ca $(step path)/config/ca.json

# Küsi sertifikaati
step ca certificate server.sisemine.ee server.crt server.key
```

Smallstepi tugevus on lihtsus. Saad sisemise CA üles mõne minutiga, ilma et peaksid PKI ekspert olema. Toetab lühiajalisi sertifikaate, automaatset uuendamist, SSO integreerimist.

## Ansible, Puppet, Chef

Konfiguratsioonihalduse tööriistad saavad samuti sertifikaate hallata. Nad ei genereeri sertifikaate ise, aga nad saavad:

- Installida CA sertifikaate truststoredesse
- Kopeerida sertifikaate õigetesse kohtadesse
- Seadistada rakendusi sertifikaate kasutama
- Käivitada uuendamisskripte

```yaml
# Ansible näide: installi CA sertifikaat
- name: Kopeeri CA sertifikaat
  copy:
    src: files/ca.crt
    dest: /usr/local/share/ca-certificates/minu-ca.crt
  
- name: Uuenda CA nimekiri
  command: update-ca-certificates
```

See lähenemine töötab hästi, kui sertifikaadid tulevad kusagilt mujalt (Vault, cert-manager) ja sa pead lihtsalt tagama, et nad jõuavad õigetesse kohtadesse.

## Monitooring ja alerting

Ükski automatiseerimine pole täiuslik. Seepärast tuleb monitoorida.

Prometheus ssl_exporter skaneerib endpointe ja kogub sertifikaatide metrikat: aegumiseni jäänud aeg, väljaandja, domeenid. Sealt saab teha Grafana dashboarde ja alertingut.

```yaml
# Prometheus alert näide
- alert: SertifikaatAegubVarsti
  expr: ssl_cert_not_after - time() < 86400 * 14
  for: 1h
  labels:
    severity: warning
  annotations:
    summary: "Sertifikaat aegub 14 päeva jooksul"
```

Lihtne lähenemine: skript, mis käib läbi kõik serverid ja kontrollib sertifikaate. Saab jooksutada croniga ja saata e-kirju, kui midagi on valesti.

## Vali õige tööriist

Valik sõltub sinu olukorrast:

Avalikud veebilehed: Certbot + Let's Encrypt. Tasuta, töökindel, lihtne.

Kubernetese keskkonnad: cert-manager. Integreeritub hästi, toetab mitmeid CA-sid.

Sisemised teenused pilvekeskkonnas: HashiCorp Vault. Võimekas, turvaline, integreeritub paljuga.

Sisemised teenused lihtsamates keskkondades: Smallstep. Kiire alustada, piisavalt võimekas.

Eksisteerivad süsteemid, kuhu automatiseerimist lisada: Ansible/Puppet + olemasolevad tööriistad. Integreeri sellega, mis juba olemas on.

Järgmises osas vaatame, kuidas TLS probleeme diagnoosida, kui midagi ei tööta.
