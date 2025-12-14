# Bildelag

Med rullegardinmenyen Bildelag i Chloros Image Viewer kan du raskt bytte mellom forskjellige versjoner av det samme bildet – fra originale opptak til bearbeidede refleksjonsutdata og beregnede indeksbilder.

## Hva er bildelag?

I Chloros refererer **lag** til de forskjellige bildeutdataene som er tilgjengelige for ett enkelt kildebilde. Når du behandler bilder, oppretter Chloros flere versjoner:

* **Originalbilder** (JPG- og RAW-filer fra kameraet ditt)
* **Refleksjonskalibrerte** utdata (hvis refleksjonskalibrering var aktivert)
* **Målbilder** (hvis bildet inneholder kalibreringsmål)
* **Indeksbilder** (NDVI, NDRE, GNDVI osv. hvis indekser ble konfigurert)

**Lagvelger-rullegardinmenyen** øverst til høyre i bildeviseren lar deg umiddelbart bytte mellom disse versjonene uten å forlate viseren.

***

## Tilgjengelige lagtyper

### JPG

* Det originale JPG-forhåndsvisningsbildet fra kameraet ditt
* Alltid tilgjengelig for alle bilder
* Ubehandlet, slik det ble tatt av kameraet
* Raskest å laste inn og vise

**Når du skal vise:**

* Rask forhåndsvisning av originalopptaket
* Kontrollere bildesammensetning og innramming
* Kontrollere opptakskvaliteten før behandling

### RAW (original)

* De originale RAW-sensordataene fra kameraet ditt
* Debayered uten etterbehandling
* Høyere bitdybde enn JPG (vanligvis 12-bit eller 14-bit sensordata)

**Når du skal vise:**

* Inspisere kvaliteten på de originale sensordataene
* Sjekke for sensorproblemer eller artefakter
* Sammenligne resultater før og etter behandling

### RAW (mål)

* Vises bare for bilder som er identifisert som inneholder kalibreringsmål
* Viser det originale RAW-bildet med oppdaget mål
* Brukes til å verifisere at måldeteksjonen var vellykket

**Når skal du vise:**

* Bekrefte at kalibreringsmålene ble oppdaget riktig
* Sjekke målbildets kvalitet
* Feilsøke kalibreringsproblemer

{% hint style=&quot;info&quot; %}
**Mållag**: Dette laget vises bare i rullegardinmenyen for bilder som inneholder kalibreringsmål. Vanlige bilder har ikke dette alternativet.
{% endhint %}

### RAW (reflektans)

* Det kalibrerte reflektansutdata-bildet
* Vignettkorrigert (hvis aktivert i behandlingen)
* Reflektans kalibrert ved hjelp av måldata (hvis aktivert)
* Multibånd TIFF med alle kamerakanaler
* Pikselverdier representerer prosentvis reflektans (når du bruker prosentmodus)
* Klar til å manipuleres med [Index/LUT Sandbox](index-lut-sandbox.md)

**Når skal du vise:**

* Inspisere kalibrerte resultater
* Verifisere kalibreringskvalitet
* Sjekke pikselverdier for vitenskapelig nøyaktighet
* Sammenligne med originalen for å se kalibreringseffekter

{% hint style=&quot;success&quot; %}
**Anbefalt**: Bruk RAW (Reflectance)-laget når du sjekker pikselverdier for vitenskapelige målinger og analyser.
{% endhint %}

### RAW (NDVI Index)... og lignende

* Beregnet vegetasjonsindeksbilde (NDVI i dette eksemplet)
* Indeksnavnet endres basert på hvilken indeks som ble konfigurert under behandlingen
* Eksempler: RAW (NDVI-indeks), RAW (NDRE-indeks), RAW (GNDVI-indeks) osv.
* Enkeltbånds gråtonebilde som viser indeksberegningsresultater
* Det vises ett lag for hver indeks som er konfigurert i prosjektinnstillingene

**Mulige indeksnavn:**

* RAW (NDVI-indeks)
* RAW (NDRE-indeks)
* RAW (GNDVI-indeks)
* RAW (OSAVI-indeks)
* RAW (EVI-indeks)
* RAW (SAVI-indeks)
* Og mange flere... (se [Multispektrale indeksformler](../project-settings/multispectral-index-formulas.md))

**Når skal du vise:**

* Undersøke indeksberegningsresultater
* Kontrollere indeksverdier
* Identifisere områder av interesse
* Verifisere indeksbilder før bruk i GIS eller analyse

***

## Bruke lagvelgeren

### Åpne rullegardinmenyen

1. Åpne et bilde i fullskjermmodus (klikk på en miniatyrbilde i bildeviseren)
2. Finn **lagrullgardinen** øverst til høyre i visningsprogrammet
3. Rullgardinen viser det valgte laget (f.eks. «JPG»)
4. Klikk på rullgardinen for å se alle tilgjengelige lag

### Bytte mellom lag

1. Klikk på lagrullgardinen for å åpne listen
2. Alle tilgjengelige lag for det aktuelle bildet vises
3. Klikk på et lagnavn for å bytte til den versjonen
4. Bildet oppdateres umiddelbart for å vise det valgte laget.

