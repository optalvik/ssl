# Osa 12 - Kvantarvutid ja post-quantum krüptograafia

## Oht silmapiiril

Kõik, millest me seni rääkisime — RSA, elliptilised kõverad, Diffie-Hellman — põhineb matemaatilistel probleemidel, mida tavalised arvutid ei suuda kiiresti lahendada. Suurte arvude tegurdamine, diskreetne logaritm — need on rasked probleemid.

Aga kvantarvutid pole tavalised arvutid. Nad kasutavad kvantmehaanikat — superpositsiooni, põimumist — viisil, mis muudab mõned rasked probleemid lihtsaks.

1994. aastal näitas Peter Shor, et kvantarvuti suudaks faktoreerida suuri arve eksponentsiaalselt kiiremini kui tavaline arvuti. See tähendab, et piisavalt võimas kvantarvuti murrab RSA minutitega. Sama kehtib elliptiliste kõverate ja Diffie-Hellmani kohta.

![Post-quantum krüptograafia selgitus](https://www.paloaltonetworks.com/content/dam/pan/en_US/images/cyberpedia/what-is-post-quantum-cryptography-pqc/Post-quantum-cryptography-explained-new.png?imwidth=800)

## Millal see juhtub?

Keegi ei tea täpselt. Praegused kvantarvutid on eksperimentaalsed — sajad kuni tuhanded kubitid, palju müra, ebastabiilsed. RSA murdmiseks on vaja miljoneid stabiilseid kubitte.

Eksperdid hindavad, et "Q-päev" — päev, mil kvantarvutid suudavad murda praegust krüptograafiat — võib saabuda 2030. ja 2040. aasta vahel. Võib-olla varem, võib-olla hiljem.

![Kvantarvutite arengu ajajoon](https://media.licdn.com/dms/image/v2/D5622AQH_NpK-zOb7cw/feedshare-shrink_800/feedshare-shrink_800/0/1723105913913?e=2147483647&v=beta&t=jyb5C_PSnAHgD42izoBqXaFFnzJnS3sqU76cF4uwDTs)

> **⚠️ HOIATUS: "Kogu nüüd, dekrüpteeri hiljem"**
> 
> Üks rünnak toimub juba praegu. Ründajad salvestavad täna krüpteeritud liiklust. Nad ei saa seda lugeda, aga nad hoiavad alles. Kui kunagi kvantarvutid saabuvad, saavad nad vana liikluse lahti teha.
> 
> Andmed, mis peavad jääma saladusse aastakümneteks — meditsiiniandmed, riigisaladused, ärisaladused — on juba ohus.

## Mis jääb turvaliseks?

Hea uudis: mitte kõik krüptograafia pole ohus.

> **✅ TURVALINE kvantarvutite vastu:**
> - **AES-256** — sümmeetriline krüpteerimine jääb turvaliseks
> - **SHA-256, SHA-3** — räsifunktsioonid on vastupidavad
> 
> **❌ HAAVATAV kvantarvutite poolt:**
> - **RSA** — murduv Shori algoritmiga
> - **ECDSA, ECDH** — elliptilised kõverad murduvad
> - **Diffie-Hellman** — klassikaline võtmevahetus murduv

Sümmeetriline krüpteerimine (AES) on suhteliselt turvaline. Grover'i algoritm kiirendab brute-force rünnakuid, aga ainult ruutjuure võrra. AES-256 muutub efektiivselt AES-128 tugevuseks, mis on endiselt piisav.

Probleem on avaliku võtme krüptograafiaga: RSA, ECDSA, ECDH, DSA. Kõik need murduvad.

## Post-quantum krüptograafia

PQC ehk post-quantum krüptograafia on uute algoritmide perekond, mis on disainitud vastu pidama kvantarvutitele. Need põhinevad teistsugustel matemaatilistel probleemidel — probleemidel, mida ka kvantarvutid ei suuda kiiresti lahendada.

NIST (USA standardiorganisatsioon) korraldas aastaid kestnud konkursi ja valis 2024. aastal välja uued standardid.

![PQC kaitseb andmeid](https://gsa-media.s3.us-east-2.amazonaws.com/gsaglobal/wp-content/uploads/2025/07/pqc-protected-page-800x248.png)

## Uued algoritmid

> **📋 NIST PQC standardid (2024):**
> 
> | Algoritm | Funktsioon | Asendab |
> |----------|------------|---------|
> | **ML-KEM (Kyber)** | Võtmevahetus | ECDH, Diffie-Hellman |
> | **ML-DSA (Dilithium)** | Digitaalallkiri | RSA, ECDSA |
> | **SLH-DSA (SPHINCS+)** | Digitaalallkiri (räsipõhine) | RSA, ECDSA |
> | **FN-DSA (FALCON)** | Digitaalallkiri (kompaktne) | RSA, ECDSA |

**ML-KEM (Kyber)** on võtmevahetuse algoritm. Seal, kus praegu kasutatakse ECDH-d, kasutatakse tulevikus ML-KEM-i. See põhineb võreprobleemidel (lattice problems) — matemaatikal, mis on raske nii tavalistele kui kvantarvutitele.

**ML-DSA (Dilithium)** on digitaalallkirja algoritm. Seal, kus praegu kasutatakse RSA-d või ECDSA-d sertifikaatide allkirjastamiseks, kasutatakse tulevikus ML-DSA-d. Samuti võrepõhine.

**SLH-DSA (SPHINCS+)** on alternatiivne allkirjaalgoritm, mis põhineb räsifunktsioonidel. See on konservatiivsem valik — räsifunktsioone mõistame väga hästi. Aga allkirjad on suuremad.

**FN-DSA (FALCON)** on veel üks allkirjaalgoritm, kompaktsema allkirjaga kui Dilithium, aga keerulisema implementatsiooniga.

## Mis muutub?

PQC algoritmid töötavad, aga nad on teistsugused. Võtmed ja allkirjad on suuremad.

> **📊 Suuruse võrdlus:**
> 
> | Kategooria | Klassikaline | Postkvant | Kasv |
> |------------|--------------|-----------|------|
> | Avalik võti | RSA-2048: 256 B | ML-KEM-768: ~1200 B | ~5x |
> | Allkiri | ECDSA: ~70 B | ML-DSA-65: ~3300 B | ~47x |

See tähendab, et mõned süsteemid vajavad kohandamist. Seadmed piiratud mäluga, aeglased võrguühendused, protokollid, mis eeldavad väikseid sertifikaate — need võivad probleeme tekitada.

## Hübriidne lähenemine

> **💡 PARIM PRAKTIKA: Hübriidne krüptograafia**
> 
> Üleminekuperioodil kasutatakse **klassikaline + PQC** koos:
> ```
> X25519 + ML-KEM-768 = Hübriidne võtmevahetus
> ```
> Kui üks algoritm osutub nõrgaks, kaitseb teine.

TLS käepigistuses kasutatakse näiteks X25519 + ML-KEM-768. Mõlemad teevad võtmevahetuse, mõlema tulemused kombineeritakse.

See on konservatiivne lähenemine. PQC algoritmid on uued, neid pole nii kaua analüüsitud kui RSA-d. Hübriidne lahendus annab kindlustunde.

## Mis praegu toimub?

> **🌐 Tööstuse hetkeseis:**
> 
> - **Chrome, Firefox** — hübriidne PQC vaikimisi lubatud
> - **Cloudflare, Google** — serveripoolne tugi olemas
> - **TLS 1.3** — toetab hübriidset võtmevahetust
> 
> See on märkamatu — kui sinu brauser ja server toetavad, kasutatakse PQC-d automaatselt.

Sertifikaadid on keerulisem lugu. PQC sertifikaadid on suuremad ja CA-d alles alustavad nende väljastamist. Üleminek võtab aega.

## Mida sina peaksid tegema?

> **📝 TEGEVUSKAVA:**
> 
> **Tavaline veebileht?**
> - Hoia tarkvara värske — PQC-tugi tuleb automaatselt
> 
> **Pikaajalised tundlikud andmed?**
> 1. Inventeeri, kus kasutatakse RSA/ECDSA
> 2. Prioritiseeri süsteemid, kus andmed peavad kaua salajas püsima
> 3. Planeeri üleminek hübriidsele PQC-le
> 
> **Uued süsteemid?**
> - Kaalu hübriidset lähenemist juba praegu

## Kokkuvõte

> **🔑 VÕTMEPUNKTID:**
> 
> - Kvantarvutid murravad RSA, ECDSA, ECDH — Q-päev tuleb 2030-2040
> - AES-256 ja SHA-256 jäävad turvaliseks
> - PQC standardid on olemas: ML-KEM, ML-DSA
> - Hübriidne lähenemine on parim praktika üleminekuks
> - "Kogu nüüd, dekrüpteeri hiljem" — oht on juba praegu reaalne

Praktiline soovitus: hoia silm peal, hoia tarkvara värske, ja kui haldad tundlikke andmeid, alusta PQC planeerimist.
