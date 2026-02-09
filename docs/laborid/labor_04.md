---
tags:
  - Praktikum
  - PQC
---

# Praktikum 4 - Post-Quantum krüptograafia praktikas

## Mida me teeme

Selles praktikumis saad käed külge post-quantum krüptograafiale. Loome PQC võtmeid ja sertifikaate, testime hübriidset TLS-i ja võrdleme klassikaliste algoritmidega.

**Vajad:** Docker (soovitatud) või Linux süsteem kompileerimiseks.

---

## Osa 1: Keskkonna ettevalmistamine

PQC algoritmid pole veel tavalises OpenSSL-is. Kasutame Open Quantum Safe (OQS) projekti.

=== "Docker (soovitatud)"

    ```bash
    mkdir -p ~/pqc-labor
    cd ~/pqc-labor
    docker pull openquantumsafe/oqs-ossl3
    docker run -it --rm -v $(pwd):/labor -w /labor openquantumsafe/oqs-ossl3 bash
    ```

    Nüüd oled konteineris, kus on PQC-toega OpenSSL.

=== "Natiivne installatsioon"

    ```bash
    sudo apt update
    sudo apt install -y build-essential cmake ninja-build libssl-dev git

    # liboqs
    cd ~ && git clone https://github.com/open-quantum-safe/liboqs.git
    cd liboqs && mkdir build && cd build
    cmake -GNinja .. && ninja && sudo ninja install

    # oqs-provider
    cd ~ && git clone https://github.com/open-quantum-safe/oqs-provider.git
    cd oqs-provider && mkdir build && cd build
    cmake -GNinja .. && ninja && sudo ninja install
    ```

### Kontrolli

```bash
openssl list -signature-algorithms | grep -i mldsa
openssl list -kem-algorithms | grep -i mlkem
```

Peaksid nägema algoritme nagu `mldsa65`, `mlkem768` - need on NIST-i standardiseeritud PQC algoritmid.

---

## Osa 2: Võtmete võrdlemine

Loome erinevaid võtmeid ja võrdleme suurusi.

### Samm 1: Klassikalised võtmed

```bash
# RSA
openssl genrsa -out rsa.key 2048

# ECDSA
openssl ecparam -genkey -name prime256v1 -out ecdsa.key
```

### Samm 2: PQC võti

```bash
openssl genpkey -algorithm mldsa65 -out mldsa.key
```

### Samm 3: Võrdle suurusi

```bash
echo "=== Võtmefailide suurused ==="
wc -c rsa.key ecdsa.key mldsa.key
```

ML-DSA võti on märgatavalt suurem - see on PQC kompromiss. Turvalisus kvantarvutite vastu tuleb suurema andmemahu hinnaga.

---

## Osa 3: PQC sertifikaadi loomine

### Samm 1: Ise-allkirjastatud PQC sertifikaat

```bash
openssl req -x509 -new -key mldsa.key \
    -out mldsa.crt -days 365 \
    -subj "/CN=pqc-test.local/O=PQC Labor/C=EE"
```

### Samm 2: Vaata tulemust

```bash
openssl x509 -in mldsa.crt -text -noout | head -30
```

Otsi ridu `Public Key Algorithm: mldsa65` ja `Signature Algorithm: mldsa65`.

### Samm 3: Võrdle sertifikaatide suurusi

```bash
# Loo RSA ja ECDSA sertifikaadid võrdluseks
openssl req -x509 -newkey rsa:2048 -nodes \
    -keyout rsa_temp.key -out rsa.crt \
    -days 365 -subj "/CN=rsa-test.local"

openssl req -x509 -newkey ec -pkeyopt ec_paramgen_curve:prime256v1 -nodes \
    -keyout ecdsa_temp.key -out ecdsa.crt \
    -days 365 -subj "/CN=ecdsa-test.local"

echo "=== Sertifikaatide suurused ==="
wc -c rsa.crt ecdsa.crt mldsa.crt
```

---

## Osa 4: PQC sertifitseerimisasutus

Loome PQC-põhise CA ja kasutame seda serveri sertifikaadi allkirjastamiseks.

### Samm 1: CA loomine

```bash
openssl genpkey -algorithm mldsa65 -out pqc_ca.key

openssl req -x509 -new -key pqc_ca.key \
    -out pqc_ca.crt -days 3650 \
    -subj "/CN=PQC Labor CA/O=Labor/C=EE"
```

### Samm 2: Serveri võti ja CSR

```bash
openssl genpkey -algorithm mldsa65 -out pqc_server.key

openssl req -new -key pqc_server.key \
    -out pqc_server.csr \
    -subj "/CN=server.pqc.local/O=Labor/C=EE"
```