**Rask bytte:**

* Nedtrekksmenyen husker ditt siste valg.
* Når du navigerer til neste bilde, prøver Chloros å vise samme lagtype.
* Hvis det laget ikke finnes på neste bilde, blir JPG standard.

### Lagets tilgjengelighet

Ikke alle lag er tilgjengelige for hvert bilde:

**Alltid tilgjengelig:**

* ✅ JPG (hvert bilde har en JPG-forhåndsvisning)

**Betinget tilgjengelig:**

* ⚠️ RAW (Original) – Bare hvis bildet ble tatt i RAW- eller RAW+JPG-modus
* ⚠️ RAW (Mål) – Bare hvis bildet inneholder oppdagede kalibreringsmål
* ⚠️ RAW (Refleksjonsgrad) – Kun etter behandling med refleksjonskalibrering aktivert
* ⚠️ RAW (\[Indeks] Indeks) – Kun etter behandling med konfigurerte indekser

***

## Lagring av lag

### Navigering mellom bilder

Når du navigerer til et annet bilde (ved hjelp av piltastene eller ved å klikke på miniatyrbilder):

**Lagpreferansen beholdes:**

* Hvis du viser «RAW (Refleksjonsgrad)», viser neste bilde «RAW (Refleksjonsgrad)» (hvis tilgjengelig)
* Hvis du viser «RAW (NDVI Indeks)», viser neste bilde «RAW (NDVI Indeks)» (hvis tilgjengelig)
* Hvis det samme laget ikke finnes, brukes JPG som standard

**Eksempel på arbeidsflyt:**

1. Åpne bilde 1, bytt til RAW (NDVI Index)
2. Trykk på → for å vise bilde 2
3. Bilde 2 viser automatisk RAW (NDVI Index)-laget
4. Fortsett å navigere – alle bildene viser NDVI-laget
5. Veldig effektivt for å gjennomgå indeksresultater på tvers av mange bilder

***

## Vanlige arbeidsflyter

### Arbeidsflyt 1: Før/etter-sammenligning

**Mål**: Sammenligne originalbildet med det kalibrerte bildet

1. Åpne det behandlede bildet i Image Viewer
2. Velg **RAW (Original)** fra rullegardinmenyen
3. Legg merke til vignettering og ukalibrerte verdier
4. Bytt til **RAW (Reflektans)** fra rullegardinmenyen
5. Sammenlign – vignettering fjernet, verdier kalibrert

### Arbeidsflyt 2: Gjennomgang av indeks

**Mål**: Gjennomgå raskt NDVI-resultater på tvers av datasettet

1. Åpne det første behandlede bildet
2. Velg **RAW (NDVI-indeks)** fra rullegardinmenyen
3. Bruk → piltasten for å navigere til neste bilde
4. NDVI-laget beholdes automatisk
5. Fortsett gjennom alle bildene og sjekk NDVI-mønstrene
6. Bytt til **RAW (NDRE Index)** for å sammenligne

### Arbeidsflyt 3: Målverifisering

**Mål**: Verifisere at alle målbilder ble oppdaget riktig

1. Naviger til et målbilde
2. Velg **RAW (Target)** fra rullegardinmenyen
3. Verifiser at kalibreringsmålene er tydelig synlige og oppdaget
4. Naviger til neste målbilde
5. Gjenta verifiseringen for alle målene

### Arbeidsflyt 4: Inspeksjon av pikselverdier

**Mål**: Kontroller refleksjonsverdiene for vitenskapelig nøyaktighet

1. Åpne det bearbeidede bildet
2. Velg **RAW (Refleksjon)**-laget
3. Aktiver **Pikselprosent**-modus (knappen øverst til høyre i verktøylinjen)
4. Flytt markøren over vegetasjonsområdene
5. Kontroller at pikselverdiene ligger innenfor forventede områder (30–70 % for NIR, 5–15 % for Red)
6. Kontroller at jord- og vannområdene har riktige verdier

***

## Forstå pikselverdier etter lag

Ulike lag viser forskjellige pikselverdier:

### JPG-lag

* **Område**: 0–255 (8-bit)
* **Betydning**: Visningsverdier, gammakorrigert
* **Bruk**: Kun visuell inspeksjon, ikke for vitenskapelig måling

### RAW (original)

* **Område**: 0–65535 (16-bit)
* **Betydning**: Rå digitale sensortall
* **Bruk**: Kontroll av sensorens ytelse, ikke kalibrert

### RAW (refleksjonsgrad)

* **Område**: 0–65 535 (16-bit TIFF) eller 0,0–1,0 (32-bit prosent)
* **Betydning**: Kalibrert prosentvis refleksjonsgrad
* **Bruk**: Vitenskapelige målinger og analyser

**For 16-bit TIFF:** Del med 65 535 for å få prosentvis refleksjonsgrad **For 32-bit prosent:** Verdiene representerer direkte prosent (0,5 = 50 % refleksjonsgrad)

