---
tags:
  - Praktikum
  - Sertifikaadid
---

# Praktikum 1 - Veebiturve ja sertifikaadid praktikas

## Mida me teeme

Selles praktikumis lood sa oma sertifitseerimisasutuse, genereerid sertifikaate ja paned püsti turvalise veebiserveri. Sa näed oma silmaga, kuidas kogu see süsteem töötab - mitte teoorias, vaid päriselt.

**Vajad:** Linuxi masinat (Ubuntu sobib hästi) ja OpenSSL-i, mis on tavaliselt juba installitud.

!!! info "Esimest korda OpenSSL-iga?"
    Kasuta [Spikrit](../spikker.md) käskude kiireks meeldetuletuseks. Selle praktikumi käigus õpid kõike samm-sammult.

---

## Osa 1: Oma CA loomine

Kõigepealt loome kaustade struktuuri, kuhu kõik failid tulevad:

```bash
mkdir -p ~/tls-labor/ca ~/tls-labor/server
cd ~/tls-labor
```

### Samm 1: CA privaatvõtme genereerimine

```bash
openssl genrsa -out ca/ca.key 4096
```

See lõi 4096-bitise RSA võtme. Mida suurem number, seda tugevam võti.

### Samm 2: CA sertifikaadi loomine

```bash
openssl req -x509 -new -nodes -key ca/ca.key -sha256 -days 3650 \
    -out ca/ca.crt \
    -subj "/C=EE/O=Minu Labor/CN=Minu Labor CA"
```

Parameetrid:

| Parameeter | Tähendus |
|------------|----------|
| `-x509` | Loo sertifikaat (mitte CSR) |
| `-new` | Uus taotlus |
| `-nodes` | Ära krüpteeri privaatvõtit parooliga |
| `-sha256` | Kasuta SHA-256 räsialgoritmi |
| `-days 3650` | Kehtib 10 aastat |
| `-subj` | Sertifikaadi subjekt |

### Samm 3: Kontrolli tulemust

```bash
openssl x509 -in ca/ca.crt -text -noout | head -20
```

Pane tähele, et Issuer ja Subject on samad - see ongi ise-allkirjastatud sertifikaat.

---

## Osa 2: Serveri sertifikaadi loomine

Nüüd loome sertifikaadi meie veebiserveri jaoks. See on kaheastmeline protsess: kõigepealt CSR (taotlus), siis allkirjastamine CA-ga.

### Samm 1: Serveri privaatvõti

```bash
openssl genrsa -out server/server.key 2048
```

### Samm 2: CSR (sertifikaadi taotlus)

```bash
openssl req -new -key server/server.key -out server/server.csr \
    -subj "/C=EE/O=Minu Labor/CN=localhost"
```

CSR sisaldab avalikku võtit ja infot, mida tahame sertifikaadile. Vaata seda:

```bash
openssl req -in server/server.csr -text -noout
```

### Samm 3: CA allkirjastab CSR-i

```bash
openssl x509 -req -in server/server.csr \
    -CA ca/ca.crt -CAkey ca/ca.key -CAcreateserial \
    -out server/server.crt -days 365 -sha256
```

### Tulemus

Nüüd on meil neli faili:

| Fail | Sisu | Turvalisus |
|------|------|------------|
| `ca/ca.key` | CA privaatvõti | ÜLISALAJANE |
| `ca/ca.crt` | CA sertifikaat | Avalik, levitame |
| `server/server.key` | Serveri privaatvõti | Salajane |
| `server/server.crt` | Serveri sertifikaat | Avalik |

### Samm 4: Kontrolli, et kõik on korras

```bash
openssl verify -CAfile ca/ca.crt server/server.crt
```

Kui näed "OK", on sertifikaadiahel terve.

---

## Osa 3: HTTPS serveri käivitamine

### Samm 1: Loo veebileht

```bash
cd ~/tls-labor/server
echo "<h1>Tere tulemast turvalisse serverisse!</h1>" > index.html
```

### Samm 2: Käivita HTTPS server

Kasutame Pythoni sisseehitatud HTTP serverit koos SSL-toega:

```bash
python3 -c "
import http.server, ssl
server = http.server.HTTPServer(('localhost', 4443), http.server.SimpleHTTPRequestHandler)
ctx = ssl.SSLContext(ssl.PROTOCOL_TLS_SERVER)
ctx.load_cert_chain('server.crt', 'server.key')
server.socket = ctx.wrap_socket(server.socket, server_side=True)
print('Server: https://localhost:4443')
server.serve_forever()
"
```

### Samm 3: Testi (teises terminalis)

```bash
# Ilma CA-d usaldamata - annab vea
curl https://localhost:4443

# CA-d usaldades - töötab
curl --cacert ~/tls-labor/ca/ca.crt https://localhost:4443
```

Esimene käsk annab vea, sest curl ei usalda meie CA-d. Teine töötab, sest ütleme curl'ile konkreetselt, et usaldagu meie CA-d.

