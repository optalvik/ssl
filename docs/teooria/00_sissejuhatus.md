# Sissejuhatus: Miks see kõik oluline on?

![Kuidas HTTPS sinu andmeid kaitseb](../assets/https_kaitseb.png)

## Miks sa peaksid hoolikama?

Iga kord, kui sa avad pangalehe, ostad midagi internetist või saadad privaatset sõnumit, usaldad sa süsteemi, mida enamik inimesi ei mõista. See süsteem kaitseb sind — sinu paroole, sinu raha, sinu privaatsust. Aga kui sa ei tea, kuidas see töötab, ei oska sa ka aru saada, kui midagi on valesti.

Kujuta ette, et istud kohvikus ja kasutad tasuta WiFi-t. Sa logid sisse oma e-postkasti. Kas keegi saab su parooli näha? Vastus sõltub sellest, kas ühendus on krüpteeritud. Kui on — näeb pealtvaataja ainult mõttetut sodi. Kui pole — näeb ta kõike.

See kursus õpetab sulle, kuidas kogu see süsteem töötab. Mitte pinnapealselt, vaid päriselt — kuidas võtmed luuakse, kuidas sertifikaadid väljastatakse, kuidas brauser otsustab, keda usaldada. Sa õpid looma oma sertifikaate, seadistama turvalisi servereid ja lahendama probleeme, kui midagi läheb valesti.

## Kellele see kursus on mõeldud?

See kursus sobib sulle, kui:

- Sa oled algaja, kes tahab aru saada, kuidas interneti turvalisus töötab
- Sa oled arendaja, kes puutub kokku HTTPS-i ja sertifikaatidega
- Sa oled DevOps insener, kes seadistab ja haldab TLS-i
- Sa lihtsalt tahad teada, mida see lukuikoon brauseris tegelikult tähendab

Eelnevaid süvateadmisi pole vaja. Alustame algusest ja liigume samm-sammult keerulisemate teemade poole.

## Mida sa õpid?

Kursus koosneb teoreetilistest osadest ja praktilistest laboritest.

**Teooria osades** saad teada:
- Mis on SSL, TLS ja HTTPS ning kuidas need sündisid
- Kuidas avaliku võtme krüptograafia töötab
- Mis on sertifikaadid ja sertifitseerimisasutused
- Kuidas TLS käepigistus toimub
- Kuidas sertifikaate hallata, automatiseerida ja probleeme lahendada
- Kuidas see kõik Eestis töötab — ID-kaart, Mobiil-ID, SK
- Mis on post-quantum krüptograafia ja miks see oluline on

**Praktilistes laborites** teed sa:
- Lood oma sertifitseerimisasutuse
- Genereerid ja analüüsid sertifikaate
- Seadistad HTTPS servereid
- Kasutad HashiCorp Vault'i automaatseks sertifikaadihalduseks
- Katsetad post-quantum algoritme

## Kuidas kursust läbida?

Soovitan alustada Osast 1 ja liikuda järjest edasi. Iga osa ehitab eelmisele peale. Laborid on mõeldud tegema pärast vastava teooria lugemist — näiteks Labor 1 pärast Osi 1-5.

Aga kui sul on juba kogemust ja tahad konkreetset teemat, võid ka otse sinna hüpata. Iga osa on piisavalt iseseisev.

Alustame sellest, kuidas see kõik alguse sai — Netscape'i inseneridest, kes 1990ndatel otsustasid interneti turvalisemaks teha.
