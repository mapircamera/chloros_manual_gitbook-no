# Bildelag

Nedtrekksmenyen «Bildelag» i Chloros Image Viewer lar deg raskt veksle mellom ulike versjoner av det samme bildet – fra de opprinnelige opptakene til bearbeidede reflektansutdata og beregnede indeksbilder.

## Hva er bildelag?

I Chloros refererer **lag** til de forskjellige bildeutdataene som er tilgjengelige for et enkelt kildebilde. Når du behandler bilder, oppretter Chloros flere versjoner:

* **Originalbilder** (JPG- og RAW-filer fra kameraet ditt)
* **Refleksjonskalibrerte** utdata (hvis refleksjonskalibrering var aktivert)
* **Målbilder** (hvis bildet inneholder kalibreringsmål)
* **Indeksbilder** (NDVI, NDRE, GNDVI osv. hvis indekser ble konfigurert)**Rullelisten for lagvalg** øverst til høyre i bildeviseren lar deg umiddelbart bytte mellom disse versjonene uten å forlate viseren.***

## Tilgjengelige lagtyper

### JPG

* Det originale JPG-forhåndsvisningsbildet fra kameraet ditt
* Alltid tilgjengelig for alle bilder
* Ubehandlet, slik det ble tatt av kameraet
* Raskest å laste inn og vise

**Når du bør se på det:**

* Rask forhåndsvisning av det opprinnelige bildet
* Kontrollere bildekomposisjon og innramming
* Verifisere bildekvalitet før behandling

### RAW (Original)

* De opprinnelige RAW-sensordataene fra kameraet ditt
* Debayered uten etterbehandling
* Høyere bitdybde enn JPG (vanligvis 12-bit eller 14-bit sensordata)

**Når du bør vise:**

* Inspisere kvaliteten på de opprinnelige sensordataene
* Kontrollere for sensorproblemer eller artefakter
* Sammenligne resultater før og etter behandling

### RAW (Mål)

* Vises kun for bilder som er identifisert som inneholdende kalibreringsmål
* Viser det originale RAW-bildet med detektert mål
* Brukes til å verifisere at måldeteksjonen var vellykket

**Når du bør se på dette:**

* Bekrefte at kalibreringsmålene ble detektert korrekt
* Kontrollere kvaliteten på målbilde
* Feilsøke kalibreringsproblemer

{% hint style="info" %}
**Mål-lag**: Dette laget vises kun i rullegardinmenyen for bilder som inneholder kalibreringsmål. Vanlige opptaksbilder vil ikke ha dette alternativet.
{% endhint %}

### RAW (refleksjon)

* Det kalibrerte utgangsbildet for refleksjon
* Vignettkorrigert (hvis aktivert i behandlingen)
* Refleksjonskalibrert ved hjelp av måldata (hvis aktivert)
* Multibånd TIFF med alle kamerakanaler
* Pikselverdiene representerer prosentvis refleksjon (når du bruker prosentmodus)
* Klar til å manipuleres med [Index/LUT Sandbox](index-lut-sandbox.md)

**Når skal det vises:**

* Inspeksjon av kalibrerte resultater
* Verifisering av kalibreringskvalitet
* Kontroll av pikselverdier for vitenskapelig nøyaktighet
* Sammenligning med originalen for å se kalibreringseffekter

{% hint style="success" %}
**Anbefalt**: Bruk RAW (refleksjonsgrad)-laget når du kontrollerer pikselverdier for vitenskapelige målinger og analyser.
{% endhint %}

### RAW (NDVI-indeks)... og lignende

* Beregnet vegetasjonsindeksbilde (NDVI i dette eksemplet)
* Indeksnavnet endres avhengig av hvilken indeks som ble konfigurert under behandlingen
* Eksempler: RAW (NDVI-indeks), RAW (NDRE-indeks), RAW (GNDVI-indeks) osv.
* Enkeltbånds gråtonebilde som viser resultatene av indeksberegningen
* Det vises ett lag for hver indeks som er konfigurert i prosjektinnstillingene

**Mulige indeksnavn:**

* RAW (NDVI-indeks)
* RAW (NDRE-indeks)
* RAW (GNDVI-indeks)
* RAW (OSAVI-indeks)
* RAW (EVI-indeks)
* RAW (SAVI-indeks)
* Og mange flere... (se [Formler for multispektrale indekser](../project-settings/multispectral-index-formulas.md))

**Når skal du vise:**

* Undersøke resultater av indeksberegninger
* Sjekke indeksverdier
* Identifisere områder av interesse
* Verifisere indeksbilder før bruk i GIS eller analyse

***

## Bruke lagvelgeren

### Åpne rullegardinmenyen

1. Åpne et bilde i fullskjermmodus (klikk på en hvilken som helst miniatyrbilde i bildeviseren)
2. Finn **lag-rullegardinmenyen** øverst til høyre i viseren
3. Rullegardinmenyen viser det valgte laget (f.eks. «JPG»)
4. Klikk på rullegardinmenyen for å se alle tilgjengelige lag

### Bytte mellom lag

