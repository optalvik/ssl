# Labor 2 - Sertifikaadi detektiiv

## Mida me teeme

Selles laboris õpid sa sertifikaate analüüsima nagu turvaekspert. Sa uurid päris sertifikaate internetist, võrdled neid, otsid probleeme ja lood tööriistu automaatseks kontrollimiseks.

---

## Osa 1: Sertifikaatide kogumine

Alustame sertifikaatide kogumisest erinevatelt saitidelt:

```bash
mkdir -p ~/sert-labor/kogutud
cd ~/sert-labor

# Kogu Google'i sertifikaat
echo | openssl s_client -connect google.com:443 -servername google.com 2>/dev/null | \
    openssl x509 -outform PEM > kogutud/google.crt

# Kogu GitHub'i sertifikaat  
echo | openssl s_client -connect github.com:443 -servername github.com 2>/dev/null | \
    openssl x509 -outform PEM > kogutud/github.crt

# Kogu mõne Eesti saidi sertifikaat
echo | openssl s_client -connect www.postimees.ee:443 -servername www.postimees.ee 2>/dev/null | \
    openssl x509 -outform PEM > kogutud/postimees.crt
```

`-servername` on oluline — see on SNI (Server Name Indication), mis ütleb serverile, millist sertifikaati sa tahad. Paljudel serveritel on mitu saiti sama IP peal.

---

## Osa 2: Põhiinfo uurimine

Vaatame, mis neis sertifikaatides kirjas on:

```bash
# Kes on subjekt (kellele sertifikaat kuulub)
for cert in kogutud/*.crt; do
    echo "=== $cert ==="
    openssl x509 -in "$cert" -noout -subject
done

# Kes on väljastaja (CA)
for cert in kogutud/*.crt; do
    echo "=== $cert ==="
    openssl x509 -in "$cert" -noout -issuer
done

# Millal aegub
for cert in kogutud/*.crt; do
    echo "=== $cert ==="
    openssl x509 -in "$cert" -noout -dates
done
```

---

## Osa 3: Sertifikaadi raport

Loome skripti, mis teeb sertifikaadist põhjaliku raporti:

```bash
cat > raport.sh << 'EOF'
#!/bin/bash

if [ -z "$1" ]; then
    echo "Kasutus: $0 sertifikaat.crt"
    exit 1
fi

CERT="$1"

echo "========================================"
echo "SERTIFIKAADI RAPORT"
echo "========================================"
echo ""

echo "--- PÕHIINFO ---"
openssl x509 -in "$CERT" -noout -subject -issuer -serial

echo ""
echo "--- KEHTIVUS ---"
openssl x509 -in "$CERT" -noout -dates

# Kontrolli, kas kehtib
if openssl x509 -in "$CERT" -noout -checkend 0 > /dev/null 2>&1; then
    echo "Staatus: KEHTIV"
else
    echo "Staatus: AEGUNUD!"
fi

# Päevi aegumiseni
AEGUB=$(openssl x509 -in "$CERT" -noout -enddate | cut -d= -f2)
AEGUB_SEK=$(date -d "$AEGUB" +%s 2>/dev/null || date -j -f "%b %d %T %Y %Z" "$AEGUB" +%s 2>/dev/null)
PRAEGU_SEK=$(date +%s)
PAEVI=$(( (AEGUB_SEK - PRAEGU_SEK) / 86400 ))
echo "Päevi aegumiseni: $PAEVI"

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

EOF

chmod +x raport.sh
```

Proovi:

```bash
./raport.sh kogutud/google.crt
./raport.sh kogutud/github.crt
```

---

## Osa 4: Sertifikaatide võrdlus

Loome tööriista kahe sertifikaadi võrdlemiseks:

