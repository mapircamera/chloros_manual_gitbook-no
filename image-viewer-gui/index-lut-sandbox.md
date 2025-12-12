# Indeks/LUT-sandkasse

Indeks/LUT-sandkassen er et interaktivt arbeidsområde i Chloros Image Viewer som lar deg eksperimentere med multispektrale indeksberegninger og fargevisualiseringer i sanntid. Dette kraftige verktøyet hjelper deg med å teste forskjellige indekser, finjustere verdier og lage publikasjonsklare visualiseringer uten å måtte behandle hele datasettet på nytt.

## Hva er indeks/LUT-sandkasse?

### Formål

Sandkassen tilbyr:

* **Indeksberegning i sanntid** – Bruk hvilken som helst vegetasjonsindeks umiddelbart
* **Interaktiv LUT-justering** – Finjuster fargegraderinger og -områder
* **Optimalisering av arbeidsflyt** – Bestem de beste innstillingene før batchbehandling

### Sandkasse vs. prosjektbehandling

**Indeks/LUT-sandkasse (interaktiv):**

* Ett bilde om gangen
* Øyeblikkelig tilbakemelding
* Eksperimentell og iterativ
* Ingen permanente endringer i filer
* Perfekt for utforsking og testing

**Prosjektbehandling (batch):**

* Hele datasettet på en gang
* Forhåndskonfigurerte innstillinger
* Permanente utdatafiler
* Tidkrevende
* Best når innstillingene er ferdigstilt

{% hint style=&quot;success&quot; %}
**Beste arbeidsflyt**: Bruk Sandbox til å eksperimentere og finne optimale indeks- og LUT-innstillinger, og bruk deretter disse innstillingene under prosjektbehandling for hele datasettet.
{% endhint %}

***

## Arbeide med indeks/LUT Sandbox

### Forstå forhåndsberegnede indekser

I Chloros kan indekser brukes under prosjektbehandling. For å bestemme hvilke indeks- og LUT-innstillinger du vil bruke på eksportene, er det enklest å bruke sandkassen i bildeviseren.

Sandkassen lar deg:

* **Bruke nye indekser og fargegraderinger (LUT-er)** for å visualisere dataene
* **Justere visualiseringsinnstillinger** interaktivt
* **Vise** allerede beregnede indeksbilder
* **Inspisere** pikselverdier på alle zoomnivåer

### Åpne sandkassen

Indeks/LUT-sandkassen er tilgjengelig i **Bildeselgeren** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Klikk på et bilde i filbrowserens bildegitter, så åpnes det i **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> -fanen
2. Klikk på **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> -fanen for å åpne venstre pop-out-sidebar hvis den ikke allerede er åpen

### Velge et bilde å bruke en indeks/LUT på

For å arbeide med en indeks i Image Viewer <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sandkassen:

1. **Åpne et bilde** fra hovedbildegitteret ved å klikke på det
2. Fanen **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> -fanen åpnes
3. Klikk på **Layer-rullegardinmenyen** (øverst til høyre i visningsprogrammet)
4. Velg laget fra rullegardinmenyen:
   * RAW (Reflektans)

### Bruke en indeks på et bilde

Når bildet er i fullskjerm og **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> -fanen er åpen:

1. Merk av for Indeks-boksen øverst i sidepanelet
2. Velg kameraets filter fra rullegardinmenyen til venstre
3. Velg ønsket indeksformel fra rullegardinmenyen til høyre
4. Dra filterkanalens fargesirkler til plasseringene i indeksformelen nedenfor
5. Når formelen er gyldig, oppdateres bildet og viser indeksverdiene
6. Flytt musepekeren rundt for å se verdiene på pekerenes plassering
7. Zoom inn for å se individuelle piksler og tilhørende verdier.

Hver indeks har et spesifikt verdifelt og en spesifikk betydning:

#### NDVI Eksempel

```
Formula: (NIR - Red) / (NIR + Red)

For Survey3W RGN camera:
NIR = 850nm band
Red = 661nm band

Result range: -1.0 to +1.0
Typical vegetation: 0.4 to 0.9
Stressed vegetation: 0.2 to 0.4
Bare soil: 0.0 to 0.2
Water: -0.1 to 0.1
```

