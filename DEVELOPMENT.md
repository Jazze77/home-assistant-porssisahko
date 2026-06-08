# 📈 Projektin kehityshistoria ja järjestelmäarkkitehtuuri

Tämä dokumentti kuvaa pörssisähköjärjestelmän kehitysmatkan ensimmäisistä prototyypeistä nykyiseen, äärimmäisen optimoituun ja vakaaseen tuotantoversioon (v2.2).

---

## 🗺️ Kehitysvaiheet kronologisesti

### Vaihe 1: Ensimmäiset askeleet (Älylampputestit)
Projekti käynnistyi hyvin yksinkertaisella kokeilulla. Tavoitteena oli oppia Home Assistantin automaatioiden perusteet ja testata laitteiden reheaikaista ohjausta:
*   **Ensimmäinen testi:** Luotiin automaatio, joka ohjasi älylamppua (on/off) tiettyjen kellonaikojen mukaan.
*   **Väriohjaus:** Laajennettiin automaatiota muuttamaan älylampun väriä lennosta, millä testattiin monimutkaisempien komentojen ja datapakettien kulkua Zigbee/Wi-Fi-verkoissa.

### Vaihe 2: Nordpool-integraatio ja visuaalinen hintavahti
Kun perusohjaus toimi, järjestelmään kytkettiin **Nordpool-sensori** reaaliaikaisten pörssisähköhintojen noutamiseksi:
*   Luotiin automaatio, joka vertasi kuluvan tunnin hintaa vuorokauden keskihintaan.
*   Älylamppu muutettiin visuaaliseksi hintavahdiksi kodin seinälle: lamppu paloi **vihreänä**, kun sähkö oli halpaa, ja muuttui **punaiseksi**, kun hinta nousi kalliiksi.

### Vaihe 3: Älypistokkeet ja ensimmäiset käyttöliittymät (The Grid)
Visuaalisesta vahdista siirryttiin todelliseen kuormanohjaukseen, kun järjestelmään liitettiin ensimmäiset WiZ-älypistokkeet:
*   **Käyttöliittymän synty:** Rakennettiin ensimmäiset Lovelace-dashboardit laitteiden ohjaukseen.
*   **The Grid (Ruudukko):** Kehitettiin dynaaminen tuntiruudukko, josta käyttäjä pystyi itse klikkailemalla valitsemaan (Grid-valinnat), mitkä tunnit laitteet olivat päällä ja mitkä pois.

### Vaihe 4: Testauskaaos (61 automaatiota ja muistirajoitukset)
Toiminnallisuuksien kasvaessa (kestoasetukset, optimaalisten jaksojen haut, simulaatiotilat, aikarajoitukset) järjestelmä monimutkaistui nopeasti. 
*   Erilaisia kokeiluja, päällekkäisiä logiikoita ja testiautomaatioita kertyi lopulta huikeat **61 kappaletta**.
*   Järjestelmä törmäsi Home Assistantin `input_text`-kenttien kiinteään **255 merkin rajoitukseen**, kun pitkien latausten tehojonoja yritettiin kirjoittaa yhteen muuttujaan. Pitkät lataukset katkesivat tai kaatuivat muistivirheisiin.

### Vaihe 5: Järjestelmän täydellinen puhdistus ja arkkitehtuurin uudistus
Havaittujen ongelmien jälkeen tehtiin radikaali päätös: koko järjestelmä pystytettiin puhtaalta pöydältä **upouudelle Home Assistant -koneelle** ja koodi kirjoitettiin kokonaan uusiksi:
*   **61 automaatiota supistettiin tasan 8 automaatioon**, jotka hoitavat kaiken taustalogiikan äärimmäisen kevyesti ja nopeasti.
*   Keksittiin ja toteutettiin **`indeksi:teho` -paritus**, joka sitoo mitatun tehon kiinteästi jakson ID-numeroon, poistaen aikasirtymävirheet lopullisesti.
*   Keksittiin **kaksoiskenttäarkkitehtuuri (`_part2` + `*`)**, joka halkaisee teholokit lennosta kahteen osaan pituuden ylittäessä 245 merkkiä, murtaen 255 merkin rajoituksen pysyvästi.

---

## 🛠️ Järjestelmän nykytila ja toiminta (Miten se toimii nyt)

Nykyinen järjestelmä on täysin tuotantovarma, automaattinen ja dynaaminen kokonaisuus, joka pyörii **8 automaation** voimin:

### Nyt käytössä olevat 8 automaatiota:
1.  **Sähköauto - Reaaliaikainen lataus (v1.1):** Ohjaa fyysistä latauspistoketta sekunnilleen valitun suunnitelman mukaan.
2.  **Sähköauto - Latauksen simulaatio (v1.2):** Mahdollistaa lataussyklien ja raportoinnin testaamisen milloin tahansa turvallisesti simuloimalla.
3.  **Lämmitin - Reaaliaikainen lataus (v2):** Ohjaa lämmittimen etäpistorasiaa pörssihintojen mukaan.
4.  **Lämmitin - Latauksen simulaatio (v2):** Lämmitysjärjestelmän täydellinen testaussimulaattori.
5.  **Nordpool Midnight Cleanup:** Aina taustalla oleva huoltotyökalu, joka siirtää keskiyöllä huomisen luonnokset kuluvan päivän aktiivisiksi ajoiksi.
6.  **Nordpool: Automaattinen jaksojen haku hintojen julkaisusta:** Herää iltapäivällä (esim. klo 14:17) heti, kun huomisen pörssihinnat julkaistaan, ja laskee valmiit ehdotukset ruudukkoon oletusarvojen mukaan.
7.  **Sähköauton Latausvahti (Turvaominaisuus):** Valvoo, että virta todella kulkee, kun pistoke on päällä.
8.  **Järjestelmän käynnistysvahti:** Varmistaa sähkökatkon tai palvelimen uudelleenkäynnistyksen jälkeen, että käynnissä olleet lataussuunnitelmat jatkuvat automaattisesti oikeasta kohdasta.

---

## 📊 Nykyiset avainominaisuudet käytännössä

*   **Täysi automaatio iltapäivisin:** Käyttäjän ei tarvitse klikkailla kestoja tai etsiä halpoja tunteja. Automaatti laskee huomisen ehdotukset valmiiksi ruudukkoon lennosta klo 14:17, ja lähettää kännykkään push-ilmoituksen: *"Ehdotukset valmiina Dashboardilla!"*. Käyttäjälle riittää illalla vain yksi "Tallenna"-napin painallus.
*   **Puhdas automaattinen kiertokulku:** Latauksen valmistuttua aamulla järjestelmä tyhjentää Lovelace-ruudukot automaattisesti takaisin puhtaiksi (harmaiksi), jolloin Dashboard pysyy aina siistinä.
*   **Älykkäät kuukausisäästökortit:** Pääsivun yläreunassa olevat dynaamiset säästökortit vertaavat suoritettuja latauksia päivän keskihintaan ja näyttävät livenä kuluvan kuukauden säästöt euroina.
*   **Taskukokoiset loppuraportit:** Heti ajon päättyessä OnePlus-puhelin piippaa korkealla prioriteetilla tarkan yhteenvedon: *"Sähköauto ladattu: 13,01 kWh | 0,05 €"*.
