# Kartmarkører

Fanen «Kart» plasserer bildene dine på et interaktivt 2D-kart basert på GPS-koordinatene deres. Den gir deg et geografisk overblikk over en opptakssesjon og er den raskeste måten – rett etter import – å fjerne bilder du ikke ønsker å behandle.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Slik åpner du fanen «Kart»

1. Åpne eller opprett et prosjekt i Chloros
2. Importer bilder som inneholder GPS-metadata
3. Klikk på **Kart**-<img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line">-fanen i venstre sidefelt
4. Kartet viser en markør på hvert bildes GPS-posisjon

{% hint style="info" %}
**GPS kreves**: Kun bilder med GPS-koordinater i EXIF-metadataene vises på kartet. Et bilde uten koordinater er fortsatt en del av prosjektet og behandles som vanlig – det har bare ingen markør.
{% endhint %}

***

## Justere bilder fra «Kart»-fanen**Kart**-<img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line">-fanen har de samme knappene for å legge til <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> og fjerne <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line"> filer som [**Filbrowser**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">-fanen. Den viser den samme prosjektfillisten, med geografiske kolonner:

| Kolonne        | Innhold                                                           |
| ------------- | ------------------------------------------------------------------ |
| **Navn**      | Filnavnet slik det ble lagret fra kameraet                             |
| **Breddegrad** | Desimalgrader, seks desimaler                                |
| **Lengdegrad** | Desimalgrader, seks desimaler                                |
| **Høyde**  | Meter, én desimal — `-` når bildet ikke inneholder høydeopplysninger |

{% hint style="info" %}
Klikk på en kolonneoverskrift for å sortere etter den; klikk igjen for å reversere rekkefølgen.
{% endhint %}

{% hint style="warning" %}
**Høyde er høyde over havet, ikke høyde over bakken.** Verdien hentes fra bildets EXIF-tag `GPSAltitude`, som refererer til gjennomsnittlig havnivå. Det er ikke flyhøyden over terrenget, og Chloros vil ikke beregne en bakkeprøveavstand ut fra denne verdien – over et felt som ligger 300 m over havnivå, registrerer en drone i 100 m AGL omtrent 400 m her. Bruk kolonnen til å oppdage avvik og bekrefte en jevn flyhøyde, ikke som en AGL-måling.
{% endhint %}

***

## Bildemarkører

Hvert bilde med GPS-data får en markør på sine koordinater.

### Visning av markører

* Markørene plasseres på de nøyaktige koordinatene som er registrert for hvert bilde
* Markører som ligger tett sammen kan visuelt overlappe hverandre når du zoomer ut – zoom inn for å skille dem fra hverandre
* Valgte og uthevede markører vises over de øvrige

### Forhåndsvisning ved musepek

* **Hold musepekeren** over en markør for å få frem et miniatyrbilde av det aktuelle bildet med filnavnet
* **Klikk**på en markør for å velge bildet og**feste** popup-vinduet – det forblir åpent til du klikker et annet sted. Mens et popup-vindu er festet, vil ikke markører du holder musepekeren over stjele det
* Dette er en rask måte å finne et bestemt bilde i en stor sesjon uten å forlate kartet

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption><p>Fanen «Kart» viser alle bilder med geotag i prosjektet</p></figcaption></figure>### Superzoom

{% hint style="success" %}
**SUPERZOOM**: Når du når den maksimale zoomnivået som fliseleverandøren har bilder for, vil videre innzooming forstørre flisene i stedet for å stoppe, slik at du kan skille mellom markører som ligger nesten oppå hverandre.
{% endhint %}

* Superzoom aktiveres kun når du er **ved** leverandørens maksimale zoomnivå for den aktuelle lokasjonen og flisene er ferdig lastet inn. Under dette nivået fungerer zoomen som vanlig
* Området er **1× til 32×** utover leverandørens eget maksimum
* En indikator i hjørnet viser gjeldende superzoom som en prosentandel, og en **×**-knapp ved siden av den tar deg tilbake til normal zoom med ett klikk
* Når du zoomer ut, går du alltid tilbake til selve kartet, slik at du aldri kan bli sittende fast i superzoom
* Zooming og panorering mens du er i superzoom overfører den resulterende forskyvningen tilbake til kartet, slik at det området utenfor sentrum du har flyttet til fortsetter å be om fliser i stedet for å bli tomt
* Markører tegnes som vektorelementer i stedet for å bli rasterisert, slik at de forblir skarpe på alle superzoom-nivåer

