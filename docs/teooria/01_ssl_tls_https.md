# Osa 1 - Mis on SSL, TLS ja HTTPS?

## Kolm lühendit, üks eesmärk

Kui räägime interneti turvalisusest, kuuleme pidevalt kolme lühendit: SSL, TLS ja HTTPS. Need kõlavad nagu tehnilised mõisted, mida ainult IT-inimesed peavad teadma, aga tegelikult puudutavad need igaüht, kes internetti kasutab.

Alustame kõige vanemast — SSL-ist. Secure Sockets Layer sündis Netscape'i laborites 1990ndate keskel. Mõte oli lihtne: luua turvaline tunnel kahe arvuti vahel, nii et kõik, mis sealt läbi läheb, on krüpteeritud. Kujuta ette, et selle asemel, et saata postkaarte, mida igaüks lugeda saab, saadad sa lukustatud kohvreid, mille võti on ainult sinul ja sinu sõbral.

SSL töötas, aga polnud täiuslik. Iga uus versioon parandas eelmise vigu, kuni lõpuks otsustati, et on aeg puhas leht võtta. Nii sündis TLS — Transport Layer Security. See on sisuliselt SSL-i järeltulija, turvalisem ja kiirem. Kui keegi tänapäeval räägib SSL-ist, mõtleb ta tegelikult TLS-i. Vana nimi lihtsalt jäi käibele, nagu me ütleme ikka "helistama", kuigi telefonidel pole ammu ketast.

## HTTP ja selle turvalisem vend

Nüüd jõuame HTTPS-ini. HTTP ehk HyperText Transfer Protocol on keel, mida sinu brauser ja veebiserverid räägivad. Kui sa külastad veebilehte, saadab sinu brauser HTTP päringuid ja saab vastuseid. Probleem on selles, et tavaline HTTP on nagu vali vestlus rahvarohkes kohvikus — igaüks läheduses kuuleb, millest te räägite.

HTTPS on HTTP + TLS. See S lõpus tähendab "Secure". Kui sa näed brauseri aadressiribal https:// ja väikest lukuikooni, tähendab see, et sinu brauser ja server on loonud krüpteeritud tunneli. Kõik, mis sealt läbi läheb — sinu paroolid, krediitkaardinumbrid, privaatsed sõnumid — on loetamatu igaühele, kes üritab vahelt lugeda.

## Kuidas lukk töötab?

See lukuikoon brauseris pole lihtsalt ilus pilt. See tähendab, et toimus edukas käepigistus — sinu brauser ja server leppisid kokku ühises saladuses, mida keegi teine ei tea.

Kujuta ette, et tahad sõbraga salakeelt kokku leppida, aga oled rahvarohkes ruumis. Te ei saa lihtsalt öelda "meie salasõna on XYZ", sest keegi kuuleb. SSL/TLS lahendab selle probleemi nutikalt: te kasutate avalikku võtit, et kokku leppida privaatvõtmes, mida ainult teie teate. See on nagu avalikult öelda midagi, millest ainult teie kahekesi aru saate.

Kui turvaline ühendus on loodud, muutub kogu suhtlus krüpteerituks. Isegi kui keegi püüab andmeid kinni, näeb ta ainult mõttetut sodi. Ilma õige võtmeta on see täiesti kasutu.

## Miks see oluline on?

Tänapäeval on raske leida tõsiseltvõetavat veebilehte, mis poleks HTTPS-i peal. Google'i brauser märgib HTTP lehti "Mitte turvalised" ja otsingumootorid eelistavad turvalisi lehti. Aga see pole ainult reeglite järgimine — see on sinu kasutajate kaitsmine.

Ilma HTTPS-ita saaks igaüks samas WiFi võrgus näha, mida sinu kasutajad teevad. Paroolid, isikuandmed, ostuinfo — kõik oleks nähtav. HTTPS-iga on see kõik lukus.

Järgmises osas vaatame, mis on PKI — see süsteem, mis otsustab, keda internetis usaldada.

![SSL TLS HTTPS](../assets/ssltslhttps.png)
