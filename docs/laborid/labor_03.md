---
tags:
  - Praktikum
  - Automatiseerimine
---

# Praktikum 3 - HashiCorp Vault PKI seadistamine Dockeriga

## Mida me teeme

Selles praktikumis ehitame üles sertifitseerimisasutuse kasutades HashiCorp Vault'i. Vault on tööriist, mida kasutavad suured ettevõtted sisemiste sertifikaatide haldamiseks. Me teeme seda Dockeriga, nii et midagi ei jää püsivalt arvutisse.

**Miks Vault, mitte OpenSSL?** Eelmistes praktikumides tegime sertifikaate käsitsi. See töötab, aga saja serveri puhul on käsitsi haldamine tee hukatusse. Vault annab sertifikaadi ühe API kutsega - automaatselt, turvaliselt, logitult.

**Vajad:** Docker, terminali.

---

## Osa 1: Vault'i käivitamine

### Samm 1: Kontrolli Dockerit

```bash
docker --version
```

Kui Docker puudub, installi see esmalt.

### Samm 2: Loo töökataloog

```bash
mkdir -p ~/vault-labor
cd ~/vault-labor
```

### Samm 3: Loo Docker Compose fail

Loo fail `docker-compose.yml`:

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

Mida see teeb: käivitab Vault'i arendusrežiimis pordil 8200, kasutab `root` kui autentimise tokenit. Arendusrežiim tähendab kiire käivitus ilma keerulise seadistamiseta - päris tootmises nii ei tee.

### Samm 4: Käivita

```bash
docker-compose up -d
docker ps
```

`-d` tähendab "detached" - Vault jookseb taustal. `docker ps` näitab, kas konteiner töötab.

---

## Osa 2: Vault CLI seadistamine

### Samm 1: Installi Vault CLI

=== "Ubuntu/Debian"

    ```bash
    wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
    echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
    sudo apt update && sudo apt install vault
    ```

=== "macOS"

    ```bash
    brew install vault
    ```

### Samm 2: Seadista ühendus

```bash
export VAULT_ADDR='http://127.0.0.1:8200'
export VAULT_TOKEN='root'
```

Need keskkonnamuutujad ütlevad Vault CLI-le, kuhu ja millise tokeniga ühenduda.

### Samm 3: Kontrolli

```bash
vault status
```

Kui näed Vault'i olekuinfot, oled ühendatud.

---

## Osa 3: PKI mootori seadistamine

Vault on modulaarne - PKI mootor tegeleb sertifikaatidega.

### Samm 1: Luba PKI

```bash
vault secrets enable pki
```

### Samm 2: Loo juur-CA

```bash
vault write pki/root/generate/internal \
    common_name="Minu Labor CA" \
    ttl=87600h
```

`ttl=87600h` on 10 aastat. Vault vastab juur-CA sertifikaadiga.

### Samm 3: Seadista URL-id

Sertifikaatidesse kirjutatakse URL-id, kust saab kontrollida tühistamist:

```bash
vault write pki/config/urls \
    issuing_certificates="$VAULT_ADDR/v1/pki/ca" \
    crl_distribution_points="$VAULT_ADDR/v1/pki/crl"
```

---

## Osa 4: Rolli loomine

Rollid määravad, kes mida teha tohib.

```bash
vault write pki/roles/veebiserver \
    allowed_domains="labor.local" \
    allow_subdomains=true \
    max_ttl="72h"
```

See lubab väljastada sertifikaate `labor.local`, `www.labor.local`, `api.labor.local` jne jaoks. Maksimaalne kehtivus 72h - lühiajalised sertifikaadid on turvalisemad, sest lekkinud võtme kahju on piiratud.

---

## Osa 5: Sertifikaadi väljastamine

### Samm 1: Küsi sertifikaati

```bash
vault write -format=json pki/issue/veebiserver \
    common_name="test.labor.local" \
    ttl="24h" > sertifikaat.json
```

Vault vastab JSON-iga, mis sisaldab sertifikaati, privaatvõtit ja CA ahelat.

### Samm 2: Eralda failidesse

