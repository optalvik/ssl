# Labor 1 - Veebiturve ja sertifikaadid praktikas

## Mida me teeme

Selles laboris lood sa oma sertifitseerimisasutuse, genereerid sertifikaate ja paned püsti turvalise veebiserveri. Sa näed oma silmaga, kuidas kogu see süsteem töötab — mitte teoorias, vaid päriselt.

Vajad Linuxi masinat (Ubuntu sobib hästi) ja OpenSSL-i, mis on tavaliselt juba installitud.

---

## Osa 1: Oma CA loomine

Kõigepealt loome kaustade struktuuri:

```bash
mkdir -p ~/tls-labor/ca ~/tls-labor/server
cd ~/tls-labor
```

Nüüd loome oma CA ehk sertifitseerimisasutuse. See on nagu oma väike "passikontor", mis hakkab väljastama sertifikaate.

```bash
# Genereeri CA privaatvõti
openssl genrsa -out ca/ca.key 4096
```

See lõi 4096-bitise RSA võtme. See number näitab võtme tugevust — mida suurem, seda raskem murda.

```bash
# Loo CA sertifikaat (ise-allkirjastatud)
openssl req -x509 -new -nodes -key ca/ca.key -sha256 -days 3650 \
    -out ca/ca.crt \
    -subj "/C=EE/O=Minu Labor/CN=Minu Labor CA"
```

Vaatame, mida need parameetrid tähendavad:
- `-x509` — loo sertifikaat (mitte CSR)
- `-new` — uus taotlus
- `-nodes` — ära krüpteeri privaatvõtit parooliga
- `-key` — kasuta seda privaatvõtit
- `-sha256` — kasuta SHA-256 räsialgoritmi
- `-days 3650` — kehtib 10 aastat
- `-subj` — sertifikaadi subjekt (kellele see kuulub)

Vaatame, mis me lõime:

```bash
openssl x509 -in ca/ca.crt -text -noout | head -20
```

Sa näed infot nagu väljastaja, kehtivusaeg, avalik võti. Pane tähele, et Issuer ja Subject on samad — see on ise-allkirjastatud sertifikaat.

---

## Osa 2: Serveri sertifikaadi loomine

Nüüd loome sertifikaadi meie veebiserveri jaoks. See on kaheastmeline protsess: kõigepealt CSR (taotlus), siis allkirjastamine.

```bash
# Genereeri serveri privaatvõti
openssl genrsa -out server/server.key 2048

# Loo CSR (sertifikaadi taotlus)
openssl req -new -key server/server.key -out server/server.csr \
    -subj "/C=EE/O=Minu Labor/CN=localhost"
```

CSR sisaldab avalikku võtit ja infot, mida tahame sertifikaadile. Vaatame seda:

```bash
openssl req -in server/server.csr -text -noout
```

Nüüd allkirjastame CSR-i oma CA-ga:

```bash
openssl x509 -req -in server/server.csr \
    -CA ca/ca.crt -CAkey ca/ca.key -CAcreateserial \
    -out server/server.crt -days 365 -sha256
```

Valmis! Meil on nüüd:
- `ca/ca.key` — CA privaatvõti (ÜLISALAJANE)
- `ca/ca.crt` — CA sertifikaat (seda levitame)
- `server/server.key` — serveri privaatvõti
- `server/server.crt` — serveri sertifikaat

Kontrollime, et sertifikaat on korras:

```bash
openssl verify -CAfile ca/ca.crt server/server.crt
```

Kui näed "OK", on kõik hästi.

---

## Osa 3: HTTPS serveri käivitamine

Pythoniga saab kiiresti HTTPS serveri püsti panna:

```bash
cd ~/tls-labor/server

# Loo lihtne veebileht
echo "<h1>Tere tulemast turvalisse serverisse!</h1>" > index.html

# Käivita HTTPS server
python3 << 'EOF'
import http.server
import ssl

server = http.server.HTTPServer(('localhost', 4443), 
    http.server.SimpleHTTPRequestHandler)

context = ssl.SSLContext(ssl.PROTOCOL_TLS_SERVER)
context.load_cert_chain('server.crt', 'server.key')

server.socket = context.wrap_socket(server.socket, server_side=True)

print("Server töötab aadressil https://localhost:4443")
server.serve_forever()
EOF
```

Ava teine terminal ja testi:

```bash
# Ilma CA-d usaldamata (näed viga)
curl https://localhost:4443

# CA-d usaldades (töötab!)
curl --cacert ~/tls-labor/ca/ca.crt https://localhost:4443
```

Esimene käsk annab vea, sest curl ei usalda meie CA-d. Teine töötab, sest me ütleme curl-ile, et usaldagu meie CA-d.

Proovi ka brauseris avada https://localhost:4443. Näed hoiatust, sest brauser ei usalda meie CA-d.

---

