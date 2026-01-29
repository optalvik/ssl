# Labor 4 - Post-Quantum krüptograafia praktikas

## Mida me siin teeme?

Selles laboris saad käed külge post-quantum krüptograafiale. Me ei räägi ainult sellest, et kvantarvutid tulevad — me loome päriselt PQC võtmeid ja sertifikaate, testime neid ja võrdleme klassikaliste algoritmidega.

Labori lõpuks oskad:
- Installida ja kasutada Open Quantum Safe (OQS) teeki
- Luua ML-DSA (Dilithium) võtmeid ja sertifikaate
- Testida hübriidset TLS ühendust
- Võrrelda klassikaliste ja PQC algoritmide jõudlust

See on praktiline labor — sa näed oma silmaga, kuidas PQC töötab ja mille poolest see erineb sellest, mida me seni kasutanud oleme.

---

## Osa 1: Keskkonna ettevalmistamine

PQC algoritmid pole veel tavalises OpenSSL-is. Me kasutame Open Quantum Safe projekti, mis on lisanud need algoritmid OpenSSL-i pluginana.

### Variant A: Docker (soovitatud)

Kõige lihtsam viis on kasutada OQS-i valmis Docker image'it:

```bash
mkdir -p ~/pqc-labor
cd ~/pqc-labor

docker pull openquantumsafe/oqs-ossl3
docker run -it --rm -v $(pwd):/labor -w /labor openquantumsafe/oqs-ossl3 bash
```

Nüüd oled konteineris, kus on PQC-toega OpenSSL.

### Variant B: Natiivne installatsioon

Kui tahad installida otse oma süsteemi:

```bash
# Sõltuvused
sudo apt update
sudo apt install -y build-essential cmake ninja-build libssl-dev git

# liboqs
cd ~
git clone https://github.com/open-quantum-safe/liboqs.git
cd liboqs
mkdir build && cd build
cmake -GNinja ..
ninja
sudo ninja install

# oqs-provider
cd ~
git clone https://github.com/open-quantum-safe/oqs-provider.git
cd oqs-provider
mkdir build && cd build
cmake -GNinja ..
ninja
sudo ninja install
```

See võtab aega ja võib tekitada probleeme sõltuvalt süsteemist. Docker on lihtsam.

### Kontrolli, kas töötab

```bash
openssl list -signature-algorithms | grep -i mldsa
openssl list -kem-algorithms | grep -i mlkem
```

Peaksid nägema algoritme nagu `mldsa65`, `mlkem768` jne. Need on NIST-i standardiseeritud PQC algoritmid.

---

## Osa 2: Klassikaliste ja PQC võtmete võrdlemine

Alustame sellest, et loome erinevaid võtmeid ja võrdleme neid.

### RSA võti (klassikaline)

```bash
openssl genrsa -out rsa.key 2048
ls -la rsa.key
```

### ECDSA võti (klassikaline)

```bash
openssl ecparam -genkey -name prime256v1 -out ecdsa.key
ls -la ecdsa.key
```

### ML-DSA võti (post-quantum)

```bash
openssl genpkey -algorithm mldsa65 -out mldsa.key
ls -la mldsa.key
```

Võrdle failisuurusi:

```bash
echo "=== Võtmefailide suurused ==="
wc -c rsa.key ecdsa.key mldsa.key
```

Sa märkad, et ML-DSA võti on suurem kui RSA või ECDSA võti. See on üks PQC algoritmide kompromiss — turvalisus kvantarvutite vastu tuleb suurema andmemahu hinnaga.

---

## Osa 3: PQC sertifikaadi loomine

Nüüd loome sertifikaadi, mis kasutab ML-DSA algoritmi.

### Ise-allkirjastatud PQC sertifikaat

```bash
openssl req -x509 -new -key mldsa.key \
    -out mldsa.crt \
    -days 365 \
    -subj "/CN=pqc-test.local/O=PQC Labor/C=EE"
```

Vaata sertifikaati:

```bash
openssl x509 -in mldsa.crt -text -noout | head -30
```

Otsi ridu `Public Key Algorithm: mldsa65` ja `Signature Algorithm: mldsa65`. Need näitavad, et sertifikaat kasutab post-quantum algoritmi.

### Sertifikaadi suuruse võrdlus