For fullstendig dokumentasjon av indeksformler, se [Multispektrale indeksformler](../project-settings/multispectral-index-formulas.md).

***

## Arbeide med LUT-er (oppslagstabeller)

### Hva er en LUT?

En **oppslagstabell (LUT)** tilordner numeriske indeksverdier til farger for visualisering:

* **Inndata**: Indeks pikselverdi (f.eks. NDVI 0,65)
* **Utdata**: RGB farge (f.eks. lysegrønn)
* **Formål**: Gjøre mønstre lettere å se og tolke

**Gråtoner vs. farge-LUT:**

* Gråtoner: Vitenskapelig og nøytralt, viser rådata
* Farge-LUT: Intuitivt og effektfullt, fremhever mønstre og forskjeller

{% hint style=&quot;success&quot; %}
**Visualiseringskraft**: Ved å bruke en farge-LUT på et gråtonet indeksbilde blir det betydelig enklere å identifisere mønstre, avvik og områder av interesse med et øyeblikk.
{% endhint %}

### Bruke en LUT på et indeksbilde

Når du har et indeksbilde som viser

1. Klikk på <img src="../.gitbook/assets/image.png" alt="" data-size="line"> &quot;+Legg til LUT&quot;-knappen
2. Velg fargegradienten
3. Juster klippets min./maks. endepunkter
4. Juster klippemodus
5. Merk av for Indeks-boksen i **Bildeselger** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> -fanen i sidepanelet for å bruke LUT-en.

### Velge en fargegradient

**Velge en gradient:**

1. I LUT-panelet finner du **den fargede gradientlinjen**
2. Hold musen over den for å se tilgjengelige forhåndsinnstilte gradienter
3. Velg ønsket gradient
4. Bildet **oppdateres umiddelbart** med nye farger når Index-boksen er merket av

{% hint style=&quot;success&quot; %}
**Beste praksis**: For vegetasjonsindekser som NDVI, er Red-Yellow-Green-gradienten mest intuitiv fordi den samsvarer med naturlige fargeassosiasjoner (grønn = sunn, gul = moderat, rød = stresset).
{% endhint %}

### Justering av fargeklasser

**Klasskontrollen** bestemmer hvor mange diskrete fargetrinn som vises i gradienten:

**Alternativer for antall klasser:**

* **2–5 klasser**: Svært brede kategorier, tydelige soner
* **6–10 klasser**: Balansert, godt egnet for klassifisering
* **11–20 klasser**: Jevne gradienter, kontinuerlig utseende
* **20+ klasser**: Nesten kontinuerlig, maksimal jevnhet

**Slik justerer du:**

1. I LUT-panelet finner du **fargeprøvefeltene under gradientlinjen**.
2. Juster antall klasser ved å legge til med +-knappen.
3. Fjern antall klasser ved å dobbeltklikke på en fargeprøve.
4. Gradienten oppdateres **i sanntid** på bildet.

**Effekt på visualisering:**

* **Færre klasser** (3-5): Skaper tydelige soner, forenklet klassifisering, lettere å skille mellom kategorier.
* **Middels antall klasser** (6–10): Balansert tilnærming, godt egnet for de fleste bruksområder.
* **Flere klasser** (15–20): Jevne overganger, detaljert variasjon, fotografisk utseende.

**Når skal det brukes:**

* **Få klasser (3–5)**: Presentasjonsbilder, klassifiseringskart, enkle rapporter.
* **Middels klasser (6-10)**: Generell analyse, balanserte detaljer, standardrapporter
* **Mange klasser (15-20)**: Vitenskapelig analyse, detaljert inspeksjon, utskrifter av publikasjonskvalitet

### Finjustering av verdivariasjoner

**Verdivariasjonskontrollene** bestemmer hvilke indeksverdier som tilordnes hvilke farger i gradienten din:

**Variasjonskontroller i LUT-panelet:**

* **Minimumsverdi**: Nedre grense for fargeskalaen
* **Maksimumsverdi**: Øvre grense for fargeskalaen
* **Mellomliggende verdier**: Fordeles automatisk mellom min og maks (basert på antall klasser)

#### Justere min-/maksverdier

**Slik justerer du verdier:**

