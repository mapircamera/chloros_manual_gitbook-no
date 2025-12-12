# Justere prosjektinnstillinger

Før du behandler bildene dine, er det viktig å konfigurere prosjektinnstillingene slik at de samsvarer med kravene til arbeidsflyten din. Prosjektinnstillingspanelet <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> gir omfattende kontroll over kalibrering, behandlingsalternativer, multispektrale indekser og eksportformater.

## Tilgang til prosjektinnstillinger

1. Åpne prosjektet i Chloros
2. Klikk på **Prosjektinnstillinger** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> i venstre sidefelt
3. Panelet Prosjektinnstillinger viser alle konfigurasjonsalternativer

{% hint style=&quot;info&quot; %}
**Innstillingene lagres automatisk** sammen med prosjektet. Når du åpner et prosjekt på nytt, gjenopprettes alle innstillingene.
{% endhint %}

***

## Rask oppsett for vanlige arbeidsflyter

### Standardinnstillinger (anbefales for de fleste brukere)

For typiske MAPIR Survey3 kameraarbeidsflyter fungerer standardinnstillingene godt:

* ✅ **Vignettkorreksjon**: Aktivert
* ✅ **Refleksjonskalibrering**: Aktivert (krever bilder av MAPIR-mål)
* ✅ **Debayer-metode**: Høy kvalitet (raskere)
* ✅ **Eksportformat**: TIFF (16-bit)

Bare importer bildene dine og start behandlingen med disse standardinnstillingene.

***

## Oversikt over prosjektinnstillinger

Panelet Prosjektinnstillinger er organisert i flere kategorier. Nedenfor finner du en oversikt over hver seksjon. For fullstendig dokumentasjon, se [Prosjektinnstillinger](../project-settings/project-settings.md).

### Målregistrering

Kontrollerer hvordan Chloros identifiserer kalibreringsmål i bildene dine.

**Viktige innstillinger:**

* **Minimum kalibreringsprøveområde**: Størrelsestærskel for måldeteksjon (standard: 25 piksler)
* **Minimum målklynging**: Likhetstærskel for gruppering av målregioner (standard: 60)

**Når du bør justere:**

* Øk prøveområdet hvis du får falske deteksjoner.
* Reduser hvis målene ikke blir oppdaget.
* Juster klynging hvis målene blir delt inn i flere deteksjoner.

### Behandling

Hovedalternativer for bildebehandling og kalibrering.

**Viktige innstillinger:**

* **Vignettkorreksjon**: Kompenserer for mørkere kanter på objektivet ✅ Anbefalt
* **Refleksjonskalibrering**: Normaliserer verdier ved hjelp av kalibreringsmål ✅ Anbefalt
* **Debayer-metode**: Algoritme for konvertering av RAW til 3-kanals multispektral
* **Minimum rekalibreringsintervall**: Tid mellom bruk av kalibreringsmål (0 = bruk alle)

**Avanserte innstillinger:**

* **Lyssensorens tidssoneforskyvning**: For PPK-tidssynkronisering (standard: 0)
* **Bruk PPK-korreksjoner**: Bruker GPS/eksponeringspinndata fra .daq-filer
* **Eksponeringspinne 1/2**: Tilordner kameraer til eksponeringspinner for oppsett med to kameraer

### Indeks (multispektrale indekser)

Konfigurer hvilke vegetasjonsindekser som skal beregnes og eksporteres.

**Slik legger du til indekser:**

1. Klikk på **«Legg til indeks»**-knappen
2. Velg en indeks fra rullegardinmenyen (NDVI, NDRE, GNDVI osv.)
3. Konfigurer visualiseringsinnstillinger (LUT-farger, verdier)
4. Legg til flere indekser etter behov

**Populære indekser:**

* **NDVI**: Generell vegetasjonshelse (vanligst)
* **NDRE**: Tidlig stressdeteksjon med RedEdge
* **GNDVI**: Klorofyllkonsentrasjonsfølsom
* **OSAVI**: Fungerer godt med synlig jord
* **EVI**: Regioner med høy bladarealindeks (LAI)

**Tilpassede formler (kun Chloros+):**

* Lag tilpassede multispektrale indeksformler
* Bruk båndmatematikk med alle bildekanaler
* Lagre tilpassede formler for gjenbruk