```bash
echo "=== Sertifikaatide suurused ==="

# Loo RSA sertifikaat võrdluseks
openssl req -x509 -newkey rsa:2048 -nodes \
    -keyout rsa_temp.key -out rsa.crt \
    -days 365 -subj "/CN=rsa-test.local"

# Loo ECDSA sertifikaat võrdluseks
openssl req -x509 -newkey ec -pkeyopt ec_paramgen_curve:prime256v1 -nodes \
    -keyout ecdsa_temp.key -out ecdsa.crt \
    -days 365 -subj "/CN=ecdsa-test.local"

wc -c rsa.crt ecdsa.crt mldsa.crt
```

ML-DSA sertifikaat on märgatavalt suurem. See mõjutab TLS käepigistust — rohkem andmeid tuleb üle võrgu saata.

---

## Osa 4: PQC sertifitseerimisasutuse loomine

Loome oma post-quantum CA ja kasutame seda serveri sertifikaadi allkirjastamiseks.

### CA võti ja sertifikaat

```bash
# CA privaatvõti
openssl genpkey -algorithm mldsa65 -out pqc_ca.key

# CA sertifikaat
openssl req -x509 -new -key pqc_ca.key \
    -out pqc_ca.crt \
    -days 3650 \
    -subj "/CN=PQC Labor CA/O=Labor/C=EE"
```

### Serveri võti ja taotlus

```bash
# Serveri privaatvõti
openssl genpkey -algorithm mldsa65 -out pqc_server.key

# Serveri sertifikaadi taotlus (CSR)
openssl req -new -key pqc_server.key \
    -out pqc_server.csr \
    -subj "/CN=server.pqc.local/O=Labor/C=EE"
```

### CA allkirjastab serveri sertifikaadi

```bash
openssl x509 -req -in pqc_server.csr \
    -CA pqc_ca.crt -CAkey pqc_ca.key \
    -CAcreateserial \
    -out pqc_server.crt \
    -days 365
```

### Kontrolli ahelat

```bash
openssl verify -CAfile pqc_ca.crt pqc_server.crt
```

Kui näed `pqc_server.crt: OK`, siis sertifikaadiahel töötab.

---

## Osa 5: Jõudluse võrdlemine

PQC algoritmid käituvad erinevalt kui klassikalised. Mõned on kiiremad võtme genereerimisel, teised allkirjastamisel. Teeme lihtsa testi.

### Võtme genereerimise kiirus

```bash
echo "=== Võtme genereerimine (10 korda) ==="

echo -n "RSA-2048: "
time (for i in {1..10}; do openssl genrsa 2048 2>/dev/null > /dev/null; done)

echo -n "ECDSA P-256: "
time (for i in {1..10}; do openssl ecparam -genkey -name prime256v1 2>/dev/null > /dev/null; done)

echo -n "ML-DSA-65: "
time (for i in {1..10}; do openssl genpkey -algorithm mldsa65 2>/dev/null > /dev/null; done)
```

### Allkirjastamise kiirus

```bash
echo "test data" > testfail.txt

echo "=== Allkirjastamine (100 korda) ==="

echo -n "RSA-2048: "
time (for i in {1..100}; do 
    openssl dgst -sha256 -sign rsa.key testfail.txt > /dev/null 2>&1
done)

echo -n "ECDSA P-256: "
time (for i in {1..100}; do 
    openssl dgst -sha256 -sign ecdsa.key testfail.txt > /dev/null 2>&1
done)

echo -n "ML-DSA-65: "
time (for i in {1..100}; do 
    openssl pkeyutl -sign -inkey mldsa.key -in testfail.txt > /dev/null 2>&1
done)
```

Tulemused sõltuvad riistvarast, aga üldiselt:
- RSA võtme genereerimine on aeglane, allkirjastamine kiire
- ECDSA on mõlemas kiire
- ML-DSA on võtme genereerimisel kiire, allkirja suurus suurem

---

## Osa 6: Hübriidne TLS

Üleminekuperioodil kasutatakse hübriidset lähenemist: klassikaline + PQC algoritm koos. Kui üks osutub nõrgaks, kaitseb teine.

### Hübriidne võtmevahetus

Käivita server hübriidse võtmevahetusega:

```bash
# Terminalis 1: Server
openssl s_server -accept 4433 \
    -cert pqc_server.crt \
    -key pqc_server.key \
    -groups x25519_mlkem768 \
    -www
```

Teises terminalis ühenda:

```bash
# Terminalis 2: Klient
openssl s_client -connect localhost:4433 \
    -CAfile pqc_ca.crt \
    -groups x25519_mlkem768
```

