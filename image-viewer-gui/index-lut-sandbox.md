# Indeks/LUT-sandkasse

Indeks/LUT-sandkassen er et interaktivt arbeidsområde i Chloros Image Viewer som lar deg eksperimentere med beregninger av multispektrale indekser og fargevisualiseringer i sanntid. Dette kraftige verktøyet hjelper deg med å teste ulike indekser, finjustere verdier og lage visualiseringer som er klare for publisering, uten å måtte behandle hele datasettet på nytt.

## Hva er Index/LUT Sandbox?

### Formål

Sandbox tilbyr:

* **Indeksberegning i sanntid** – Bruk hvilken som helst vegetasjonsindeks umiddelbart
* **Interaktiv LUT-justering** – Finjuster fargegradienter og -områder
* **Optimalisering av arbeidsflyt** – Bestem de beste innstillingene før batchbehandling

### Sandkasse vs. prosjektbehandling

**Indeks/LUT-sandkasse (interaktiv):**

* Ett bilde om gangen
* Umiddelbar tilbakemelding
* Eksperimentell og iterativ
* Ingen permanente endringer i filene
* Perfekt for utforsking og testing

**Prosjektbehandling (batch):**

* Hele datasettet på en gang
* Forhåndskonfigurerte innstillinger
* Permanente utdatafiler
* Tidkrevende
* Best når innstillingene er ferdigstilt

{% hint style="success" %}
**Beste arbeidsflyt**: Bruk Sandbox til å eksperimentere og finne optimale indeks- og LUT-innstillinger, og bruk deretter disse innstillingene under prosjektbehandling for hele datasettet.
{% endhint %}

***

## Arbeide med indeks-/LUT-sandkassen

### Forstå forhåndsberegnede indekser

I Chloros kan indekser brukes under prosjektbehandlingen. For å bestemme hvilke indeks- og LUT-innstillinger du vil bruke på eksportene, er det enklest å bruke bildesynersandkassen.

Sandkassen lar deg:

* **Bruke nye indekser og fargegradienter (LUT-er)** for å visualisere dataene
* **Justere visualiseringsinnstillinger** interaktivt
* **Se** allerede beregnede indeksbilder
* **Inspisere** pikselverdier på alle zoomnivåer

### Åpne sandkassen

Du får tilgang til indeks-/LUT-sandkassen i **bildeviserens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sidefeltet:

1. Klikk på et bilde i bildeoversikten i filbrowseren, så åpnes det i **bildesviseren** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> -fanen
2. Klikk på **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> for å åpne sidepanelet til venstre hvis det ikke allerede er åpent

### Velge et bilde å bruke en indeks/LUT på

For å jobbe med en indeks i <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sandkassen:

1. **Åpne et bilde** fra hovedbildegitteret ved å klikke på det
2. **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> -fanen åpnes
3. Klikk på **Lag-rullegardinmenyen** (øverst til høyre i visningsvinduet)
4. Velg laget fra rullegardinmenyen:
   * RAW (Refleksjon)

### Bruke et indeks på et bilde

Når bildet er i fullskjerm og **Bildeviser** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> er åpen:

1. Merk av i indeksboksen øverst i sidepanelet
2. Velg kameraets filter fra rullegardinmenyen til venstre
3. Velg ønsket indeksformel fra rullegardinmenyen til høyre
4. Dra filterkanalens fargesirkler til plasseringene i indeksformelen nedenfor
5. Når formelen er gyldig, oppdateres bildet og viser indeksverdiene
6. Flytt musepekeren rundt for å se verdiene på pekerenes plassering
7. Zoom inn for å se enkeltpiksler og tilhørende verdier

Hver indeks har et spesifikt verdispekter og en spesifikk betydning:

#### Eksempel på NDVI

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

* **Inndata**: Indekspikselverdi (f.eks. NDVI 0,65)
* **Utdata**: RGB farge (f.eks. lysegrønn)
* **Formål**: Gjøre mønstre lettere å se og tolke**Gråtoner vs. farge-LUT:**