1. I LUT-panelet finner du inndatafeltene **Min Value** og **Max Value**
2. Klikk på feltet **Minimum**
3. Skriv inn ønsket minimumsverdi (f.eks. `0.2`)
4. Trykk **Enter** eller klikk utenfor feltet
5. Gjenta for feltet **Maksimum** (f.eks. `0.9`)
6. Visualiseringen **oppdateres umiddelbart**

{% hint style=&quot;info&quot; %}
**Automatisk skalering**: Når du først bruker en LUT, setter Chloros automatisk min/maks til det faktiske datarekkevidden i bildet. Du kan deretter begrense dette området for å fokusere på bestemte verdier av interesse.
{% endhint %}

**Eksempel på NDVI-områdejusteringer:**

* **Fullt område**: `-1.0` til `1.0` (vis alle mulige verdier)
* **Fokusert på vegetasjon**: `0.2` til `0.9` (utelukk bar jord og vann)
* **Kun sunn vegetasjon**: `0.5` til `0.9` (fremhev kun frodige planter)
* **Stressdeteksjon**: `0.2` til `0.5` (fremhev problemområder)
* **Tilpasset område**: Juster basert på dine observerte pikselverdier

**Hvorfor justere områder?**

* **Øk kontrasten** i området du er interessert i
* **Ekskluder irrelevante verdier** (f.eks. vannmasser, bar jord)
* **Standardiser visualiseringen** på tvers av flere bilder eller datoer
* **Fremhev subtile forskjeller** innenfor et smalt verdispekter

### Klippe ut verdier utenfor området

Når pikselverdiene faller utenfor det definerte min./maks.-området, kan du kontrollere hvordan de vises ved hjelp av **klippemoduser**.

#### **Tilgjengelige alternativer for klippemodus:**

#### 1. Minimum og maksimum

* Piksler **under minimum** → vises med den **første fargen** i gradienten (f.eks. rød)
* Piksler **over maksimum** → vises med den **siste fargen** i gradienten (f.eks. grønn)
* **Bruksområde**: Fremhev ekstreme verdier, vis hele dataintervallet med mettede farger ved grensene
* **Eksempel**: NDVI-verdier under 0,2 vises alle i rødt, verdier over 0,9 vises alle i grønt

#### 2. Gjennomsiktig bakgrunn

* Piksler **utenfor området** blir **helt gjennomsiktige**
* Bare piksler **innenfor området** viser fargegradient
* **Bruksområde**: GIS-overlegg, isolering av bestemte verdier, fremheving av kun områder av interesse
* **Eksempel**: Vis kun NDVI 0,4-0,7 i farge, alt annet gjennomsiktig

{% hint style=&quot;warning&quot; %}
**Begrensning av gjennomsiktighet**: Gjennomsiktige piksler vises som bakgrunnsfarge i visningsprogrammet. Når de eksporteres under behandling, bevares gjennomsiktigheten i PNG-format, men ikke i JPG.
{% endhint %}

#### 3. Indeksbakgrunn

* Piksler **utenfor området** vises i **gråtoner** (viser rå indeksverdier)
* Piksler **innenfor området** viser **fargegradient**
* **Bruksområde**: Subtil utheving, opprettholder konteksten samtidig som områder av interesse fremheves
* **Eksempel**: Fargeutheving av stresset vegetasjon (NDVI 0,3-0,5) mens sunne områder vises i grått

#### 4. Original bakgrunn

* Piksler **utenfor området** vises som **det originale multispektrale bildet**
* Piksler **innenfor området** viser **fargegradient**
* **Bruksområde**: Mest intuitivt – kombinerer naturlig bildekontekst med analytisk fargeoverlegg
* **Eksempel**: Se det faktiske utseendet på åkeren/avlingen med fargekodede stressområder overlagt

### Velge riktig klippemodus

| Klippemodus              | Best for                                   | Visualiseringsstil          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimum og maksimum**    | Full datavisning, vitenskapelig analyse     | Alle piksler farget           |
| **Gjennomsiktig bakgrunn** | GIS-overlegg, isolering av bestemte områder    | Farge på området, tomt utenfor |
| **Indeksbakgrunn**       | Subtil vektlegging, opprettholder datakontekst  | Farge på område, grått utenfor  |
| **Original bakgrunn**    | Rapporter, presentasjoner, intuitiv analyse | Farge på område, foto utenfor |

