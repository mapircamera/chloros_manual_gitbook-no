# Index/LUT-sandkasse

Index/LUT-sandkassen er det interaktive arbeidsområdet i sidepanelet i Chloros Image Viewer. Du velger en formel, knytter kameraets kanaler til den, fargelegger den med en gradient og justerer verdibåndet — og bildet oppdateres i sanntid mens du gjør dette. Siden versjon 1.2.0 kan du også **lagre det du har laget**, enten for ett bilde eller for hele prosjektet, uten å måtte behandle det på nytt.

## Hva Sandbox er til

| Index/LUT Sandbox (interaktiv)        | Prosjektbehandling (batch)       |
| -------------------------------------- | -------------------------------- |
| Ett bilde om gangen, umiddelbar tilbakemelding  | Hele datasettet i én omgang     |
| Eksperimentelt og iterativt             | Forhåndskonfigurerte innstillinger          |
| Renderer i sanntid; lagrer kun når du ber om det  | Skriver alltid ut produktfiler      |
| Perfekt for å finne de riktige innstillingene | Best når innstillingene er endelige |

{% hint style="success" %}
**Den vanlige arbeidsflyten**: finjuster i Sandboxen til visualiseringen ser ut slik du ønsker, og eksporter deretter enten direkte fra Sandboxen, eller kopier de samme indeks- og LUT-innstillingene til [Prosjektinnstillinger](../project-settings/project-settings.md), slik at de blir innbakt i hvert bilde ved neste behandlingsomgang.
{% endhint %}

***

## Åpne Sandbox

1. Klikk på et bilde i rutenettet – det åpnes i fullskjerm i **Image Viewer**-fanen <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Klikk på **Image Viewer**-ikonet <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> for å skyve ut venstre sidepanel hvis det ikke allerede er åpent
3. Velg et flerbåndslag fra lagmenyen øverst til høyre — **RAW (Reflectance)** er det vanlige valget, fordi indeksverdier beregnet på kalibrert reflektans er sammenlignbare mellom bilder

Sidepanelet viser, fra topp til bunn:

* bildets navn og kameramodellen
* **Eksporter/Lagre bilde(r)**-knappen — vises når**Indeks**eller**LUT** er merket av
* avmerkingsboksene **Indeks**og**LUT**
* konfigurasjonspanelet for indeks
* **Cursor Values**-panelet med avlesning, histogram og GSD-kontroll

{% hint style="warning" %}
**Ikke tilgjengelig for monokameraer.** På et LATTICE M3M-bilde med ett bånd er begge avmerkingsboksene deaktivert, med verktøytipset _«Ikke tilgjengelig for mono (M3M)-sensorer»_ — en flerbåndsindeks er udefinert på ett bånd. For å beregne indekser fra M3M-kameraer må du kombinere to eller flere til en justert flerbåndsstabel og bruke LATTICE-indeksmotoren.
{% endhint %}

***

## Bruke en indeks

1. Merk av i **Indeks**-boksen øverst i sidepanelet
2. Velg kameraets filter fra rullegardinmenyen til venstre (`RGN`, `OCN`, `NGB`, `RGB`, `RE`, `NIR`)
3. Velg en indeksformel fra rullegardinmenyen til høyre – 27 innebygde formler, pluss eventuelle egendefinerte formler du har lagret
4. Formelen vises som matematisk uttrykk nedenfor, med en tom sirkel ved hvert båndfelt. **Dra en farget kanalsirkel inn på en plass** for å knytte den til formelen
5. Når alle plassene som formelen bruker er knyttet til, oppdateres bildet og viser indeksverdier
6. Flytt markøren over bildet for å lese verdiene; **Markørverdier**-panelet legger til en indeksrad med verdien under markøren

Dobbeltklikk på en tilknyttet plass for å fjerne den. En ufullstendig formel er en normal tilstand mens du drar, ikke en feil — bildet oppdateres ganske enkelt ikke før formelen er fullstendig.

Kanalsirklene er fargekodet: rød = Red, grønn = Green, blå = Blue, oransje = Orange, cyan = Cyan, lilla = NIR, magenta = RE. De samme fargene brukes for kanalprikkene og histogramkurvene i panelet «Cursor Values».

### Eksempel på NDVI

