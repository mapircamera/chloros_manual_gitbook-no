# Åpne et bilde i fullskjermmodus

Chloros Image Viewer har et dedikert fullskjermgrensesnitt for visning, analyse og bearbeiding av multispektrale bilder. Uansett om du viser originale bilder eller bearbeidede resultater, tilbyr Image Viewer kraftige verktøy for inspeksjon og analyse.

## Åpne Image Viewer

### Fra filbrowseren

Den vanligste måten å åpne et bilde i Image Viewer på:

1. Forsikre deg om at du er i fanen **File Browser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Klikk på et hvilket som helst **miniatyrbilde** i bildegitteret
3. Bildet åpnes i **hovedforhåndsvisningsområdet** (midt på skjermen)
4. Bildet er nå lastet inn og klart for visning i fullskjermmodus

### Åpne fanen Image Viewer

Når et bilde er lastet inn i forhåndsvisningsområdet:

1. Klikk på **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> i venstre sidefelt
2. Fanen Bildeselger åpnes og viser det valgte bildet i fullskjermmodus
3. Avanserte visnings- og analyseverktøy blir tilgjengelige i venstre sidefelt

***

## Oversikt over grensesnittet til Bildeselger

### Hovedvisningsområdet

Den største delen av skjermen viser bildet ditt:

* **Full oppløsning**: Bilder vises i opprinnelig oppløsning
* **Zoombar**: Bruk kontrollene eller mushjulet til å zoome
* **Panorerbar**: Klikk og dra for å flytte rundt når du har zoomet
* **Bildeforhold opprettholdes**: Bildene skaleres proporsjonalt

***

## Visningsalternativer

### Grunnleggende bildnavigering

#### Bla gjennom bilder

Naviger gjennom bildesettet ditt ved hjelp av hurtigtaster eller knapper:

* **Neste bilde**: Klikk på →-knappen eller trykk på **→** (høyre pil)-tasten
* **Forrige bilde**: Klikk på ←-knappen eller trykk på **←** (venstre pil)-tasten
* **Gå til et bestemt bilde**: Gå tilbake til filbrowseren og klikk på ønsket miniatyrbilde

#### Zoomkontroller

Juster forstørrelsen for å se på bildedetaljer:

**Zoom inn:**

* Klikk på **+** (pluss)-knappen
* Trykk på **+** eller **=**-tasten
* Rull mushjulet **opp**

**Zoom ut:**

* Klikk på **−** (minus)-knappen
* Trykk på **−** (minus)-tasten
* Rull mushjulet **ned**

**Tilpass til skjerm:**

* Klikk på **↔** (Tilpass)-knappen
* Trykk på **0** (null)-tasten
* Dobbeltklikk på bildet

#### Panorering når du zoomer

Når du zoomer utover skjermstørrelsen:

1. Flytt musepekeren over bildet
2. Klikk og **hold venstre museknapp nede**
3. **Dra** for å flytte bildet rundt
4. Slipp for å stoppe panorering

**Alternativ**: Bruk piltastene for å panorere i små trinn

***

## Inspeksjon av pikselverdi

### Visning av pikselverdier ved markøren

Når du beveger musepekeren over bildet, vises pikselverdiene i sanntid:

**Plassering av verdivisning:**

* **Flytende tall og rød linje i indeks LUT-gradientlegenden på høyre side**
* **Når du zoomer inn ytterligere, vises flytende verdi nær markøren og uthevet piksel**
* Viser verdier for piksel **under markøren eller uthevet**
* Oppdateres når du beveger musen

***

## Bildetyper du kan vise

### Originalbilder (før behandling)

**RAW + JPG-bilder fra kamera:**

* Viser RAW-data som forhåndsvisning
* Viser originale, ukorrigerte verdier
* Nyttig for å sjekke bildekvaliteten før behandling

### Kalibrerte refleksjonsbilder

**Etter behandling:**

* Vignettkorrigert
* Refleksjonskalibrert
* Multibånd TIFF (Red, Green, NIR, etc.)
* Vitenskapelige data klare for analyse

### Indeksbilder

**NDVI, NDRE, GNDVI, etc. (\_NDVI.tif-filer):**