```bash
cat > vordle.sh << 'EOF'
#!/bin/bash

if [ "$#" -ne 2 ]; then
    echo "Kasutus: $0 sert1.crt sert2.crt"
    exit 1
fi

CERT1="$1"
CERT2="$2"

echo "Võrdlen: $CERT1 vs $CERT2"
echo ""

# Väljastaja
echo "--- VÄLJASTAJA ---"
ISSUER1=$(openssl x509 -in "$CERT1" -noout -issuer)
ISSUER2=$(openssl x509 -in "$CERT2" -noout -issuer)
echo "$CERT1: $ISSUER1"
echo "$CERT2: $ISSUER2"
if [ "$ISSUER1" = "$ISSUER2" ]; then
    echo "=> SAMA väljastaja"
else
    echo "=> ERINEV väljastaja"
fi

echo ""
echo "--- VÕTME SUURUS ---"
SIZE1=$(openssl x509 -in "$CERT1" -noout -text | grep "Public-Key" | grep -oE '[0-9]+')
SIZE2=$(openssl x509 -in "$CERT2" -noout -text | grep "Public-Key" | grep -oE '[0-9]+')
echo "$CERT1: $SIZE1 bitti"
echo "$CERT2: $SIZE2 bitti"

echo ""
echo "--- KEHTIVUSAEG ---"
PAEVI1=$( (openssl x509 -in "$CERT1" -noout -enddate | cut -d= -f2 | xargs -I{} date -d {} +%s 2>/dev/null) || echo "0")
PAEVI2=$( (openssl x509 -in "$CERT2" -noout -enddate | cut -d= -f2 | xargs -I{} date -d {} +%s 2>/dev/null) || echo "0")
PRAEGU=$(date +%s)
echo "$CERT1: $(( (PAEVI1 - PRAEGU) / 86400 )) päeva"
echo "$CERT2: $(( (PAEVI2 - PRAEGU) / 86400 )) päeva"

EOF

chmod +x vordle.sh
./vordle.sh kogutud/google.crt kogutud/github.crt
```

---

## Osa 5: Probleemide tuvastamine

Loome skripti, mis otsib sertifikaadist võimalikke probleeme:

```bash
cat > kontrolli.sh << 'EOF'
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

# Kas aegub varsti (30 päeva)?
if ! openssl x509 -in "$CERT" -noout -checkend 2592000 > /dev/null 2>&1; then
    echo "[HOIATUS] Sertifikaat aegub 30 päeva jooksul"
    PROBLEEME=$((PROBLEEME + 1))
fi

# Kas võti on piisavalt tugev?
KEYSIZE=$(openssl x509 -in "$CERT" -noout -text | grep "Public-Key" | grep -oE '[0-9]+')
if [ "$KEYSIZE" -lt 2048 ] 2>/dev/null; then
    echo "[HOIATUS] Võtme suurus ($KEYSIZE) on alla 2048 biti"
    PROBLEEME=$((PROBLEEME + 1))
fi

# Kas kasutab SHA-1 (nõrk)?
SIG=$(openssl x509 -in "$CERT" -noout -text | grep "Signature Algorithm" | head -1)
if echo "$SIG" | grep -qi "sha1"; then
    echo "[HOIATUS] Kasutab nõrka SHA-1 allkirja algoritmi"
    PROBLEEME=$((PROBLEEME + 1))
fi

# Kas on ise-allkirjastatud?
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

EOF

chmod +x kontrolli.sh
./kontrolli.sh kogutud/google.crt
```

---

## Osa 6: Massiline skaneerimine

Skaneeri mitu serverit korraga:

```bash
cat > skaneeri.sh << 'EOF'
#!/bin/bash

SERVERID="google.com github.com facebook.com amazon.com microsoft.com"

echo "Sertifikaatide skaneerimine"
echo "=========================="
echo ""

for SERVER in $SERVERID; do
    echo "--- $SERVER ---"
    
    # Proovi ühenduda
    CERT_INFO=$(echo | openssl s_client -connect "$SERVER:443" -servername "$SERVER" 2>/dev/null)
    
    if [ -z "$CERT_INFO" ]; then
        echo "  Ei saa ühendust!"
        continue
    fi
    
    # Ekstrakti info
    echo "$CERT_INFO" | openssl x509 -noout -subject -issuer -dates 2>/dev/null | sed 's/^/  /'
    
    echo ""
done

EOF

chmod +x skaneeri.sh
./skaneeri.sh
```