1. Klikk på lag-rullegardinmenyen for å åpne listen
2. Alle tilgjengelige lag for det aktuelle bildet vises
3. Klikk på et hvilket som helst lagnavn for å bytte til den versjonen
4. Bildet oppdateres umiddelbart for å vise det valgte laget

**Rask bytte:**

* Nedtrekksmenyen husker ditt siste valg
* Når du navigerer til neste bilde, prøver Chloros å vise samme lagtype
* Hvis det laget ikke finnes på neste bilde, blir standardinnstillingen JPG

### Lagtilgjengelighet

Ikke alle lag er tilgjengelige for hvert bilde:

**Alltid tilgjengelig:*** ✅ JPG (hvert bilde har en JPG-forhåndsvisning)

**Betinget tilgjengelig:**

* ⚠️ RAW (Original) – Kun hvis bildet ble tatt i RAW- eller RAW+JPG-modus
* ⚠️ RAW (Mål) – Kun hvis bildet inneholder oppdagede kalibreringsmål
* ⚠️ RAW (Reflektans) – Kun etter behandling med reflektanskalibrering aktivert
* ⚠️ RAW (\[Indeks] Indeks) – Kun etter behandling med konfigurerte indekser

***

## Lagets vedvarende tilstand

### Navigere mellom bilder

Når du navigerer til et annet bilde (ved hjelp av piltastene eller ved å klikke på miniatyrbilder):**Laginnstillingen beholdes:**

* Hvis du viser «RAW (Reflektans)», viser neste bilde «RAW (Reflektans)» (hvis tilgjengelig)
* Hvis du viser «RAW (NDVI Indeks)», viser neste bilde «RAW (NDVI Indeks)» (hvis tilgjengelig)
* Hvis det samme laget ikke finnes, blir standardinnstillingen JPG

**Eksempel på arbeidsflyt:**

1. Åpne bilde 1, bytt til RAW (NDVI Index)
2. Trykk → for å vise bilde 2
3. Bilde 2 viser automatisk RAW (NDVI Index)-laget
4. Fortsett å navigere – alle bilder viser NDVI-laget
5. Veldig effektivt for å gjennomgå indeksresultater på tvers av mange bilder

***

## Vanlige arbeidsflyter

### Arbeidsflyt 1: Før/etter-sammenligning

**Mål**: Sammenligne originalbilde med kalibrert bilde

1. Åpne det behandlede bildet i Image Viewer
2. Velg **RAW (Original)** fra rullegardinmenyen
3. Legg merke til vignettering og ukalibrerte verdier
4. Bytt til **RAW (Reflectance)** fra rullegardinmenyen
5. Sammenlign – vignettering fjernet, verdier kalibrert

### Arbeidsflyt 2: Gjennomgang av indeks

**Mål**: Gjennomgå raskt NDVI-resultater i hele datasettet

1. Åpne det første behandlede bildet
2. Velg **RAW (NDVI-indeks)** fra rullegardinmenyen
3. Bruk → piltasten for å navigere til neste bilde
4. NDVI-laget beholdes automatisk
5. Fortsett gjennom alle bildene og sjekk NDVI-mønstrene
6. Bytt til **RAW (NDRE Index)** for å sammenligne

### Arbeidsflyt 3: Verifisering av mål

**Mål**: Verifisere at alle målbilder ble oppdaget riktig

1. Naviger til et målbilde
2. Velg **RAW (Target)** fra rullegardinmenyen
3. Verifiser at kalibreringsmålene er tydelig synlige og oppdaget
4. Naviger til neste målbilde
5. Gjenta verifiseringen for alle mål

### Arbeidsflyt 4: Inspeksjon av pikselverdier

**Mål**: Sjekk refleksjonsverdiene for vitenskapelig nøyaktighet

1. Åpne det behandlede bildet
2. Velg **RAW (Refleksjon)**-laget
3. Aktiver **Pikselprosent**-modus (knappen øverst til høyre i verktøylinjen)
4. Flytt markøren over vegetasjonsområder
5. Kontroller at pikselverdiene ligger innenfor forventede områder (30–70 % for NIR, 5–15 % for Red)
6. Kontroller at jord- og vannområder har passende verdier

***

## Forstå pikselverdier etter lag

Ulike lag viser forskjellige pikselverdiområder:

### JPG-lag

* **Område**: 0–255 (8-bit)
* **Betydning**: Visningsverdier, gammakorrigert
* **Bruk**: Kun visuell inspeksjon, ikke for vitenskapelig måling

### RAW (Original)

* **Område**: 0–65535 (16-bit)
* **Betydning**: Rå digitale tall fra sensoren
* **Bruk**: Kontroll av sensorens ytelse, ikke kalibrert

### RAW (refleksjonsgrad)

* **Område**: 0–65 535 (16-bit TIFF) eller 0,0–1,0 (32-bit prosent)
* **Betydning**: Kalibrert refleksjonsgrad i prosent
* **Bruk**: Vitenskapelige målinger og analyser**For 16-bit TIFF:**Del med 65 535 for å få prosent refleksjonsgrad**For 32-bit prosent:** Verdiene representerer direkte prosent (0,5 = 50 % refleksjonsgrad)

