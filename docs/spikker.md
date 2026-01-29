---
title: Spikker
description: Kiired käsud ja viited
---

# Spikker

Kõik vajalikud käsud ühes kohas.

[:material-file-pdf-box: Lae alla PDF spikker](assets/cheat_sheet.pdf){ .md-button }

---

## OpenSSL käsud

### Sertifikaadi info

```bash
# Vaata sertifikaadi sisu
openssl x509 -in cert.crt -text -noout

# Kontrolli aegumist
openssl x509 -in cert.crt -noout -dates

# Vaata SAN välja
openssl x509 -in cert.crt -noout -text | grep -A1 "Subject Alternative Name"
```

### Serveri kontrollimine

```bash
# Kontrolli serverit
openssl s_client -connect example.com:443 -servername example.com

# Näita sertifikaadiahel
openssl s_client -connect example.com:443 -showcerts

# Kontrolli protokolle
openssl s_client -connect example.com:443 -tls1_2
openssl s_client -connect example.com:443 -tls1_3
```

### Võtme ja sertifikaadi vastavus

```bash
# Kas võti ja sert klapivad?
openssl x509 -noout -modulus -in cert.crt | md5sum
openssl rsa -noout -modulus -in key.key | md5sum
# Peavad olema samad!
```

---

## Formaatide teisendamine

=== "PEM ↔ DER"

    ```bash
    # PEM -> DER
    openssl x509 -in cert.pem -outform DER -out cert.der
    
    # DER -> PEM
    openssl x509 -in cert.der -inform DER -out cert.pem
    ```

=== "PEM ↔ P12"

    ```bash
    # PEM -> P12 (võti + sert)
    openssl pkcs12 -export -out cert.p12 -inkey key.key -in cert.crt
    
    # P12 -> PEM
    openssl pkcs12 -in cert.p12 -out cert.pem -nodes
    ```

=== "JKS ↔ P12"

    ```bash
    # JKS -> P12
    keytool -importkeystore -srckeystore keystore.jks \
      -destkeystore keystore.p12 -deststoretype PKCS12
    
    # P12 -> JKS
    keytool -importkeystore -srckeystore keystore.p12 \
      -srcstoretype PKCS12 -destkeystore keystore.jks
    ```

---

## Sertifikaadi loomine

### Ise-allkirjastatud (testimiseks)

```bash
# Ühe käsuga
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem \
  -days 365 -nodes -subj "/CN=localhost"
```

### CSR + allkirjastamine

```bash
# Genereeri võti
openssl genrsa -out server.key 2048

# Loo CSR
openssl req -new -key server.key -out server.csr -subj "/CN=example.com"

# Allkirjasta CA-ga
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key \
  -CAcreateserial -out server.crt -days 365
```

---

## Vault käsud

```bash
# Login
export VAULT_ADDR='http://127.0.0.1:8200'
vault login

# PKI väljastamine
vault write pki/issue/webserver common_name="example.com"

# Sertifikaadi tühistamine
vault write pki/revoke serial_number="..."
```

---

## Kasulikud tööriistad

| Tööriist | URL | Otstarve |
|----------|-----|----------|
| SSL Labs | ssllabs.com/ssltest | Serveri testimine |
| crt.sh | crt.sh | CT logide otsing |
| badssl.com | badssl.com | TLS testimine |

---

## Levinud vead

!!! failure "Certificate has expired"
    Sertifikaat on aegunud. Kontrolli: `openssl x509 -in cert.crt -noout -dates`

!!! failure "Hostname mismatch"
    Sertifikaadi nimi ei klapi. Kontrolli SAN välja.

!!! failure "Unable to get local issuer certificate"
    Ahel on katki — vahesertifikaat puudub.

!!! failure "Key values mismatch"
    Võti ja sertifikaat ei kuulu kokku. Kontrolli modulus hash'e.