---

## Osa 7: Usaldusahela kontrollimine

Vaatame, kuidas usaldusahel välja näeb:

```bash
# Näita kogu ahel
echo | openssl s_client -connect google.com:443 -servername google.com -showcerts 2>/dev/null

# Loe ahelast sertifikaadid välja
echo | openssl s_client -connect google.com:443 -servername google.com -showcerts 2>/dev/null | \
    awk '/BEGIN CERTIFICATE/,/END CERTIFICATE/{ if(/BEGIN CERTIFICATE/){a++}; print > "ahel_"a".crt"}'

# Vaata, mitu sertifikaati ahelas on
ls ahel_*.crt 2>/dev/null

# Uuri igaühte
for cert in ahel_*.crt; do
    echo "=== $cert ==="
    openssl x509 -in "$cert" -noout -subject -issuer
done

# Koristus
rm -f ahel_*.crt
```

---

## Osa 8: CT logide päring

Certificate Transparency logidest saab uurida, mis sertifikaate on domeenile väljastatud:

```bash
# Kasuta crt.sh API-t
DOMEEN="postimees.ee"

echo "Otsing domeenile: $DOMEEN"
curl -s "https://crt.sh/?q=%.$DOMEEN&output=json" | \
    python3 -c "
import json, sys
data = json.load(sys.stdin)
for cert in data[:10]:  # Näita 10 esimest
    print(f\"ID: {cert['id']}, Nimi: {cert['common_name']}, Väljastaja: {cert['issuer_name'][:50]}...\")
"
```

See näitab, mis sertifikaate on sellele domeenile kunagi väljastatud. Kasulik anomaaliate tuvastamiseks — kui näed sertifikaati, mida sina ei taotlenud, on probleem.

---

## Osa 9: Automatiseeritud monitooring

Loo skript, mida saab croni-ga jooksutada:

```bash
cat > monitor.sh << 'EOF'
#!/bin/bash

SERVERID="google.com github.com minusait.ee"
HOIATA_PAEVI=30
LOGIFAIL="/tmp/sert_monitor.log"

echo "$(date): Alustan monitooringut" >> "$LOGIFAIL"

for SERVER in $SERVERID; do
    # Saa sertifikaat
    AEGUB=$(echo | openssl s_client -connect "$SERVER:443" -servername "$SERVER" 2>/dev/null | \
        openssl x509 -noout -enddate 2>/dev/null | cut -d= -f2)
    
    if [ -z "$AEGUB" ]; then
        echo "$(date): VIGA - Ei saa ühendust: $SERVER" >> "$LOGIFAIL"
        continue
    fi
    
    # Arvuta päevad
    AEGUB_SEK=$(date -d "$AEGUB" +%s 2>/dev/null)
    PRAEGU_SEK=$(date +%s)
    PAEVI=$(( (AEGUB_SEK - PRAEGU_SEK) / 86400 ))
    
    if [ "$PAEVI" -lt 0 ]; then
        echo "$(date): KRIITILINE - $SERVER sertifikaat on AEGUNUD!" >> "$LOGIFAIL"
    elif [ "$PAEVI" -lt "$HOIATA_PAEVI" ]; then
        echo "$(date): HOIATUS - $SERVER aegub $PAEVI päeva pärast" >> "$LOGIFAIL"
    else
        echo "$(date): OK - $SERVER ($PAEVI päeva)" >> "$LOGIFAIL"
    fi
done

EOF

chmod +x monitor.sh
./monitor.sh
cat /tmp/sert_monitor.log
```

---

## Kokkuvõte

Selles laboris sa:
- Kogusid sertifikaate erinevatelt saitidelt
- Lõid analüüsitööriistu
- Õppisid tuvastama probleeme
- Uurisid usaldusahelaid
- Lõid monitooringuskripti

Need oskused on turvaauditi ja igapäevase halduse jaoks olulised. Kui midagi läheb valesti, tead nüüd, kuidas uurida.
