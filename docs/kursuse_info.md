---
tags:
  - TLS
  - Turvalisus
---

# Kursuse info

**Usalda Tabalukku — SSL/TLS ja sertifikaadid praktikas**

| | |
|---|---|
| **Autor** | Maria Talvik · IT-õpetaja |
| **Sihtrühm** | IT-süsteemide nooremspetsialist (tase 4) |
| **Maht** | ~8 akadeemilist tundi (+ ~4h boonusmaterjale) |
| **Hindamine** | Kujundav (enesekontrolliküsimused + praktikumid) |
| **Litsents** | [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) |

## Eesmärk

Kursuse läbinu mõistab TLS/SSL protokollide ja PKI toimimist ning suudab iseseisvalt luua, hallata ja diagnoosida sertifikaate.

## Eeldused

Arvutikasutuse baasteadmised, Linuxi käsurea tundmine (kataloogid, failid), arvutivõrkude alused (IP, DNS, HTTP).

## Seos õppekavaga

Info- ja kommunikatsioonitehnoloogia erialade riiklik õppekava (RT I, 14.04.2020, 3):

| Moodul | EKAP | Seos kursusega |
|--------|------|----------------|
| Küberturvalisus | 8 | Sertifikaadid, krüpteerimine, PKI |
| Arvutivõrgud | 16 | TLS/SSL, HTTPS, turvaline ühendus |
| Rakendusserverid | 8 | Sertifikaatide paigaldamine, HTTPS |

## Struktuur ja maht

| Osa | Maht | Teemad |
|-----|------|--------|
| Teooria (osad 0–9) | 5 h | SSL/TLS, PKI, käepigistus, sertifikaadid, vormingud, CA, elutsükkel, automatiseerimine |
| Teooria (osa 10) | 1 h | Tõrkeotsing ja diagnostika |
| Praktikum 1 | 1 h | Oma CA loomine, sertifikaadid, HTTPS server |
| Praktikum 2 | 1 h | Sertifikaatide analüüs ja monitooring |

*Boonusmaterjalid (~4h) — mTLS, post-quantum krüptograafia, Eesti PKI, HashiCorp Vault labor, PQC labor.*

## Õpiväljundid

| Nr | Kursuse läbinu... | Bloomi tase | Osad |
|----|-------------------|-------------|------|
| 1 | selgitab SSL/TLS ja HTTPS-i tööpõhimõtteid ning eristab versioone | mõistmine | 0–1 |
| 2 | kirjeldab PKI toimimist, CA rolli ja usaldusahelat | mõistmine | 2 |
| 3 | analüüsib TLS käepigistuse etappe | analüüs | 3 |
| 4 | loob sertifikaate OpenSSL-iga (CA, serveri-, kliendisertifikaadid) | rakendamine | 4, P1 |
| 5 | seadistab HTTPS veebiserveri | rakendamine | P1 |
| 6 | eristab sertifikaadivorminguid (PEM, DER, P12, JKS) ja teisendab neid | rakendamine | 5 |
| 7 | hindab sisemiste ja väliste CA kasutusjuhte | hindamine | 6 |
| 8 | rakendab automatiseeritud sertifikaadihaldust (Let's Encrypt) | rakendamine | 9 |
| 9 | diagnoosib levinumaid TLS/sertifikaadivigu | analüüs | 10, P2 |

## Üldpädevused

| Pädevus | Seos |
|---------|------|
| Digipädevus | Turvaline andmeedastus, sertifikaadihaldus |
| Õpipädevus | Iseseisev töö, probleemilahendus |
| Matemaatikapädevus | Krüptograafia alused |
| Suhtluspädevus | Erialane terminoloogia eesti ja inglise keeles |

## Hindamine ja tagasiside

Iga teooriaosa lõpus on enesekontrolliküsimused (kokkupandavad), praktikumid sisaldavad kontrollitava tulemusega ülesandeid. Spikker toetab iseseisvat kordamist.

## Kuidas kasutada

- **Järjest** — alusta osast 0, tee praktikumid pärast vastavat teooriat
- **Teemapõhiselt** — mine otse vajaliku osa juurde
- **Spikrina** — kiire meeldetuletus igapäevatöös
- **Õpetaja juhendamisel** — sobib kontaktõppe toetamiseks

## Tehnilised nõuded

Materjal töötab kõigis kaasaegsetes veebilehitsejates (Chrome, Firefox, Safari, Edge) ja on mobiilisõbralik. Praktikumide jaoks on vajalik Linuxi käsurida (võib kasutada ka WSL-i või virtuaalmasinat).