```

Formula: (NIR - Red) / (NIR + Red)

For a Survey3W RGN camera:
  NIR = 850 nm band
  Red = 661 nm band

Result range:          -1.0 to +1.0
Typical vegetation:     0.4 to 0.9
Stressed vegetation:    0.2 to 0.4
Bare soil:              0.0 to 0.2
Water:                 -0.1 to 0.1
```

For den fullstendige formelreferansen – alle tre forhåndsinnstilte lister og hvilke navn som fungerer hvor – se [Multispektrale indeksformler](../project-settings/multispectral-index-formulas.md).

### Med «Indeks» merket av, men uten LUT

Bildet tegnes i **gråtoner**, strukket mellom de to terskelverdiene. Dette er bevisst: indeksbildet er skalardata, og gråtoner er den mest naturtro gjengivelsen av det. Legg til en LUT når du ønsker farge.

***

## Arbeide med LUT-er (oppslagstabeller)

En **oppslagstabell** tilordner indeksverdier til farger: inndata NDVI 0,65, utdata en bestemt grønnfarge. Den endrer ikke dataene – den endrer måten du leser dem på.

### Legge til en LUT

1. Klikk på **«+ Add LUT»**-knappen under formelen <img src="../.gitbook/assets/image (1) (1) (1).png" alt="" data-size="line">
2. Velg en fargegradient
3. Angi minimums- og maksimumsverdier for klipping
4. Velg en klippemodus
5. Merk av i **LUT**-boksen i sidepanelet for å gjengi den

LUT-avmerkingsboksen forblir deaktivert inntil en LUT faktisk er konfigurert på indeksen.

### Velge en fargegradient

Hold musepekeren over **gradientlinjen**for å åpne listen over forhåndsinnstillinger — Chloros leveres med**syv** forhåndsinnstilte gradienter:

| # | Gradient                            | Form                                                               |
| - | ----------------------------------- | ------------------------------------------------------------------- |
| 1 | Red → Gul → Green (**standard**)  | Divergerende — samsvarer med den vanlige oppfatningen av vegetasjon: grønt = sunt |
| 2 | Lilla → Gul → Green             | Divergerende, med en tydelig lav ende                                  |
| 3 | Brun → Hvit → Blue                | Divergerende rundt et lyst midtpunkt                                   |
| 4 | Svart → Lilla → Rosa → Blekgul | Sekvensiell, mørk til lys                                           |
| 5 | Red → Gul → Blue                 | Avvikende rundt et lyst midtpunkt                                   |
| 6 | Lilla → Blue → Green → Gul      | Sekvensiell, mørk til lys                                           |
| 7 | Orange → Hvit → Lilla             | Divrundt et lyst midtpunkt                                   |

En **divergerende**gradient plasserer en nøytral farge i midten av vinduet, noe som fungerer godt når midtpunktet har en bestemt betydning (en terskel, en referansedato). En**sekvensiell** gradient går monotont fra mørkt til lyst, noe som fungerer godt for en mengde som bare har «mer» og «mindre».

Hver forhåndsinnstilling har syv fargestopp. Klikk på en forhåndsinnstilling, så oppdateres bildet umiddelbart (når LUT-boksen er avmerket).

### Redigere fargestoppene

Under gradientlinjen er det en rad med fargeprøver, én per stopp:

* **Endre en farge**: klikk på en fargeprøve for å åpne fargevelgeren (fargehjul, RGB/HSV-glidebrytere eller en heksakode som `#FF0000`)
* **Legg til et stopp**: klikk på**+**-knappen på slutten av raden – et hvitt stopp legges til
* **Fjern et stopp**:**dobbeltklikk** på fargeprøven
* **Behold en redigert gradient**: klikk på lagringsikonet ved siden av gradientlinjen for å legge den redigerte gradienten til listen over forhåndsinnstillinger, slik at du kan velge den igjen

Gradienten du har konfigurert på en indeks, lagres sammen med den indeksen i prosjektinnstillingene, slik at den beholdes når prosjektet lukkes og åpnes på nytt.

**Færre stopp**gir tydelige soner som kan tolkes som en klassifisering;**flere stopp** gir jevne, nesten fotografiske overganger. Tre til fem stopp passer til presentasjonslysbilder og klassifiseringskart; seks til ti passer til generell analyse; femten eller flere passer til detaljert inspeksjon og publiseringsfigurer.

### Innstilling av verdiområdet