* Gråtoner: Vitenskapelig og nøytralt, viser rådata
* Farge-LUT: Intuitivt og effektfullt, fremhever mønstre og forskjeller

{% hint style="success" %}
**Visualiseringskraft**: Å bruke en farge-LUT på et gråtonet indeksbilde gjør det dramatisk enklere å identifisere mønstre, avvik og områder av interesse med et øyeblikk.
{% endhint %}

### Bruke en LUT på et indeksbilde

Når du har et indeksbilde som viser

1. Klikk på <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> &quot;+Legg til LUT&quot;-knappen
2. Velg fargegradienten
3. Juster klippingspunktets min./maks. endepunkter
4. Juster klippingsmodus
5. Merk av for Indeks-boksen i **Bildeviser** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> for å bruke LUT

### Velge en fargegradient

**Velge en gradient:**

1. I LUT-panelet finner du den**fargede gradientlinjen**

2. Hold musen over den for å se tilgjengelige gradientinnstillinger
3. Velg ønsket gradient
4. Bildet **oppdateres umiddelbart** med nye farger når boksen for indeks er merket av

{% hint style="success" %}
**Beste praksis**: For vegetasjonsindekser som NDVI er gradienten Red-Gul-Green mest intuitiv, fordi den samsvarer med naturlige fargeassosiasjoner (grønn = sunn, gul = moderat, rød = stresset).
{% endhint %}

### Justere fargeklasser

**Kontrollen Klasser**bestemmer hvor mange diskrete fargetrinn som vises i gradienten din:**Alternativer for antall klasser:*** **2–5 klasser**: Svært brede kategorier, tydelige soner
* **6–10 klasser**: Balansert, godt egnet for klassifisering
* **11–20 klasser**: Jevne gradienter, kontinuerlig utseende
* **20+ klasser**: Nesten kontinuerlig, maksimal jevnhet**Slik justerer du:**

1. I LUT-panelet finner du**fargeprøvefeltene under gradientlinjen**

2. Juster antall klasser ved å legge til med +-knappen
3. Fjern antall klasser ved å dobbeltklikke på en fargeprøve
4. Gradienten oppdateres **i sanntid** på bildet**Effekt på visualisering:*** **Færre klasser** (3–5): Skaper tydelige soner, forenklet klassifisering, lettere å skille mellom kategorier
* **Middels antall klasser** (6–10): Balansert tilnærming, godt egnet for de fleste bruksområder
* **Flere klasser** (15–20): Jevne overganger, detaljert variasjon, fotografisk utseende**Når skal det brukes:*** **Få klasser (3–5)**: Presentasjonsbilder, klassifiseringskart, enkle rapporter
* **Middels antall klasser (6–10)**: Generell analyse, balanserte detaljer, standardrapporter
* **Mange klasser (15–20)**: Vitenskapelig analyse, detaljert inspeksjon, resultater av publikasjonskvalitet

### Finjustering av verdier

**Verdikontrollene**bestemmer hvilke indeksverdier som tilordnes hvilke farger i gradienten din:**Verdikontroller i LUT-panelet:*** **Minimumsverdi**: Nedre grense for fargeskalaen
* **Maksimumsverdi**: Øvre grense for fargeskalaen
* **Mellomverdier**: Fordeles automatisk mellom min og maks (basert på antall klasser)

#### Justere min./maks. verdier

**Slik justerer du verdibånd:**

1. I LUT-panelet finner du inndatafeltene**Min. verdi**og**Maks. verdi**

2. Klikk på feltet**Min. verdi**

3. Skriv inn ønsket minimumsverdi (f.eks. `0.2`)
4. Trykk **Enter** eller klikk utenfor feltet
5. Gjenta for **Maks. verdi**-feltet (f.eks. `0.9`)
6. Visualiseringen **oppdateres umiddelbart**{% hint style="info" %}**Automatisk skalering**: Når du først bruker en LUT, setter Chloros automatisk min/maks til det faktiske dataintervallet i bildet. Du kan deretter begrense dette intervallet for å fokusere på bestemte verdier av interesse.
{% endhint %}

