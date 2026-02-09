# TLS käepigistus

<figure markdown="span">
  ![TLS käepigistuse protsess](../assets/handshake.png)
  <figcaption>Joonis 3.1. TLS käepigistuse protsess (Talvik, 2025). Loodud tehisintellekti abil.</figcaption>
</figure>

## Kaks võõrast kohtuvad

Iga kord, kui sa avad turvalise veebilehe, toimub kulisside taga kiire tants. Sinu brauser ja server — kaks masinat, kes pole kunagi varem kohtunud — peavad mõne sekundi jooksul kokku leppima, kuidas turvaliselt suhelda. Seda tantsu nimetatakse TLS käepigistuseks.[^rfc8446]

Mõtle sellest nagu kahe spioonina kohtumine. Nad ei tunne teineteist, nad ei usalda kedagi, aga neil on vaja vahetada salajast infot. Kuidas nad seda teevad nii, et keegi kolmas ei saaks pealt kuulata?

| Samm | Suund | Tegevus |
|------|-------|---------|
| 1 | Brauser → Server | "Tere! Ma toetan TLS 1.2, 1.3 ja neid šifreid..." |
| 2 | Server → Brauser | "OK, TLS 1.3. Siin on mu sertifikaat." |
| 3 | Brauser | Kontrollib sertifikaati (CA, nimi, kehtivus) |
| 4 | Brauser ↔ Server | Võtmevahetus (Diffie-Hellman / ECDHE) |
| 5 | Mõlemad | Genereerivad sessioonivõtme |
| 6 | Mõlemad | Turvaline tunnel on avatud |

*Tabel 3.1. TLS käepigistuse sammud*

## Esimene samm: "Tere, ma tahan turvaliselt rääkida"

Kõik algab sellest, et sinu brauser saadab serverile teate: "Tere! Ma olen brauser, ma toetan neid TLS versioone ja neid krüpteerimisviise. Siin on ka juhuslik number, mille ma just genereerisin."

See on nagu siseneda ruumi ja öelda: "Ma räägin eesti, inglise ja vene keelt. Kumb sulle sobib?"

## Teine samm: Server vastab ja näitab passi

Server vaatab, mida brauser pakub, ja valib variandi, mis mõlemale sobib. Siis saadab ta vastuse: "Olgu, räägime TLS 1.3 ja kasutame seda krüpteerimist. Siin on minu sertifikaat — minu digitaalne pass. Ja siin on minu enda juhuslik number."

Brauser kontrollib sertifikaati nelja asja suhtes: kas see on kehtiv (pole aegunud), kas selle on allkirjastanud usaldatud CA, kas sertifikaadi nimi vastab veebilehe aadressile ja kas sertifikaat pole tühistatud (OCSP/CRL kontroll).

Kui midagi on valesti — sertifikaat aegunud, vale nimi, tundmatu CA — näed sa seda punast hoiatust brauseris.

## Kolmas samm: Saladuse loomine

Nüüd tuleb nutikus osa. Brauser ja server peavad kokku leppima saladuses, mida keegi kolmas ei tea. Aga kuidas seda teha, kui keegi võib pealt kuulata?

Lahendus on matemaatiline ime nimega võtmevahetus. Kujuta ette, et sina ja sõber seisate rahvarohke tänava kahes otsas. Te mõlemad valite salaja värvi ja segatee selle avaliku värviga. Siis vahetate segatud värve. Nüüd lisab igaüks teise segatud värvile oma salajase värvi. Matemaatiliselt jõuate mõlemad sama lõppvärvini, aga keegi, kes nägi ainult segatud värve tänaval, ei suuda seda lõppvärvi arvutada.[^ristic]

!!! info "Diffie-Hellman lihtsalt"
    Mõlemad pooled valivad salajase väärtuse ja arvutavad sellest avaliku väärtuse. Avalikke väärtusi vahetatakse. Matemaatiliselt jõuavad mõlemad sama jagatud saladuseni, aga keegi, kes nägi ainult avalikke väärtusi, ei suuda seda arvutada.

TLS-is kasutatakse selleks Diffie-Hellmani algoritmi või selle elliptiliste kõverate versiooni (ECDHE). Tulemuseks on jagatud saladus, mida ainult brauser ja server teavad.

## Neljas samm: Turvaline tunnel on avatud

Nüüd, kui mõlemal poolel on sama saladus, genereeritakse sellest sessioonivõtmed. Need võtmed krüpteerivad kogu ülejäänud suhtluse.

| Tüüp | Kiirus | Kasutus TLS-is |
|------|--------|----------------|
| **Asümmeetriline** (RSA, ECDH) | Aeglane | Ainult käepigistus |
| **Sümmeetriline** (AES) | Kiire | Kogu andmevahetus |

*Tabel 3.2. Kaks krüptograafia tüüpi TLS-is*

Seega: aeglast meetodit kasutatakse ainult selleks, et kokku leppida kiires meetodis. Käepigistus on läbi. Nüüd liigub kõik — HTML, pildid, paroolid, krediitkaardid — krüpteeritud tunnelis.

## TLS 1.2 versus 1.3

| Omadus | TLS 1.2 | TLS 1.3 |
|--------|---------|---------|
| Käepigistuse kiirus | 2 edasi-tagasi (2-RTT) | 1 edasi-tagasi (1-RTT) |
| Nõrgad šifrid | Lubatud | Eemaldatud |
| Forward secrecy | Valikuline | Kohustuslik |
| 0-RTT režiim | Ei | Jah |

