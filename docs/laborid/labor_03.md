# Labor 3 - HashiCorp Vault PKI seadistamine Dockeriga

## Mida me siin teeme?

Selles laboris ehitame üles päris sertifitseerimisasutuse kasutades HashiCorp Vault'i. See pole mingi mänguasi — Vault on tööriist, mida kasutavad suured ettevõtted oma sisemiste sertifikaatide haldamiseks. Aga me teeme seda Dockeriga, nii et sa ei pea midagi püsivalt oma arvutisse installima.

Labori lõpuks oskad sa:
- Käivitada Vault'i Dockeris
- Seadistada Vault'i sertifikaate väljastama
- Luua ja kasutada sertifikaate päris HTTPS serveri jaoks
- Mõista, miks automaatne sertifikaadihaldus on parem kui käsitsi tegemine

## Miks Vault, mitte lihtsalt OpenSSL?

Eelmistes laborites tegime sertifikaate OpenSSL-iga käsitsi. See töötab, aga kujuta ette, et sul on sada serverit. Iga kord, kui sertifikaat aegub, pead sa käsitsi uue tegema, selle õigesse kohta kopeerima, serveri taaskäivitama. Üks unustatud sertifikaat ja teenus on maas.

Vault lahendab selle probleemi. Sa ütled Vault'ile "anna mulle sertifikaat serveri X jaoks" ja Vault annab. Automaatselt, turvaliselt, logitult. Kui sertifikaat aegub, küsid lihtsalt uue. Pole faile, mida kaduma kaotada, pole käsitsi kopeerimist.

See on nagu vahe selle vahel, kas sa käid iga kord passibüroos passi uuendamas või on sul automaatne süsteem, mis saadab uue passi koju enne kui vana aegub.

---

## Osa 1: Vault'i käivitamine

### Eeldused

Sul peab olema Docker installitud. Kontrolli:

```bash
docker --version
```

Kui Docker puudub, installi see esmalt oma operatsioonisüsteemi juhiste järgi.

### Docker Compose fail

Loo endale töökataloog ja sinna fail nimega `docker-compose.yml`:

```bash
mkdir -p ~/vault-labor
cd ~/vault-labor
```

Loo fail `docker-compose.yml` järgmise sisuga:

```yaml
version: '3.8'

services:
  vault:
    image: hashicorp/vault:latest
    container_name: vault
    ports:
      - "8200:8200"
    environment:
      VAULT_DEV_ROOT_TOKEN_ID: root
      VAULT_DEV_LISTEN_ADDRESS: 0.0.0.0:8200
    cap_add:
      - IPC_LOCK
    command: "server -dev"
```

See fail ütleb Dockerile: käivita Vault arendusrežiimis, kuula pordil 8200, kasuta "root" kui autentimise tokenit. Arendusrežiim tähendab, et Vault käivitub kohe ilma keerulise seadistamiseta — päris tootmises sa seda ei kasutaks, aga õppimiseks on ideaalne.

### Käivitamine

```bash
docker-compose up -d
```

`-d` tähendab "detached" — Vault jookseb taustal ja sa saad terminali edasi kasutada.

Kontrolli, kas töötab:

```bash
docker ps
```

Peaksid nägema vault konteinerit jooksmas.

---

## Osa 2: Vault CLI seadistamine

Vault'iga suhtlemiseks vajad käsurea tööriista. Selle saad installida:

Ubuntu/Debian:
```bash
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vault
```

macOS:
```bash
brew install vault
```

Seadista keskkond:

```bash
export VAULT_ADDR='http://127.0.0.1:8200'
export VAULT_TOKEN='root'
```

Need käsud ütlevad Vault CLI-le, kuhu ühenduda ja millise tokeniga autentida.

Kontrolli ühendust:

```bash
vault status
```

Kui näed infot Vault'i oleku kohta, oled ühendatud.

---

## Osa 3: PKI mootori seadistamine

