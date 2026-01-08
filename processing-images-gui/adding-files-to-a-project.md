# Legge til filer i et prosjekt

Når du har opprettet eller åpnet et prosjekt i Chloros, er neste trinn å legge til multispektrale bilder for å starte behandlingen. Filbrowseren<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> gjør det enkelt å importere bilder og administrere datasettet ditt.

## Åpne filbrowseren

1. Åpne eller opprett et prosjekt i Chloros
2. Klikk på **Filbrowser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> i venstre sidefelt
3. Filbrowser-panelet viser filisten for prosjektet ditt

{% hint style="info" %}
**Støttede filtyper**: Chloros støtter RAW+JPG- og JPG-bildefiler fra MAPIR Survey3W og Survey3N kameraer. Kun RAW+JPG anbefales.
{% endhint %}

***

## Legge til bilder i prosjektet ditt

Det er to hovedmåter å legge til bilder i prosjektet ditt på:

### Metode 1: Legg til filer

Bruk dette alternativet for å importere enkeltstående bildefiler eller et lite utvalg av filer.

1. Klikk på **«Legg til filer»** <img src="../.gitbook/assets/image.png" alt="" data-size="line"> øverst i filbrowserpanelet
2. Naviger til mappen som inneholder bildene dine
3. Velg en eller flere bildefiler (hold **Ctrl** nede for å velge flere filer)
4. Klikk på **«Åpne»** for å importere de valgte filene

### Metode 2: Legg til mappe

Bruk dette alternativet for å importere alle bildene fra en mappe samtidig.

1. Klikk på **&quot;Legg til mappe&quot;** <img src="../.gitbook/assets/image (1).png" alt="" data-size="line"> øverst i filbrowserpanelet.
2. Naviger til og velg mappen som inneholder bildene fra opptakssesjonen.
3. Klikk på **«Velg mappe»** for å importere alle støttede bilder fra den mappen.

***

## Forstå filbrowser-tabellen

Når bildene er importert, vises de i en tabell med følgende kolonner:

### Filnavn

* Opprinnelig filnavn fra kameraet
* Opprettholder kameraets navngivningskonvensjon (f.eks. IMG\_0001.RAW)

### Tidsstempel

* Dato og klokkeslett da bildet ble tatt
* Hentet fra EXIF-metadata i bildet
* Brukes til PPK-synkronisering og kalibreringsmåldeteksjon

### Kameramodell

* Automatisk oppdaget kamera- og filterkonfigurasjon
* Eksempler: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Brukes til å bruke riktige behandlingsprofiler

### Målkolonne (avkryssingsboks)

* Merk av i denne boksen for bilder som inneholder kalibreringsmål
* Fremskynder måldeteksjonen betydelig under behandlingen
* Se [Velge målbilder](choosing-target-images.md) for detaljer

### Vise bildemetadata

Ved å klikke på veksleknappen øverst til høyre over tabellen vises metadataene for det valgte bildet i bildeområdet.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Administrere filer i prosjektet ditt

### Fjerne filer

Slik fjerner du uønskede bilder fra prosjektet ditt:

1. Velg ett eller flere bilder i tabellen Filbrowser
2. Klikk på **&quot;Fjern valgt&quot;** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line"> .
3. Bekreft fjerningen (filene slettes ikke fra disken, bare fjernes fra prosjektet).

### Sortering og filtrering

* **Sorter etter kolonne**: Klikk på en kolonneoverskrift for å sortere bildene.
* **Sorter etter tidsstempel**: Nyttig for å organisere kronologiske bildesekvenser.
* **Kamerafiltermodell**: Grupper bilder etter kameratype hvis du bruker flere kameraer.

***

## Forhåndsvisning av bilder

### Visning av hele bildet

Klikk på en miniatyrbilde i filbrowseren for å vise det i hovedforhåndsvisningsområdet:

1. Bildet vises i forhåndsvisningspanelet i midten
2. Bruk zoomkontrollene til å se på bildedetaljer
3. Naviger mellom bildene ved hjelp av piltastene

### Hurtignavigering

* **Forrige bilde**: Klikk på venstre pil eller trykk på ←-tasten
* **Neste bilde**: Klikk på høyre pil eller trykk på →-tasten
* **Zoom inn/ut**: Bruk mushjulet eller zoomknappene
* **Panorer**: Klikk og dra på bildet når du har zoomet inn

***

## Håndtering av dupliserte filer

Chloros oppdager og ignorerer automatisk dupliserte filer:

* Filer med identiske filnavn hoppes over
* Forhindrer utilsiktet dobbeltbehandling
* Advarselsmelding vises når duplikater oppdages

{% hint style="warning" %}
**Viktig**: Ikke gi originale bildefiler nytt navn eller endre dem før du importerer dem. Chloros er avhengig av originale filnavn og metadata for riktig behandling.
{% endhint %}

***

## Blandede kameradataset

Hvis prosjektet ditt inneholder bilder fra flere MAPIR-kameraer:

1. Chloros oppdager automatisk hver kameramodell
2. Hver kameratype behandles med sin riktige kalibreringsprofil
3. Filbrowseren viser kameramodellen i kolonnen Kameramodell
4. Behandlingen bruker riktige innstillinger for hver kameratype

**Eksempel på scenario**: Survey3W RGN + Survey3N OCN dobbeltkameraoppsett

***

## Beste praksis

### Organiser før import

* Oppbevar kalibreringsmålbilder i samme mappe som undersøkelsesbilder
* Behold den opprinnelige mappestrukturen fra kameraet/SD-kortet
* Ikke bland datasett fra forskjellige økter i ett prosjekt

### Filnavngiving

* Behold de opprinnelige kamerafilnavnene (IMG\_0001.RAW osv.)
* Ikke gi filene nye navn før import
* Originale navn inneholder viktige metadata

### Kalibreringsmålbilder

* Inkluder alltid 1–2 kalibreringsmålbilder per økt
* Ta bilder av målene før og etter fotograferingsøkten
* Plasser målene i samme lysforhold som fotograferingsområdet
* Merk målbildene ved å merke av for Target (Mål) for å øke behandlingshastigheten

***

## Vanlige problemer og løsninger

### Bilder vises ikke etter import

**Mulige årsaker:**

* Filformatet støttes ikke (bare RAW+JPG og JPG fra MAPIR-kameraer)
* Bildene er fra kameraer som ikke er MAPIR (se [Støttede kameraer](../supported-cameras.md))
* Filkorrupsjon eller ufullstendig overføring fra SD-kort

**Løsning**: Kontroller filformatet og kameraets modellkompatibilitet

### Kameramodell oppdages ikke

**Mulige årsaker:**

* Modifiserte EXIF-metadata
* Bilder redigert i ekstern programvare
* Ufullstendig filoverføring

**Løsning**: Importer originale, umodifiserte filer fra kamera/SD-kort på nytt

### Manglende tidsstempler

**Mulige årsaker:**

* Kameraets klokke er ikke riktig innstilt
* EXIF-data fjernet av ekstern programvare

**Løsning**: Kontroller at kameraets tidsinnstillinger var riktige under opptaket

***

## Neste trinn

Når filene er importert:

1. **Gå gjennom fillisten** – Kontroller at alle bildene er lastet inn riktig
2. **Kontroller kameramodeller** – Kontroller at kameraet er riktig gjenkjent
3. **Merk målbilder** – Se [Velge målbilder](choosing-target-images.md)
4. **Juster innstillinger** – Konfigurer behandlingsalternativer i [Prosjektinnstillinger](adjusting-project-settings.md)
5. **Start behandlingen** – Se [Starte behandlingen](starting-the-processing.md)

For detaljert informasjon om prosjektkonfigurasjon, se [Justere prosjektinnstillinger](adjusting-project-settings.md).