For alle tilgjengelige indekser og formler, se [Multispektrale indeksformler](../project-settings/multispectral-index-formulas.md).

### Eksport

Kontrollerer utdatafilformat og kvalitet.

**Tilgjengelige formater:**

* **TIFF (16-bit)**: Anbefales for GIS og vitenskapelig analyse (0-65 535-området)
* **TIFF (32-bit, prosent)**: Flytende refleksjonsverdier (0,0-1,0-området)
* **PNG (8-bit)**: Tapsfri komprimering for visualisering (område 0–255)
* **JPG (8-bit)**: Minste filer, tapsrik komprimering (område 0–255)

***

## Lagre og laste inn innstillinger

### Lagre prosjektmal

Opprett gjenbrukbare maler for konsistente arbeidsflyter:

1. Konfigurer alle ønskede innstillinger i panelet Prosjektinnstillinger.
2. Bla til delen **«Lagre prosjektmal»** nederst.
3. Skriv inn et beskrivende navn på malen (f.eks. «Survey3N\_RGN\_Agriculture»).
4. Klikk på lagringsikonet.

**Fordeler:**

* Bruk identiske innstillinger på flere prosjekter
* Del konfigurasjoner med teammedlemmer
* Oppretthold konsistens for gjentatte undersøkelser

### Last inn mal på nytt prosjekt

Når du oppretter et nytt prosjekt:

1. Velg **«Nytt prosjekt»** fra hovedmenyen
2. Velg alternativet **«Last inn fra mal»**
3. Velg den lagrede malen
4. Alle innstillinger blir automatisk brukt

### Arbeidskatalog

Innstillingen **«Lagre prosjektmappe»** angir hvor nye prosjekter opprettes som standard:

* **Standardplassering**: `C:\Users\[Username]\Chloros Projects`
* **Endre plassering**: Klikk på redigeringsikonet og velg ny mappe
* **Når du bør endre**:
  * Nettverksstasjon for teamsamarbeid
  * Annen stasjon med mer lagringsplass
  * Organisert mappestruktur etter år/kunde

***

## PPK (Post-Processed Kinematic) oppsett

Hvis du bruker MAPIR DAQ-opptakere med GPS for presis geolokalisering:

### Forutsetninger

* MAPIR DAQ med GPS (GNSS)-modul
* .daq-loggfil med eksponeringspinneoppføringer
* Kamera koblet til DAQ-eksponeringspinner under opptakssesjonen

### Konfigurasjonstrinn

1. Plasser .daq-loggfilen i prosjektmappen din.
2. I prosjektinnstillingene aktiverer du avmerkingsboksen **«Bruk PPK-korreksjoner»**.
3. Angi **«Lyssensorens tidssoneforskjell»** om nødvendig (standard: 0 for UTC).
4. Tilordne kameraer til eksponeringspinner:
   * **Enkelt kamera**: Tildeles automatisk til pin 1
   * **To kameraer**: Tildel hvert kamera manuelt til riktig pin

**Tildeling av eksponeringspinner:**

* **Eksponeringspin 1**: Velg kameramodell fra rullegardinmenyen
* **Eksponeringspin 2**: Velg andre kamera eller «Ikke bruk»
* Samme kamera kan ikke tildeles begge pinnene

{% hint style=&quot;warning&quot; %}
**Viktig**: Eksponeringspinner må tilordnes riktig til sine respektive kameraer. Feil tilordning vil resultere i feil geolokaliseringsdata.
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

1. Lag en mal med standardinnstillingene dine
2. Bruk konsistent kalibreringsmåloppsett for hver økt
3. Behandle hver dato som et eget prosjekt
4. Bruk identiske innstillinger for sammenlignbare resultater
5. Eksporter i samme format for tidsanalyse

### Store datasett

For prosjekter med mange bilder (500+):

* Vurder å dele opp i mindre prosjekter etter dato eller område.
* Bruk Chloros+ parallellbehandling for raskere resultater.
* Vurder CLI eller API for batchautomatisering.
* Juster minimumskalibreringsintervall for å redusere måldeteksjonstiden.

***

## Kontrollere innstillingene

Før du starter behandlingen, må du gå gjennom disse viktige innstillingene:

* [ ] Kameramodell riktig oppdaget i filbrowseren
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