Terskelkontrollen er en **glidebryter med to håndtak**som går fra −1 til +1, med et redigerbart tekstfelt i hver ende for eksakte verdier, samt en**AUTO**-knapp.

* Dra i en av håndtakene, eller skriv inn et tall i boksen og trykk Enter
* **AUTO**setter området til**

2. og 98. persentil** av bildets gyldige indeksverdier — et godt utgangspunkt som ignorerer ekstremverdier. Chloros avrunder resultatet adaptivt, til 4 desimaler for et svært smalt område, 3 for et smalt område og 2 ellers
* Enhver manuell justering har forrang fremfor AUTO inntil du trykker på AUTO igjen

Eksempel på NDVI-vinduer:

| Mål                                    | Min  | Maks |
| --------------------------------------- | ---- | --- |
| Vis alt                                 | −1,0 | 1,0 |
| Kun vegetasjon, ekskluder jord og vann | 0,2  | 0,9 |
| Kun sunn vegetasjon                 | 0,5  | 0,9 |
| Fremhev stress                        | 0,2  | 0,5 |

Ved å innsnevre vinduet øker du kontrasten innenfor området du er interessert i, og alt annet havner utenfor området — hvor **klippemodus** avgjør hva som skjer med det.

***

## Klippemodus

Når indeksverdien til en piksel faller utenfor min./maks.-vinduet, avgjør klippemodus hvordan den skal tegnes.

| Nedtrekksmenyets etikett                  | Lagret verdi      | Piksler utenfor området tegnes som                                                                                                |
| ------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Minimum og maksimum** (standard) | `clip`            | Den nærmeste endefargen i gradienten — verdier under minimumet får den første fargen, verdier over maksimumet får den siste |
| **Gjennomsiktig bakgrunn**      | `transparent`     | Fullstendig gjennomsiktig (ekte alfa)                                                                                                  |
| **Indeksbakgrunn**| `indexColor`      | Gråtoner, strukket over bildets**fulle** indeksområde, slik at strukturer utenfor området fremdeles er synlige i grått                |
| **Original bakgrunn**         | `backgroundColor` | Selve det underliggende bildet, slik at fargeoverlegget ligger oppå den virkelige scenen                                                |

| Modus                       | Best egnet for                               | Utseende                                      |
| -------------------------- | -------------------------------------- | ----------------------------------------- |
| **Minimum og maksimum**      | Full datavisning, vitenskapelig analyse | Hver piksel er farget                      |
| **Gjennomsiktig bakgrunn** | GIS-overlegg, isolering av et verdibånd   | Farge inne i vinduet, ingenting utenfor |
| **Indeksbakgrunn**       | Fremheving samtidig som datakonteksten bevares    | Farge inne, grått utenfor               |
| **Original bakgrunn**    | Rapporter og presentasjoner              | Farge inne, fotografi utenfor         |

{% hint style="info" %}
**Piksler uten data er alltid gjennomsiktige, i alle moduser.** En piksel hvis indeks ikke er endelig (en 0/0-divisjon) eller er nøyaktig −1,0 eller +1,0 (metningsvakter, der det ene båndet viser null mens det andre ikke gjør det), behandles som «ingen data» i stedet for som en ekstremverdi. Dette holder utbrente høylys og døde skygger utenfor fargeskalaen din, i stedet for å gjengi dem som de mest ekstreme verdiene i bildet. Den samme regelen definerer hvilke piksler som brukes i AUTO-tersklene og indeks-histogrammet, slik at alle tre stemmer overens.
{% endhint %}

Gjennomsiktigheten bevares når eksporten lagres som PNG. Den kan ikke gjengis i JPG.

***

## Avlesning av verdier mens du justerer**Cursor Values**-panelet under konfigurasjonspanelet er måleinstrumentet for Sandbox:

* Flytt markøren over bildet og avles kildeverdiene per kanal, samt indeksverdien i sin egen rad
* Slå på **INDEX**-knappen over histogrammet for å se fordelingen av indeksverdiene i bildet, med de to klippeterskelverdiene tegnet som oransje stiplede linjer og markørens verdi som en hvit linje — dette er den raskeste måten å velge et vindu som faktisk inneholder dataene dine
* Slå på **CURSOR** for å se markørlinjer ved verdiene under pekeren
* Zoom inn over 60× (mindre hvis en GSD-blokkstørrelse er angitt) for å markere enkeltvis viste piksler med en flytende verdi

