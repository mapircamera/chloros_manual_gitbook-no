# Legge til filer i et prosjekt

Når du har opprettet eller åpnet et prosjekt i Chloros, er neste trinn å legge til multispektrale bilder for å starte behandlingen. Fanen «File Browser» <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> gjør det enkelt å importere bilder og administrere datasettet ditt.

## Åpne filbrowseren

1. Åpne eller opprett et prosjekt i Chloros
2. Klikk på **Filbrowser**-ikonet <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> i venstre sidefelt
3. Filbrowser-panelet viser prosjektets filliste

{% hint style="info" %}
**Støttede filtyper**:

* **Survey3W / Survey3N**: RAW+JPG-par og JPG-bilder (RAW+JPG anbefales)
* **LATTICE**: `.tif` / `.tiff`-opptak — lagret av Chloros-kamerakontroll eller av en LATTICE-hub
* **Lyssensordata**: `.daq`-opptak (DAQ-U/M/E) og DAQ-M `.csv`-nedstrålingslogger — importert sammen med bilder for å drive reflektanskalibrering
{% endhint %}

***

## Legge til bilder i prosjektet ditt

Det finnes to hovedmåter å legge til bilder i prosjektet ditt på:

### Metode 1: Legg til filer

Bruk dette alternativet til å importere enkeltstående bildefiler eller et lite utvalg av filer.

1. Klikk på **«Legg til filer»**-<img src="../.gitbook/assets/image (3).png" alt="" data-size="line">knappen øverst i Filbrowser-panelet
2. Naviger til mappen som inneholder bildene dine
3. Velg én eller flere bildefiler (hold nede **Ctrl** for å velge flere filer)
4. Klikk på **«Åpne»** for å importere de valgte filene

### Metode 2: Legg til mappe

Bruk dette alternativet for å importere alle bildene fra en mappe på en gang. Du kan velge **flere mapper** i én dialogboks.

1. Klikk på **«Legg til mappe»**-<img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line">-knappen øverst i Filbrowser-panelet
2. Naviger til og velg mappen(e) som inneholder bildene fra opptakssesjonen din
3. Klikk på **«Velg mappe»** for å importere alle støttede bilder

{% hint style="info" %}
**Filer som ikke lastes inn, rapporteres.** Hvis en mappe inneholder filer som Chloros gjenkjenner, men ikke kan laste inn, får du en advarsel – bildene forsvinner ikke usynlig fra rutenettet.
{% endhint %}

***

## Importere LATTICE-opptaksmapper

LATTICE-opptak lagres med **én undermappe per eksportnivå** – for eksempel `raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` – med den tilhørende nedstrømningsfilen `.daq` i roten:

```
output/
├── raw/           capture_<timestamp>_SN<serial>_raw.tif
├── debayered/     capture_<timestamp>_SN<serial>_debayered.tif
├── preview/       capture_<timestamp>_SN<serial>_display.tif
└── *.daq          the downwelling reading matched to the capture
```

