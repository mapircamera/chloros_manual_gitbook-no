# Justere prosjektinnstillinger

Før du behandler bildene dine, er det viktig å konfigurere prosjektinnstillingene slik at de samsvarer med kravene i arbeidsflyten din. <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> gir deg full kontroll over kalibrering, behandlingsalternativer, multispektrale indekser og eksportformater.

## Åpne prosjektinnstillinger

1. Åpne prosjektet ditt i Chloros
2. Klikk på **Prosjektinnstillinger** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> i venstre sidefelt
3. Panelet Prosjektinnstillinger viser alle konfigurasjonsalternativer

{% hint style="info" %}
**Innstillingene lagres automatisk** sammen med prosjektet. Når du åpner et prosjekt på nytt, gjenopprettes alle innstillingene.
{% endhint %}

***

## Hurtigoppsett for vanlige arbeidsflyter

### Standardinnstillinger (anbefales for de fleste brukere)

For typiske MAPIR Survey3 kameraprosesser fungerer standardinnstillingene godt:

* ✅ **Vignettkorrigering**: Aktivert
* ✅ **Refleksjonskalibrering**: Aktivert (krever bilder av MAPIR-mål)
* ✅ **Debayer-metode**: Standard (Rask, Middels kvalitet)
* ✅ **Eksportformat**: TIFF (16-bit)

Bare importer bildene dine og start behandlingen med disse standardinnstillingene.

***

## Oversikt over prosjektinnstillinger

Panelet for prosjektinnstillinger er organisert i flere kategorier. Nedenfor finner du en oppsummering av hver seksjon. For fullstendig dokumentasjon, se [Prosjektinnstillinger](../project-settings/project-settings.md).

### Målgjenkjenning

Styrer hvordan Chloros identifiserer kalibreringsmål i bildene dine.

**Viktige innstillinger:*** **Minimumsareal for kalibreringsprøve**: Størrelsestærskel for måldeteksjon (standard: 25 piksler)
* **Minimumsmålgruppering**: Likhetstærskel for gruppering av målområder (standard: 60)**Når du bør justere:**

* Øk prøvearealet hvis du får falske deteksjoner
* Reduser hvis mål ikke blir oppdaget
* Juster gruppering hvis mål blir delt opp i flere oppdagelser

### Behandling

Hovedalternativer for bildebehandling og kalibrering.

**Viktige innstillinger:*** **Vignettkorrigering**: Kompenserer for mørkere kanter på bildet ✅ Anbefalt
* **Refleksjonskalibrering**: Normaliserer verdier ved hjelp av kalibreringsmål ✅ Anbefalt
* **Debayer-metode**: Algoritme for konvertering av RAW til 3-kanals multispektral
* **Minimalt rekalibreringsintervall**: Tid mellom bruk av kalibreringsmål (0 = bruk alle)**Avanserte innstillinger:*** **Tidssoneforskyvning for lyssensor**: For PPK-tidssynkronisering (standard: 0)
* **Bruk PPK-korreksjoner**: Bruker GPS-/eksponeringspin-data fra .daq-filer
* **Eksponeringspin 1/2**: Tilordner kameraer til eksponeringspinner for oppsett med to kameraer

### Debayer-metode

Vi tilbyr for øyeblikket to debayering-metoder i Chloros:

#### Standard (rask, middels kvalitet)

Standard debayer behandler raskt, men viser farestøy fra debayering, noe som resulterer i mindre nøyaktige og mer støyende bilder.

#### Texture Aware (Langsom, høyeste kvalitet) \[Kun Chloros+]

Texture Aware bruker en kantbevisst debayer av høy kvalitet kombinert med en AI/ML-støyreduksjonsmodell som fjerner nesten all støy fra debayering. Texture Aware-modellen krever GPU-minne (VRAM) for å kjøre. Vi anbefaler å bruke den når du har &gt;4 GB VRAM tilgjengelig for raskere behandling.

### Indeks (multispektrale indekser)

Konfigurer hvilke vegetasjonsindekser som skal beregnes og eksporteres.

**Slik legger du til indekser:**

1. Klikk på**&quot;Legg til indeks&quot;**-knappen
2. Velg en indeks fra rullegardinmenyen (NDVI, NDRE, GNDVI, osv.)
3. Konfigurer visualiseringsinnstillinger (LUT-farger, verdier)
4. Legg til flere indekser etter behov

**Populære indekser:*** **NDVI**: Generell vegetasjonshelse (vanligst)
* **NDRE**: Tidlig stressdeteksjon sammen med RedEdge
* **GNDVI**: Følsom for klorofyllkonsentrasjon
* **OSAVI**: Fungerer godt med synlig jord
* **EVI**: Områder med høy bladarealindeks (LAI)**Egendefinerte formler (kun Chloros+):**

* Opprett tilpassede multispektrale indeksformler
* Bruk båndmatematikk med alle bildekanaler
* Lagre tilpassede formler for gjenbruk

For alle tilgjengelige indekser og formler, se [Multispektrale indeksformler](../project-settings/multispectral-index-formulas.md).

### Eksport

Styrer utdatafilformat og kvalitet.

**Tilgjengelige formater:*** **TIFF (16-bit)**: Anbefales for GIS og vitenskapelig analyse (0–65 535)
* **TIFF (32-bit, prosent)**: Refleksjonsverdier med flytende desimaler (0,0–1,0)
* **PNG (8-bit)**: Tapsfri komprimering for visualisering (0–255)
* **JPG (8-bit)**: Minste filer, komprimering med tap (0–255)***