Otsi väljundist rida `Server Temp Key`. Kui näed `X25519MLKEM768`, toimib hübriidne võtmevahetus.

See tähendab:
- Klassikaline X25519 (elliptiline kõver) teeb võtmevahetuse
- ML-KEM-768 (post-quantum) teeb samuti võtmevahetuse
- Mõlema tulemused kombineeritakse

Isegi kui üks algoritm murdub, jääb teine kaitsma.

---

## Osa 7: Allkirja suuruste võrdlus

Üks PQC suurimaid kompromisse on allkirja suurus. Vaatame:

```bash
echo "test" > test.txt

# RSA allkiri
openssl dgst -sha256 -sign rsa.key -out rsa.sig test.txt
echo -n "RSA-2048 allkiri: "
wc -c < rsa.sig

# ECDSA allkiri
openssl dgst -sha256 -sign ecdsa.key -out ecdsa.sig test.txt
echo -n "ECDSA P-256 allkiri: "
wc -c < ecdsa.sig

# ML-DSA allkiri
openssl pkeyutl -sign -inkey mldsa.key -in test.txt -out mldsa.sig
echo -n "ML-DSA-65 allkiri: "
wc -c < mldsa.sig
```

Tulemused on umbes:
- RSA-2048: ~256 baiti
- ECDSA P-256: ~70 baiti
- ML-DSA-65: ~3300 baiti

ML-DSA allkiri on umbes 50 korda suurem kui ECDSA! See on oluline, kui mõtled sertifikaadiahelatele — iga sertifikaat sisaldab allkirja.

---

## Osa 8: NIST algoritmide ülevaade

Siin on kokkuvõte NIST-i standardiseeritud PQC algoritmidest:

| Algoritm | NIST nimi | Tüüp | Kasutus |
|----------|-----------|------|---------|
| Kyber | ML-KEM | Võtmevahetus | TLS, VPN |
| Dilithium | ML-DSA | Allkiri | Sertifikaadid, koodi allkirjastamine |
| SPHINCS+ | SLH-DSA | Allkiri | Pikaajalised allkirjad |
| FALCON | FN-DSA | Allkiri | Piiratud ressursiga seadmed |

Turvatase valikud:
- `-512` / `-44`: AES-128 ekvivalent
- `-768` / `-65`: AES-192 ekvivalent
- `-1024` / `-87`: AES-256 ekvivalent

---

## Osa 9: Päris maailm

Kas PQC on juba kasutusel? Jah!

### Brauserid

Chrome ja Firefox kasutavad juba hübriidset võtmevahetust (X25519Kyber768) vaikimisi. Sa ei pea midagi tegema — kui külastad saiti, mis toetab PQC-d, kasutatakse seda automaatselt.

Kontrolli oma brauseris:
1. Mine lehele nagu cloudflare.com
2. Ava arendaja tööriistad (F12)
3. Vaata Security sakki
4. Otsi "Key exchange" infot

### Serverid

Cloudflare, Google ja teised suured toetavad PQC-d. Kui sinu server kasutab uut tarkvara, võib PQC-tugi juba olemas olla.

---

## Osa 10: Puhastamine

```bash
# Kui kasutasid Dockerit
exit  # väljud konteinerist

# Kustuta failid
cd ~/pqc-labor
rm -f *.key *.crt *.csr *.sig *.txt *.srl
```

---

## Kokkuvõte

Selles laboris õppisid:

| Teema | Mida tegid |
|-------|------------|
| OQS seadistamine | Docker või natiivne installatsioon |
| PQC võtmed | ML-DSA võtmete genereerimine |
| PQC sertifikaadid | CA ja serveri sertifikaadid |
| Jõudlus | Klassikaliste ja PQC algoritmide võrdlus |
| Hübriidne TLS | X25519 + ML-KEM kombinatsioon |
| Suurused | Võtmete ja allkirjade suuruste võrdlus |

Peamised õppetunnid:

1. **PQC töötab** — need pole enam eksperimentaalsed algoritmid
2. **Suurused on suuremad** — võtmed, sertifikaadid, allkirjad
3. **Hübriidne on tark** — klassikaline + PQC annab parima kaitse
4. **Brauserid juba toetavad** — üleminek on käimas

Mida edasi teha:

- Jälgi NIST-i ja IETF-i uudiseid PQC kohta
- Hoia oma tarkvara (OpenSSL, brauserid) värskena
- Kui haldad pikaajalisi andmeid, alusta PQC planeerimist
- Katseta hübriidseid lahendusi testimiskeskkonnas
