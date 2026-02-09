---
tags:
  - Sertifikaadid
---

# Sertifikaadi failivormingud

## Sama sisu, erinevad pakendid

Kui hakkad sertifikaatidega tööle,[^openssl] kohtad peagi failinimede džunglit: .pem, .crt, .cer, .key, .p12, .pfx, .der, .jks... See tundub alguses kaosena, aga tegelikult on loogika olemas.

Mõtle sellest nagu toidupakendist. Sul on sama lõuna - võileib ja õun. Sa võid selle panna plastikkarpi, paberikotti, fooliumisse või termoskotti. Sisu on sama, aga pakend erinev. Iga pakend sobib erinevateks olukordadeks: plastikarpi saab pesta, paberikott on kergem, foolium hoiab sooja.

Sertifikaadivormingud töötavad samamoodi. Sama matemaatiline info - võtmed ja sertifikaadid - on pakitud erinevatesse vormingutesse, sest erinevad süsteemid eelistavad erinevaid pakendeid.

## PEM - kõige levinum

PEM on nagu sertifikaatide lingua franca.[^ristic] Kui avad PEM-faili tekstiredaktoris, näed midagi sellist:

```
-----BEGIN CERTIFICATE-----
MIIDXTCCAkWgAwIBAgIJAJC1HiIAZAiUMA0Gcg...
palju ridu base64 teksti...
-----END CERTIFICATE-----
```

See "BEGIN CERTIFICATE" ja "END CERTIFICATE" vahel on base64 kodeeringus sertifikaat. Base64 tähendab, et binaarandmed on muudetud tavalisteks tähtedeks ja numbriteks, nii et neid saab kopeerida, kleepida, e-kirjaga saata.

PEM-faile näeb laienditega .pem, .crt, .cer, .key. Jah, sama formaat, erinevad laiendid - see tekitab alguses segadust. Tavaliselt .key sisaldab privaatvõtit, .crt või .cer sertifikaati, .pem võib olla kumbki või mõlemad.

Üks PEM-fail võib sisaldada mitut sertifikaati järjest. See on mugav usaldusahela jaoks - paned serveri sertifikaadi ja vahesertifikaadid ühte faili.

## DER - binaarne vend

DER on sama info, mis PEM, aga binaarkujul. Seda ei saa tekstiredaktoris avada - näed ainult loetamatut sodi. Aga see on kompaktsem ja mõned süsteemid (eriti Windows ja Java) eelistavad seda.

Konverteerimine PEM ja DER vahel on lihtne:

```bash
# PEM -> DER
openssl x509 -in sertifikaat.pem -outform DER -out sertifikaat.der

# DER -> PEM
openssl x509 -in sertifikaat.der -inform DER -outform PEM -out sertifikaat.pem
```

## PKCS#12 / PFX - kõik ühes kotis

Mõnikord on vaja transportida sertifikaati koos privaatvõtmega. Näiteks viid sertifikaadi ühest serverist teise või impordid brauserisse.

PKCS#12 (failid .p12 või .pfx) lahendab selle. See on nagu lukustatud kohver, mis sisaldab sertifikaati, privaatvõtit ja vahel ka usaldusahelat. Kohver on parooliga kaitstud, nii et isegi kui keegi selle kätte saab, ei saa ta ilma paroolita sisu kätte.

```bash
# Loo PKCS#12 fail
openssl pkcs12 -export -out sertifikaat.p12 \
    -inkey privaatvoti.key \
    -in sertifikaat.crt \
    -certfile vahesertifikaat.crt

# Küsitakse parool, millega fail krüpteeritakse
```

Windows armastab PKCS#12 faile. Kui tahad sertifikaati Windowsi importida, on .pfx sageli lihtsaim tee.

## JKS - Java maailm

Java on oma rada käinud ja kasutab oma formaati nimega JKS (Java KeyStore). See on nagu PKCS#12, aga Java-spetsiifiline.

Java maailmas räägitakse kahest asjast: keystore ja truststore. Keystore hoiab sinu enda sertifikaati ja privaatvõtit - see on sinu identiteet. Truststore hoiab CA sertifikaate - see on nimekiri neist, keda sa usaldad. Tehniliselt on mõlemad samad JKS-failid, lihtsalt erineva sisuga.

Tänapäeval soovitab isegi Oracle kasutada PKCS#12 formaati JKS asemel, sest see on universaalsem. Aga vanu süsteeme on palju ja JKS-i kohtab ikka.

