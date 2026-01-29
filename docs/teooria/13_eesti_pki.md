# Osa 13 - PKI Eestis: ID-kaart, Mobiil-ID ja SK

## Eesti kui digitaalne eeskuju

Eesti on üks maailma kõige digitaalsemaid riike. E-valimised, e-residentsus, digiretseptid, e-maksuamet — kõik see töötab sertifikaatide ja digitaalallkirjade peal. Kui sa oled Eestis ja kasutad ID-kaarti pangaga ühendumiseks või dokumendi allkirjastamiseks, kasutad sa täpselt seda PKI süsteemi, millest see kursus räägib.

Aga kes on need CA-d, keda Eesti usaldab? Kuidas ID-kaardi sertifikaat sinu seadmesse jõuab? Ja miks mõni välismaine teenus Eesti ID-kaarti ei tunnista?

## SK ID Solutions — Eesti sertifitseerimiskeskus

Eesti PKI selgroog on SK ID Solutions (varem tuntud kui AS Sertifitseerimiskeskus). See on eraettevõte, mis väljastab sertifikaate ID-kaartidele, Mobiil-ID-le ja Smart-ID-le.

SK ei ole riigiga lihtne. See on erasektori ettevõte, mida omavad Eesti pangad ja telekommunikatsiooniettevõtted. Aga riik usaldab SK-d — SK juursertifikaadid on Eesti riigi usaldusnimekirjas ja enamiku Euroopa riikide süsteemides tunnustatud.

SK haldab mitut CA-d:

**EE Certification Centre Root CA** — juur-CA, millele kõik muu toetub. Selle võtit hoitakse üliturvaliselt, offline.

**ESTEID-SK** — see CA väljastab sertifikaate ID-kaartidele. Iga ID-kaart sisaldab kahte sertifikaati: üks autentimiseks, teine digitaalallkirjastamiseks.

**ESTEID2018** — uuem versioon ID-kaardi sertifikaatidest.

**MID-SK** — Mobiil-ID sertifikaadid.

**NQ-SK** — kvalifitseeritud ajatemplite teenus.

## ID-kaart ja selle sertifikaadid

Eesti ID-kaart on rohkem kui plastikaart nimega. See on krüptograafiline seade, mis sisaldab:

**Autentimissertifikaat** — seda kasutatakse sinu isiku tõestamiseks. Kui sa logid sisse pangakontole ID-kaardiga, näitab sinu kaart seda sertifikaati pangale. Pank kontrollib, kas sertifikaat on kehtiv ja kas SK on selle allkirjastanud. Kui jah, siis pank teab, et sa oled sina.

**Allkirjastamissertifikaat** — seda kasutatakse digitaalallkirjastamiseks. Kui sa allkirjastad lepingut DigiDoc-is, kasutab tarkvara seda sertifikaati. Allkiri on matemaatiliselt seotud sinu privaatvõtmega, mis on kaardil ja ei lahku sealt kunagi.

Mõlemad sertifikaadid on seotud sinu isikukoodiga. Sertifikaadis on kirjas sinu nimi, isikukood ja kehtivusaeg. Aga privaatvõti on ainult kaardi kiibis — seda ei saa välja lugeda.

PIN-koodid kaitsevad neid sertifikaate. PIN1 on autentimiseks, PIN2 on allkirjastamiseks. Kui keegi saab sinu kaardi, aga ei tea PIN-koode, ei saa ta sinu nimel midagi teha.

## Kuidas ID-kaardiga autentimine töötab?

Kui sa logid sisse riigi teenusesse ID-kaardiga:

1. Veebileht küsib sinu brauserilt sertifikaati
2. Brauser küsib ID-kaardilt (läbi kaardilugeja) sertifikaati
3. Sina sisestad PIN1 koodi
4. Kaart annab sertifikaadi ja tõestab, et tal on vastav privaatvõti
5. Server kontrollib, kas sertifikaat on kehtiv (OCSP päring SK-le)
6. Kui kõik on korras, oled sisse logitud

See on mTLS (mutual TLS), millest rääkisime varem — mõlemad pooled tõestavad oma identiteeti.

## Mobiil-ID ja Smart-ID

Mobiil-ID kasutab sama põhimõtet, aga privaatvõti on SIM-kaardil, mitte ID-kaardil. Sertifikaadid väljastab SK, lihtsalt teine CA (MID-SK).

Smart-ID on uuem lahendus, kus võtmed on telefoni turvaelementides ja sertifikaadid pilves. See on mugavam (ei vaja erilist SIM-i), aga põhimõte on sama.

Mõlemad on Eestis ja paljudes Euroopa riikides tunnustatud digitaalallkirja vahenditena.

## eIDAS ja Euroopa tunnustamine

Eesti digitaalallkiri on tunnustatud kogu Euroopa Liidus tänu eIDAS määrusele. See Euroopa regulatsioon ütleb, et liikmesriigid peavad tunnustama teiste riikide kvalifitseeritud digitaalallkirju.

SK sertifikaadid on "kvalifitseeritud", mis tähendab, et nad vastavad eIDAS nõuetele. Seega, kui sa allkirjastad Eesti ID-kaardiga lepingu, on see juriidiliselt siduv kogu Euroopas.