### Opprette egendefinerte LUT-farger

For full kontroll over visualiseringen kan du opprette **egendefinerte fargegraderinger** ved å redigere individuelle fargestopp.

**Slik oppretter du en tilpasset overgang:**

1. I LUT-panelet finner du **forhåndsvisningslinjen for overganger**
2. Se etter **fargeprøvefeltene** under overgangen
3. **Klikk på et fargestopp** for å velge det
4. En **fargevelger** åpnes
5. Velg en ny farge ved å bruke:
   * **Fargehjul**: Visuell fargevalg
   * **RGB/HSV-glidebrytere**: Presis fargekontroll
   * **Hex-kodeinngang**: Nøyaktig fargespesifikasjon (f.eks. `#FF0000` for rød)
6. Klikk utenfor fargevelgeren **for å bruke den nye fargen**
7. Gradienten **oppdateres umiddelbart** på bildet

**Legge til eller fjerne fargestopp:**

* **Legg til et stopp**: Klikk på +-ikonet for å legge til en ny fargeprøve på slutten
* **Fjern et stopp**: Dobbeltklikk på fargefeltet for å fjerne fargeprøven

**Tilpasningsstrategier:**

* **Inverter gradient**: Vend fargerekkefølgen for å reversere betydningen (f.eks. grønn=lav, rød=høy)
* **Merkevarefarger**: Tilpass fargepaletten til organisasjonen din for rapporter
* **Fargeblindvennlig**: Bruk kombinasjoner av oransje-blått eller lilla-gult
* **Utskriftsoptimalisering**: Velg farger som fungerer både ved farge- og gråtonet utskrift
* **Flere terskler**: Bruk forskjellige farger ved bestemte verditerskler for klassifisering

{% hint style=&quot;info&quot; %}
**Lagre tilpassede fargegraderinger**: Tilpassede fargegraderinger kan lagres og gjenbrukes. Klikk på lagringsikonet i LUT-panelet for å lagre dine tilpassede fargevalg for fremtidig bruk.
{% endhint %}

***

## Interaktiv arbeidsflyt

### Oppdateringer i sanntid

Alle LUT-justeringer i sandkassen oppdaterer bildet **øyeblikkelig og interaktivt**:

* **Bytt lag** → Bildet endres umiddelbart
* **Velg gradient** → Fargene oppdateres umiddelbart
* **Juster verdifeltet** → Kontrasten endres i sanntid
* **Endre klasser** → Gradientens glatthet oppdateres umiddelbart
* **Endre klipping** → Bakgrunnsvisningen endres umiddelbart
* **Rediger farger** → Tilpasset gradient brukes umiddelbart

**Ingen «Bruk»-knapp nødvendig** – alle endringer er live og interaktive!

{% hint style=&quot;success&quot; %}
**Live tilbakemelding**: Den øyeblikkelige visuelle tilbakemeldingen lar deg raskt eksperimentere med forskjellige innstillinger til du finner den optimale visualiseringen for dine analysebehov.
{% endhint %}

### Iterativ forbedringsarbeidsflyt

**Typisk LUT-optimaliseringsarbeidsflyt:**

1. **Velg indekslag** (f.eks. RAW (refleksjonsgrad))
2. **Bruk indeks** – Velg kamerafilter og indeksformel, dra fargede sirkler til riktig sted i indeksformelen
3. **Bruk LUT-gradient** – Start med forhåndsinnstillingen Red-Yellow-Green
4. **Kontroller pikselverdier** – Flytt markøren rundt, noter verdivariasjoner
5. **Juster min/maks** – Begrens for å fokusere på vegetasjon (f.eks. 0,2 til 0,9)
6. **Velg klipping** – Prøv «Original bakgrunn» for kontekst
7. **Finjuster farger** – Tilpass gradienten om nødvendig for spesifikk vektlegging
8. **Fullfør innstillingene** – Dokumenter innstillingene og kopier til prosjektinnstillinger for eksportbehandling

### Inspeksjon av pikselverdier

Det er avgjørende å forstå faktiske pikselverdier for å kunne angi effektive LUT-verdier:

**Slik inspiserer du verdier:**

1. Pikselverdier vises når bildet har enten Indeks eller både Indeks og LUT **merket av**.
2. **Flytt markøren** over forskjellige områder av bildet
3. **Observer pikselverdiene** som vises i legenden når du holder markøren over
4. Zoom inn for å se individuelle piksler markert med en flytende verdi
5. **Noter** verdierangene for forskjellige funksjoner:
   * **Sunn vegetasjon**: f.eks. NDVI 0,55-0,85
   * **Stresset vegetasjon**: f.eks. NDVI 0,30-0,50
   * **Bare jord**: f.eks. NDVI 0,05–0,25
   * **Vann** (hvis til stede): f.eks. NDVI -0,05 til 0,10

**Bruke pikselverdier til å angi LUT-intervaller:**

Etter å ha inspisert pikselverdiene, justerer du LUT min/maks tilsvarende:

**Eksempel på scenario:**

* **Observasjon**: Jordverdier = 0,05-0,25, Stresset = 0,25-0,50, Sunn = 0,50-0,85
* **Mål**: Visualiser kun plantens helse (unntatt jord)
* **LUT-innstillinger**: Min = `0.25`, Maks = `0.85`
* **Klipping**: «Original bakgrunn» for å se jord i naturlig farge
* **Resultat**: Fargegradienten gjelder kun vegetasjon, jord vises som originalbilde

{% hint style=&quot;info&quot; %}
**Dynamisk område**: Ulike avlinger, årstider og vekststadier vil ha forskjellige verdier. Kontroller alltid pikselverdiene i ditt spesifikke datasett før du angir LUT-områder.
{% endhint %}

***

## Tilpassede indekser (Chloros+)

### Opprette tilpassede indeksformler

{% hint style=&quot;info&quot; %}
**Hvor du oppretter**: Tilpassede indekser kan konfigureres i **Prosjektinnstillinger** før behandling, samt i sidefeltet i Image Viewer-sandkassen.
{% endhint %}

**Slik oppretter du en tilpasset indeks:**

1. **Åpne Prosjektinnstillinger** (før behandling) eller Image Viewer-sandkassenes sidefelt
2. Naviger til **Indeksformel-rullegardinmenyen**
3. Se etter **«Tilpasset»**-alternativet (du må være logget inn med Chloros+-lisens)
4. **Definer formelen** ved hjelp av båndvariabler:
   * Båndnavn: `NIR`, `Red`, `Green`, `Blue`, `RedEdge` osv.
   * Operatører: `+`, `-`, `*`, `/`, `^` (eksponent)
   * Funksjoner: `sqrt()`, `abs()` osv. (hvis støttet)
   * Parenteser: `()` for rekkefølgen av operasjoner
5. **Gi indeksen et navn** (f.eks. «MyIndex» eller «CustomNDVI»)
6. **Lagre konfigurasjonen**

**Eksempler på tilpassede formler:**

```
Modified NDVI with offset:
(NIR - Red) / (NIR + Red + 0.5)

Simple ratio:
NIR / Red

Complex multi-band:
(NIR - Red) / (NIR + Red - Blue)

Exponential index:
(NIR / Red) ^ 2
```

{% hint style=&quot;warning&quot; %}
**Formelvalidering**: Sørg for at formelen bruker bånd som er tilgjengelige i kameraet. For eksempel er RedEdge bare tilgjengelig på kameraer med et RedEdge-filter.
{% endhint %}

***

## Neste trinn

Nå som du forstår indeks/LUT-sandkassen:

* **Bruk på behandling**: Bruk oppdagede innstillinger i [Prosjektinnstillinger](../project-settings/project-settings.md)
* **Batchbehandling**: Bruk optimaliserte indekser på hele datasett
* **Lær mer**: Les [Multispektrale indeksformler](../project-settings/multispectral-index-formulas.md)

Relatert dokumentasjon:

* [**Bildelag**](image-layers.md) – Lagadministrasjon og visualisering
* [**Åpne et bilde i fullskjerm**](opening-an-image-full-screen.md) – Grunnleggende om bildevisningen
* [**Behandle bilder (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Full behandlingsarbeidsflyt
