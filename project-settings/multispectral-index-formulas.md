---
description: This page lists some multispectral indices that Chloros uses.
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---
# Multispektrale indeksformler

Indeksformlene nedenfor bruker en kombinasjon av Survey3-filterets gjennomsnittlige transmisjonsområder:

<table><thead><tr><th align="center">Survey3-filterfarge</th><th width="196.199951171875" align="center">Survey3-filternavn</th><th width="159.800048828125" align="center">Transmisjonsområde (FWHM)</th><th align="center">Gjennomsnittlig transmisjon</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476-512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865 nm</td><td align="center">850 nm</td></tr></tbody></table>

Når disse formlene brukes, kan navnet ende på «\_1» eller «\_2», som tilsvarer hvilket NIR-filter, enten NIR1 eller NIR2, som ble brukt.

***

## EVI – Forbedret vegetasjonsindeks

Denne indeksen ble opprinnelig utviklet for bruk med MODIS-data som en forbedring av NDVI ved å optimalisere vegetasjonssignalet i områder med høy bladarealindeks (LAI). Den er mest nyttig i regioner med høy LAI hvor NDVI kan bli mettet. Den bruker det blå refleksjonsområdet til å korrigere for bakgrunnssignaler fra jorda og redusere atmosfæriske påvirkninger, inkludert aerosolspredning.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

EVI-verdiene bør ligge mellom 0 og 1 for vegetasjonspiksler. Lyse elementer som skyer og hvite bygninger, sammen med mørke elementer som vann, kan føre til unormale pikselverdier i et EVI-bilde. Før du oppretter et EVI-bilde, bør du maskere ut skyer og lyse elementer fra refleksjonsbildet, og eventuelt sette terskelverdien for pikselverdiene fra 0 til 1.

_Referanse: Huete, A., et al. «Overview of the Radiometric and Biophysical Performance of the MODIS Vegetation Indices.» Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 – Skogdekkeindeks 1

Denne indeksen skiller skogkroner fra andre typer vegetasjon ved hjelp av multispektrale refleksjonsbilder som inkluderer et rødt kantbånd.

$$
FCI1 = Red * RedEdge
$$

Skogkledde områder vil ha lavere FCI1-verdier på grunn av lavere refleksjonsgrad fra trær og tilstedeværelsen av skygger i kronetaket.

_Referanse: Becker, Sarah J., Craig S.T. Daughtry og Andrew L. Russ. «Robuste skogdekkeindekser for multispektrale bilder.» Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 – Skogdekkeindeks 2

Denne indeksen skiller skogkroner fra andre typer vegetasjon ved hjelp av multispektrale refleksjonsbilder som ikke inkluderer et rødt kantbånd.

$$
FCI2 = Red * NIR
$$

Skogkledde områder vil ha lavere FCI2-verdier på grunn av lavere refleksjonsgrad for trær og tilstedeværelsen av skygger i kronetaket.

_Referanse: Becker, Sarah J., Craig S.T. Daughtry og Andrew L. Russ. «Robuste skogdekkeindekser for multispektrale bilder.» Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI – Global miljøovervåkingsindeks

Denne ikke-lineære vegetasjonsindeksen brukes til global miljøovervåking fra satellittbilder og forsøker å korrigere for atmosfæriske effekter. Den ligner på NDVI, men er mindre følsom for atmosfæriske effekter. Den påvirkes av bar jord og anbefales derfor ikke til bruk i områder med sparsom eller moderat tett vegetasjon.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Hvor:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Referanse: Pinty, B., og M. Verstraete. GEMI: et ikke-lineært indeks for overvåking av global vegetasjon fra satellitter. Vegetation 101 (1992): 15-20._

***

## GARI - Green Atmosfærisk resistent indeks

Denne indeksen er mer følsom for et bredt spekter av klorofyllkonsentrasjoner og mindre følsom for atmosfæriske effekter enn NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Gamma-konstanten er en vektingsfunksjon som avhenger av aerosolforholdene i atmosfæren. ENVI bruker en verdi på 1,7, som er den anbefalte verdien fra Gitelson, Kaufman og Merzylak (1996, side 296).

_Referanse: Gitelson, A., Y. Kaufman og M. Merzylak. «Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS.» Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI - Green Klorofyllindeks

Denne indeksen brukes til å estimere bladklorofyllinnholdet i et bredt spekter av plantearter.

$$
GCI = {NIR \over Green} - 1
$$

Bred NIR og grønne bølgelengder gir en bedre prediksjon av klorofyllinnholdet, samtidig som det gir større følsomhet og et høyere signal-støy-forhold.

_Referanse: Gitelson, A., Y. Gritz og M. Merzlyak. «Relationships Between Leaf Chlorophyll Content and Spectral Reflectance and Algorithms for Non-Destructive Chlorophyll Assessment in Higher Plant Leaves.» Journal of Plant Physiology 160 (2003): 271-282._