Vault on modulaarne — ta koosneb "mootoritest", mis teevad erinevaid asju. PKI mootor tegeleb sertifikaatidega.

### PKI lubamine

```bash
vault secrets enable pki
```

See käsk ütleb Vault'ile: luba PKI mootor vaikimisi teel (`pki/`).

### Juur-CA loomine

Nüüd loome oma sertifitseerimisasutuse. See on see "valitsus", kes hakkab passe väljastama:

```bash
vault write pki/root/generate/internal \
    common_name="Minu Labor CA" \
    ttl=87600h
```

`ttl=87600h` on 10 aastat — nii kaua kehtib meie juur-CA. Päris elus oleks see võib-olla veel pikem.

Vault vastab sulle juur-CA sertifikaadiga. See on see dokument, mida teised peavad usaldama, et usaldada kõiki selle CA poolt väljastatud sertifikaate.

### URL-ide seadistamine

Sertifikaatidesse kirjutatakse URL-id, kust saab kontrollida, kas sertifikaat on tühistatud:

```bash
vault write pki/config/urls \
    issuing_certificates="$VAULT_ADDR/v1/pki/ca" \
    crl_distribution_points="$VAULT_ADDR/v1/pki/crl"
```

---

## Osa 4: Rolli loomine

Vault'is määravad "rollid", kes mida teha tohib. Me loome rolli, mis lubab väljastada sertifikaate domeenile `labor.local` ja selle alamdomeenidele:

```bash
vault write pki/roles/veebiserver \
    allowed_domains="labor.local" \
    allow_subdomains=true \
    max_ttl="72h"
```

See ütleb: roll nimega "veebiserver" tohib väljastada sertifikaate `labor.local`, `www.labor.local`, `api.labor.local` jne jaoks. Sertifikaadid kehtivad maksimaalselt 72 tundi.

Miks nii lühike aeg? Sest lühiajalised sertifikaadid on turvalisemad. Kui võti lekib, on kahju piiratud. Ja kui sul on automaatne süsteem, mis uuendab sertifikaate, pole pikk kehtivusaeg vaja.

---

## Osa 5: Sertifikaadi väljastamine

Nüüd küsime Vault'ilt sertifikaadi:

```bash
vault write pki/issue/veebiserver \
    common_name="test.labor.local" \
    ttl="24h"
```

Vault vastab JSON-iga, mis sisaldab:
- `certificate` — sinu sertifikaat
- `private_key` — sinu privaatvõti
- `ca_chain` — CA sertifikaat

### Failidesse salvestamine

Et sertifikaate kasutada, salvesta need failidesse:

```bash
vault write -format=json pki/issue/veebiserver \
    common_name="test.labor.local" \
    ttl="24h" > sertifikaat.json

cat sertifikaat.json | jq -r '.data.private_key' > server.key
cat sertifikaat.json | jq -r '.data.certificate' > server.crt
cat sertifikaat.json | jq -r '.data.ca_chain[0]' > ca.crt
```

Nüüd on sul kolm faili: privaatvõti, sertifikaat ja CA sertifikaat.

---

## Osa 6: HTTPS serveri testimine

Loome lihtsa veebilehe ja käivitame HTTPS serveri:

```bash
echo "<h1>Tere tulemast turvalisele lehele!</h1><p>See sertifikaat tuli Vault'ist.</p>" > index.html
```

Käivita server (vajab Python 3):

```bash
python3 << 'EOF'
import http.server
import ssl

server = http.server.HTTPServer(('0.0.0.0', 4443), http.server.SimpleHTTPRequestHandler)
context = ssl.SSLContext(ssl.PROTOCOL_TLS_SERVER)
context.load_cert_chain('server.crt', 'server.key')
server.socket = context.wrap_socket(server.socket, server_side=True)
print("Server käivitatud: https://localhost:4443")
server.serve_forever()
EOF
```

### Testimine

Teises terminalis:

```bash
curl --cacert ca.crt https://localhost:4443
```

