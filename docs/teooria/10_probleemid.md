# Osa 10 - TLS probleemide lahendamine

## Kui midagi ei tööta

TLS probleemid on salakavalad. Veatead on sageli krüptilised, põhjused peidetud. "Handshake failed" ei ütle sulle, miks käepigistus ebaõnnestus. "Certificate verify failed" ei ütle, mis sertifikaadiga täpselt viga on.

Aga mustrid on olemas. Samad vead korduvad, samad põhjused ilmnevad. Kui tead, kuhu vaadata, saad probleemid tavaliselt kiiresti lahendatud.

> **🔧 DIAGNOSTIKA TÖÖRIISTAD:**
> 
> | Tööriist | Kasutus |
> |----------|---------|
> | `openssl s_client` | TLS ühenduse testimine |
> | `openssl x509` | Sertifikaadi analüüs |
> | `openssl verify` | Usaldusahela kontroll |
> | `curl -v` | HTTP(S) päringute debug |
> | `testssl.sh` | Põhjalik turvaanalüüs |

---

## Sertifikaat aegunud

> **❌ VIGA:** `certificate has expired`

See on klassika. Kõik töötas, siis äkki enam mitte. Kasutajad näevad brauseris punast hoiatust.

**Diagnoos:**
```bash
# Vaata sertifikaadi kuupäevi
openssl x509 -in sertifikaat.crt -noout -dates

# Või serverist otse
echo | openssl s_client -connect server:443 2>/dev/null | \
    openssl x509 -noout -dates
```

**Lahendus:** Hangi uus sertifikaat.

**Ennetamine:** Seadista monitooring, mis hoiatab 30/14/7 päeva ette.

---

## Vale hostname

> **❌ VIGA:** `hostname mismatch` / `certificate is not valid for this name`

Sertifikaat on kehtiv, aga nimi ei klapi. See juhtub, kui sertifikaadi CN või SAN ei vasta aadressile.

**Diagnoos:**
```bash
# Vaata, mis nimed sertifikaadis on
openssl x509 -in sertifikaat.crt -noout -text | grep -A1 "Subject Alternative Name"
```

> **💡 NÄIDE:**
> 
> Sul on sertifikaat `www.minusait.ee` jaoks, aga keegi üritab minna `minusait.ee` (ilma www).
> 
> **Lahendus:** Lisa SAN-i mõlemad nimed või kasuta wildcard `*.minusait.ee`

---

## Usaldusahel katki

> **❌ VIGA:** `unable to get local issuer certificate` / `certificate verify failed`

Kõige sagedamini on server seadistatud saatma ainult lõppsertifikaati, aga mitte vahesertifikaati.

**Diagnoos:**
```bash
# Vaata tervet ahelat
openssl s_client -connect server:443 -showcerts

# Kontrolli, kas ahel on terve
openssl verify -CAfile ca.crt -untrusted intermediate.crt server.crt
```

**Lahendus:**
```bash
# Kombineeri sertifikaadid õiges järjekorras
cat server.crt intermediate.crt > fullchain.crt
```

> **📋 AHELA JÄRJEKORD:**
> ```
> 1. Serveri sertifikaat     (server.crt)
> 2. Vahesertifikaat         (intermediate.crt)
> 3. [Veel vahesertifikaate kui on]
> # Juur-CA EI PANDA kaasa (brauseril juba olemas)
> ```

---

## Privaatvõti ei klapi

> **❌ VIGA:** `key values mismatch` / `private key does not match certificate`

Server on konfigureeritud kasutama privaatvõtit, mis ei kuulu sertifikaadi juurde.

**Diagnoos:**
```bash
# Võrdle moduluseid - PEAVAD olema SAMAD
openssl rsa -noout -modulus -in server.key | openssl md5
openssl x509 -noout -modulus -in server.crt | openssl md5
```

> **⚠️ TÜÜPILINE PÕHJUS:**
> 
> Kiirustades tehtud sertifikaadi uuendamine — kasutati vana võtmefaili uue sertifikaadiga.

**Lahendus:** Leia õige privaatvõti või genereeri uus paar ja hangi uus sertifikaat.

---

