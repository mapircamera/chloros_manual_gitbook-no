---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Formler for multispektrale indekser

Indeksformlene nedenfor bruker en kombinasjon av gjennomsnittlige transmisjonsområder for Survey3-filteret:

<table><thead><tr><th align="center">Survey3-filterfarge</th><th width="196.199951171875" align="center">Survey3 Filternavn</th><th width="159.800048828125" align="center">Transmisjonsområde (FWHM)</th><th align="center">Gjennomsnittlig transmisjon</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB – Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN – Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN – Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN – Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN – NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR – NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>Når disse formlene brukes, kan navnet ende på «\_1» eller «\_2», noe som tilsvarer hvilket NIR-filter enten NIR1 eller NIR2 ble brukt.

For LATTICE M3C-kameraer (Bayer trippelbåndpass) bruker den samme indeksmotoren M3C-filterbåndene:

| M3C-filter | Bånd 1 (sentrum/FWHM) | Bånd 2 (sentrum/FWHM) | Bånd 3 (sentrum/FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

LATTICE M3M-kameraer er enkeltbånds (ett smalbåndsfilter per kamera), så flerbåndsindekser beregnes ikke for et enkelt M3M-bilde. For å beregne indekser med M3M må du kombinere to eller flere kameraer til en justert flerbåndsstabel og bruke LATTICE-indeksmotoren (`chloros-cli lattice index`, eller GUI-ens live indekskalkulator).

***

## Hvor hvert indeksnavn fungerer

Chloros har **tre** indeksflater, og deres forhåndsinnstilte lister er ikke identiske. Bruk denne delen til å sjekke om et navn vil fungere der du planlegger å bruke det.

| Hvor du befinner deg | Hvilken liste gjelder | Antall |
| --- | --- | --- |
| Prosjektinnstillinger → Indeks → Legg til indeks (GUI) | Flate 1 | 27 |
| Bildeviser [Indeks/LUT-sandkasse](../image-viewer-gui/index-lut-sandbox.md) (GUI) | Flate 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | Flate 2 | 22 |
| SDK `process_folder(indices=[...])` | Overflate 2 | 22 |
| `chloros-cli lattice index --preset` | Overflate 3 | 22 (et annet 22) |
| Fanen «Kameraer» – Live Index Calculator | Overflate 3 | 22 (et annet 22) |

Overflatene 1 og 2 behandler **ett bilde om gangen fra ett kamera**, ved hjelp av symbolplassene `x`/`y`/`z`(/`a`) knyttet til det aktuelle kameraets filterkanaler. Overflate 3 behandler en**justert flerbåndsstabel** — flere LATTICE-kameraer samregistrert i én kube — og refererer til kanalene ved hjelp av navn med små bokstaver.

### 1. GUI-prosjektinnstillinger / Bildesviserens «sandbox»-rullegardinmeny — 27 formler

Rullegardinmenyen viser dem i denne rekkefølgen (det er innsettingsrekkefølgen, ikke alfabetisk):

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

I GUI-en drar du kameraets filterkanaler inn i formelens båndplasser, slik at enhver formel kan brukes med hvilken som helst båndtilordning kameraet ditt støtter. Egendefinerte formler du har lagret, legges til under denne listen.

De **fem GUI-eksklusive** formlene – de som listen CLI/SDK `--indices` ikke godtar – er implementert som:

| Forhåndsinnstilling kun i GUI | Formel (som implementert) | Plasser |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a (fire plasser) |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

Den tiltenkte tilordningen for hver av dem er angitt i sin egen seksjon lenger ned på denne siden (for eksempel forventer GARI at x=Green, y=NIR, z=Blue, a=Red). GARI er den eneste formelen i Chloros som bruker et fjerde felt.

### 2. CLI / SDK `--indices` navnutvidelse — 22 forhåndsinnstillinger

Alternativet `chloros-cli process --indices` (og parameteren SDK `indices`) godtar følgende forhåndsinnstilte navn:

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**Ukjente indeksnavn hoppes over uten varsel.** Et navn utenfor denne listen (inkludert de fem formlene som kun finnes i GUI: `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` og eventuelle egendefinerte formler du har lagret i GUI-en) blir utelatt med kun en loggmelding — kjøringen fortsetter uten den indeksen, og selve kjøringen rapporteres fortsatt som vellykket. Meldingen skrives ut som:

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

Navnene sammenlignes uten å skille mellom store og små bokstaver etter at mellomrom er fjernet, så `ndvi`, `NDVI` og ` NDVI ` er den samme forhåndsinnstillingen. En forhåndsinnstilling hoppes også over hvis den krever et bånd som kameraets filter ikke støtter.
{% endhint %}

De eksakte formlene slik de er implementert (symbolene `x`/`y`/`z` er båndplasser; standardtilordningen vises per forhåndsinnstilling):

| Forhåndsinnstilling | Formel (slik den er implementert) | Standardfilter | Plasser (x, y, z) |
| --- | --- | --- | --- |
| NDVI | `(y-x)/(y+x)` | RGN | Red, NIR |
| GNDVI | `(y-x)/(y+x)` | RGN | Green, NIR |
| NDRE | `(y-x)/(y+x)` | RE | RE, NIR |
| OSAVI | `(y-x)/(y+x+0.16)` | RGN | Red, NIR |
| SAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Red, NIR |
| MSAVI2 | `(2*y+1-sqrt((2*y+1)*(2*y+1)-8*(y-x)))/2` | RGN | Red, NIR |
| EVI | `2.5*(y-x)/(y+6*x-7.5*z+1)` | RGN | Red, NIR, Blue |
| MSR | `((y/x)-1)/(sqrt(y/x)+1)` | RGN | Red, NIR |
| TDVI | `1.5*(y-x)/sqrt(y*y+x+0.5)` | RGN | Red, NIR |
| LAI | `3.618*(2.5*(y-x)/(y+6*x-7.5*z+1))-0.118` | RGN | Red, NIR, Blue |
| GCI | `(y/x)-1` | RGN | Green, NIR |
| GRVI | `y/x` | RGN | Green, NIR |
| GSAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Green, NIR |
| GOSAVI | `(y-x)/(y+x+0.16)` | RGN | Green, NIR |
| NLI | `((y*y)-x)/((y*y)+x)` | RGN | Red, NIR |
| MNLI | `((y*y-x)*(1+0.5))/((y*y)+x+0.5)` | RGN | Red, NIR |
| RDVI | `(y-x)/sqrt(y+x)` | RGN | Red, NIR |
| WDRVI | `(0.2*y-x)/(0.2*y+x)` | RGN | Red, NIR |
| CVI | `(z/y)/(x/y)` | RGB | Red, Green, Blue |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | RGB | Red, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red, Green, Blue |

#### Hvordan et forhåndsinnstilt navn blir til båndposisjoner

Når du angir et navn uten ytterligere spesifikasjoner, for eksempel `NDVI`, må Chloros avgjøre hvilken kanal i hvilken fil hvert symbol leser fra. Det bruker denne tabellen, som tilordner en filterkode til matriseplasseringen for hver kanal:

| Filterkode | Kanal → matriseindeks |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (`Red` godtas som et alias for Orange, også 0) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

Forhåndsinnstillingens **standardfilter** (kolonnen «Standardfilter» ovenfor) brukes når prosjektet inneholder bilder med det filteret. Hvis det ikke gjør det, skanner Chloros gjennom filtrene som faktisk finnes i prosjektet i rekkefølgen `RGN, OCN, NGB, RGB, RE, NIR` og velger det første som kan levere alle kanalene forhåndsinnstillingen trenger. Hvis ingen kan det, blir forhåndsinnstillingen utelatt for den aktuelle kjøringen. Dette er grunnen til at `NDVI`, som ble forespurt på et OCN-datasett fremdeles gir et fornuftig resultat — den knytter seg til OCNs posisjoner Orange og NIR.

LATTICE M3C-modellstrenger inneholder filteret med et `F`-prefiks (`LATT-M3C-L41-FRGN`), men prefikset fjernes når filterkoden leses av bildet, så et FRGN-kamera løser opp gjennom raden `RGN` ovenfor og trenger ingen spesiell håndtering.

### 3. LATTICE-indeksmotor (`lattice index --preset`, live indekskalkulator) — 22 forhåndsinnstillinger

LATTICE-motoren fungerer på justerte flerbåndsstabler (live-matriser eller eksporterte flerbånds-TIFF-filer) og bruker kanalnavn med små bokstaver (`red`, `green`, `blue`, `red_edge`, `nir`). Listen over forhåndsinnstillinger skiller seg fra de to ovennevnte:

| Forhåndsinnstilling | Formel | Kanaler |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | rød, nir |
| GNDVI | `(nir - green) / (nir + green)` | grønn, nir |
| BNDVI | `(nir - blue) / (nir + blue)` | blå, nir |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | rød\_kant, nir |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | blå, grønn, nir |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | rød, nir |
| OSAVI | `1.5 * (nir - red) / (nir + red + 0.16)` | rød, NIR |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | rød, NIR |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | blå, rød, nir |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | rød, nir |
| CVI | `(nir / green) - (red / green)` | rød, grønn, nir |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | rød, nir |
| TDVI | `sqrt((nir - red) / (nir + red) + 0.5)` | rød, NIR |
| LAI | `3.618 * ((nir - red) / (nir + 6*red - 7.5*green + 1)) - 0.118` | rød, grønn, NIR |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | rød, grønn, blå |
| NGRDI | `(green - red) / (green + red)` | rød, grønn |
| VARI | `(green - red) / (green + red - blue)` | rød, grønn, blå |
| TGI | `green - 0.39*red - 0.61*blue` | rød, grønn, blå |
| EXG | `2*green - red - blue` | rød, grønn, blå |
| CIRE | `(nir / red_edge) - 1` | rød\_kant, nir |
| CIGREEN | `(nir / green) - 1` | grønn, nir |
| NDWI | `(green - nir) / (green + nir)` | grønn, nir |

Kjør `chloros-cli lattice index --list-presets` for å skrive ut denne tabellen fra den installerte versjonen, og `--list-gradients` for de tilgjengelige fargegradientene. Kanalsymbolene er store og små bokstaver, og må samsvare med forhåndsinnstillingens navn med små bokstaver (f.eks. `--channel red=Red_660 --channel nir=NIR_850`).

***

## CVI

Som implementert i GUI og CLI/SDK-forhåndsinnstillingslisten, er CVI formelen for forholdet mellom forhold:

$$
CVI = {(z / y) \over (x / y)}
$$

med standard kanaltilordning RGB x=Red, y=Green, z=Blue. I brukergrensesnittet kan du dra hvilken som helst av kameraets kanaler inn i x/y/z-plassene. Merk at LATTICE-indeksmotorens forhåndsinnstilling `CVI` bruker en annen formel, nemlig `(NIR / Green) - (Red / Green)` — sjekk tabellene ovenfor for overflaten du bruker.

***

## ENDVI – Enhanced Normalized Difference Vegetation Index

Denne indeksen bruker den blå kanalen i tillegg til NIR og den grønne, og er populær blant NGB-filtrerte kameraer der det blå båndet erstatter det røde.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

Implementeringen er symbolformelen `((x+y)-(2*z))/((x+y)+(2*z))` — tilordne kameraets NIR- og Green-kanaler til x/y-plassene og Blue til z (for et NGB-kamera: x=NIR, y=Green, z=Blue).

***

## EVI – Forbedret vegetasjonsindeks

Denne indeksen ble opprinnelig utviklet for bruk med MODIS-data som en forbedring i forhold til NDVI ved å optimalisere vegetasjonssignalet i områder med høy bladarealindeks (LAI). Den er mest nyttig i regioner med høy LAI, der NDVI kan bli mettet. Den bruker det blå refleksjonsområdet til å korrigere for bakgrunnssignaler fra jord og til å redusere atmosfæriske påvirkninger, inkludert aerosolspredning.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

EVI-verdiene bør ligge mellom 0 og 1 for vegetasjonspiksler. Lyse elementer som skyer og hvite bygninger, sammen med mørke elementer som vann, kan føre til unormale pikselverdier i et EVI-bilde. Før du oppretter et EVI-bilde, bør du maskere ut skyer og lyse elementer fra refleksjonsbildet, og eventuelt sette en terskel for pikselverdiene fra 0 til 1.

_Referanse: Huete, A., et al. «Overview of the Radiometric and Biophysical Performance of the MODIS Vegetation Indices.» Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 – Skogdekkeindeks 1

_Kun GUI — ikke tilgjengelig som forhåndsinnstilling for CLI/SDK `--indices`._

Denne indeksen skiller skogtak fra andre vegetasjonstyper ved hjelp av multispektrale refleksjonsbilder som inkluderer et «red edge»-bånd.

$$
FCI1 = Red * RedEdge
$$

Skogkledde områder vil ha lavere FCI1-verdier på grunn av trærnes lavere reflektans og tilstedeværelsen av skygger i trekronene.

_Referanse: Becker, Sarah J., Craig S.T. Daughtry og Andrew L. Russ. «Robust forest cover indices for multispectral images.» Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## FCI2 – Skogdekkeindeks 2

_Kun GUI – ikke tilgjengelig som forhåndsinnstilling for CLI/SDK `--indices`._

Denne indeksen skiller skogtak fra andre typer vegetasjon ved hjelp av multispektrale refleksjonsbilder som ikke inkluderer et «red edge»-bånd.

$$
FCI2 = Red * NIR
$$

Skogkledde områder vil ha lavere FCI2-verdier på grunn av trærnes lavere reflektans og tilstedeværelsen av skygger i trekronene.

_Referanse: Becker, Sarah J., Craig S.T. Daughtry og Andrew L. Russ. «Robust forest cover indices for multispectral images.» Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## GEMI – Global Environmental Monitoring Index

_Kun GUI – ikke tilgjengelig som forhåndsinnstilling for CLI/SDK eller `--indices`._

Dette ikke-lineære vegetasjonsindekset brukes til global miljøovervåking basert på satellittbilder og forsøker å korrigere for atmosfæriske effekter. Det ligner på NDVI, men er mindre følsomt for atmosfæriske effekter. Det påvirkes av bar jord; derfor anbefales det ikke brukt i områder med sparsom eller moderat tett vegetasjon.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Hvor:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Referanse: Pinty, B., og M. Verstraete. GEMI: et ikke-lineært indeks for å overvåke global vegetasjon fra satellitter. Vegetation 101 (1992): 15–20._

***

## GARI – Green Atmosfærisk motstandsdyktig indeks

_Kun GUI — ikke tilgjengelig som forhåndsinnstilling for CLI/SDK `--indices`._

Denne indeksen er mer følsom for et bredt spekter av klorofyllkonsentrasjoner og mindre følsom for atmosfæriske effekter enn NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Gammakonsanten er en vektingsfunksjon som avhenger av aerosolforholdene i atmosfæren. ENVI bruker verdien 1,7, som er den anbefalte verdien fra Gitelson, Kaufman og Merzylak (1996, side 296).

_Referanse: Gitelson, A., Y. Kaufman og M. Merzylak. «Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS.» Remote Sensing of Environment 58 (1996): 289–298._

***

## GCI – Green Klorofyllindeks

Denne indeksen brukes til å estimere klorofyllinnholdet i bladene hos et bredt spekter av plantearter.

$$
GCI = {NIR \over Green} - 1
$$

Bruk av brede NIR- og grønne bølgelengder gir en bedre prediksjon av klorofyllinnholdet, samtidig som det gir større følsomhet og et høyere signal-støy-forhold.

_Referanse: Gitelson, A., Y. Gritz og M. Merzlyak. «Sammenhenger mellom klorofyllinnholdet i bladene og spektral reflektans, samt algoritmer for ikke-destruktiv klorofyllmåling i bladene til høyere planter.» Journal of Plant Physiology 160 (2003): 271–282._

***

## GLI – Green Bladindeks

Denne indeksen ble opprinnelig utviklet for bruk med et digitalt RGB-kamera for å måle hvete-dekke, der de røde, grønne og blå digitale tallene (DN-er) varierer fra 0 til 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLI-verdiene varierer fra -1 til +1. Negative verdier representerer jord og ikke-levende elementer, mens positive verdier representerer grønne blader og stengler.

_Referanse: Louhaichi, M., M. Borman og D. Johnson. «Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat.» Geocarto International 16, nr. 1 (2001): 65–70._

***

## GNDVI – Green Normalized Difference Vegetation Index

Dette indekset ligner på NDVI, bortsett fra at det måler det grønne spektrumet fra 540 til 570 nm i stedet for det røde spektrumet. Dette indekset er mer følsomt for klorofyllkonsentrasjonen enn NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Referanse: Gitelson, A., og M. Merzlyak. «Remote Sensing of Chlorophyll Concentration in Higher Plant Leaves.» Advances in Space Research 22 (1998): 689–692._

***

## GOSAVI – Green Optimalisert jordjustert vegetasjonsindeks

Denne indeksen ble opprinnelig utviklet med farge-infrarød fotografering for å forutsi nitrogenbehovet for mais. Den ligner på OSAVI, men erstatter det grønne båndet med det røde.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Referanse: Sripada, R., et al. «Determining In-Season Nitrogen Requirements for Corn Using Aerial Color-Infrared Photography.» Doktoravhandling, North Carolina State University, 2005._

***

## GRVI – Green-forholdet vegetasjonsindeks

Denne indeksen er følsom for fotosyntesehastigheten i skogkronene, da refleksjonsgraden for grønt og rødt påvirkes sterkt av endringer i bladpigmentene.

$$
GRVI = {NIR \over Green }
$$

_Referanse: Sripada, R., et al. «Luftfotografering med farge-infrarødt for å bestemme nitrogenbehovet hos mais tidlig i sesongen.» Agronomy Journal 98 (2006): 968–977._

***

## GSAVI – Green Jordjustert vegetasjonsindeks

Denne indeksen ble opprinnelig utviklet med farge-infrarødt fotografi for å forutsi nitrogenbehovet for mais. Den ligner på SAVI, men erstatter det grønne båndet med det røde.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Referanse: Sripada, R., et al. «Determining In-Season Nitrogen Requirements for Corn Using Aerial Color-Infrared Photography.» Doktoravhandling, North Carolina State University, 2005._

***

## LAI – Bladarealindeks

Dette indekset brukes til å estimere løvdekke og til å forutsi avlingens vekst og avkastning. ENVI beregner det grønne LAI ved hjelp av følgende empiriske formel fra Boegh et al. (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Der EVI er:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Høye LAI-verdier ligger vanligvis mellom omtrent 0 og 3,5. Når motivet imidlertid inneholder skyer og andre lyse elementer som gir mettede piksler, kan LAI-verdiene overstige 3,5. Ideelt sett bør du maskere bort skyer og lyse elementer fra scenen før du oppretter et LAI-bilde.

_Referanse: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde og A. Thomsen. «Airborne Multi-spectral Data for Quantifying Leaf Area Index, Nitrogen Concentration and Photosynthetic Efficiency in Agriculture.» Remote Sensing of Environment 81, nr. 2–3 (2002): 179–193._

***

## LCI - Bladklorofyllindeks

_Kun GUI — ikke tilgjengelig som forhåndsinnstilling for CLI/SDK `--indices`._

Denne indeksen brukes til å estimere klorofyllinnholdet i høyere planter, og er følsom for variasjoner i refleksjonsgrad forårsaket av klorofyllabsorpsjon.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Referanse: Datt, B. «Remote Sensing of Water Content in Eucalyptus Leaves.» Journal of Plant Physiology 154, nr. 1 (1999): 30–36._

***

## MNLI – Modifisert ikke-lineær indeks

Denne indeksen er en videreutvikling av den ikke-lineære indeksen (NLI) som innlemmer den jordjusterte vegetasjonsindeksen (SAVI) for å ta hensyn til jordbakgrunnen. ENVI bruker en justeringsfaktor for kronetakets bakgrunn (_L_) på 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Referanse: Yang, Z., P. Willis og R. Mueller. «Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy.» Proceedings of the Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 – Modifisert jordjustert vegetasjonsindeks 2

Denne indeksen er en enklere versjon av indeksen MSAVI foreslått av Qi m.fl. (1994), som er en forbedring av jordjustert vegetasjonsindeks (SAVI). Den reduserer jordstøy og øker det dynamiske området for vegetasjonssignalet. MSAVI2 er basert på en induktiv metode som ikke bruker en konstant _L_-verdi (som i SAVI) for å fremheve sunn vegetasjon.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Referanse: Qi, J., A. Chehbouni, A. Huete, Y. Kerr og S. Sorooshian. «A Modified Soil Adjusted Vegetation Index.» Remote Sensing of Environment 48 (1994): 119–126._

***

## MSR – Modifisert enkeltforhold

Denne indeksen er en modifisering av det enkle forholdet NIR/Red, utformet for å linearisere forholdet til biofysiske parametere, og er mer følsom enn NDVI ved høyere vegetasjonstetthet.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_Referanse: Chen, J. «Evaluation of Vegetation Indices and a Modified Simple Ratio for Boreal Applications.» Canadian Journal of Remote Sensing 22 (1996): 229–242._

***

## NDRE – Normaliserte forskjeller RedEdge

Denne indeksen ligner på NDVI, men sammenligner kontrasten mellom NIR og RedEdge i stedet for Red, som ofte oppdager vegetasjonsstress tidligere.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – Normalized Difference Vegetation Index

Dette indekset er et mål på sunn, grønn vegetasjon. Kombinasjonen av den normaliserte differanseformuleringen og bruken av klorofyllens områder med høyest absorpsjon og refleksjonsevne gjør det robust under et bredt spekter av forhold. Det kan imidlertid bli mettet i tett vegetasjon når LAI blir høyt.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Verdien av denne indeksen varierer fra -1 til 1. Det vanlige området for grønn vegetasjon er 0,2 til 0,8.

_Referanse: Rouse, J., R. Haas, J. Schell og D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. Third ERTS Symposium, NASA (1973): 309–317._

***

## NLI – Ikke-lineær indeks

Denne indeksen forutsetter at forholdet mellom mange vegetasjonsindekser og biofysiske overflateparametere er ikke-lineært. Den lineariserer forhold til overflateparametere som har en tendens til å være ikke-lineære.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Referanse: Goel, N., og W. Qin. «Influences of Canopy Architecture on Relationships Between Various Vegetation Indices and LAI and Fpar: A Computer Simulation.» Remote Sensing Reviews 10 (1994): 309–347._

***

## OSAVI – Optimalisert jordjustert vegetasjonsindeks

Denne indeksen er basert på den jordjusterte vegetasjonsindeksen (SAVI). Den bruker en standardverdi på 0,16 for justeringsfaktoren for bakgrunn i trekronetaket. Rondeaux (1996) fastslo at denne verdien gir større variasjon i jordforholdene enn SAVI ved lav vegetasjonsdekning, samtidig som den viser økt følsomhet for vegetasjonsdekning på over 50 %. Denne indeksen egner seg best i områder med relativt sparsom vegetasjon der jorden er synlig gjennom trekronene.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Referanse: Rondeaux, G., M. Steven og F. Baret. «Optimization of Soil-Adjusted Vegetation Indices.» Remote Sensing of Environment 55 (1996): 95–107._

***

## RDVI – Renormalized Difference Vegetation Index

Dette indekset bruker forskjellen mellom bølgelengder i nær-infrarødt og rødt, sammen med NDVI, for å fremheve sunn vegetasjon. Det er ufølsomt for effektene av jord og solens innfallsvinkel.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Referanse: Roujean, J., og F. Breon. «Estimating PAR Absorbed by Vegetation from Bidirectional Reflectance Measurements.» Remote Sensing of Environment 51 (1995): 375–384._

***

## SAVI – Jordjustert vegetasjonsindeks

Denne indeksen ligner på NDVI, men den demper effekten av jordpiksler. Den bruker en justeringsfaktor for bakgrunn i trekronetaket, _L_, som er en funksjon av vegetasjonstettheten og ofte krever forhåndskunnskap om vegetasjonsmengden. Huete (1988) foreslår en optimal verdi på _L_=0,5 for å ta hensyn til førsteordens variasjoner i jordbakgrunnen. Denne indeksen brukes best i områder med relativt sparsom vegetasjon der jorden er synlig gjennom kronetaket.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Referanse: Huete, A. «A Soil-Adjusted Vegetation Index (SAVI).» Remote Sensing of Environment 25 (1988): 295–309._

***

## TDVI – Transformed Difference Vegetation Index

Denne indeksen er nyttig for overvåking av vegetasjonsdekket i urbane miljøer. Den blir ikke mettet på samme måte som NDVI og SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Referanse: Bannari, A., H. Asalhi og P. Teillet. «Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping» i Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, bind 5 (2002)._

***

## VARI - Indeks for synlig, atmosfærisk motstandsdyktig vegetasjon

Denne indeksen er basert på ARVI og brukes til å estimere andelen vegetasjon i et bilde med lav følsomhet for atmosfæriske effekter.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Referanse: Gitelson, A., et al. «Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction.» International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – Vegetasjonsindeks med bredt dynamisk område

Denne indeksen ligner på NDVI, men den bruker en vektingskoeffisient (_a_) for å redusere forskjellen mellom bidragene fra signalene i det nær-infrarøde og det røde spektrumet til NDVI. WDRVI er særlig effektiv i områder med moderat til høy vegetasjonstetthet når NDVI overstiger 0,6. NDVI har en tendens til å flate ut når vegetasjonsandelen og bladarealindeksen (LAI) øker, mens WDRVI er mer følsom overfor et bredere spekter av vegetasjonsandeler og overfor endringer i LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Vektingskoeffisienten (_a_) kan variere fra 0,1 til 0,2. Henebry, Viña og Gitelson (2004) anbefaler en verdi på 0,2.

_Referanser_

_Gitelson, A. «Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.» Journal of Plant Physiology 161, nr. 2 (2004): 165–173._

_Henebry, G., A. Viña og A. Gitelson. «The Wide Dynamic Range Vegetation Index and its Potential Utility for Gap Analysis.» Gap Analysis Bulletin 12: 50–56._