Proovi ka brauseris avada `https://localhost:4443` - näed hoiatust, sest brauser ei usalda meie CA-d.

---

## Osa 4: CA usaldamine süsteemis

Kui tahad, et kogu süsteem usaldaks sinu CA-d:

```bash
# Ubuntu/Debian
sudo cp ~/tls-labor/ca/ca.crt /usr/local/share/ca-certificates/minu-labor.crt
sudo update-ca-certificates
```

Nüüd kontrolli - peaks töötama ilma `--cacert` parameetrita:

```bash
curl https://localhost:4443
```

---

## Osa 5: Sertifikaadi uurimine

Erinevad OpenSSL käsud näitavad sertifikaadist erinevat infot:

```bash
# Kõik detailid
openssl x509 -in server/server.crt -text -noout

# Ainult subjekt ja väljastaja
openssl x509 -in server/server.crt -noout -subject -issuer

# Kehtivusajad
openssl x509 -in server/server.crt -noout -dates

# Sõrmejälg
openssl x509 -in server/server.crt -noout -fingerprint -sha256
```

---

## Osa 6: Serverist sertifikaadi lugemine

Sertifikaati saab lugeda otse serverist, ilma faili omamata:

```bash
# Näita sertifikaati
echo | openssl s_client -connect localhost:4443 2>/dev/null | \
    openssl x509 -text -noout
```

`s_client` ühendub serveriga nagu brauser, teeb TLS käepigistuse ja saab sertifikaadi kätte.

```bash
# Näita tervet sertifikaadikett
openssl s_client -connect localhost:4443 -showcerts
```

```bash
# Kontrolli, kas aegub 30 päeva jooksul
echo | openssl s_client -connect localhost:4443 2>/dev/null | \
    openssl x509 -noout -checkend 2592000
```

`-checkend 2592000` kontrollib, kas sertifikaat kehtib veel 2592000 sekundit (30 päeva).

---

## Osa 7: Võtme leke ja taastumine

Simuleerime olukorda, kus privaatvõti lekib. Mida teha?

### Samm 1: Genereeri uus võtmepaar

```bash
cd ~/tls-labor
openssl genrsa -out server/server_uus.key 2048
```

### Samm 2: Loo uus CSR

```bash
openssl req -new -key server/server_uus.key -out server/server_uus.csr \
    -subj "/C=EE/O=Minu Labor/CN=localhost"
```

### Samm 3: Allkirjasta CA-ga

```bash
openssl x509 -req -in server/server_uus.csr \
    -CA ca/ca.crt -CAkey ca/ca.key -CAcreateserial \
    -out server/server_uus.crt -days 365 -sha256
```

### Samm 4: Asenda vanad failid ja taaskäivita server

```bash
mv server/server_uus.key server/server.key
mv server/server_uus.crt server/server.crt
```

Päris maailmas peaksid ka vana sertifikaadi tühistama (CRL-i lisama).

---

## Osa 8: Subject Alternative Names (SAN)

Praegune sertifikaat töötab ainult `localhost` jaoks. Kui tahame mitut nime, on vaja SAN-e.

### Samm 1: Loo konfiguratsioonifail

```bash
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
```

See konfiguratsioonifail ütleb, et sertifikaat peab kehtima nelja aadressi jaoks: `localhost`, `server.local`, kõik `*.labor.local` alamdomeenid ja IP `127.0.0.1`.

### Samm 2: Loo uus CSR SAN-idega

```bash
openssl req -new -key server/server.key -out server/server_san.csr \
    -subj "/C=EE/O=Minu Labor/CN=localhost" \
    -config server/san.cnf
```

### Samm 3: Allkirjasta (NB! -extensions on oluline)

```bash
openssl x509 -req -in server/server_san.csr \
    -CA ca/ca.crt -CAkey ca/ca.key -CAcreateserial \
    -out server/server.crt -days 365 -sha256 \
    -extensions v3_req -extfile server/san.cnf
```

### Samm 4: Kontrolli SAN-e

```bash
openssl x509 -in server/server.crt -noout -text | grep -A1 "Subject Alternative Name"
```

---

## Osa 9: Võtme ja sertifikaadi paaritamine

Kuidas kontrollida, kas võti ja sertifikaat kuuluvad kokku?

```bash
openssl x509 -in server/server.crt -noout -modulus | openssl md5
openssl rsa -in server/server.key -noout -modulus | openssl md5
```

Kui MD5 räsid on samad, on võti ja sertifikaat paar. See on kasulik siis, kui sul on palju faile ja pole kindel, mis millega kokku käib.

---

## Kokkuvõte

Selles praktikumis sa:

- Lõid oma sertifitseerimisasutuse (CA)
- Genereerisid serveri sertifikaadi ja allkirjastasid selle
- Käivitasid HTTPS serveri
- Õppisid sertifikaate uurima ja serverist lugema
- Harjutasid võtmeleket ja taastumist
- Lõid SAN-idega sertifikaadi

Need on samad sammud, mida DevOps insenerid teevad iga päev - lihtsalt automatiseeritult ja suuremas mahus.