**Eksempel på justeringer av NDVI-området:*** **Fullt område**: `-1.0` til `1.0` (vis alle mulige verdier)
* **Fokusert på vegetasjon**: `0.2` til `0.9` (ekskluder bar jord og vann)
* **Kun sunn vegetasjon**: `0.5` til `0.9` (fremhev kun livskraftige planter)
* **Stressdeteksjon**: `0.2` til `0.5` (fremhev problemområder)
* **Tilpasset område**: Juster basert på dine observerte pikselverdier**Hvorfor justere områder?*** **Øk kontrasten** i området du er interessert i
* **Ekskluder irrelevante verdier** (f.eks. vannforekomster, bar jord)
* **Standardiser visualiseringen** på tvers av flere bilder eller datoer
* **Fremhev subtile forskjeller** innenfor et smalt verdispekter

### Klippe ut verdier utenfor området

Når pikselverdier faller utenfor det definerte min./maks.-området, kan du kontrollere hvordan de vises ved hjelp av **klippemoduser**.

#### **Tilgjengelige alternativer for klippemodus:**

#### 1. Minimum og maksimum

* Piksler **under minimum**→ vises med den**første fargen** i gradienten (f.eks. rød)
* Piksler **over maksimum**→ vises med den**siste fargen** i gradienten (f.eks. grønn)
* **Bruksområde**: Fremhev ekstreme verdier, vis hele dataintervallet med mettede farger ved grensene
* **Eksempel**: NDVI-verdier under 0,2 vises alle som røde, verdier over 0,9 vises alle som grønne

#### 2. Gjennomsiktig bakgrunn

* Piksler **utenfor området**blir**helt gjennomsiktige*** Bare piksler **innenfor området** viser fargegradient
* **Bruksområde**: GIS-overlegg, isolering av spesifikke verdier, fremheving av kun områder av interesse
* **Eksempel**: Vis kun NDVI 0,4–0,7 i farge, alt annet gjennomsiktig

{% hint style="warning" %}
**Begrensning av gjennomsiktighet**: Gjennomsiktige piksler vises som bakgrunnsfargen i visningsprogrammet. Når filen eksporteres under behandling, beholdes gjennomsiktigheten i PNG-format, men ikke i JPG.
{% endhint %}

#### 3. Indeksbakgrunn

* Piksler **utenfor området**vises i**gråtoner** (viser rå indeksverdier)
* Piksler **innenfor området**viser**fargegradient*** **Bruksområde**: Subtil utheving, bevarer konteksten samtidig som områder av interesse fremheves
* **Eksempel**: Fargeutheving av stresset vegetasjon (NDVI 0,3–0,5) mens sunne områder vises i grått

#### 4. Original bakgrunn

* Piksler **utenfor området**viser det**originale multispektrale bildet*** Piksler **innenfor området**viser**fargegradient*** **Bruksområde**: Mest intuitivt – kombinerer naturlig bildesammenheng med analytisk fargeoverlegg
* **Eksempel**: Se det faktiske utseendet på åkeren/avlingen med fargekodede stressområder lagt over

### Velge riktig klippemodus

| Klippemodus              | Best egnet for                                   | Visualiseringsstil          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimum og maksimum**    | Full datavisning, vitenskapelig analyse     | Alle piksler fargelagte           |
| **Gjennomsiktig bakgrunn** | GIS-overlegg, isolering av spesifikke områder    | Farge innenfor området, blank utenfor |
| **Indeksbakgrunn**       | Subtil vektlegging, opprettholder datakonteksten  | Farge på området, grått utenfor  |
| **Original bakgrunn**    | Rapporter, presentasjoner, intuitiv analyse | Farge på området, foto utenfor |

### Opprette egendefinerte LUT-farger

