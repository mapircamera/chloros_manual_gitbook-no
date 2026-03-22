# Kartmarkører

Fanen «Kart» viser bildene dine på et interaktivt 2D-kart basert på deres GPS-koordinater. Dette gir deg en geografisk oversikt over opptakssesjonen din og hjelper deg med å visualisere det geografiske dekningsområdet. Det er også nyttig når du først importerer bildene dine, slik at du raskt kan fjerne bilder du ikke trenger å behandle.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Åpne fanen Kart

1. Åpne eller opprett et prosjekt i Chloros
2. Importer bilder som inneholder GPS-metadata
3. Klikk på **Kart** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> i venstre sidefelt
4. Kartet viser markører på hvert bildes GPS-posisjon

{% hint style="info" %}
**GPS kreves**: Bare bilder med innebygde GPS-koordinater i EXIF-metadataene vises på kartet. Sørg for at GPS er aktivert på kameraet ditt under opptak.
{% endhint %}

***

## Justere bilder fra kartfanen**Kart**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> har de samme funksjonene for å legge til  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  og fjern  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  -knapper som [**Filbrowser**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Den viser også den samme prosjektfil-tabellisten, men med andre kolonneoverskrifter:

### Filnavn

* Opprinnelig filnavn fra kameraet
* Opprettholder kameraets navnekonvensjon (f.eks. IMG\_0001.RAW)

### Breddegrad

* Bildets breddegrad

### Lengdegrad

* Bildets lengdegrad

### Høyde

* Bildets høyde

{% hint style="info" %}
Ved å klikke på kolonneoverskriftene i tabellen sorteres også raddataene
{% endhint %}

***

## Bildemarkører

Hvert bilde med GPS-data vises som en markør på kartet:

### Visning av markører

* Markører angir de nøyaktige GPS-koordinatene der hvert bilde ble tatt
* Markører som ligger tett sammen kan grupperes når du zoomer ut
* Zoom inn for å se de enkelte bildeplasseringene

{% hint style="success" %}
SUPER-ZOOM: Når du når det maksimale zoomnivået fra kartfliseleverandøren, forstørres flisen ved ytterligere zoom, slik at du kan se markører som ligger tett sammen.
{% endhint %}

### Forhåndsvisning ved musepek

* **Hold musen** over en markør for å se en miniatyrforhåndsvisning av det bildet
* Dette gir rask visuell identifisering uten å forlate kartvisningen
* Nyttig for å finne bestemte bilder i en stor opptakssesjon

***

## Kartfliseleverandører

{% hint style="success" %}
**Automatisk valg**: Chloros velger automatisk den flisetjenesten som gir det beste zoomnivået for din nåværende kartposisjon. Du kan bytte mellom leverandører manuelt hvis ønskelig.
{% endhint %}

Kart-fanen støtter to fliseleverandører for bakgrunnskartbildene:

### Google Maps

* Standard satellitt- og kartbilder fra Google
* Best for generell verdensomspennende dekning

### ESRI

* Satellitt- og luftbilder fra ESRI ArcGIS
* Tilbyr ofte bilder med høyere oppløsning i visse regioner

***

## Kartflisetyper

Du kan velge kartlagstype (fra venstre til høyre):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="original">### Terreng

Viser høydeprofiler og kartfliser med detaljer (veier osv.)

### Kart

Viser standard (lavere båndbredde) kartfliser med detaljer (veier osv.)

### Satellitt

Viser detaljerte (høyere båndbredde) satellittkartfliser

### Hybrid

Viser satellittkartfliser med tilleggsdetaljer (veier osv.)

***

## Kartnavigering

### Zoomkontroller

* **Zoom inn/ut**: Bruk musens rullehjul eller zoomknappene
* **Fullskjerm**: Vis kartet i fullskjerm

### Panorering

* **Panorering**: Klikk og dra for å bevege deg rundt på kartet***

## Bruksområder

### Visualisering av flyrute

* Se dekningsområdet for droneopptak
* Identifiser hull i bildedekningen
* Verifiser gjennomføringen av flyruten

### Gjennomgang av bakkemåling

* Se den romlige fordelingen av bakkebaserte opptak
* Finn kalibreringsmålbilder i forhold til måleområdet
* Planlegg flere opptakssteder

### Kvalitetskontroll

* Identifiser raskt bilder som er tatt på uventede steder
* Kontroller GPS-nøyaktigheten i hele datasettet
* Kryssreferer bildeplasseringer med feltnotater

***

## Feilsøking

### Ingen markører vises

**Mulige årsaker:**

* Bildene inneholder ikke GPS-metadata
* GPS var deaktivert på kameraet under opptaket
* EXIF-data ble fjernet av ekstern programvare

**Løsning**: Kontroller at GPS er aktivert på kameraet ditt, og importer originalfilene på nytt

### Markører på feil sted

**Mulige årsaker:**

* Kameraets GPS hadde dårlig satellittposisjon
* GPS-avvik under opptaket

**Løsning**: Dette er vanligvis et problem knyttet til opptakstidspunktet; vurder å bruke PPK/RTK GPS for presisjonsapplikasjoner