```bash
cat sertifikaat.json | jq -r '.data.private_key' > server.key
cat sertifikaat.json | jq -r '.data.certificate' > server.crt
cat sertifikaat.json | jq -r '.data.ca_chain[0]' > ca.crt
```

`jq` on JSON töötlemise tööriist. `-r` annab "raw" väljundi ilma jutumärkideta.

---

## Osa 6: HTTPS serveri testimine

### Samm 1: Loo veebileht

```bash
echo "<h1>Tere tulemast!</h1><p>Sertifikaat Vault'ist.</p>" > index.html
```

### Samm 2: Käivita HTTPS server

```bash
python3 -c "
import http.server, ssl
server = http.server.HTTPServer(('0.0.0.0', 4443), http.server.SimpleHTTPRequestHandler)
ctx = ssl.SSLContext(ssl.PROTOCOL_TLS_SERVER)
ctx.load_cert_chain('server.crt', 'server.key')
server.socket = ctx.wrap_socket(server.socket, server_side=True)
print('Server: https://localhost:4443')
server.serve_forever()
"
```

### Samm 3: Testi (teises terminalis)

```bash
curl --cacert ca.crt https://localhost:4443
```

Kui näed HTML vastust, töötab!

---

## Osa 7: Vahe-CA loomine

Päris tootmises ei kasutata juur-CA-d otse. Vahe-CA (intermediate CA) lisab turvakihi - kui see kompromiteeritakse, saab selle tühistada ilma juur-CA-d puutumata.

### Samm 1: Luba vahe-PKI

```bash
vault secrets enable -path=pki_int pki
vault secrets tune -max-lease-ttl=43800h pki_int
```

### Samm 2: Loo vahe-CA taotlus

```bash
vault write -format=json pki_int/intermediate/generate/internal \
    common_name="Minu Labor Vahe-CA" \
    | jq -r '.data.csr' > vahe_ca.csr
```

### Samm 3: Juur-CA allkirjastab vahe-CA

```bash
vault write -format=json pki/root/sign-intermediate \
    csr=@vahe_ca.csr \
    format=pem_bundle \
    ttl="43800h" \
    | jq -r '.data.certificate' > vahe_ca.crt
```

`@vahe_ca.csr` tähendab "loe sisu failist".

### Samm 4: Impordi vahe-CA

```bash
vault write pki_int/intermediate/set-signed certificate=@vahe_ca.crt
```

Nüüd on kaheastmeline CA hierarhia. Sertifikaate väljastaksid `pki_int` teelt.

---

## Osa 8: Automaatne uuendamine

Vault'i võlu on automaatne uuendamine. Loo skript:

```bash
cat > uuenda.sh << 'SKRIPT'
#!/bin/bash
export VAULT_ADDR='http://127.0.0.1:8200'
export VAULT_TOKEN='root'

vault write -format=json pki/issue/veebiserver \
    common_name="test.labor.local" \
    ttl="24h" > sertifikaat.json

cat sertifikaat.json | jq -r '.data.private_key' > server.key
cat sertifikaat.json | jq -r '.data.certificate' > server.crt

echo "Sertifikaat uuendatud: $(date)"
# Päris elus: systemctl reload nginx
SKRIPT

chmod +x uuenda.sh
```

Croniga ajastamiseks:

```
0 3 * * * /home/kasutaja/vault-labor/uuenda.sh
```

---

## Osa 9: Puhastamine

```bash
cd ~/vault-labor
docker-compose down
rm -f server.key server.crt ca.crt sertifikaat.json index.html
rm -f vahe_ca.csr vahe_ca.crt uuenda.sh
```

---

## Kokkuvõte

| Aspekt | OpenSSL käsitsi | Vault |
|--------|-----------------|-------|
| Sertifikaadi loomine | Mitu käsku, mitu faili | Üks API kutse |
| Võtmete hoidmine | Failid kettal | Vault haldab |
| Uuendamine | Käsitsi mäletamine | Automaatne |
| Audit | Puudub | Sisseehitatud |
| Ligipääsukontroll | Failiõigused | Poliitikad |

See on põhjus, miks suured organisatsioonid kasutavad Vault'i - sajad sertifikaadid sajal serveril vajavad automatiseerimist.
