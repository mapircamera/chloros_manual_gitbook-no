# Legge til filer i et prosjekt

Når du har opprettet eller åpnet et prosjekt i Chloros, er neste trinn å legge til multispektrale bilder for å starte behandlingen. Fanen «Filbrowser»<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> gjør det enkelt å importere bilder og administrere datasettet ditt.

## Åpne filbrowseren

1. Åpne eller opprett et prosjekt i Chloros
2. Klikk på **filbrowser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> i venstre sidefelt
3. Filbrowser-panelet viser prosjektets filliste

{% hint style="info" %}
**Støttede filtyper**: Chloros støtter RAW+JPG- og JPG-bildefiler fra MAPIR, Survey3W og Survey3N-kameraer. Kun RAW+JPG anbefales.
{% endhint %}

***

## Legge til bilder i prosjektet ditt

Det er to hovedmåter å legge til bilder i prosjektet ditt på:

### Metode 1: Legg til filer

Bruk dette alternativet til å importere enkeltstående bildefiler eller et lite utvalg av filer.

1. Klikk på **&quot;Legg til filer&quot;** <img src="../.gitbook/assets/image.png" alt="" data-size="line"> øverst i filbrowseren
2. Naviger til mappen som inneholder bildene dine
3. Velg én eller flere bildefiler (hold **Ctrl** nede for å velge flere filer)
4. Klikk på **«Åpne»** for å importere de valgte filene

### Metode 2: Legg til mappe

Bruk dette alternativet for å importere alle bildene fra en mappe på en gang.

1. Klikk på **&quot;Legg til mappe&quot;** <img src="../.gitbook/assets/image (1).png" alt="" data-size="line"> øverst i filbrowseren
2. Naviger til og velg mappen som inneholder bildene fra opptakssesjonen
3. Klikk på **«Velg mappe»** for å importere alle støttede bilder fra den mappen***

## Forstå filbrowser-tabellen

Når bildene er importert, vises de i en tabell med følgende kolonner:

### Filnavn

* Opprinnelig filnavn fra kameraet
* Opprettholder kameraets navnekonvensjon (f.eks. IMG\_0001.RAW)

### Tidsstempel

* Dato og klokkeslett da bildet ble tatt
* Hentet fra bildets EXIF-metadata
* Brukes til PPK-synkronisering og deteksjon av kalibreringsmål

### Kameramodell

* Automatisk oppdaget kamera- og filterkonfigurasjon
* Eksempler: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Brukes til å anvende riktige behandlingsprofiler

### Målkolonne (avkryssingsboks)

* Merk av i denne boksen for bilder som inneholder kalibreringsmål
* Fremskynder målgjenkjenningen betydelig under behandlingen
* Se [Velge målbilder](choosing-target-images.md) for detaljer

### Vise bildemetadata

Ved å klikke på veksleknappen øverst til høyre over tabellen vises metadataene for det valgte bildet i bildeområdet.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Administrere filer i prosjektet ditt

### Fjerne filer

Slik fjerner du uønskede bilder fra prosjektet ditt:

1. Velg ett eller flere bilder i tabellen Filbrowser
2. Klikk på **&quot;Fjern valgte&quot;** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line"> -knappen
3. Bekreft fjerningen (filene slettes ikke fra disken, men fjernes kun fra prosjektet)

### Sortering og filtrering

* **Sorter etter kolonne**: Klikk på en hvilken som helst kolonneoverskrift for å sortere bilder
* **Sortering etter tidsstempel**: Nyttig for å organisere kronologiske opptaksserier
* **Filter for kameramodell**: Grupper bilder etter kameratype hvis du bruker flere kameraer***

## Forhåndsvisning av bilder

### Visning av fullstendig bilde

Klikk på et hvilket som helst miniatyrbilde i filbrowseren for å vise det i hovedforhåndsvisningsområdet:

1. Bildet vises i det sentrale forhåndsvisningspanelet
2. Bruk zoomkontrollene til å inspisere bildedetaljer
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

## Blandede kameradatamengder

Hvis prosjektet ditt inneholder bilder fra flere MAPIR-kameraer:

1. Chloros oppdager automatisk hver kameramodell
2. Hver kameratype behandles med sin tilhørende kalibreringsprofil
3. Filbrowseren viser kameramodellen i kolonnen «Kameramodell»
4. Behandlingen bruker riktige innstillinger for hver kameratype

**Eksempelscenario**: Survey3W RGN + Survey3N OCN oppsett med to kameraer***

## Beste praksis

### Organiser før import

* Oppbevar kalibreringsmålbilder i samme mappe som kartleggingsbilder
* Behold den opprinnelige mappestrukturen fra kameraet/SD-kortet
* Ikke bland datasett fra forskjellige økter i ett prosjekt

### Filnavngivning

* Behold de opprinnelige filnavnene fra kameraet (IMG\_0001.RAW, osv.)
* Ikke gi filene nye navn før import
* Originale navn inneholder viktig metadata

### Kalibreringsmålbilder

* Inkluder alltid 1–2 kalibreringsmålbilder per økt
* Ta bilder av målene før og etter opptakssesjonen
* Plasser målene under de samme lysforholdene som opptaksområdet
* Merk målbildene ved å bruke avkrysningsruten «Target» for å øke behandlingshastigheten

***

## Vanlige problemer og løsninger

### Bilder vises ikke etter import

**Mulige årsaker:**

* Filformatet støttes ikke (kun RAW+JPG og JPG fra MAPIR-kameraer)
* Bildene er fra kameraer som ikke er av typen MAPIR (se [Støttede kameraer](../supported-cameras.md))
* Filskade eller ufullstendig overføring fra SD-kort

**Løsning**: Kontroller kompatibiliteten mellom filformat og kameramodell

### Kameramodell oppdages ikke

**Mulige årsaker:**

* Endrede EXIF-metadata
* Bilder redigert i ekstern programvare
* Ufullstendig filoverføring

**Løsning**: Importer originale, uendrede filer fra kamera/SD-kort på nytt

### Manglende tidsstempler

**Mulige årsaker:**

* Kameraklokken er ikke riktig innstilt
* EXIF-data fjernet av ekstern programvare

**Løsning**: Kontroller at kameratidsinnstillingene var riktige under opptaket***

## Neste trinn

Når filene er importert:

1. **Gjennomgå fillisten** – Sørg for at alle bildene er lastet inn riktig
2. **Sjekk kameramodeller** – Kontroller at kameraet er gjenkjent riktig
3. **Merk målbilder** – Se [Velge målbilder](choosing-target-images.md)
4. **Juster innstillinger** – Konfigurer behandlingsalternativer i [Prosjektinnstillinger](adjusting-project-settings.md)
5. **Start behandlingen** – Se [Starte behandlingen](starting-the-processing.md)

For detaljert informasjon om prosjektkonfigurasjon, se [Justere prosjektinnstillinger](adjusting-project-settings.md).