### Samm 3: CA allkirjastab

```bash
openssl x509 -req -in pqc_server.csr \
    -CA pqc_ca.crt -CAkey pqc_ca.key -CAcreateserial \
    -out pqc_server.crt -days 365
```

### Samm 4: Kontrolli ahelat

```bash
openssl verify -CAfile pqc_ca.crt pqc_server.crt
```

`pqc_server.crt: OK` - sertifikaadiahel töötab.

---

## Osa 5: Jõudluse võrdlemine

### Võtme genereerimine (10 korda)

```bash
echo -n "RSA-2048:   "; time (for i in {1..10}; do openssl genrsa 2048 2>/dev/null > /dev/null; done)
echo -n "ECDSA P-256: "; time (for i in {1..10}; do openssl ecparam -genkey -name prime256v1 2>/dev/null > /dev/null; done)
echo -n "ML-DSA-65:   "; time (for i in {1..10}; do openssl genpkey -algorithm mldsa65 2>/dev/null > /dev/null; done)
```

### Allkirjastamine (100 korda)

```bash
echo "test data" > testfail.txt

echo -n "RSA-2048:    "; time (for i in {1..100}; do openssl dgst -sha256 -sign rsa.key testfail.txt > /dev/null 2>&1; done)
echo -n "ECDSA P-256: "; time (for i in {1..100}; do openssl dgst -sha256 -sign ecdsa.key testfail.txt > /dev/null 2>&1; done)
echo -n "ML-DSA-65:   "; time (for i in {1..100}; do openssl pkeyutl -sign -inkey mldsa.key -in testfail.txt > /dev/null 2>&1; done)
```

Üldised mustrid: RSA võtme genereerimine aeglane, allkirjastamine kiire. ECDSA mõlemas kiire. ML-DSA võtme genereerimisel kiire, allkirja suurus suurem.

---

## Osa 6: Hübriidne TLS

Üleminekuperioodil kasutatakse klassikaline + PQC koos. Kui üks osutub nõrgaks, kaitseb teine.

### Samm 1: Käivita server (terminal 1)

```bash
openssl s_server -accept 4433 \
    -cert pqc_server.crt -key pqc_server.key \
    -groups x25519_mlkem768 \
    -www
```

### Samm 2: Ühenda (terminal 2)

```bash
openssl s_client -connect localhost:4433 \
    -CAfile pqc_ca.crt \
    -groups x25519_mlkem768
```

Otsi väljundist `Server Temp Key`. `X25519MLKEM768` tähendab, et toimib hübriidne võtmevahetus - klassikaline X25519 ja post-quantum ML-KEM-768 koos.

---

## Osa 7: Allkirja suuruste võrdlus

```bash
echo "test" > test.txt

openssl dgst -sha256 -sign rsa.key -out rsa.sig test.txt
openssl dgst -sha256 -sign ecdsa.key -out ecdsa.sig test.txt
openssl pkeyutl -sign -inkey mldsa.key -in test.txt -out mldsa.sig

echo "RSA-2048:    $(wc -c < rsa.sig) baiti"
echo "ECDSA P-256: $(wc -c < ecdsa.sig) baiti"
echo "ML-DSA-65:   $(wc -c < mldsa.sig) baiti"
```

Oodatavad tulemused: RSA ~256B, ECDSA ~70B, ML-DSA ~3300B. ML-DSA allkiri on ~50x suurem kui ECDSA!

---

## Osa 8: NIST algoritmide ülevaade

| Algoritm | NIST nimi | Tüüp | Kasutus |
|----------|-----------|------|---------|
| Kyber | ML-KEM | Võtmevahetus | TLS, VPN |
| Dilithium | ML-DSA | Allkiri | Sertifikaadid, koodi allkirjastamine |
| SPHINCS+ | SLH-DSA | Allkiri | Pikaajalised allkirjad |
| FALCON | FN-DSA | Allkiri | Piiratud ressursiga seadmed |

Turvatasemed: `-512`/`-44` = AES-128, `-768`/`-65` = AES-192, `-1024`/`-87` = AES-256.

---

## Osa 9: Puhastamine

```bash
# Kui kasutasid Dockerit
exit

# Kustuta failid
cd ~/pqc-labor
rm -f *.key *.crt *.csr *.sig *.txt *.srl
```

---

## Kokkuvõte

Peamised õppetunnid:

1. **PQC töötab** - need pole enam eksperimentaalsed algoritmid
2. **Suurused on suuremad** - võtmed, sertifikaadid, allkirjad
3. **Hübriidne on tark** - klassikaline + PQC annab parima kaitse
4. **Brauserid juba toetavad** - Chrome ja Firefox kasutavad hübriidset võtmevahetust vaikimisi
