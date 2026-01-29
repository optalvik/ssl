# Osa 11 - Boonusteemad

## Minna sügavamale

Siiani oleme käsitlenud TLS põhitõdesid — kuidas sertifikaadid töötavad, kuidas neid hallata, kuidas probleeme lahendada. Aga maailm on laiem. Siin on mõned täiendavad teemad, mis võivad kasuks tulla.

## Mutual TLS (mTLS)

Tavalises TLS-is autentib end ainult server. Server näitab sertifikaati, klient kontrollib, ühendus luuakse. Aga klient jääb anonüümseks — server ei tea, kes ta on.

mTLS ehk vastastikune TLS keerab selle ümber: ka klient peab ennast tõestama. Mõlemal poolel on sertifikaat, mõlemad kontrollivad teineteist.

See on kasulik mitmes olukorras. Microservice'ide vaheline suhtlus: iga teenus autentib end teisele, nii et võõrad ei saa vahelesegamiseks. Zero-trust võrgud: ära usalda kedagi automaatselt, nõua kõigilt tõestust. API-d, kus turvalisus on kriitiline: pangad, tervishoid, valitsusasutused.

Konfigureerimine tähendab seda, et serveril pead määrama CA, mille sertifikaate klientidelt aktsepteerid:

```nginx
# Nginx mTLS
ssl_client_certificate /etc/ssl/ca.crt;
ssl_verify_client on;
```

Kliendil pead määrama sertifikaadi, mida kasutada:

```bash
# curl koos kliendi sertifikaadiga
curl --cert klient.crt --key klient.key https://server/
```

mTLS komplitseerib sertifikaatide haldust — sertifikaate on rohkem, kõik peavad olema kehtivad. Aga turvalisus on märkimisväärselt parem.

## HTTP Strict Transport Security (HSTS)

Oled ehk märganud, et mõnikord saad veebisaidile ka ilma https:// sisestamata. Brauser lihtsalt suunab sind HTTPS-ile. Kuidas ta teab, et peab seda tegema?

Üks võimalus on HSTS. Server saadab päise:

```
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

See ütleb brauserile: järgmise aasta jooksul ühenda selle saidiga alati ainult HTTPS-iga. Ära isegi ürita HTTP-d.

See kaitseb downgrade rünnakute eest. Ründaja ei saa sind HTTP-le suunata ja siis liiklust pealt kuulata, sest brauser keeldub HTTP-d kasutamast.

Veel parem: HSTS preload. Saad oma domeeni lisada brauseritesse sisse ehitatud nimekirja. Siis ei pea brauser isegi korra HTTP-ga ühenduma, et HSTS päist saada — ta teab juba ette.

## Certificate Transparency

Mis siis, kui CA väljastab sertifikaadi ilma sinu loata? See on juhtunud. CA-d on häkitud, CA töötajad on altkäemaksu võtnud, valitsused on sundinud CA-sid vale sertifikaate väljastama.

Certificate Transparency (CT) on lahendus. Iga väljastatud sertifikaat logitakse avalikesse logidesse. Igaüks saab neid logisid kontrollida. Kui keegi väljastab sertifikaadi sinu domeenile, näed sa seda logist.

Brauserid nõuavad nüüd, et sertifikaadid oleksid CT logides. Sertifikaadiga tuleb kaasa SCT (Signed Certificate Timestamp), mis tõestab, et ta on logitud.

Sa saad monitoorida oma domeene:

```bash
# Otsi sertifikaate crt.sh-st
curl "https://crt.sh/?q=minusait.ee&output=json" | jq
```

Kui näed sertifikaati, mida sina ei taotlenud, on probleem. Teavita CA-d, uuri, reageeri.

## OCSP Stapling

Kuidas brauser teab, kas sertifikaat on tühistatud? Traditsiooniliselt küsib ta CA-lt OCSP vastust — "Kas see sertifikaat on veel kehtiv?"

Probleem: see lisab latentsust ja annab CA-le infot sinu surfamisharjumuste kohta.

OCSP stapling lahendab selle. Server ise küsib perioodiliselt OCSP vastuse ja "klammerdab" selle sertifikaadi külge. Brauser saab värske kinnituse otse serverilt, ilma CA-ga rääkimata.

```nginx
# Nginx OCSP stapling
ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8;
```

## TLS terminatsioon ja passthrough

Suurtes keskkondades on tihti load balancerid rakenduste ees. Küsimus: kus TLS lõppeb?

TLS terminatsioon load balanceris: kliendid ühenduvad turvalise ühendusega load balanceriga, load balancer dekrüpteerib ja edastab tavalise HTTP-ga backend serveritele. Lihtne seadistada, backend serverid ei pea TLS-i tegelema, aga liiklus load balanceri ja backendi vahel on krüpteerimata.

TLS passthrough: load balancer ei dekrüpteeri, ta lihtsalt suunab krüpteeritud liikluse backend serverile. Backend peab ise TLS-i tegema. Turvalisem, aga keerulisem.

Re-encryption: load balancer termineerib kliendi TLS-i, siis loob uue TLS-ühenduse backendiga. Parim mõlemast maailmast, aga kõige keerulisem.

## Wildcard vs SAN sertifikaadid

Kui sul on palju alamdomeene, on kaks võimalust.

Wildcard sertifikaat (*.minusait.ee) katab kõik ühetasemelised alamdomeenid: www.minusait.ee, api.minusait.ee, admin.minusait.ee. Ta ei kata mitmetasemelisi (test.api.minusait.ee) ega põhidomeeni (minusait.ee ilma alamdomeenita).

SAN (Subject Alternative Name) sertifikaat loetleb konkreetsed nimed: minusait.ee, www.minusait.ee, api.minusait.ee. Sa saad lisada ka teisi domeene (teineleht.ee).

Wildcard on mugavam, kui alamdomeene on palju ja nad muutuvad tihti. SAN on täpsem — sa tead täpselt, mis kaetud on. Mõned organisatsioonid eelistavad SAN-i turvakaalutlustel: kui wildcard sertifikaat lekib, katab see kõiki võimalikke alamdomeene, mitte ainult olemasolevaid.

## Oma CA loomine

Kui tahad sisemist CA-d, on OpenSSL-iga võimalik alustada:

```bash
# Genereeri CA võti
openssl genrsa -out ca.key 4096

# Loo CA sertifikaat
openssl req -x509 -new -nodes -key ca.key -sha256 -days 3650 \
    -out ca.crt -subj "/CN=Minu Sisemine CA"

# Genereeri serveri võti
openssl genrsa -out server.key 2048

# Loo CSR
openssl req -new -key server.key -out server.csr \
    -subj "/CN=server.sisemine.ee"

# Allkirjasta CA-ga
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key \
    -CAcreateserial -out server.crt -days 365 -sha256
```

See on alus. Päris CA jaoks tahad kahekihilist struktuuri (juur + vahe), automaatset väljastamist, tühistamist, logimist. Siin tulevad mängu Vault, Smallstep ja teised tööriistad.

## Sertifikaadi kinnitamine (pinning)

Mõnikord tahad usaldada ainult konkreetset sertifikaati, mitte kõiki, mida CA võiks väljastada. See on pinning.

Rakendus teab ette, milline sertifikaat (või milline avalik võti) peab olema. Kui server saadab midagi muud, keeldub rakendus ühendumast.

See kaitseb olukorra vastu, kus ründaja saab kätte vale sertifikaadi usaldatud CA-lt. Aga see teeb sertifikaatide uuendamise keeruliseks — pead rakendust uuendama enne sertifikaadi uuendamist.

Mobiilirakendustes on pinning levinud. Veebibrauserites on sellest pigem loobutud (liiga palju probleeme tekitas), asendatud Certificate Transparency-ga.

Järgmises osas vaatame tulevikku — kuidas kvantarvutid mõjutavad krüptograafiat ja mis on post-quantum cryptography.