For full kontroll over visualiseringen din kan du opprette **egendefinerte fargegradienter** ved å redigere individuelle fargepunkter.**Slik oppretter du en tilpasset gradient:**

1. I LUT-panelet finner du**forhåndsvisningslinjen for gradienter**

2. Se etter**fargeprøvefeltene** under gradienten
3. **Klikk på et fargestopp** for å velge det
4. En **fargevelger** åpnes
5. Velg en ny farge ved hjelp av:
   * **Fargehjulet**: Visuelt fargevalg
   * **RGB/HSV-glidebrytere**: Presis fargekontroll
   * **Inntasting av heksakode**: Eksakt fargespesifikasjon (f.eks. `#FF0000` for rød)
6. Klikk utenfor fargevelgeren **for å bruke den nye fargen**

7. Gradienten**oppdateres umiddelbart** på bildet**Legge til eller fjerne fargepunkter:*** **Legg til et punkt**: Klikk på +-ikonet for å legge til en ny fargeprøve på slutten
* **Fjern et punkt**: Dobbeltklikk på fargefeltet for å fjerne fargeprøven**Tilpasningsstrategier:*** **Inverter gradient**: Vend farge rekkefølgen for å reversere betydningen (f.eks. grønn=lav, rød=høy)
* **Merkevarefarger**: Tilpass organisasjonens fargepalett for rapporter
* **Fargeblind-vennlig**: Bruk oransje-blå eller lilla-gule kombinasjoner
* **Utskriftsoptimalisering**: Velg farger som fungerer både ved farge- og gråtoneutskrift
* **Flere terskler**: Bruk forskjellige farger ved bestemte verditerskler for klassifisering

{% hint style="info" %}
**Lagring av tilpassede gradienter**: Tilpassede gradienter kan lagres og gjenbrukes. Klikk på lagringsikonet i LUT-panelet for å lagre dine tilpassede fargevalg for fremtidig bruk.
{% endhint %}

***

## Interaktiv arbeidsflyt

### Oppdateringer i sanntid

Alle LUT-justeringer i sandkassen oppdaterer bildet **umiddelbart og interaktivt**:

* **Bytt lag** → Bildet endres umiddelbart
* **Velg gradient** → Fargene oppdateres umiddelbart
* **Juster verdivariasjon** → Kontrasten endres i sanntid
* **Endre klasser** → Gradientens jevnhet oppdateres umiddelbart
* **Endre klipping** → Bakgrunnsvisningen endres umiddelbart
* **Rediger farger** → Tilpasset gradient brukes umiddelbart**Ingen «Bruk»-knapp nødvendig** – alle endringer er live og interaktive!

{% hint style="success" %}
**Live-tilbakemelding**: Den umiddelbare visuelle tilbakemeldingen lar deg raskt eksperimentere med ulike innstillinger til du finner den optimale visualiseringen for dine analysebehov.
{% endhint %}

### Iterativ forbedringsprosess

**Typisk LUT-optimaliseringsprosess:**

1.**Velg indekslag** (f.eks. RAW (refleksjon))
2. **Bruk indeks** – Velg kamerafilter og indeksformel, dra fargede sirkler til riktig plassering i indeksformelen
3. **Bruk LUT-gradient** – Start med forhåndsinnstillingen Red-Yellow-Green
4. **Undersøk pikselverdier** – Flytt markøren rundt, legg merke til verdivariasjoner
5. **Juster min/maks** – Begrens for å fokusere på vegetasjon (f.eks. 0,2 til 0,9)
6. **Velg klipping** – Prøv «Original bakgrunn» for kontekst
7. **Finjuster farger** – Tilpass gradienten om nødvendig for spesifikk vektlegging
8. **Fullfør innstillingene**– Dokumenter innstillingene og kopier til prosjektinnstillinger for eksportbehandling

### Inspeksjon av pikselverdier

Det er avgjørende å forstå de faktiske pikselverdiene for å angi effektive LUT-områder:**Slik inspiserer du verdiene:**