`--cacert ca.crt` ütleb curl'ile, et ta usaldaks meie CA-d. Ilma selleta saaksid hoiatuse, sest meie CA pole avalikult usaldatud.

Kui näed HTML vastust, töötab kõik!

---

## Osa 7: Vahe-CA loomine (tootmise parim praktika)

Päris tootmises ei kasutata juur-CA-d otse sertifikaatide väljastamiseks. Selle asemel luuakse vahe-CA (intermediate CA). Kui vahe-CA kompromiteeritakse, saab selle tühistada ilma juur-CA-d puutumata.

### Vahe-PKI lubamine

```bash
vault secrets enable -path=pki_int pki
vault secrets tune -max-lease-ttl=43800h pki_int
```

### Vahe-CA taotluse loomine

```bash
vault write -format=json pki_int/intermediate/generate/internal \
    common_name="Minu Labor Vahe-CA" \
    | jq -r '.data.csr' > vahe_ca.csr
```

### Juur-CA allkirjastab vahe-CA

```bash
vault write -format=json pki/root/sign-intermediate \
    csr=@vahe_ca.csr \
    format=pem_bundle \
    ttl="43800h" \
    | jq -r '.data.certificate' > vahe_ca.crt
```

### Vahe-CA sertifikaadi importimine

```bash
vault write pki_int/intermediate/set-signed certificate=@vahe_ca.crt
```

Nüüd on sul kaheastmeline CA hierarhia. Sertifikaate väljastaksid `pki_int` teelt, mitte `pki` teelt.

---

## Osa 8: Automaatne uuendamine

Vault'i võlu on automaatne uuendamine. Siin on näide skriptist, mis uuendab sertifikaati:

```bash
#!/bin/bash
# uuenda_sertifikaat.sh

export VAULT_ADDR='http://127.0.0.1:8200'
export VAULT_TOKEN='root'

vault write -format=json pki/issue/veebiserver \
    common_name="test.labor.local" \
    ttl="24h" > sertifikaat.json

cat sertifikaat.json | jq -r '.data.private_key' > server.key
cat sertifikaat.json | jq -r '.data.certificate' > server.crt

echo "Sertifikaat uuendatud: $(date)"

# Siin võiks olla käsk teenuse taaskäivitamiseks
# systemctl reload nginx
```

Selle skripti võid lisada cron'i, et see jookseks iga päev:

```bash
0 3 * * * /home/kasutaja/vault-labor/uuenda_sertifikaat.sh
```

---

## Osa 9: Puhastamine

Kui labor on läbi:

```bash
cd ~/vault-labor
docker-compose down
rm -f server.key server.crt ca.crt sertifikaat.json index.html
rm -f vahe_ca.csr vahe_ca.crt
```

---

## Kokkuvõte

Selles laboris õppisid:

| Teema | Mida tegid |
|-------|------------|
| Vault käivitamine | Docker Compose'iga arendusrežiimis |
| PKI seadistamine | Juur-CA ja rollide loomine |
| Sertifikaadi väljastamine | API kaudu automaatselt |
| HTTPS server | Vault'i sertifikaadiga töötav server |
| Vahe-CA | Tootmise parim praktika |
| Automaatne uuendamine | Skript sertifikaatide uuendamiseks |

Vault vs käsitsi OpenSSL:

| Aspekt | OpenSSL käsitsi | Vault |
|--------|-----------------|-------|
| Sertifikaadi loomine | Mitu käsku, faile | Üks API kutse |
| Võtmete hoidmine | Failid kettal | Vault haldab |
| Uuendamine | Käsitsi mäletamine | Automaatne |
| Audit | Puudub | Sisseehitatud |
| Ligipääsukontroll | Failiõigused | Poliitikad |

See on põhjus, miks suured organisatsioonid kasutavad Vault'i — mitte sellepärast, et OpenSSL ei tööta, vaid sellepärast, et sajad sertifikaadid sajal serveril vajavad automatiseerimist.