Aga praktikas on nüansse. Mõni välismaine teenus ei pruugi Eesti ID-kaarti toetada — mitte sellepärast, et sertifikaat poleks kehtiv, vaid sellepärast, et nende tarkvara lihtsalt ei tea, kuidas Eesti kaarte lugeda. See on tehniline probleem, mitte õiguslik.

## DigiDoc ja allkirjastamine

DigiDoc on Eesti digitaalallkirja formaat ja tarkvara. Kui sa allkirjastad faili DigiDoc-iga, tekib .asice või .bdoc konteiner, mis sisaldab:

- Originaalfaili(d)
- Sinu digitaalallkirja
- Ajatemplit (tõestab, millal allkiri tehti)
- Sertifikaatide ahelat

See konteiner on matemaatiliselt kindel. Kui keegi muudab faili pärast allkirjastamist, muutub allkiri kehtetuks. Kui keegi üritab võltsida ajatemplit, on see tuvastatav.

DigiDoc4 klient on tasuta tarkvara, mida võid alla laadida id.ee lehelt. See töötab Windowsis, Macis ja Linuxis.

## Kuidas ise SK sertifikaate kasutada?

Kui sa arendad tarkvara, mis peab Eesti ID-kaarti toetama, pead sa teadma mõningaid asju.

**Usaldusahel** — sinu tarkvara peab usaldama SK juursertifikaate. Need on alla laetavad SK veebilehelt (sk.ee). Pead need lisama oma truststoresse.

**OCSP kontroll** — sinu tarkvara peaks kontrollima, kas sertifikaat on kehtiv, mitte ainult kas see pole aegunud. SK pakub OCSP teenust. See on oluline, sest kui inimene kaotab ID-kaardi, tühistatakse sertifikaat — ja sinu tarkvara peab seda teadma.

**Kaardilugeja tugi** — kui tahad, et kasutajad saaksid sinu veebilehele ID-kaardiga sisse logida, pead implementeerima WebCrypto või kasutama mõnda olemasolevat teeki. See on tehniliselt keeruline — lihtsam on kasutada olemasolevaid lahendusi nagu TARA (Riigi Autentimisteenuse).

## TARA — Riigi Autentimisteenuse

Kui sa ei taha ise ID-kaardi, Mobiil-ID ja Smart-ID tuge implementeerida, on TARA (Riigi Autentimisteenuse) sinu sõber.

TARA on RIA (Riigi Infosüsteemi Ameti) teenus, mis pakub autentimist-kui-teenust. Sinu rakendus suunab kasutaja TARA lehele, kasutaja valib autentimisviisi (ID-kaart, Mobiil-ID, Smart-ID, pangalink), autentib end, ja TARA saadab sinu rakendusele tagasi info kasutaja kohta.

See on OpenID Connect protokollil põhinev — standardne, hästi dokumenteeritud, lihtne implementeerida. Enamik Eesti avaliku sektori veebiteenuseid kasutab TARA-t.

Erasektori jaoks on alternatiiviks Dokobit, SK ise, või teised teenusepakkujad, kes pakuvad sarnast autentimine-kui-teenust äriklientidele.

## Praktiline näide: sertifikaadi kontrollimine

Kui sul on ID-kaart ja kaardilugeja, saad oma sertifikaate vaadata:

```bash
# MacOS/Linux - loe sertifikaate kaardilt
pkcs11-tool -L  # näita saadaolevaid slotte

# Või kasuta DigiDoc4 klienti - seal on visuaalne liides
```

Eesti ID-kaardi sertifikaate saab vaadata ka veebis: mine id.ee lehele ja kontrolli oma kaardi staatust.

## SK sertifikaatide usaldamine oma serveris

Kui sa tahad, et sinu server usaldaks Eesti ID-kaarte:

```bash
# Lae alla SK juursertifikaadid
wget https://www.sk.ee/upload/files/EE_Certification_Centre_Root_CA.pem.crt

# Lisa oma truststoresse
# Linux:
sudo cp EE_Certification_Centre_Root_CA.pem.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates

# Või Java jaoks:
keytool -importcert -alias sk-root -file EE_Certification_Centre_Root_CA.pem.crt \
    -keystore truststore.jks -storepass changeit
```

## Kokkuvõte

Eesti PKI on üks maailma arenenumaid. ID-kaart, Mobiil-ID ja Smart-ID põhinevad samadel põhimõtetel, mida see kursus õpetab — avaliku võtme krüptograafia, sertifikaadid, CA-d, usaldusahelad.

Kui sa mõistad, kuidas TLS ja PKI töötavad, mõistad sa ka, kuidas Eesti digitaalne identiteet töötab. See pole mingi Eesti-spetsiifiline maagia — see on sama matemaatika ja samad protokollid, lihtsalt hästi implementeeritud ja laialt kasutusele võetud.

| Komponent | Mis see on | Kes haldab |
|-----------|-----------|------------|
| ID-kaart | Füüsiline kaart krüptokiibiga | PPA, SK sertifikaadid |
| Mobiil-ID | SIM-kaardil põhinev identiteet | Mobiilioperaatorid, SK |
| Smart-ID | Telefoni põhinev identiteet | SK |
| TARA | Autentimine-kui-teenus | RIA |
| DigiDoc | Allkirjastamise tarkvara ja formaat | RIA |
| SK | Sertifitseerimisasutus | Eraettevõte |

Järgmises osas vaatame tulevikku — kvantarvutid ja post-quantum krüptograafia.