```bash
# Konverteeri PKCS#12 -> JKS (kui vaja)
keytool -importkeystore \
    -srckeystore sertifikaat.p12 \
    -srcstoretype PKCS12 \
    -destkeystore sertifikaat.jks \
    -deststoretype JKS
```

## Võtmefailid

Privaatvõtit hoitakse tavaliselt eraldi failis laiendiga .key. See on PEM-vormingus:

```
-----BEGIN PRIVATE KEY-----
MIIEvgIBADANBgkqhkiG9w0BAQEFAASC...
-----END PRIVATE KEY-----
```

Mõnikord näed "BEGIN RSA PRIVATE KEY" või "BEGIN EC PRIVATE KEY" - need on vanemad vormingud, mis näitavad võtme tüüpi. Uuem "BEGIN PRIVATE KEY" on üldisem ja sisaldab info võtme tüübi kohta sees.

Privaatvõtit võib ka parooliga krüpteerida:

```
-----BEGIN ENCRYPTED PRIVATE KEY-----
```

See on hea mõte, eriti kui fail võib valedesse kätesse sattuda. Aga see tähendab ka seda, et rakendus peab teadma parooli, et võtit kasutada.

## CSR - taotlus

Kui taotled sertifikaati, saadad CA-le CSR-faili (Certificate Signing Request). See on samuti PEM-vormingus:

```
-----BEGIN CERTIFICATE REQUEST-----
```

CSR sisaldab sinu avalikku võtit ja infot, mida tahad sertifikaadile panna. See on allkirjastatud sinu privaatvõtmega, mis tõestab, et sul on vastav privaatvõti.

## Praktiline näide

Kujuta ette, et seadistad Nginx veebiserverit. Sul on vaja:

1. Privaatvõtme fail (.key)
2. Sertifikaadi fail koos usaldusahelaga (.crt)

Nginx konfiguratsioon näeb välja umbes nii:

```
ssl_certificate /etc/ssl/certs/minusait.crt;
ssl_certificate_key /etc/ssl/private/minusait.key;
```

Sertifikaadifail sisaldab nii sinu sertifikaati kui ka vahesertifikaate, ühes failis järjest.

Kui seadistad Java rakendust, on lugu teine. Seal on vaja JKS või PKCS#12 faili, kuhu on pakitud nii sertifikaat kui võti:

```
server.ssl.key-store=/etc/ssl/minusait.p12
server.ssl.key-store-password=salajane
```

## Kokkuvõtteks

| Laiend | Mis see on | Tüüpiline kasutus |
|--------|-----------|-------------------|
| .pem | Tekstivormis sertifikaat/võti | Linux, universaalne |
| .crt, .cer | Sama mis .pem (tavaliselt) | Sertifikaadifailid |
| .key | Privaatvõti (PEM vormingus) | Võtmefailid |
| .der | Binaarses vormis sertifikaat | Windows, Java |
| .p12, .pfx | Sertifikaat + võti ühes (parooliga) | Transport, Windows |
| .jks | Java KeyStore | Java rakendused |
| .csr | Sertifikaadi taotlus | Saatmiseks CA-le |

*Tabel 5.1. Sertifikaadi failivormingud ja laiendid*

Järgmises osas vaatame sisemiste ja väliste sertifikaatide vahet - millal kasutad Let's Encrypt'i ja millal lood oma CA.

---

## Enesekontroll

??? question "1. Mis vahe on PEM ja DER vormingul?"
    PEM on tekstipõhine (base64), loetav tekstiredaktoriga, algab "BEGIN CERTIFICATE". DER on binaarne, kompaktsem, aga ei saa kopeerida/kleepida. Sisu on sama, ainult kodeering erineb.

??? question "2. Milleks on PKCS#12 (.p12/.pfx) formaat?"
    PKCS#12 pakib sertifikaadi, privaatvõtme ja vahel usaldusahela ühte parooliga kaitstud faili. Kasulik transportimiseks serverite vahel ja Windowsi importimiseks.

??? question "3. Mis vahe on keystore ja truststore kontseptsioonil?"
    Keystore hoiab sinu identiteeti (privaatvõti + sertifikaat). Truststore hoiab CA sertifikaate - neid, keda sa usaldad. Erinevad turvanõuded: keystore on ülitundlik, truststore sisaldab ainult avalikke sertifikaate.

[^openssl]: OpenSSL Project. *OpenSSL dokumentatsioon*. https://www.openssl.org/docs/
[^ristic]: Ristić, I. (2022). *Bulletproof TLS and PKI*. Feisty Duck. https://www.feistyduck.com/books/bulletproof-tls-and-pki/