## Osa 4: CA usaldamine süsteemis

Kui tahad, et kogu süsteem usaldaks sinu CA-d:

```bash
# Ubuntu/Debian
sudo cp ~/tls-labor/ca/ca.crt /usr/local/share/ca-certificates/minu-labor.crt
sudo update-ca-certificates

# Kontrolli, kas töötab
curl https://localhost:4443
```

Nüüd töötab ilma `--cacert` parameetrita.

---

## Osa 5: Sertifikaadi uurimine

Õpi sertifikaate lugema:

```bash
# Näita kõik detailid
openssl x509 -in server/server.crt -text -noout

# Ainult subjekt ja väljastaja
openssl x509 -in server/server.crt -noout -subject -issuer

# Kehtivusajad
openssl x509 -in server/server.crt -noout -dates

# Avaliku võtme info
openssl x509 -in server/server.crt -noout -pubkey

# Sõrmejälg (fingerprint)
openssl x509 -in server/server.crt -noout -fingerprint -sha256
```

---

## Osa 6: Serverist sertifikaadi lugemine

Sa ei pea sertifikaadifaili omama, et seda näha. Saad seda otse serverist küsida:

```bash
# Ühenda serveriga ja näita sertifikaati
echo | openssl s_client -connect localhost:4443 2>/dev/null | \
    openssl x509 -text -noout

# Näita tervet sertifikaadikett
openssl s_client -connect localhost:4443 -showcerts

# Kontrolli, kas sertifikaat aegub varsti (30 päeva jooksul)
echo | openssl s_client -connect localhost:4443 2>/dev/null | \
    openssl x509 -noout -checkend 2592000
```

---

## Osa 7: Võtme leke ja taastumine

Simuleerime olukorda, kus võti lekib. Mida teha?

```bash
cd ~/tls-labor

# 1. Genereeri UUS võtmepaar
openssl genrsa -out server/server_uus.key 2048

# 2. Loo UUS CSR
openssl req -new -key server/server_uus.key -out server/server_uus.csr \
    -subj "/C=EE/O=Minu Labor/CN=localhost"

# 3. Allkirjasta CA-ga
openssl x509 -req -in server/server_uus.csr \
    -CA ca/ca.crt -CAkey ca/ca.key -CAcreateserial \
    -out server/server_uus.crt -days 365 -sha256

# 4. Asenda vanad failid
mv server/server_uus.key server/server.key
mv server/server_uus.crt server/server.crt

# 5. Taaskäivita server (peata vana ja käivita uuesti)
```

Päris maailmas peaksid ka vana sertifikaadi tühistama (CRL-i lisama), aga see on keerulisem harjutus.

---

## Osa 8: Subject Alternative Names

Praegune sertifikaat töötab ainult "localhost" jaoks. Mis siis, kui tahame, et ta töötaks mitme nimega?

```bash
# Loo konfiguratsioonifail SAN-idega
cat > server/san.cnf << 'EOF'
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req

[req_distinguished_name]

[v3_req]
basicConstraints = CA:FALSE
keyUsage = digitalSignature, keyEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = localhost
DNS.2 = server.local
DNS.3 = *.labor.local
IP.1 = 127.0.0.1
EOF

# Genereeri uus CSR SAN-idega
openssl req -new -key server/server.key -out server/server_san.csr \
    -subj "/C=EE/O=Minu Labor/CN=localhost" \
    -config server/san.cnf

# Allkirjasta (NB! -extensions on oluline)
openssl x509 -req -in server/server_san.csr \
    -CA ca/ca.crt -CAkey ca/ca.key -CAcreateserial \
    -out server/server.crt -days 365 -sha256 \
    -extensions v3_req -extfile server/san.cnf

# Kontrolli, kas SAN-id on olemas
openssl x509 -in server/server.crt -noout -text | grep -A1 "Subject Alternative Name"
```

Nüüd töötab sertifikaat localhost, server.local, *.labor.local ja IP 127.0.0.1 jaoks.

---

## Osa 9: Võtmete ja sertifikaatide võrdlemine

Kuidas kontrollida, kas võti ja sertifikaat kuuluvad kokku?

```bash
# Võrdle moduluseid (peavad olema samad)
openssl x509 -in server/server.crt -noout -modulus | openssl md5
openssl rsa -in server/server.key -noout -modulus | openssl md5
```

Kui MD5 hashid on samad, on võti ja sertifikaat paar.

---

## Kokkuvõte

Selles laboris sa:
- Lõid oma sertifitseerimisasutuse
- Genereerisid serveri sertifikaadi
- Käivitasid HTTPS serveri
- Õppisid sertifikaate uurima
- Harjutasid võtmeleket ja taastumist
- Lõid SAN-idega sertifikaadi

Need on samad sammud, mida DevOps insenerid teevad iga päev — lihtsalt automatiseeritult ja suuremas mahus.
