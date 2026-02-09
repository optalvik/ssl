---
tags:
  - Praktikum
  - Turvalisus
---

# Praktikum 2 - Sertifikaadi detektiiv

## Mida me teeme

Selles praktikumis õpid sa sertifikaate analüüsima nagu turvaekspert. Kogud päris sertifikaate internetist, uurid neid ja lood tööriistu automaatseks kontrollimiseks.

---

## Osa 1: Sertifikaatide kogumine

Alustame sertifikaatide kogumisest erinevatelt saitidelt.

### Samm 1: Loo töökataloog

```bash
mkdir -p ~/sert-labor/kogutud
cd ~/sert-labor
```

### Samm 2: Kogu sertifikaate

`s_client` ühendub serveriga, teeb TLS käepigistuse ja annab sertifikaadi. Me salvestame selle PEM-failiks:

```bash
echo | openssl s_client -connect google.com:443 -servername google.com 2>/dev/null | \
    openssl x509 -outform PEM > kogutud/google.crt
```

`-servername` on SNI (Server Name Indication) - see ütleb serverile, millist sertifikaati sa tahad. Oluline, sest paljudel serveritel on mitu saiti sama IP peal.

Kogu veel mõne saidi sertifikaat:

```bash
echo | openssl s_client -connect github.com:443 -servername github.com 2>/dev/null | \
    openssl x509 -outform PEM > kogutud/github.crt

echo | openssl s_client -connect www.postimees.ee:443 -servername www.postimees.ee 2>/dev/null | \
    openssl x509 -outform PEM > kogutud/postimees.crt
```

---

## Osa 2: Põhiinfo uurimine

Nüüd vaatame, mis neis sertifikaatides kirjas on. `for`-tsükkel käib läbi kõik kogutud sertifikaadid.

### Kes on omanik?

```bash
for cert in kogutud/*.crt; do
    echo "=== $(basename $cert) ==="
    openssl x509 -in "$cert" -noout -subject
done
```

### Kes väljastas?

```bash
for cert in kogutud/*.crt; do
    echo "=== $(basename $cert) ==="
    openssl x509 -in "$cert" -noout -issuer
done
```

### Millal aegub?

```bash
for cert in kogutud/*.crt; do
    echo "=== $(basename $cert) ==="
    openssl x509 -in "$cert" -noout -dates
done
```

---

## Osa 3: Sertifikaadi raportiskript

Loome skripti, mis koondab kogu olulise info ühte raportisse. Ehitame selle samm-sammult üles.

### Samm 1: Loo skriptifail ja põhistruktuur

```bash
cat > raport.sh << 'SKRIPT'
#!/bin/bash

if [ -z "$1" ]; then
    echo "Kasutus: $0 sertifikaat.crt"
    exit 1
fi

CERT="$1"
echo "========================================"
echo "SERTIFIKAADI RAPORT"
echo "========================================"
SKRIPT

chmod +x raport.sh
```

See kontrollib, kas kasutaja andis argumendiks sertifikaadifaili. Kui ei andnud, näitab kasutusjuhendit.

### Samm 2: Lisa põhiinfo ja kehtivuse kontroll

Avame faili ja lisame järgmised osad juurde:

```bash
cat >> raport.sh << 'SKRIPT'

echo "--- PÕHIINFO ---"
openssl x509 -in "$CERT" -noout -subject -issuer -serial

echo ""
echo "--- KEHTIVUS ---"
openssl x509 -in "$CERT" -noout -dates

if openssl x509 -in "$CERT" -noout -checkend 0 > /dev/null 2>&1; then
    echo "Staatus: KEHTIV"
else
    echo "Staatus: AEGUNUD!"
fi
SKRIPT
```

`-checkend 0` kontrollib, kas sertifikaat kehtib praegusel hetkel.

### Samm 3: Lisa SAN-id ja võtmeinfo

```bash
cat >> raport.sh << 'SKRIPT'

echo ""
echo "--- ALTERNATIIVSED NIMED (SAN) ---"
openssl x509 -in "$CERT" -noout -text | grep -A1 "Subject Alternative Name" | tail -1 | tr ',' '\n'

echo ""
echo "--- VÕTME INFO ---"
openssl x509 -in "$CERT" -noout -text | grep -A2 "Public Key Algorithm"

echo ""
echo "--- ALLKIRJA ALGORITM ---"
openssl x509 -in "$CERT" -noout -text | grep "Signature Algorithm" | head -1

echo ""
echo "--- SÕRMEJÄLG ---"
openssl x509 -in "$CERT" -noout -fingerprint -sha256
SKRIPT
```