*Tabel 3.3. TLS 1.2 ja TLS 1.3 võrdlus*

Vana TLS 1.2 käepigistus vajas kaks edasi-tagasi reisi. Brauser saatis teate, ootas vastust, saatis veel teate, ootas jälle. See võttis aega.

TLS 1.3, mis sai standardiks 2018. aastal, lahendas selle.[^rfc8446] Käepigistus toimub ühe reisiga. Brauser saadab esimese teatega juba oma võtmevahetuse info, nii et server saab kohe vastata täieliku infoga. Tulemus: kiirem ühendus.

TLS 1.3 eemaldas ka kõik nõrgad krüpteerimisviisid. Vanas versioonis said valida halbade variantide vahel. Uues versioonis on ainult head valikud. See teeb seadistamise lihtsamaks — raskem on kogemata midagi valesti panna.

## Edastussaladus — miks see oluline on?

Üks TLS 1.3 parimaid omadusi on kohustuslik edastussaladus (forward secrecy). See tähendab, et isegi kui keegi saab kunagi kätte serveri privaatvõtme, ei saa ta vanu salvestatud ühendusi lahti krüpteerida.

!!! warning "Miks see oluline on?"
    Isegi kui ründaja saab kunagi tulevikus kätte serveri privaatvõtme, ei saa ta vanu salvestatud ühendusi lahti krüpteerida. Iga ühenduse jaoks genereeritakse uued ajutised võtmed, mis kustutatakse pärast kasutamist.

See on nagu kasutada iga vestluse jaoks uut salakeelt ja siis see ära unustada.

## Kui käepigistus ebaõnnestub

| Veateade | Põhjus |
|----------|--------|
| `certificate has expired` | Sertifikaat aegunud |
| `hostname mismatch` | Sertifikaadi nimi ei klapi URL-iga |
| `unable to get local issuer` | Vahesertifikaat puudub |
| `self signed certificate` | CA pole usaldatud |
| `no shared cipher` | Protokolli/šifri mitteühilduvus |

*Tabel 3.4. Levinud TLS veateated ja nende põhjused*

Mõnikord näed brauseris hoiatust, et turvaline ühendus ebaõnnestus. Need hoiatused on olulised. Nad kaitsevad sind rünnakute eest, kus keegi üritab end serveri ja sinu vahele sokutada (man-in-the-middle). Kui näed sellist hoiatust päris pangalehel, on see tõsine ohumärk.

---

## Kokkuvõte

TLS käepigistus on protsess, kus brauser ja server lepivad kokku turvalises ühenduses. Server tõestab identiteeti sertifikaadiga, pooled vahetavad võtmeid (Diffie-Hellman/ECDHE) ja genereerivad sessioonivõtme. TLS 1.3 on kiirem (1-RTT), turvalisem (ainult tugevad šifrid) ja kohustab edastussaladust.

---

## Enesekontroll

??? question "1. Miks ei saa brauser ja server lihtsalt kohe krüpteeritud andmeid saatma hakata — miks on käepigistust vaja?"
    Nad pole kunagi varem kohtunud ja neil pole jagatud saladust. Käepigistus lahendab küsimusse: kuidas leppida kokku krüpteerimisv õtmes nii, et keegi kolmas ei saaks seda teada, isegi kui ta kogu suhtlust pealt kuulab.

??? question "2. Kolleeg ütleb: 'Ma sain brauseris sertifikaadi hoiatuse, aga vajutasin edasi, sest leht töötab ju.' Miks see on ohtlik?"
    Hoiatus tähendab, et käepigistuse usalduskontroll ebaõnnestus — server ei suutnud tõestada oma identiteeti. Edasi vajutamine tähendab, et sa võid suhelda ründajaga (man-in-the-middle), kes näeb kogu sinu liiklust.

??? question "3. Miks kasutab TLS 1.3 ainult ühte edasi-tagasi teekonda (1-RTT), aga TLS 1.2 vajab kahte?"
    TLS 1.3 saadab võtmevahetuse andmed juba esimese sõnumiga kaasa, sest nõrkade šifrite tugi on eemaldatud ja valikud on väiksemad. TLS 1.2 pidi esmalt kokku leppima parameetrites ja alles siis võtit vahetama.

??? question "4. Sinu ülemus küsib: 'Kas keegi saab meie HTTPS liiklust salvestada ja hiljem lahti krüpteerida, kui meie privaatvõti lekib?' Mida vastaksid TLS 1.3 kohta?"
    TLS 1.3 kasutab ainult efemeerseid võtmeid (PFS — Perfect Forward Secrecy). Iga sessioon saab uue võtme, mis kustutatakse pärast sessiooni. Isegi kui serveri privaatvõti lekib, ei saa varasemaid sessioone lahti krüpteerida.

[^rfc8446]: Rescorla, E. (2018). *The Transport Layer Security (TLS) Protocol Version 1.3*. RFC 8446. https://datatracker.ietf.org/doc/html/rfc8446
[^ristic]: Ristić, I. (2022). *Bulletproof TLS and PKI*. Feisty Duck. https://www.feistyduck.com/books/bulletproof-tls-and-pki/