En praktisk fremgangsmåte:

1. Noter verdiene over frisk vegetasjon, stresset vegetasjon, bar jord og vann
2. Se hvor disse klyngene befinner seg på indeks-histogrammet
3. Angi min./maks.-verdier for å avgrense klyngen du er interessert i
4. Velg en beskjæringsmodus — _Original Background_ holder scenen synlig rundt den

***

## Eksportere fra Sandbox

Alt ovenfor er en live-forhåndsvisning inntil du lagrer det. Knappen **Eksporter/Lagre bilde(r)** øverst i sidefeltet åpner et vindu som glir over sidefeltet (i stedet for å dekke bildet, slik at du fortsatt kan se hva du tar stilling til).

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>### Alternativer

| Alternativ                          | Effekt                                                                                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Bruk på gjeldende bilde**      | Lagrer nøyaktig det viste bildet med disse innstillingene                                                                                                |
| **Bruk på alle prosjektbilder** | Kjører den samme konfigurasjonen på nytt på hvert bilde i prosjektet. Bilder uten de båndene denne indeksen trenger, hoppes over og behandles ikke som feil |
| **Indeks-/LUT-gradientlinje**      | Skriver også et eget forklaringsbilde per eksport, med verdivariasjonen merket                                                                     |
| **Indekshistogram**             | Skriver også et eget histogrambilde per eksport, som viser dataenes min./maks.-verdier og klippeterskelverdiene                                               |

Hvis **GSD-blokkstørrelsen** på bildefanen er over 1, vil ruten vise dette før du bekrefter: eksporten lagrer det du ser, inkludert blokk-gjennomsnitt. Sett GSD-kontrollen tilbake til 1 først hvis du ønsker full oppløsning.

### Hvor filene lagres

Hvert klikk på **Eksporter**tildeler en**ny mappe som aldri gjenbrukes**:

```
<project folder>/Sandbox_Exports/<IndexName>_<Index|LUT>_<NNN>/
```

Eksempler: `Sandbox_Exports/NDVI_LUT_001/`, deretter `Sandbox_Exports/NDVI_LUT_002/` for neste kjøring. Nummereringen genereres ved å skanne innholdet på disken, slik at den overlever omstarter og mapper du sletter manuelt. Ingenting blir noensinne overskrevet — hele poenget med Sandbox er å sammenligne ett forsøk med det forrige.

Inne i mappen, per bilde:

| Fil                                                   | Innhold                                                   |
| ------------------------------------------------------ | ---------------------------------------------------------- |
| `<source name>_<IndexName>_<Index\|LUT>.png`           | Det renderte bildet, piksel for piksel slik det ble vist i visningsprogrammet |
| `<source name>_<IndexName>_<Index\|LUT>_legend.png`    | Sidecar-filen med gradientstangen, hvis forespurt                     |
| `<source name>_<IndexName>_<Index\|LUT>_histogram.png` | Sidecar-filen med indeks-histogrammet, hvis forespurt                  |

De to sidecar-filene skrives alltid i **full oppløsning**, selv når hovedbildet er blokk-gjennomsnittsberegnet: en blokkstørrelse tilsvarer skjermoppløsningen, og begge sidecar-filene leser de faktiske indeksverdiene per piksel. De viser også mer enn versjonene på skjermen – begge angir strekkvinduet _og_ de faktiske minimums- og maksimumsverdiene for dataene, slik at en lagret forklaring fortsatt er lesbar måneder senere uten at prosjektet er åpent.

### Fremdrift og resultater

En eksport av hele prosjektet tar noen minutter, så kjøringen rapporterer tilbake via en live fremdriftskanal i stedet for å blokkere:

* En fremdriftslinje viser `current / total` og filen som skrives
* Når den er ferdig, viser ruten hvor mange bilder som ble eksportert, hvor mange som ble hoppet over, og stien til utdatamappen
* Bilder som ble hoppet over, vises med årsaken (opptil fem vises, deretter en «+N flere»-linje). Den vanligste årsaken er et lag som ikke har de kanalene denne indeksen trenger
* Hvis **ingen** bilder i prosjektet kan bruke indeksen, rapporterer kjøringen feil i stedet for å etterlate en tom mappe