### Samm 4: Proovi

```bash
./raport.sh kogutud/google.crt
./raport.sh kogutud/github.crt
```

---

## Osa 4: Probleemide tuvastamine

Loome skripti, mis otsib sertifikaadist tüüpilisi turvaprobleeme.

### Samm 1: Skripti alus ja aegumiskontroll

```bash
cat > kontrolli.sh << 'SKRIPT'
#!/bin/bash

CERT="$1"
PROBLEEME=0

echo "Kontrollin: $CERT"
echo ""

# Kas aegunud?
if ! openssl x509 -in "$CERT" -noout -checkend 0 > /dev/null 2>&1; then
    echo "[KRIITILINE] Sertifikaat on AEGUNUD!"
    PROBLEEME=$((PROBLEEME + 1))
fi

# Kas aegub 30 päeva jooksul?
if ! openssl x509 -in "$CERT" -noout -checkend 2592000 > /dev/null 2>&1; then
    echo "[HOIATUS] Sertifikaat aegub 30 päeva jooksul"
    PROBLEEME=$((PROBLEEME + 1))
fi
SKRIPT

chmod +x kontrolli.sh
```

### Samm 2: Lisa võtme tugevuse ja algoritmi kontroll

```bash
cat >> kontrolli.sh << 'SKRIPT'

# Kas võti on piisavalt tugev?
KEYSIZE=$(openssl x509 -in "$CERT" -noout -text | grep "Public-Key" | grep -oE '[0-9]+')
if [ "$KEYSIZE" -lt 2048 ] 2>/dev/null; then
    echo "[HOIATUS] Võtme suurus ($KEYSIZE) on alla 2048 biti"
    PROBLEEME=$((PROBLEEME + 1))
fi

# Kas kasutab nõrka SHA-1?
SIG=$(openssl x509 -in "$CERT" -noout -text | grep "Signature Algorithm" | head -1)
if echo "$SIG" | grep -qi "sha1"; then
    echo "[HOIATUS] Kasutab nõrka SHA-1 allkirja algoritmi"
    PROBLEEME=$((PROBLEEME + 1))
fi

# Kas ise-allkirjastatud?
SUBJ=$(openssl x509 -in "$CERT" -noout -subject)
ISSU=$(openssl x509 -in "$CERT" -noout -issuer)
if [ "$SUBJ" = "$ISSU" ]; then
    echo "[INFO] See on ise-allkirjastatud sertifikaat"
fi

echo ""
if [ $PROBLEEME -eq 0 ]; then
    echo "Tulemus: Probleeme ei leitud"
else
    echo "Tulemus: Leiti $PROBLEEME probleemi"
fi
SKRIPT
```

### Samm 3: Proovi

```bash
./kontrolli.sh kogutud/google.crt
```

---

## Osa 5: Mitme serveri skaneerimine

Nüüd skaneerime mitut serverit korraga.

### Samm 1: Loo serverite nimekiri ja tsükkel

```bash
cat > skaneeri.sh << 'SKRIPT'
#!/bin/bash

SERVERID="google.com github.com facebook.com amazon.com"

echo "Sertifikaatide skaneerimine"
echo "=========================="

for SERVER in $SERVERID; do
    echo ""
    echo "--- $SERVER ---"
    CERT_INFO=$(echo | openssl s_client -connect "$SERVER:443" -servername "$SERVER" 2>/dev/null)

    if [ -z "$CERT_INFO" ]; then
        echo "  Ei saa ühendust!"
        continue
    fi

    echo "$CERT_INFO" | openssl x509 -noout -subject -issuer -dates 2>/dev/null | sed 's/^/  /'
done
SKRIPT

chmod +x skaneeri.sh
```

`sed 's/^/  /'` lisab iga rea ette tühikud, et väljund oleks loetavam.

### Samm 2: Jooksuta

```bash
./skaneeri.sh
```

Lisa oma lemmikservereid nimekirja ja proovi uuesti!

---

## Osa 6: Usaldusahela uurimine

Vaatame, kuidas usaldusahel välja näeb päris serveril.

### Samm 1: Kuva kogu ahel

```bash
echo | openssl s_client -connect google.com:443 -servername google.com -showcerts 2>/dev/null
```