## Lagre og laste inn innstillinger

### Lagre prosjektmal

Opprett gjenbrukbare maler for konsistente arbeidsflyter:

1. Konfigurer alle ønskede innstillinger i panelet Prosjektinnstillinger
2. Bla ned til seksjonen **«Lagre prosjektmal»** nederst
3. Skriv inn et beskrivende navn på malen (f.eks. «Survey3N\_RGN\_Agriculture»)
4. Klikk på lagringsikonet

**Fordeler:**

* Bruk identiske innstillinger på tvers av flere prosjekter
* Del konfigurasjoner med teammedlemmer
* Oppretthold konsistens for gjentatte undersøkelser

### Last inn mal på nytt prosjekt

Når du oppretter et nytt prosjekt:

1. Velg **«Nytt prosjekt»** fra hovedmenyen
2. Velg alternativet **«Last inn fra mal»**

3. Velg den lagrede malen din
4. Alle innstillinger blir automatisk brukt

### Arbeidskatalog

Innstillingen **&quot;Lagre prosjektmappe&quot;** angir hvor nye prosjekter opprettes som standard:

* **Standardplassering**: `C:\Users\[Username]\Chloros Projects`
* **Endre plassering**: Klikk på redigeringsikonet og velg ny mappe
* **Når du bør endre**:
  * Nettverksstasjon for teamsamarbeid
  * Annen stasjon med mer lagringsplass
  * Organisert mappestruktur etter år/kunde

***

## PPK-oppsett (Post-Processed Kinematic)

Hvis du bruker MAPIR DAQ-opptakere med GPS for presis geolokalisering:

### Forutsetninger

* MAPIR DAQ med GPS (GNSS)-modul
* .daq-loggfil med oppføringer for eksponeringspinner
* Kamera koblet til DAQ-eksponeringspinner under opptakssesjonen

### Konfigurasjonstrinn

1. Plasser .daq-loggfilen i prosjektmappen din
2. I prosjektinnstillingene aktiverer du avkrysningsruten **«Apply PPK corrections»**

3. Angi**«Light sensor timezone offset»** om nødvendig (standard: 0 for UTC)
4. Tilordne kameraer til eksponeringspinner:
   * **Enkelt kamera**: Tilordnes automatisk til pinne 1
   * **To kameraer**: Tilordne hvert kamera manuelt til riktig pinne**Tilordning av eksponeringspinner:*** **Eksponeringspinne 1**: Velg kameramodell fra rullegardinmenyen
* **Eksponeringspinne 2**: Velg det andre kameraet eller «Ikke bruk»
* Samme kamera kan ikke tilordnes begge pinnene

{% hint style="warning" %}
**Viktig**: Eksponeringspinnene må være riktig tilordnet sine respektive kameraer. Feil tilordning vil føre til feil geolokaliseringsdata.
{% endhint %}

***

## Avanserte scenarier

### Prosjekter med flere kameraer

Når du behandler bilder fra flere MAPIR-kameraer i ett prosjekt:

1. Chloros oppdager automatisk hver kameramodell
2. Hvert kamera får riktig behandlingsprofil
3. PPK: Tilordne hvert kamera manuelt til riktig eksponeringspinne
4. Alle kameraer bruker samme eksportformat og indekser

**Eksempel**: Survey3W RGN + Survey3N OCN rigg med to kameraer

### Tidsforløp eller undersøkelser over flere datoer

For gjentatte undersøkelser av samme område over tid:

1. Opprett en mal med standardinnstillingene dine
2. Bruk konsistent oppsett av kalibreringsmål for hver økt
3. Behandle hver dato som et eget prosjekt
4. Bruk identiske innstillinger for sammenlignbare resultater
5. Eksporter i samme format for tidsanalyse

### Store datasett

For prosjekter med mange bilder (500+):

* Vurder å dele opp i mindre prosjekter etter dato eller område
* Bruk Chloros+ parallellbehandling for raskere resultater
* Vurder CLI eller API for batchautomatisering
* Juster minimum intervall for rekalibrering for å redusere måldeteksjonstiden

***

## Verifisere innstillingene dine

Før du starter behandlingen, må du gå gjennom disse viktige innstillingene:

* [ ] Kameramodell oppdaget riktig i filbrowseren
* [ ] Vignettkorreksjon aktivert
* [ ] Refleksjonskalibrering aktivert
* [ ] Minst ett kalibreringsmålbilde importert
* [ ] Ønskede multispektrale indekser lagt til
* [ ] Eksportformat som passer for arbeidsflyten din
* [ ] PPK-innstillinger konfigurert (hvis du bruker .daq med eksponeringshendelser)

***

## Neste trinn

Når innstillingene er konfigurert:

1. **Merk kalibreringsmålbilder** – Se [Velge målbilder](choosing-target-images.md)
2. **Start behandlingen** – Se [Starte behandlingen](starting-the-processing.md)
3. **Overvåk fremdriften** – Se [Overvåke behandlingen](monitoring-the-processing.md)

For fullstendige detaljer om alle tilgjengelige innstillinger, se referansedokumentasjonen [Prosjektinnstillinger](../project-settings/project-settings.md).