***

## Kartfliseleverandører

{% hint style="success" %}
**Automatisk valg**: Chloros velger den flisetjenesten som tilbyr det beste zoomnivået for der bildene dine befinner seg. Du kan bytte manuelt når som helst.
{% endhint %}

| Leverandør        | Merknader                                                                                                                                                             |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Google Maps** | Bred verdensomspennende dekning; støtter alle fire flisetyper                                                                                                            |
| **Esri ArcGIS**| Ofte luftbilder med høyere oppløsning i bestemte regioner. Flisetypen**Terrain** tilbys ikke for Esri, og knappen for denne er deaktivert når Esri er valgt |

***

## Kartflisetyper

Velg kartlagstype med knappene (fra venstre til høyre):

![](<../.gitbook/assets/image (14).png>)

| Type                 | Viser                                                                |
| -------------------- | -------------------------------------------------------------------- |
| **Terreng**          | Høydeavskygging med kartdetaljer (veier, etiketter). Kun Google       |
| **Kart**              | Standard gatekartfliser — alternativet med lavest båndbredde              |
| **Satellitt**        | Detaljerte satellittbilder, uten etiketter — alternativet med høyest båndbredde |
| **Hybrid** (standard) | Satellittbilder med veier og etiketter lagt over                |

Fanen «Kart» åpnes på **Hybrid**. Valget ditt overføres til leverandørbytte der leverandøren støtter dette.

***

## Kartnavigering

* **Zoom**: musens rullehjul eller zoomknappene på kartet
* **Panorering**: klikk og dra
* **Fullskjerm**: fullskjermkontrollen utvider kartet til hele vinduet***

## Bruksområder

### Gjennomgang av flyrute

* Se dekningsområdet for en droneøkt på et øyeblikk
* Oppdag hull der en passering ble utelatt
* Bekreft at flyvningen fulgte det planlagte mønsteret

### Gjennomgang av bakkemåling

* Se hvordan bakkebaserte opptak er fordelt
* Finn kalibreringsmålrammer i forhold til måleområdet
* Bestem hvor det er behov for flere opptak

### Kvalitetskontroll

* Finn bilder som er tatt på uventede steder, og fjern dem før behandling
* Sorter etter høyde for å oppdage bilder som er tatt i feil høyde, eller der GPS-posisjonen var dårlig
* Sammenlign bildeplasseringer med feltnotater

***

## Feilsøking

### Ingen markører vises

**Mulige årsaker**

* Bildene inneholder ingen GPS-metadata
* GPS var deaktivert på kameraet under opptaket
* EXIF-data ble fjernet av annen programvare før import

**Hva du bør gjøre**: Kontroller at GPS er aktivert på kameraet, og importer originalfilene på nytt. Du kan sjekke om en bestemt fil har koordinater ved å lete etter den i filtabellen på «Kart»-fanen – et bilde uten koordinater har ingen rad der.

### Markørene er på feil sted

**Mulige årsaker**: dårlig satellittposisjonering ved opptakstidspunktet, eller GPS-avvik under opptaket.**Hva du bør gjøre**: dette er et problem knyttet til opptakstidspunktet, ikke noe Chloros kan korrigere i etterkant. For presisjonsarbeid bør du bruke en PPK/RTK-GPS-arbeidsflyt – se innstillingen**Bruk PPK-korreksjoner** i [Prosjektinnstillinger](../project-settings/project-settings.md).

### Kartet er tomt eller flisene slutter å lastes inn

Fliseleverandørene er nettbaserte tjenester. Hvis flisene slutter å komme inn, må du sjekke maskinens nettverkstilkobling og deretter prøve å bytte leverandør. Hvis du hadde zoomet veldig langt inn, trykker du på **×**-tilbakestillingsknappen for å gå tilbake til et normalt zoomnivå og la kartet be om flisene på nytt.

***

## Relaterte sider

* [**Bildegitter**](image-grid.md) — det samme bildesettet som miniatyrbilder
* [**Åpne et bilde i fullskjerm**](opening-an-image-full-screen.md) — se nærmere på ett bilde
* [**Legge til filer i et prosjekt**](../processing-images-gui/adding-files-to-a-project.md) — knappene for å legge til/fjerne filer som denne fanen deler