***

## GLI - Green Bladindeks

Denne indeksen ble opprinnelig utviklet for bruk med et digitalt RGB-kamera for å måle hveteavling, hvor de digitale tallene (DN) for rødt, grønt og blått varierer fra 0 til 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLI-verdiene varierer fra -1 til +1. Negative verdier representerer jord og ikke-levende elementer, mens positive verdier representerer grønne blader og stengler.

_Referanse: Louhaichi, M., M. Borman og D. Johnson. «Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat.» Geocarto International 16, nr. 1 (2001): 65-70._

***

## GNDVI - Green Normaliserte forskjellsvegetasjonsindeks

Denne indeksen ligner NDVI, bortsett fra at den måler det grønne spekteret fra 540 til 570 nm i stedet for det røde spekteret. Denne indeksen er mer følsom for klorofyllkonsentrasjon enn NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Referanse: Gitelson, A., og M. Merzlyak. «Remote Sensing of Chlorophyll Concentration in Higher Plant Leaves.» Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green Optimalisert jordjustert vegetasjonsindeks

Denne indeksen ble opprinnelig utviklet med farge-infrarød fotografering for å forutsi nitrogenbehovet for mais. Den ligner på OSAVI, men erstatter det grønne båndet med rødt.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Referanse: Sripada, R., et al. «Determining In-Season Nitrogen Requirements for Corn Using Aerial Color-Infrared Photography.» Ph.D.-avhandling, North Carolina State University, 2005._

***

## GRVI - Green Ratio Vegetation Index

Denne indeksen er følsom for fotosyntesehastigheten i skogkronene, da grønn og rød refleksjonsgrad påvirkes sterkt av endringer i bladpigmentene.

$$
GRVI = {NIR \over Green }
$$

_Referanse: Sripada, R., et al. «Luftfoto med infrarødt lys for å bestemme nitrogenbehovet i mais tidlig i sesongen.» Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Green Jordjustert vegetasjonsindeks

Denne indeksen ble opprinnelig utviklet med farge-infrarød fotografering for å forutsi nitrogenbehovet for mais. Den ligner på SAVI, men erstatter det grønne båndet med rødt.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Referanse: Sripada, R., et al. «Determining In-Season Nitrogen Requirements for Corn Using Aerial Color-Infrared Photography.» Ph.D.-avhandling, North Carolina State University, 2005._

***

## LAI – Bladarealindeks