### RAW (Indeksbilder)

* **Område**: Varierer etter indeks (typisk -1,0 til +1,0 for normaliserte indekser)
* **Betydning**: Resultat av indeksberegning
* **Eksempler**:
  * NDVI: -1 til +1 (vegetasjon typisk 0,4 til 0,9)
  * NDRE: -1 til +1 (stressdeteksjon)
  * EVI: 0 til 1 (forbedret vegetasjon)

***

## Tips og beste praksis

### Effektiv bytte mellom lag

* **Kjennskap til hurtigtaster**: Selv om det ikke finnes hurtigtaster for lag, fungerer navigasjonspilene (←/→) på tvers av alle lag
* **Konsistente arbeidsflyter**: Velg ett lag (f.eks. NDVI) og gjennomgå hele datasettet før du bytter til et annet
* **Raske sammenligninger**: Veksle mellom Original og Reflektans for å verifisere behandlingskvaliteten

### Ytelseshensyn

* **JPG lastes inn raskest**: Brukes for rask navigering gjennom mange bilder
* **RAW-lag lastes inn saktere**: Høyere oppløsning og bitdybde
* **Indekslag**: Lignende hastighet som Refleksjonslag
* **Første innlasting er tregest**: Påfølgende visninger av samme lag blir lagret i hurtigbufferen og er raskere

### Kvalitetsverifisering

* **Sjekk alltid RAW (Original)**: Kontroller kvaliteten på kildedataene før du stoler på de behandlede resultatene
* **Sammenlign lag**: Bruk lagbytte for å bekrefte at behandlingen fungerte riktig
* **Sjekk indeksområder**: Bruk Pixel Percent-modus med indekslag for å verifisere at verdiene er rimelige***

## Feilsøking

### Lag ikke tilgjengelig

**Problem**: Forventet lag vises ikke i rullegardinmenyen**Mulige årsaker:**

* Bildet ble ikke behandlet (bare JPG og RAW (Original) er tilgjengelig)
* Refleksjonskalibrering var deaktivert under behandlingen
* Spesifikk indeks ble ikke konfigurert i prosjektinnstillingene
* Bildet er et mål-kun-bilde (ingen indekser generert for mål)

**Løsninger:**

1. Kontroller at bildet ble behandlet (sjekk utdatamappen for behandlede filer)
2. Sjekk prosjektinnstillingene for å bekrefte at indeksene ble konfigurert
3. Behandle på nytt med ønskede indekser aktivert

### Feil lag vises

**Problem**: Bildet åpnes i et uventet lag**Årsak**: Lagpreferansen fra forrige bilde ble overført, men det laget finnes ikke i det aktuelle bildet**Løsning**: Chloros faller automatisk tilbake til JPG når det foretrukne laget ikke er tilgjengelig – dette er normal oppførsel

### Kan ikke se kalibreringsmål

**Problem**: RAW-laget (mål) viser ikke måldeteksjon**Mulige årsaker:**

* Målene ble ikke oppdaget under behandlingen
* Bildet inneholder faktisk ikke mål
* Innstillingene for måldeteksjon er for strenge

**Løsninger:**

1. Sjekk feillogg for meldinger om «Mål funnet»
2. Kontroller at bildet faktisk inneholder synlige kalibreringsmål
3. Juster innstillingene for måldeteksjon i prosjektinnstillingene
4. Se [Velge målbilder](../processing-images-gui/choosing-target-images.md)

***

## Relaterte funksjoner

### Verktøy i bildeviseren

Når du viser et lag, kan du bruke:

* **Zoomkontroller**: Forstørr for å inspisere detaljer
* **Panorer**: Klikk og dra for å bevege deg rundt i det forstørrede bildet
* **Inspeksjon av pikselverdier**: Se verdier ved markørens plassering
* **Navigasjonspiler**: Flytt mellom bilder mens du beholder laget
* **Pikselprosentmodus**: Veksle mellom DN- og prosentvisning

Se [Åpne et bilde i fullskjerm](opening-an-image-full-screen.md) for fullstendig dokumentasjon om bildeviseren.

### Indeks/LUT-sandkasse

For interaktiv indekstesting og visualisering:

* **Indeksberegning i sanntid**: Test forskjellige indeksformler
* **LUT-fargemapping**: Bruk fargegradienter på gråtonede indekser
* **Eksporter visualiseringer**: Lagre fargede indeksbilder

Se [Indeks/LUT-sandkasse](index-lut-sandbox.md) for detaljer.

***

## Neste trinn

Nå som du forstår bildelag:

* [**Åpne et bilde i fullskjerm**](opening-an-image-full-screen.md) – Komplett Image Viewer-veiledning
* [**Index/LUT Sandbox**](index-lut-sandbox.md) – Interaktiv indeksvisualisering
* [**Multispektrale indeksformler**](../project-settings/multispectral-index-formulas.md) – Referanse for tilgjengelige indekser
* [**Fullføre behandlingen**](../processing-images-gui/finishing-the-processing.md) – Forstå behandlede resultater