## TLS versioon ei sobi

> **❌ VIGA:** `protocol version mismatch` / `no protocols available`

Vanad kliendid ei toeta uusi TLS versioone. Uued serverid ei toeta vanu.

**Diagnoos:**
```bash
# Testi konkreetset TLS versiooni
openssl s_client -connect server:443 -tls1_2
openssl s_client -connect server:443 -tls1_3
```

> **📊 TLS VERSIOONID:**
> 
> | Versioon | Staatus | Toeta? |
> |----------|---------|--------|
> | SSL 3.0 | Surnud | ❌ Mitte kunagi |
> | TLS 1.0 | Aegunud | ❌ Keela |
> | TLS 1.1 | Aegunud | ❌ Keela |
> | TLS 1.2 | OK | ✅ Minimaalne |
> | TLS 1.3 | Parim | ✅ Eelistatud |

---

## Cipher mismatch

> **❌ VIGA:** `no shared cipher` / `handshake failure`

Server ja klient ei leia ühist krüpteerimisalgoritmi.

**Diagnoos:**
```bash
# Vaata, mis ciphereid server toetab
nmap --script ssl-enum-ciphers -p 443 server

# Või testssl.sh-ga
./testssl.sh server:443
```

---

## Kiire debugimise spikker

> **🚀 KIIRE DIAGNOSTIKA:**
> 
> ```bash
> # 1. Kas server vastab üldse?
> curl -v https://server/ 2>&1 | head -30
> 
> # 2. Mis sertifikaadi server saadab?
> echo | openssl s_client -connect server:443 2>/dev/null | \
>     openssl x509 -noout -subject -issuer -dates
> 
> # 3. Kas ahel on terve?
> echo | openssl s_client -connect server:443 -CAfile ca.crt
> 
> # 4. Kas võti ja sertifikaat klapivad?
> openssl rsa -noout -modulus -in server.key | md5sum
> openssl x509 -noout -modulus -in server.crt | md5sum
> 
> # 5. Põhjalik analüüs
> ./testssl.sh server:443
> ```

---

## Levinud veateated ja lahendused

> **📋 VEATEADETE SÕNASTIK:**
> 
> | Veateade | Tähendus | Lahendus |
> |----------|----------|----------|
> | `certificate has expired` | Sertifikaat aegunud | Uuenda sertifikaat |
> | `unable to get local issuer certificate` | CA puudub truststorest | Lisa CA või vahesertifikaat |
> | `certificate verify failed` | Midagi valesti sertifikaadiga | Kontrolli aegumist, nime, ahelat |
> | `self signed certificate` | Ise-allkirjastatud | Lisa truststoresse või hangi päris sert |
> | `hostname mismatch` | Nimi ei klapi | Hangi õige nimega sertifikaat |
> | `no shared cipher` | Šifrid ei ühildu | Kontrolli server/klient konfiguratsiooni |
> | `wrong version number` | TLS vs HTTP port | Kontrolli porti (443 vs 80) |
> | `key values mismatch` | Vale võtmepaar | Leia õige võti või genereeri uus |

---

## Java-spetsiifiline debug

> **☕ JAVA TLS DEBUG:**
> 
> ```bash
> # Lisa käivitusparameetritesse:
> -Djavax.net.debug=ssl,handshake
> 
> # Väljund näitab:
> # - Milliseid sertifikaate saadeti
> # - Milliseid CA-sid usaldati  
> # - Kus täpselt käepigistus katkes
> ```

---

## Ennetamine

> **✅ PARIMAD PRAKTIKAD:**
> 
> 1. **Monitooring** — jälgi sertifikaatide aegumist
> 2. **Automaatika** — Certbot, cert-manager, Vault
> 3. **Alertid** — 30/14/7 päeva hoiatused
> 4. **Testimine** — regulaarne testssl.sh skannimine
> 5. **Dokumentatsioon** — kirjuta üles, mis kus on

Parim debugging on see, mida pole vaja teha. Seadista süsteemid nii, et probleemid ei juhtuks.

Järgmises osas vaatame täiendavaid teemasid: mTLS, HSTS, Certificate Transparency ja muid.