1. Pikselverdiene vises når bildet har enten**boksen for indeks**, eller både**boksen for indeks**og**boksen for LUT** merket av.
2. **Flytt markøren** over forskjellige områder av bildet
3. **Observer pikselverdiene** som vises i forklaringen når du holder markøren over dem
4. Zoom inn for å se individuelle piksler uthevet med en flytende verdi
5. **Noter ned** verdier for ulike elementer:
   * **Sunn vegetasjon**: f.eks. NDVI 0,55–0,85
   * **Stresset vegetasjon**: f.eks. NDVI 0,30–0,50
   * **Bare jord**: f.eks. NDVI 0,05–0,25
   * **Vann** (hvis til stede): f.eks. NDVI -0,05 til 0,10**Bruk av pikselverdier til å angi LUT-intervaller:**Etter å ha undersøkt pikselverdiene, justerer du LUT-min/maks tilsvarende:**Eksempelscenario:*** **Observasjon**: Jordverdier = 0,05–0,25, Stresset = 0,25–0,50, Frisk = 0,50–0,85
* **Mål**: Visualiser kun plantens helse (ekskluder jord)
* **LUT-innstillinger**: Min = `0.25`, Maks = `0.85`
* **Klipping**: «Original bakgrunn» for å se jord i naturlig farge
* **Resultat**: Fargegradienten gjelder kun vegetasjon, jord vises som originalbildet

{% hint style="info" %}
**Dynamisk område**: Ulike avlinger, årstider og vekststadier vil ha forskjellige verdier. Kontroller alltid pikselverdiene i ditt spesifikke datasett før du angir LUT-områder.
{% endhint %}

***

## Tilpassede indekser (Chloros+)

### Opprette egendefinerte indeksformler

{% hint style="info" %}
**Hvor skal du opprette**: Egendefinerte indekser kan konfigureres i**Prosjektinnstillinger** før behandling, samt i sidefeltet i Image Viewer-sandkassen.
{% endhint %}

**Slik oppretter du en egendefinert indeks:**

1.**Åpne prosjektinnstillinger** (før behandling) eller sidepanelet i Image Viewer-sandkassen
2. Naviger til **rullegardinmenyen for indeksformler**

3. Se etter alternativet**«Tilpasset»** (du må være logget inn med Chloros+-lisens)
4. **Definer formelen** ved hjelp av båndvariabler:
   * Båndnavn: `NIR`, `Red`, `Green`, `Blue`, `RedEdge`, osv.
   * Operatører: `+`, `-`, `*`, `/`, `^` (eksponent)
   * Funksjoner: `sqrt()`, `abs()`, osv. (hvis støttet)
   * Parenteser: `()` for rekkefølge av operasjoner
5. **Gi indeksen et navn** (f.eks. «MyIndex» eller «CustomNDVI»)
6. **Lagre konfigurasjonen**

**Eksempler på egendefinerte formler:**

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

{% hint style="warning" %}
**Formelvalidering**: Sørg for at formelen din bruker bånd som er tilgjengelige i kameraet ditt. For eksempel er RedEdge bare tilgjengelig på kameraer med et RedEdge-filter.
{% endhint %}

***

## Neste trinn

Nå som du forstår indeks-/LUT-sandkassen:

* **Bruk i behandlingen**: Bruk innstillingene du har funnet i [Prosjektinnstillinger](../project-settings/project-settings.md)
* **Batchbehandling**: Bruk optimaliserte indekser på hele datasett
* **Lær mer**: Les [Multispektrale indeksformler](../project-settings/multispectral-index-formulas.md)

Relatert dokumentasjon:

* [**Bildelag**](image-layers.md) – Lagadministrasjon og visualisering
* [**Åpne et bilde i fullskjerm**](opening-an-image-full-screen.md) – Grunnleggende om bildeviseren
* [**Behandling av bilder (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Fullstendig behandlingsarbeidsflyt