Denne indeksen brukes til å estimere løvdekke og forutsi avlingens vekst og utbytte. ENVI beregner grønn LAI ved hjelp av følgende empiriske formel fra Boegh et al (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Der EVI er:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Høye LAI-verdier varierer vanligvis fra omtrent 0 til 3,5. Men når scenen inneholder skyer og andre lyse elementer som gir mettede piksler, kan LAI-verdiene overstige 3,5. Ideelt sett bør du maskere ut skyer og lyse elementer fra scenen før du oppretter et LAI-bilde.

_Referanse: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde og A. Thomsen. «Airborne Multi-spectral Data for Quantifying Leaf Area Index, Nitrogen Concentration and Photosynthetic Efficiency in Agriculture.» Remote Sensing of Environment 81, nr. 2-3 (2002): 179-193._

***

## LCI – Bladklorofyllindeks

Denne indeksen brukes til å estimere klorofyllinnholdet i høyere planter, som er følsomme for variasjoner i refleksjonsgrad forårsaket av klorofyllabsorpsjon.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Referanse: Datt, B. «Fjernmåling av vanninnholdet i eukalyptusblader.» Journal of Plant Physiology 154, nr. 1 (1999): 30-36._

***

## MNLI – Modifisert ikke-lineær indeks

Denne indeksen er en forbedring av den ikke-lineære indeksen (NLI) som inkluderer den jordjusterte vegetasjonsindeksen (SAVI) for å ta hensyn til jordbakgrunnen. ENVI bruker en justeringsfaktor for bakgrunnsdekke (_L_) på 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Referanse: Yang, Z., P. Willis og R. Mueller. «Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy.» Proceedings of the Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 – Modifisert jordjustert vegetasjonsindeks 2

Denne indeksen er en enklere versjon av MSAVI-indeksen foreslått av Qi et al. (1994), som forbedrer den jordjusterte vegetasjonsindeksen (SAVI). Den reduserer jordstøy og øker det dynamiske området for vegetasjonssignalet. MSAVI2 er basert på en induktiv metode som ikke bruker en konstant _L_-verdi (som med SAVI) for å fremheve sunn vegetasjon.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Referanse: Qi, J., A. Chehbouni, A. Huete, Y. Kerr og S. Sorooshian. «A Modified Soil Adjusted Vegetation Index.» Remote Sensing of Environment 48 (1994): 119-126._

***

## NDRE – Normaliserte forskjeller RedEdge

Denne indeksen ligner NDVI, men sammenligner kontrasten mellom NIR og RedEdge i stedet for Red, som ofte oppdager vegetasjonsstress tidligere.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – Normaliserte forskjellsvegetasjonsindeks

Denne indeksen er et mål på sunn, grønn vegetasjon. Kombinasjonen av den normaliserte forskjellsformuleringen og bruken av de høyeste absorpsjons- og refleksjonsområdene for klorofyll gjør den robust over et bredt spekter av forhold. Den kan imidlertid bli mettet i tette vegetasjonsforhold når LAI blir høy.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Verdien av denne indeksen varierer fra -1 til 1. Det vanlige området for grønn vegetasjon er 0,2 til 0,8.

_Referanse: Rouse, J., R. Haas, J. Schell og D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. Third ERTS Symposium, NASA (1973): 309-317._

***

## NLI – ikke-lineær indeks

Denne indeksen forutsetter at forholdet mellom mange vegetasjonsindekser og biofysiske parametere på overflaten er ikke-lineært. Den lineariserer forholdet til overflateparametere som har en tendens til å være ikke-lineære.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Referanse: Goel, N. og W. Qin. «Influences of Canopy Architecture on Relationships Between Various Vegetation Indices and LAI and Fpar: A Computer Simulation.» Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI – Optimalisert jordjustert vegetasjonsindeks

Denne indeksen er basert på den jordjusterte vegetasjonsindeksen (SAVI). Den bruker en standardverdi på 0,16 for justeringsfaktoren for bakgrunnsbelysning. Rondeaux (1996) fastslo at denne verdien gir større jordvariasjon enn SAVI for lav vegetasjonsdekke, samtidig som den viser økt følsomhet for vegetasjonsdekke på over 50 %. Denne indeksen brukes best i områder med relativt sparsom vegetasjon hvor jorden er synlig gjennom vegetasjonsdekket.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Referanse: Rondeaux, G., M. Steven og F. Baret. «Optimization of Soil-Adjusted Vegetation Indices.» Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI – Renormalized Difference Vegetation Index

Denne indeksen bruker forskjellen mellom nær-infrarøde og røde bølgelengder, sammen med NDVI, for å fremheve sunn vegetasjon. Den er ufølsom for effekten av jord og solens geometri.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Referanse: Roujean, J., og F. Breon. «Estimering av PAR absorbert av vegetasjon fra toveis refleksjonsmålinger.» Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI – Jordjustert vegetasjonsindeks

Denne indeksen ligner på NDVI, men den undertrykker effekten av jordpiksler. Den bruker en justeringsfaktor for bakgrunnsdekke, _L_, som er en funksjon av vegetasjonstetthet og ofte krever forhåndskunnskap om vegetasjonsmengder. Huete (1988) foreslår en optimal verdi på _L_=0,5 for å ta hensyn til førsteordens variasjoner i jordbakgrunnen. Denne indeksen brukes best i områder med relativt sparsom vegetasjon hvor jorden er synlig gjennom kronetaket.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Referanse: Huete, A. «A Soil-Adjusted Vegetation Index (SAVI).» Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI – Transformed Difference Vegetation Index

Denne indeksen er nyttig for å overvåke vegetasjonsdekket i urbane miljøer. Den blir ikke mettet som NDVI og SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Referanse: Bannari, A., H. Asalhi og P. Teillet. «Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping» I Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, bind 5 (2002)._

***

## VARI – Synlig atmosfærisk resistent indeks

Denne indeksen er basert på ARVI og brukes til å estimere andelen vegetasjon i et område med lav følsomhet for atmosfæriske effekter.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Referanse: Gitelson, A., et al. «Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – Vegetasjonsindeks med bredt dynamisk område

Denne indeksen ligner på NDVI, men bruker en vektingskoeffisient (_a_) for å redusere forskjellen mellom bidragene fra nær-infrarøde og røde signaler til NDVI. WDRVI er spesielt effektiv i scener med moderat til høy vegetasjonstetthet når NDVI overstiger 0,6. NDVI har en tendens til å flate ut når vegetasjonsfraksjonen og bladarealindeksen (LAI) øker, mens WDRVI er mer følsom for et bredere spekter av vegetasjonsfraksjoner og for endringer i LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Vektingskoeffisienten (_a_) kan variere fra 0,1 til 0,2. Henebry, Viña og Gitelson (2004) anbefaler en verdi på 0,2.

_Referanser_

_Gitelson, A. «Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.» Journal of Plant Physiology 161, nr. 2 (2004): 165-173._

_Henebry, G., A. Viña og A. Gitelson. «The Wide Dynamic Range Vegetation Index and its Potential Utility for Gap Analysis.» Gap Analysis Bulletin 12: 50-56._
