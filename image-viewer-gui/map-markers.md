# Kartmarkører

Fanen Kart viser bildene dine på et interaktivt 2D-kart basert på GPS-koordinatene deres. Dette gir en geografisk oversikt over opptakssesjonen din og hjelper deg med å visualisere den romlige dekningen. Det er også nyttig når du først importerer bildene dine, for å raskt fjerne bilder du ikke trenger å behandle.

## Åpne fanen Kart

1. Åpne eller opprett et prosjekt i Chloros.
2. Importer bilder som inneholder GPS-metadata.
3. Klikk på fanen **Kart** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> i venstre sidefelt.
4. Kartet viser markører på GPS-posisjonen til hvert bilde.

{% hint style=&quot;info&quot; %}
**GPS påkrevd**: Bare bilder med innebygde GPS-koordinater i EXIF-metadataene vil vises på kartet. Sørg for at kameraet har GPS aktivert under opptak.
{% endhint %}

***

## Justere bilder fra kartfanen

Fanen **Kart**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> har samme funksjoner for å legge til  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  og fjern  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  filknappene som [**Filbrowser**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Den viser også den samme prosjektfiltabellen, men med andre kolonneoverskrifter:

### Filnavn

* Opprinnelig filnavn fra kameraet
* Opprettholder kameraets navngivningskonvensjon (f.eks. IMG\_0001.RAW)

### Breddegrad

* Bildets breddegrad

### Lengdegrad

* Bildets lengdegrad

### Høyde

* Bildets høyde

{% hint style=&quot;info&quot; %}
Ved å klikke på kolonneoverskriftene i tabellen sorteres også raddataene.
{% endhint %}

***

## Bildemarkører

Hvert bilde med GPS-data vises med en markør på kartet:

### Markørvisning

* Markørene angir de nøyaktige GPS-koordinatene der hvert bilde ble tatt.
* Markører kan grupperes når du zoomer ut.
* Zoom inn for å se de enkelte bildenes plasseringer.

{% hint style=&quot;success&quot; %}
SUPER-ZOOM: Når du når det maksimale zoomnivået fra kartfliseleverandøren, forstørres flisen ved ytterligere zoom, slik at du kan se markører som ligger tett sammen.
{% endhint %}

### Forhåndsvisning ved å holde musepekeren over

* **Hold musen** over en markør for å se en miniatyrforhåndsvisning av det bildet.
* Dette gjør det mulig å identifisere bildet raskt uten å forlate kartvisningen.
* Nyttig for å finne bestemte bilder i en stor opptakssesjon.

***

## Kartfliseleverandører

{% hint style=&quot;success&quot; %}
**Automatisk valg**: Chloros velger automatisk den fliseleverandøren som gir det beste zoomnivået for din nåværende kartposisjon. Du kan manuelt bytte mellom leverandører hvis du ønsker det.
{% endhint %}

Kartfanen støtter to fliseleverandører for bakgrunnskartbildene:

### Google Maps

* Standard satellitt- og kartbilder fra Google
* Best for generell dekning over hele verden

### ESRI

* Satellitt- og luftbilder fra ESRI ArcGIS
* Gir ofte bilder med høyere oppløsning i visse regioner

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

### Panoreringkontroller

* **Panorering**: Klikk og dra for å bevege deg rundt på kartet***

## Bruksområder

### Visualisering av flyrute

* Se dekningsområdet for droneopptakssesjoner
* Identifiser hull i bildedekningen
* Kontroller gjennomføringen av flyruten

### Gjennomgang av bakkeundersøkelse

* Se den romlige fordelingen av bakkebaserte opptak
* Finn kalibreringsmålbilder i forhold til undersøkelsesområdet
* Planlegg flere opptakssteder

### Kvalitetskontroll

* Identifiser raskt bilder som er tatt på uventede steder.
* Kontroller GPS-nøyaktigheten i hele datasettet.
* Kryssreferer bildesteder med feltnotater.

***

## Feilsøking

### Ingen markører vises

**Mulige årsaker:**

* Bildene inneholder ikke GPS-metadata.
* GPS var deaktivert på kameraet under opptaket.
* EXIF-data ble fjernet av ekstern programvare.

**Løsning**: Kontroller at GPS er aktivert på kameraet og importer originalfilene på nytt

### Markører på feil sted

**Mulige årsaker:**

* Kameraets GPS hadde dårlig satellittposisjonering
* GPS-avvik under opptak

**Løsning**: Dette er vanligvis et problem knyttet til opptakstidspunktet. Vurder å bruke PPK/RTK GPS for presisjonsapplikasjoner