`-showcerts` näitab kõiki ahelasse kuuluvaid sertifikaate, mitte ainult esimest.

### Samm 2: Eralda sertifikaadid failidesse

```bash
echo | openssl s_client -connect google.com:443 -servername google.com -showcerts 2>/dev/null | \
    awk '/BEGIN CERTIFICATE/,/END CERTIFICATE/{ if(/BEGIN CERTIFICATE/){a++}; print > "ahel_"a".crt"}'
```

See `awk` käsk lõikab iga `BEGIN/END CERTIFICATE` ploki eraldi failiks.

### Samm 3: Uuri iga sertifikaati

```bash
for cert in ahel_*.crt; do
    echo "=== $cert ==="
    openssl x509 -in "$cert" -noout -subject -issuer
    echo ""
done
```

Näed, kuidas iga sertifikaat on allkirjastatud järgmise poolt - see ongi usaldusahel.

### Koristus

```bash
rm -f ahel_*.crt
```

---

## Osa 7: CT logide päring

Certificate Transparency logidest saab uurida, mis sertifikaate on domeenile kunagi väljastatud. See on kasulik anomaaliate tuvastamiseks - kui näed sertifikaati, mida sina ei taotlenud, on probleem.

```bash
DOMEEN="postimees.ee"
curl -s "https://crt.sh/?q=%.$DOMEEN&output=json" | \
    python3 -c "
import json, sys
data = json.load(sys.stdin)
for cert in data[:10]:
    print(f\"ID: {cert['id']}, Nimi: {cert['common_name']}, Väljastaja: {cert['issuer_name'][:50]}...\")
"
```

`crt.sh` on tasuta CT logi otsimootor. JSON väljund annab meile struktureeritud andmed, mida saab skriptiga töödelda.

---

## Osa 8: Lihtne monitooringuskript

Loo skript, mis kontrollib serverite sertifikaate ja logib tulemused. Seda saab hiljem croniga ajastada.

### Samm 1: Skripti alus

```bash
cat > monitor.sh << 'SKRIPT'
#!/bin/bash

SERVERID="google.com github.com"
HOIATA_PAEVI=30
LOGIFAIL="/tmp/sert_monitor.log"

echo "$(date): Alustan monitooringut" >> "$LOGIFAIL"
SKRIPT

chmod +x monitor.sh
```

### Samm 2: Lisa kontrollitsükkel

```bash
cat >> monitor.sh << 'SKRIPT'

for SERVER in $SERVERID; do
    AEGUB=$(echo | openssl s_client -connect "$SERVER:443" -servername "$SERVER" 2>/dev/null | \
        openssl x509 -noout -enddate 2>/dev/null | cut -d= -f2)

    if [ -z "$AEGUB" ]; then
        echo "$(date): VIGA - Ei saa ühendust: $SERVER" >> "$LOGIFAIL"
        continue
    fi

    AEGUB_SEK=$(date -d "$AEGUB" +%s 2>/dev/null)
    PRAEGU_SEK=$(date +%s)
    PAEVI=$(( (AEGUB_SEK - PRAEGU_SEK) / 86400 ))

    if [ "$PAEVI" -lt 0 ]; then
        echo "$(date): KRIITILINE - $SERVER on AEGUNUD!" >> "$LOGIFAIL"
    elif [ "$PAEVI" -lt "$HOIATA_PAEVI" ]; then
        echo "$(date): HOIATUS - $SERVER aegub $PAEVI päeva pärast" >> "$LOGIFAIL"
    else
        echo "$(date): OK - $SERVER ($PAEVI päeva)" >> "$LOGIFAIL"
    fi
done
SKRIPT
```

### Samm 3: Jooksuta ja vaata tulemust

```bash
./monitor.sh
cat /tmp/sert_monitor.log
```

Croniga ajastamiseks lisa rida `crontab -e` käsuga:

```
0 8 * * * /home/kasutaja/sert-labor/monitor.sh
```

See jooksutab skripti iga päev kell 8 hommikul.

---

## Kokkuvõte

Selles praktikumis sa:

- Kogusid sertifikaate erinevatelt saitidelt
- Lõid raportiskripti sertifikaadi analüüsimiseks
- Ehitasid probleemide tuvastamise tööriista
- Skaneerisid mitut serverit korraga
- Uurisid usaldusahelaid
- Lõid monitooringuskripti

Need oskused on turvaauditi ja igapäevase halduse jaoks olulised.
