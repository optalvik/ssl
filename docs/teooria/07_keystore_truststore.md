# Osa 7 - Keystore ja truststore

## Kaks erinevat asja

Java maailmas — ja see levib ka mujale — räägitakse pidevalt kahest asjast: keystore ja truststore. Alguses tundub, et need on sama asi erinevate nimedega. Nad näevad välja sarnased, nad on sageli samas failivormingus, neid hallatakse sama tööriistaga. Aga neil on fundamentaalselt erinev roll.

Keystore hoiab sinu identiteeti. Seal on sinu privaatvõti ja sinu sertifikaat. See on nagu rahakott, kus hoiad oma isikutunnistust ja võtmeid.

Truststore hoiab teiste identiteete. Seal on CA sertifikaadid — need, keda sa usaldad. See on nagu nimekiri usaldusväärsetest asutustest, kelle väljastatud dokumente sa aktsepteerid.

Kui server alustab TLS ühendust, võtab ta keystorest oma sertifikaadi ja näitab seda klientidele. Kui klient saab sertifikaadi, kontrollib ta truststorest, kas allkirjastanud CA on usaldatud.

## Miks need eraldi on?

Praktiline põhjus: neil on erinevad turvanõuded.

Keystore sisaldab privaatvõtit — see on ülitundlik. Kui keegi saab sinu privaatvõtme, saab ta esineda sinuna. Keystorei peab hoidma turvaliselt, piiratud ligipääsuga, võimalusel parooliga kaitstult.

Truststore sisaldab ainult avalikke sertifikaate. Need pole salajased — CA sertifikaadid on avalikult kättesaadavad. Truststore peab olema ajakohane, aga see pole nii kriitiline turvaoht kui keystore.

Mõnikord tahad neid eraldi hallata. Näiteks üks meeskond haldab serverite identiteete (keystoored), teine meeskond haldab, milliseid CA-sid usaldatakse (truststoored). Eraldi failid teevad selle lihtsamaks.

## Java vaikimisi truststore

Java tuleb kaasa vaikimisi truststorega nimega cacerts. See asub kusagil Java installi sees, tavaliselt $JAVA_HOME/lib/security/cacerts. Seal on eelinstallitud kõik suured avalikud CA-d — samad, mida brauserid usaldavad.

See tähendab, et Java rakendused usaldavad vaikimisi kõiki avalikke veebisaite. Kui ühendud https://google.com-iga, töötab see kohe, sest Google'i sertifikaadi allkirjastanud CA on cacerts failis olemas.

Aga kui tahad ühenduda sisemise teenusega, mis kasutab sisemise CA sertifikaati, tuleb see CA cacertsi lisada. Või luua eraldi truststore ja öelda rakendusele, et kasutagu seda.

## Failivormingud

Traditsiooniliselt kasutas Java oma formaati JKS (Java KeyStore). See on binaarfail, mis sisaldab sertifikaate ja võtmeid, kaitstud parooliga.

Tänapäeval eelistatakse PKCS#12 formaati (.p12), mis on standardsem ja töötab ka väljaspool Java maailma. Java toetab mõlemat.

Keytool on Java tööriist, millega keystoore ja truststoore hallatakse:

```bash
# Vaata, mis truststores on
keytool -list -keystore $JAVA_HOME/lib/security/cacerts -storepass changeit

# Lisa CA sertifikaat truststoresse
keytool -importcert -alias minu-ca -file ca.crt \
    -keystore truststore.jks -storepass salajane

# Loo keystore sertifikaadi ja võtmega
keytool -importkeystore \
    -srckeystore minu.p12 -srcstoretype PKCS12 \
    -destkeystore keystore.jks -deststoretype JKS
```

Vaikimisi parool cacerts faili jaoks on "changeit". Jah, tõesti. See pole turvameetme, see on lihtsalt koht, kuhu parool panna.

## Kuidas rakendused neid kasutavad

Java rakendused saavad keystorei ja truststorei süsteemiomaduste kaudu:

```
-Djavax.net.ssl.keyStore=/tee/keystore.jks
-Djavax.net.ssl.keyStorePassword=parool
-Djavax.net.ssl.trustStore=/tee/truststore.jks
-Djavax.net.ssl.trustStorePassword=parool
```

Või konfiguratsioonis, sõltuvalt rakendusest. Spring Boot näiteks:

```yaml
server:
  ssl:
    key-store: classpath:keystore.p12
    key-store-password: parool
    trust-store: classpath:truststore.p12
    trust-store-password: parool
```

## Väljaspool Java maailma

Kuigi terminid tulevad Java-st, kasutavad sarnast kontseptsiooni ka teised süsteemid.

Nginx ja Apache ei kasuta keystoori — nad võtavad sertifikaadi ja võtme otse failidest. Aga truststorei kontseptsioon on olemas: sa määrad CA sertifikaadi, mida klientide verifitseerimiseks kasutada.

Elasticsearch, Kafka, MongoDB ja paljud teised kasutavad sarnast mudelit. Konfiguratsioon näeb välja midagi sellist:

```yaml
xpack.security.transport.ssl.keystore.path: elastic.p12
xpack.security.transport.ssl.truststore.path: ca.p12
```

Mõned süsteemid eelistavad eraldi PEM-faile JKS/P12 failide asemel. Lõpptulemus on sama — kusagil on sinu identiteet, kusagil on nimekiri usaldatavatest.

## Levinud vead

Kõige tavalisem viga: truststore puudumine. Sa seadistad serveri oma sertifikaadiga, kõik töötab. Siis proovid ühenduda teise serveriga, mis kasutab sisemise CA sertifikaati, ja ühendus ebaõnnestub. Põhjus: sinu truststore ei sisalda seda CA-d.

Teine levinud viga: vale truststore. Sul on mitu Java versiooni, igaühel oma cacerts. Sa lisad CA ühe cacertsi, aga rakendus kasutab teist Java versiooni teise cacertsiga.

Kolmas viga: paroolid. Keystore parool ja võtme parool võivad olla erinevad. Mõnikord peab mõlemad määrama. Mõnikord uuendad parooli, aga konfiguratsioon jääb vanaks.

Neljas viga: aegunud sertifikaadid truststores. CA sertifikaadid aeguvad ka. Kui su truststore sisaldab aegunud CA sertifikaati, lakkavad selle CA allkirjastatud serverid töötamast.

## Debugimine

Kui TLS ühendused ebaõnnestuvad, on Java debug-režiim suureks abiks:

```
-Djavax.net.debug=ssl,handshake
```

See prindib konsooli kõik TLS käepigistuse detailid: milliseid sertifikaate saadeti, milliseid CA-sid usaldati, kus täpselt ühendus katkes.

Teine kasulik tööriist on openssl s_client:

```bash
openssl s_client -connect server:443 -showcerts
```

See näitab, millise sertifikaadi server saadab ja kas usaldusahel on terve.

Järgmises osas räägime sertifikaatide elukaarist — kuidas neid luuakse, uuendatakse ja tühistatakse.