### RAW (indeksbilder)

* **Område**: Varierer etter indeks (vanligvis -1,0 til +1,0 for normaliserte indekser)
* **Betydning**: Resultat av indeksberegning
* **Eksempler**:
  * NDVI: -1 til +1 (vegetasjon vanligvis 0,4 til 0,9)
  * NDRE: -1 til +1 (stressdeteksjon)
  * EVI: 0 til 1 (forbedret vegetasjon)

***

## Tips og beste praksis

### Effektiv lagbytte

* **Korttastkunnskap**: Selv om det ikke finnes noen korttast for lag, fungerer navigasjonspilene (←/→) på tvers av alle lag.
* **Konsistente arbeidsflyter**: Velg ett lag (f.eks. NDVI) og gjennomgå hele datasettet før du bytter til et annet.
* **Raske sammenligninger**: Veksle mellom Original og Reflektans for å verifisere behandlingskvaliteten.

### Ytelseshensyn

* **JPG lastes raskest**: Brukes for rask navigering gjennom mange bilder.
* **RAW-lag lastes saktere**: Høyere oppløsning og bitdybde.
* **Indekslag**: Lignende hastighet som refleksjonslag.
* **Første lasting er tregest**: Senere visninger av samme lag blir lagret i hurtigbufferen og er raskere.

### Kvalitetsverifisering

* **Sjekk alltid RAW (Original)**: Kontroller kildedatakvaliteten før du stoler på behandlede resultater
* **Sammenlign lag**: Bruk lagbytte for å validere at behandlingen fungerte korrekt
* **Kontroller indeksområder**: Bruk Pixel Percent-modus med indekslag for å kontrollere at verdiene er rimelige

***

## Feilsøking

### Lag ikke tilgjengelig

**Problem**: Forventet lag vises ikke i rullegardinmenyen

**Mulige årsaker:**

* Bildet ble ikke behandlet (bare JPG og RAW (Original) er tilgjengelig)
* Refleksjonskalibrering ble deaktivert under behandlingen
* Spesifikk indeks ble ikke konfigurert i prosjektinnstillingene
* Bildet er et mål-bare-bilde (ingen indekser generert for mål)

**Løsninger:**

1. Kontroller at bildet ble behandlet (sjekk utdata-mappen for behandlede filer)
2. Sjekk prosjektinnstillingene for å bekrefte at indeksene ble konfigurert
3. Behandle på nytt med ønskede indekser aktivert

### Feil lag vises

**Problem**: Bildet åpnes i uventet lag

**Årsak**: Lagpreferanse fra forrige bilde ble overført, men det laget finnes ikke i det aktuelle bildet

**Løsning**: Chloros faller automatisk tilbake til JPG når det foretrukne laget ikke er tilgjengelig – dette er normal oppførsel

### Kan ikke se kalibreringsmål

**Problem**: RAW-laget (mål) viser ikke måldeteksjon.

**Mulige årsaker:**

* Målene ble ikke oppdaget under behandlingen.
* Bildet inneholder faktisk ikke mål.
* Innstillingene for måldeteksjon er for strenge.

**Løsninger:**

1. Sjekk feilsøkingsloggen for meldinger om «Mål funnet».
2. Kontroller at bildet faktisk inneholder synlige kalibreringsmål.
3. Juster innstillingene for måldeteksjon i prosjektinnstillingene.
4. Se [Velge målbilder](../processing-images-gui/choosing-target-images.md).

***

## Relaterte funksjoner

### Verktøy for bildevisning

Når du viser et lag, kan du bruke:

* **Zoomkontroller**: Forstørr for å se på detaljer.
* **Panorering**: Klikk og dra for å flytte rundt på det forstørrede bildet.
* **Pikselverdiinspeksjon**: Se verdier på cursorens plassering
* **Navigasjonspiler**: Flytt mellom bilder mens du beholder laget
* **Pikselprosentmodus**: Veksle mellom DN- og prosentvisning

Se [Åpne et bilde i fullskjermmodus](opening-an-image-full-screen.md) for fullstendig dokumentasjon om bildevisningsverktøyet.

### Indeks/LUT-sandkasse

For interaktiv indeksering og visualisering:

* **Indeksberegning i sanntid**: Test forskjellige indeksformler
* **LUT-fargekartlegging**: Bruk fargegraderinger på gråtonede indekser
* **Eksporter visualiseringer**: Lagre fargede indeksbilder

Se [Indeks/LUT Sandbox](index-lut-sandbox.md) for detaljer.

***

## Neste trinn

Nå som du forstår bildelag:

* [**Åpne et bilde i fullskjerm**](opening-an-image-full-screen.md) – Komplett guide til bildevisningen
* [**Index/LUT Sandbox**](index-lut-sandbox.md) – Interaktiv indeksvisualisering
* [**Multispektrale indeksformler**](../project-settings/multispectral-index-formulas.md) – Tilgjengelige indeksreferanser
* [**Fullføre behandlingen**](../processing-images-gui/finishing-the-processing.md) – Forstå behandlede resultater