**Velg «Legg til mappe» i roten til opptakene** (`output/` ovenfor). Når den valgte mappen ikke inneholder bilder selv, men har undermapper, går Chloros automatisk ned i dem — undermappene på det aktuelle nivået og rotmappen `.daq` hentes alle på én gang.**Slik importeres opptak:*** Hvert opptak importeres som et **enkelt bilde**, gruppert etter opptak (ikke én oppføring per nivå). De andre nivåene i det samme opptaket vises som visningsmoduser for det ene bildet.
* **Behandlingen starter alltid fra råbildet.** De andre nivåene kan vises, men bare `raw` føres gjennom behandlingsprosessen — å behandle et allerede behandlet produkt på nytt ville føre til at korreksjonene ble brukt to ganger, så Chloros avvises. En reimportert eksport kan aldri ta en opptakets råbildeplass.
* En opptaksmappe som er lagret **uten** råbildeimport, vises normalt, men behandlingen hopper over den og oppgir dette i loggen. (Flagget CLI `--input-level` kan tvinge frem et inngangspunkt i dette tilfellet — se [CLI-referansen](../reference/cli-reference.md#what-a-captures-folder-looks-like).)**LATTICE-hub-økter** importeres på samme måte: pek på «Add Folder» i sesjonsmappen som er kopiert fra hubben (den inneholder `raw/` samt `previews/`), sammen med eventuelle DAQ-M `.csv`-logger. Hvis kalibreringen av kameraet eller DAQ-enheten ikke er lagret i hurtigbufferen på maskinen din ennå, henter Chloros den automatisk ved hjelp av serienummeret ved import (krever internettilgang én gang).***

## Forståelse av filbrowsertabellen

Når bildene er importert, vises de i en tabell med følgende kolonner:

### Filnavn

* Opprinnelig filnavn fra kameraet
* Opprettholder kameraets navngivningskonvensjon (f.eks. IMG\_0001.RAW eller capture\_20260816\_101500\_SN213800234\_raw.tif)

### Tidsstempel

* Dato og klokkeslett da bildet ble tatt
* Hentet fra bildets EXIF-metadata
* Brukes til lyssensormatching, PPK-synkronisering og planlegging av kalibreringsmål

### Kameramodell

* Automatisk gjenkjent kamera- og filterkonfigurasjon
* Eksempler på Survey3: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* LATTICE-eksempler: LATT-M3M-L41-F550, LATT-M3C-L87-FRGN
* Brukes til å anvende riktige behandlingsprofiler

### Målkolonne (avkryssingsrute)

* Merk av i denne boksen for bilder som inneholder kalibreringsmål
* Når minst ett bilde er merket av, **skannes kun de merkede bildene** for mål
* Se [Velge målbilder](choosing-target-images.md) for detaljer

### Vise bildemetadata

Når du klikker på veksleknappen øverst til høyre over tabellen, vises metadataene for det valgte bildet i bilderutenettet.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Lyssensorfiler i prosjektet ditt

* `.daq`- og `.csv`-filer vises i filbrowseren, men er ikke klikkbare bilder — de angir nedadgående irradians for reflektanskalibrering.
* Hver importert `.daq`/`.csv`-fil vises i **Prosjektinnstillinger → DAQ-lyssensor**, der du kan se hvilken diffusorhettekorreksjon som gjelder for hver fil. Se [Justere prosjektinnstillinger](adjusting-project-settings.md).
* Opptak du gjør i fanen **Lyssensorer** legges automatisk til i det åpne prosjektet – ingen manuell import er nødvendig.***

## Administrere filer i prosjektet

### Fjerne filer

Slik fjerner du uønskede bilder fra prosjektet:

1. Velg ett eller flere bilder i tabellen i filbrowseren
2. Klikk på **«Fjern valgte»**-<img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">knappen
3. Bekreft fjerningen (filene slettes ikke fra disken, men fjernes kun fra prosjektet)

### Sortering og filtrering

* **Sorter etter kolonne**: Klikk på en hvilken som helst kolonneoverskrift for å sortere bilder
* **Sortering etter tidsstempel**: Nyttig for å organisere kronologiske opptaksserier
* **Filter etter kameramodell**: Gruppér bilder etter kameratype hvis du bruker flere kameraer***

## Forhåndsvisning av bilder

### Vise hele bildet

Klikk på en hvilken som helst miniatyrbilde i filbrowseren for å vise det i hovedforhåndsvisningsområdet:

1. Bildet vises i det sentrale forhåndsvisningspanelet
2. Bruk zoomkontrollene for å se nærmere på bildedetaljer
3. Naviger mellom bildene ved hjelp av piltastene

### Hurtignavigering

* **Forrige bilde**: Klikk på venstre pil eller trykk på ←-tasten
* **Neste bilde**: Klikk på høyre pil eller trykk på →-tasten
* **Zoom inn/ut**: Bruk mushjulet eller zoomknappene
* **Panorer**: Klikk og dra på bildet når du har zoomet inn***

## Håndtering av dupliserte filer

Chloros oppdager og ignorerer automatisk dupliserte filer:

* Filer med identiske filnavn hoppes over
* Forhindrer utilsiktet dobbel behandling
* Advarselsmelding vises når duplikater oppdages

{% hint style="warning" %}
**Viktig**: Ikke endre navn på eller modifiser de originale bildefilene før import. Chloros er avhengig av originale filnavn og metadata for korrekt behandling.
{% endhint %}

***

## Datasett med blandede kameraer

Hvis prosjektet ditt inneholder bilder fra flere MAPIR-kameraer:

1. Chloros oppdager automatisk hver kameramodell — Survey3, LATTICE eller en blanding
2. Hver kameratype behandles med sin tilhørende kalibreringsprofil
3. Filbrowseren viser kameramodellen i kolonnen «Kameramodell»
4. Hvert kamera får sin egen utdatamappestruktur når det behandles

**Eksempler på scenarier**: Survey3W RGN + Survey3N OCN oppsett med to kameraer, eller et LATTICE-array med en RGB-master og flere smalbåndsmoduler***

## Anbefalte fremgangsmåter

### Organiser før import

* Oppbevar kalibreringsmålbildene i samme mappe som kartleggingsbildene
* Oppbevar lyssensorfilene `.daq` / `.csv` fra hver opptakssesjon sammen med bildene fra den aktuelle sesjonen
* Bevar den opprinnelige mappestrukturen fra kameraet/SD-kortet/huben
* Ikke bland datasett fra forskjellige opptakssesjoner i ett prosjekt

### Filnavngivning

* Behold de opprinnelige filnavnene fra kameraet (IMG\_0001.RAW, capture\_..., osv.)
* Ikke gi filene nye navn før import
* De opprinnelige navnene inneholder viktig metadata

### Kalibreringsmålbilder

* Inkluder alltid 1–2 kalibreringsmålbilder per sesjon (Survey3; for LATTICE kan en DAQ-opptak erstatte dette — se [Velge målbilder](choosing-target-images.md))
* Ta bilder av målene før og etter opptakssesjonen
* Plasser målene under de samme lysforholdene som opptaksområdet
* Merk målbildene ved å bruke avmerkingsboksen «Mål»

***

## Vanlige problemer og løsninger

### Bilder vises ikke etter import

**Mulige årsaker:**

* Filformatet støttes ikke (se listen over støttede filtyper øverst på denne siden)
* Bildene er fra kameraer som ikke er av typen MAPIR (se [Støttede kameraer](../supported-cameras.md))
* Filskade eller ufullstendig overføring fra SD-kort

**Løsning**: Kontroller at filformatet og kameramodellen er kompatible, og se på advarselen om filinnlasting for å finne ut nøyaktig hvilke filer som mislyktes

### Kameramodell oppdages ikke

**Mulige årsaker:**

* Endrede EXIF-metadata
* Bilder redigert i ekstern programvare
* Ufullstendig filoverføring

**Løsning**: Importer originale, uendrede filer på nytt fra kameraet/SD-kortet

### Manglende tidsstempler

**Mulige årsaker:**

* Kameraklokken er ikke riktig innstilt
* EXIF-data fjernet av ekstern programvare

**Løsning**: Kontroller at klokkeinnstillingene på kameraet var riktige under opptaket

### Gjenåpnet prosjekt rapporterer manglende filer

Hvis kildefilene er flyttet eller slettet siden prosjektet sist ble åpnet, vil Chloros vise deg **hvilke** filer som mangler, i stedet for å åpne et tomt rutenett. Gjenopprett filene til de opprinnelige stiene, eller fjern de manglende oppføringene og importer på nytt.***

## Neste trinn

Når filene er importert:

1. **Gjennomgå fillisten** – Sørg for at alle bildene er lastet inn riktig
2. **Sjekk kameramodeller** – Kontroller at kameraene er gjenkjent riktig
3. **Merk målbilder** – Se [Velge målbilder](choosing-target-images.md)
4. **Juster innstillingene** – Konfigurer behandlingsalternativene i [Prosjektinnstillinger](adjusting-project-settings.md)
5. **Start behandlingen** – Se [Starte behandlingen](starting-the-processing.md)

For detaljert informasjon om prosjektkonfigurasjon, se [Justere prosjektinnstillinger](adjusting-project-settings.md).