* Enkeltbånds gråtonebilder
* Pikselverdier representerer indeksberegningsresultater
* Område vanligvis -1 til +1 for normaliserte indekser
* Kan bruke farge-LUT-er for visualisering

***

## Indeks- og LUT-applikasjon

Bruk multispektrale indekser og farge-Look-Up-tabeller:

1. Finn **Index/LUT Sandbox** i **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sidefeltet
2. Velg vegetasjonsindeks (NDVI, NDRE, etc.)
3. Velg multispektral formel, eller lag din egen tilpassede formel (kun Chloros+)
4. Bruk fargeluttgradient for visualisering
5. Juster verdier og terskler

Se [Indeks/LUT Sandbox](index-lut-sandbox.md) for detaljerte instruksjoner.

***

## Tastatursnarveier

### Navigering

* **→** (høyre pil): Neste bilde
* **←** (venstre pil): Forrige bilde
* **Hjem**: Første bilde i listen
* **End**: Siste bilde i listen

### Zoom

* **+** eller **=**: Zoom inn
* **−**: Zoom ut
* **0** (null): Tilpass til skjermen
* **Mushjul**: Zoom inn/ut

### Visningskontroller

* **P**: Veksle mellom pikselprosentmodus
* **L**: Veksle mellom lagpaneler
* **Esc**: Lukk fullskjerm eller gå tilbake til filbrowseren

### Annet

* **Ctrl+S**: Lagre gjeldende bilde
* **F**: Fullskjermmodus (hvis tilgjengelig)

***

### Kontrollere indeksberegninger

Kontroller at indeksene er beregnet riktig:

1. Åpne NDVI eller et annet indeksbilde
2. Kontroller vegetasjonsområdene:
   * **NDVI**: Bør vise 0,4–0,9 for sunne planter
   * **NDRE**: Høyere verdier for kraftig vekst
   * **GNDVI**: Ligner på NDVI, men klorofyllfølsom
3. Kontroller ikke-vegetasjon:
   * **Jord**: Nær 0 eller svakt negativt
   * **Vann**: Negative verdier (-0,5 til 0)

***

## Feilsøking av visningsproblemer

### Bildet åpnes ikke

**Mulige årsaker:**

* Filen ble ødelagt under behandlingen
* Filformatet støttes ikke
* Utilstrekkelig minne for store bilder

**Løsninger:**

1. Prøv å åpne i ekstern visningsprogram for å kontrollere filens integritet
2. Kontroller at filformatet samsvarer med forventet type
3. Lukk andre programmer for å frigjøre minne
4. Prøv et mindre/annet bilde

### Svart eller hvitt bilde

**Mulige årsaker:**

* Verdier utenfor skjermens kapasitet
* 32-biters flytende bilde med uvanlige verdier
* Feil i indeksberegningen

**Løsninger:**

1. Kontroller pikselverdiene – hvis alle er svært lave eller svært høye, juster visningsområdet.
2. Prøv å åpne i QGIS eller lignende med automatisk justering av området.
3. Kontroller feilsøkingsloggen fra behandlingen for feil.

### Pikselverdiene virker feil

**Mulige årsaker:**

* Visning av feil bilde (original vs. behandlet)
* Kalibreringen ble ikke brukt riktig
* Lyssensordata ble ikke inkludert i inndata
* Prosentmodus ble slått feil

**Løsninger:**

1. Kontroller at du viser behandlet utdata (sjekk filnavnsuffikset)
2. Sjekk statusen til prosentmodusknappen
3. Sammenlign med bilder fra samme datasett som du vet er gode

***

## Neste trinn

Nå som du kan vise bilder i fullskjerm:

* [**Bildelag**](image-layers.md) – Lær om multibåndvisualisering
* [**Indeks/LUT-sandkasse**](index-lut-sandbox.md) – Bruk tilpassede indekser og fargekartlegging
* [**Multispektrale indeksformler**](../project-settings/multispectral-index-formulas.md) – Forstå tilgjengelige indekser

For behandlingsarbeidsflyt, se:

* [**Behandling av bilder (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Komplett behandlingsveiledning
