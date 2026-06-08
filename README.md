# Home Assistant Nordpool Älykäs Lataus- ja Lämmitysjärjestelmä (v2.2)

Täysin automaattinen, paikallinen ja erittäin tehokas energianoptimointijärjestelmä Home Assistantille. Järjestelmä seuraa reaaliaikaisia pörssisähkön hintoja (Nordpool) ja ajastaa suuren kuormituksen laitteet – erityisesti **sähköauton (EV)** ja **kodin lämmitysjärjestelmän** – vuorokauden halvimmille tunneille.

Järjestelmä kiertää Home Assistantin tekstimuuttujien tiukan 255 merkin rajoituksen käyttämällä kustomoitua **kaksoiskenttä- ja indeksiparituskoneistoa**.

## 📊 Keskeiset ominaisuudet
*   **Dynaaminen 15 minuutin ajastus:** Hakemisto ja ajastus pohjautuvat tarkkoihin 15 minuutin pörssisähkön hintajaksoihin.
*   **Automaattinen hintasynkronointi:** Noin klo `14:17` (tai heti, kun huomisen hinnat vahvistetaan), järjestelmä laskee tulevan yön optimaaliset latausikkunat automaattisesti käyttäjän oletusasetusten mukaan.
*   **Keskiyön huoltoautomaatio (Midnight Cleanup):** Siirtää huomisen luonnosjaksot automaattisesti kuluvan päivän aktiivisiksi ajoiksi täsmälleen klo `00:00:00`.
*   **Kaksoiskenttämuisti (`_part2` + `*`):** Kiertää Home Assistantin tilatietojen 255 merkin rajan jakamalla teholokit lennosta kahteen eri tekstikenttään kustomoidun katkomerkin avulla.
*   **Indeksiparitus (`idx:teho`):** Estää listojen siirtymävirheet, jos automaatio käynnistetään kesken varttitunnin jakson, sitomalla tehomittaukset kiinteästi jaksojen ID-numeroihin.
*   **Live-kuukausisäästölaskuri:** Visualisoi toteutuneet kumulatiiviset kuukausisäästöt (€) suoraan päädashboardilla verrattuna päivän keskihintaan.
*   **Reaaliaikaiset push-ilmoitukset:** Lähettää korkean prioriteetin raportin ja tarkan kulutusyhteenvedon (`kWh` ja `€`) suoraan käyttäjän älypuhelimeen heti lataussyklin valmistuttua.

---

## 📂 Repositorion rakenne

Tämä repositorio sisältää tuotantovalmiit koodit ja käyttöliittymäkonfiguraatiot suoraan Home Assistantin ytimestä:

```text
├── automations.yaml       # Taustaprosessit, reaaliaikainen ohjaus & synkronointi
├── configuration.yaml     # Custom-pohjaiset template-sensorit, arkistot & helperit
├── scripts.yaml           # Matemaattiset hakualgoritmit ja loppuraportit
└── dashboards/            # Käyttöliittymäkoodit (HA Raw Configuration Editorista)
    ├── 0.debug.txt        # Diagnostiikka- ja testausnäkymä
    ├── 1.main.txt         # Pääsivu live-seurannalla ja säästökorteilla
    ├── 2.sahkoauto.txt    # Sähköauton erikoistunut ajastusmatriisi asetuksineen
    └── 3.lammitin.txt     # Lämmittimen ohjausruudukko dynaamisella kuormituksella
```

---

## 🛠️ Järjestelmän arkkitehtuuri ja toiminta