Det kan kun kjøre én sandkasseeksport om gangen. Å starte en ny mens en pågår, avvises med en tydelig melding i stedet for å la to kjøringer konkurrere om den samme prosjektfilen.

### Rutenettet henter kjøringen

Hver fullført kjøring vises som en egen knapp i [bildegitteret](image-grid.md) på verktøylinjen, merket `<IndexName> <Index|LUT> <NNN>`. Slik sammenligner du kjøringer: eksporter to ganger med forskjellige gradienter eller terskler, og veksle deretter mellom de to knappene i rutenettet.

***

## Egendefinerte indeksformler (Chloros+)

{% hint style="info" %}
**Hvor du oppretter dem**: i sidepanelet i Sandbox, eller i**Prosjektinnstillinger** før behandling. Begge skriver til den samme listen på prosjektnivå.
{% endhint %}

1. Åpne kalkulatoren for egendefinerte formler fra rullegardinmenyen for indeksformler (krever pålogging med et kvalifiserende Chloros+-abonnement)
2. Skriv formelen ved hjelp av **bånd- og spor-symbolene** `x`, `y`, `z`, `a`, `b`, `c` — ikke båndnavn
3. Tilgjengelige operatører: `+`, `-`, `*`, `/`, `^` og `()` for gruppering
4. Tilgjengelige funksjoner: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
5. Gi den et navn og lagre den – den vises nederst i formelmenyen, og du tilordner kanalene ved å dra kanal-sirklene, akkurat som med en innebygd forhåndsinnstilling

```

Modified NDVI with an offset:   (y-x)/(y+x+0.5)
Simple ratio:                   y/x
Three-band difference:          (y-x)/(y+x-z)
Squared ratio:                  (y/x)^2
```

{% hint style="warning" %}
**Egendefinerte formler er kun tilgjengelige i GUI.** Alternativet CLI/SDK `--indices` utvider de 22 innebygde forhåndsinnstillingene og hopper automatisk over alt annet, inkludert dine egendefinerte formler. For å behandle en tilpasset formel i batch, må du konfigurere den i prosjektinnstillingene og kjøre behandlingen, eller bruke eksportfunksjonen «Bruk på alle prosjektbilder» i Sandbox.
{% endhint %}

***

## Feilsøking

### «Dette laget har ikke kanalene som denne indeksen trenger»

Formelen leser en kanalposisjon som det gjeldende laget ikke har — for eksempel en indeks med tre spor på en fil med én eller to kanaler. Bytt til et flerbåndslag (reflektans eller debayert), eller velg en indeks som passer til kameraets filter.

### «Kunne ikke nå bakenden for bildebehandling»

Bakenden svarer ikke. Sjekk fanen «Log»; hvis backend-en starter på nytt, gjenoppretter Sandbox seg selv når den er tilbake.

### Bildet endret seg ikke da jeg dro en sirkel

Formelen er ikke fullført ennå. En ufullstendig formel behandles som en normal tilstand midt i en dra-bevegelse – ingenting renderes, og ingenting rapporteres som en feil. Fyll ut alle feltene som formelen bruker.

### Hele bildet har én farge

Klippvinduet ditt ligger sannsynligvis langt utenfor dataene. Trykk på **AUTO**for å justere det til 2.- eller 98.-persentilen, eller slå på**INDEX**-histogrammet for å se hvor dataene faktisk ligger.

### Eksporterte farger stemmer ikke overens med det jeg så

Det burde de – eksportbanen er en bevisst gjenspeiling av live-forhåndsvisningen, inkludert alfa i klippemodus, og blokkgjennomsnittet blir brukt _etter_ fargeleggingen, akkurat slik visningsprogrammet gjør det. Hvis de avviker, må du sjekke at GSD-blokkstørrelsen ikke har endret seg mellom visning og eksport.

***

## Neste trinn

* [**Bildelag**](image-layers.md) — hvilket lag indeksen skal kjøres på, og hva verdiene betyr
* [**Åpne et bilde i fullskjerm**](opening-an-image-full-screen.md) — detaljert beskrivelse av markørvisning, histogram og GSD-kontroll
* [**Formler for multispektrale indekser**](../project-settings/multispectral-index-formulas.md) — alle forhåndsinnstillinger, på alle overflater
* [**Prosjektinnstillinger**](../project-settings/project-settings.md) — å lagre innstillingene du har funnet i en behandlingskjøring