### Datan kulku järjestelmässä
1. **Haku:** `sensor.nordpool_oma` vastaanottaa uudet hintatiedot.
2. **Laskenta:** Skripti lukee oletusasetukset (`kesto` ja `aikavali`) ja eristää halvimmat jaksot.
3. **Luonnos:** Valinnat piirtyvät dynaamisesti Dashboardin ruudukkoon luonnoslistaksi.
4. **Lukitus:** Käyttäjä tallentaa valinnat tai automaatio lukitsee ne aktiiviseksi suoritusjonoksi.
5. **Seuranta:** WiZ-etäpistorasiat (`sensor.sahkoauto_teho_laskettu` ja `_lammitin_teho_laskettu`) mittaavat tehoa livenä.
6. **Raportointi:** Ajon päättyessä tekstipuskurit yhdistyvät, lokit puretaan, arkistot päivittyvät ja kännykkään lähetetään push-ilmoitus.

---

## 🚀 Käyttöönotto puhtaaseen järjestelmään

Pystyttääksesi tämän pörssisähköjärjestelmän uuteen Home Assistant -ympäristöön, seuraa näitä vaiheita:

### Vaihe 1: Alusta järjestelmän helperit (muuttujat)
Säästääksesi aikaa käyttöliittymän klikkailussa, kopioi `configuration.yaml` -tiedostosta löytyvä laaja helper-segmentti suoraan tuotantokoneesi `configuration.yaml` -tiedostoon. Tämä luo kaikki tarvittavat `input_text`, `input_number`, `input_datetime`, `input_boolean` ja `input_select` -muuttujat automaattisesti uudelleenkäynnistyksen yhteydessä.

### Vaihe 2: Yhdistä kooditiedostot
1. Liitä sensorimääritykset omaan `configuration.yaml` -tiedostoosi.
2. Liitä automaatiot omaan `automations.yaml` -tiedostoosi.
3. Liitä laskenta- ja raporttiskriptit omaan `scripts.yaml` -tiedostoosi.
4. Mene Home Assistantissa valikkoon **Kehitystyökalut > YAML** ja lataa kaikki komponentit uudelleen.

### Vaihe 3: Pystytä Dashboardit
1. Luo uusi tyhjä työpöytä tai välilehti Lovelace UI -käyttöliittymään.
2. Klikkaa oikeasta yläkulmasta kolmea pistettä -> **Muokkaa työpöytää** (Edit Dashboard).
3. Klikkaa uudelleen kolmea pistettä -> **Raw-konfiguraatiomuokkain** (Raw Configuration Editor).
4. Liitä vastaava raakateksti `dashboards/` -kansiosta löytyvistä tiedostoista.
*Huom: Varmista, että olet asentanut HACS:n kautta kortit `custom:flex-table-card` ja `custom:button-card`.*

### Vaihe 4: Rekisteröi kännykkäilmoitukset
Varmistaaksesi push-ilmoitusten toiminnan, asenna virallinen **Home Assistant Companion App** puhelimeesi, yhdistä se palvelimeesi ja salli ilmoitukset sovelluksen asetuksista. Päivitä `notify.mobile_app_oman_laitteesi_nimi` -kohdat `scripts.yaml` -tiedostossa vastaamaan puhelimesi rekisteröityä nimeä.

---

## 🛡️ Tärkeä turvallisuus- ja laitteistohuomautus
Jos ajat Home Assistantia mini-PC-koneella tai vanhemmalla pöytätietokoneella, **älä kytke USB Zigbee-koordinaattoria suoraan tietokoneen emolevyn USB-portteihin**. USB 3.0 -portit ja sisäiset komponentit aiheuttavat **voimakasta 2,4 GHz radiotaajuushäiriötä**, joka estää sensorien parituksen ja pätkii signaalia [C1].

Kytke koordinaattori (esim. *Sonoff Zigbee 3.0 USB Dongle Plus-E*) aina **1,5–2 metrin pituiseen USB-jatkojohtoon**, jotta saat tuotua antennin fyysisesti etäälle tietokoneen keskusyksiköstä [C1].

---
## 📄 Lisenssi
Tämä arkkitehtuuri on vapaasti jaettavissa ja muokattavissa avoimen lähdekoodin automaatiokäytäntöjen mukaisesti. Optimoi ruudukkosi ja säästä sähkölaskussa!
