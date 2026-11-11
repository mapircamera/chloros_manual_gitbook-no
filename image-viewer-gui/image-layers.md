# Bildelag

**Nedtrekksmenyen for lag** øverst til høyre i bildeviseren lar deg veksle mellom alle versjonene av bildet du ser på – fra kildeopptaket via hvert bearbeidet produkt til de beregnede indeksbildene – uten å forlate viseren.

## Hva er bildelag?

Et «lag» i Chloros

er én **produktfil**knyttet til ett kildebilde. Import gir deg kildefilene; bearbeidingen legger til et lag for hvert produkt som kjøringen genererte. Eksporterte filer beholder kildens filnavn – det er**mappen** som identifiserer produktet, og lagnavnet er Chloros

’ betegnelse på den mappen.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## Laglisten

### Alltid til stede

| Lag | Hva det er |
| --- | --- |
| **JPG**(eller**PNG

**/**TIFF

**) | Grunnfilen som fulgte med opptaket. Survey3

-import gir en `.JPG` ved siden av hver `.RAW`; LATTICE-opptak gir en PNG

- eller TIFF

-forhåndsvisning. Merket etter hva som faktisk ble importert |
| **RAW (Original)** | Den rå kildebilden, debayert for visning uten at det er gjort noen korreksjoner. Tilgjengelig fra importøyeblikket – den trenger ingen behandling |

Et LATTICE-opptak der basefilen **er** råbildet, har ingen egen baseoppføring: `RAW (Original)` dekker allerede dette.

### Survey3

-behandlingsprodukter

| Lag | Skrevet til | Eksisterer når |
| --- | --- | --- |
| **RAW (Mål)** | — | Bildet ble identifisert som inneholdende et kalibreringsmål |
| **RAW (Refleksjonsgrad)** | `Reflectance_Calibrated_Images/` | Refleksjonsgradskalibrering ble gjennomført med hell på dette bildet |
| **Vignettkorrigert**| `Vignette_Corrected_Images/` | Bildet kunne ikke refleksjonskalibreres**og** *vignettkorreksjon* var aktivert |
| **Sensorrespons**| `Sensor_Response_Images/` | Bildet kunne ikke kalibreres for reflektans**og** *vignettkorreksjon* var slått av |
| **Hvitbalanse** | `White_Balanced_Images/` | Et produkt med hvitbalanse ble lagret |

{% hint style="info" %}
**Vignettkorrigering og sensorrespons er alternativer, aldri begge deler.** Det finnes nøyaktig ett ukalibrert reserveprodukt per kjøring, for hver kameramodell, og *Vignettkorrigering*-bryteren velger hvilket. Se [Prosjektinnstillinger](../project-settings/project-settings.md).
{% endhint %}

### LATTICE-nivåer

LATTICE fanger opp fan-out til disse i én enkelt behandlingsomgang. Hvilke som finnes, avhenger av eksportbryterne per produkt i Prosjektinnstillinger og av hva som gjelder for kameraet.

| Lag | Skrives til | Gjelder for |
| --- | --- | --- |
| **RAW (Debayered)** | `Debayered_Images/` | RGB

- og multispektralt |
| **RAW (forhåndsvisning)** | `Preview_Images/` | Multispektralt (falskfargeutvidelse) |
| **Hvitbalansert** | `Preview_Images/` | RGB

-hovedkameraer — RGB

-forhåndsvisningen er registrert under dette navnet, slik at den stemmer overens med Survey3

-laget med samme navn |
| **RAW (Radiance)** | `Radiance_Images/` | Kun multispektralt |
| **RAW (reflektans)** | `Reflectance_Calibrated_Images/` | Kun multispektralt, og kun når en samsvarende `.daq`-nedstrålingsregistrering eller et QA-godkjent mål i bildet dekker bildet |

RGB

-hovedkameraer har ingen radiometri per bånd, så radiance og reflectance hoppes over for dem som **ikke aktuelt** — loggen angir dette i stedet for å mislykkes uten melding.

{% hint style="info" %}
**Rå- og debayered-lag samsvarer med produktene (nytt i 1.3.0).**LATTICE-råfiler og debayered-filer lagres i sensorens opprinnelige orientering, og registrerer i XMP den orienteringen som tilhører dem; de andre produktene lagres med den orienteringen allerede anvendt. Visningsprogrammet og bildeoversikten viser**RAW (Original)**og**RAW (Debayered)** i henhold til denne oppføringen, slik at bildet ikke lenger snus når du veksler mellom et produktlag og et rålag. Dette gjelder kun visningen — filene blir aldri overskrevet. Filer uten orienteringsoppføring (tatt før versjon 1.3.0 eller skrevet av annen programvare) vises slik de er lagret. Se [Utdataformater for bilder](../output-image-formats.md#orientation).
{% endhint %}

Klemmasker (`*_clampmask.tif` i en `clamp_masks/`-mappe) er ikke lag: Chloros

hopper over dem ved import. Se [Utdataformater for bilder](../output-image-formats.md#clamp-masks).

### Indeks-, LUT- og sandkasse-lag

| Lagmønster | Eksempel | Hvor det kommer fra |
| --- | --- | --- |
| **RAW (`<INDEX>`-indeks)** | `RAW (NDVI Index)` | Ett per indeks konfigurert i prosjektinnstillingene, beregnet under behandlingen |
| **`<INDEX>` LUT** | `NDVI LUT` | Den fargemappede versjonen av et indeks |
| **Sandkasse (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Én per [Indeks/LUT Sandkasse](index-lut-sandbox.md) eksportkjøring |

Hvis det samme indeksnavnet er konfigurert mer enn én gang med forskjellige innstillinger, får den andre og de påfølgende et nummer i navnet (`RAW (NDVI2 Index)`), slik at lagene forblir skillebare.

***

## Bruke lagvelgeren

1. Åpne et bilde i fullskjerm ved å klikke på et miniatyrbilde i rutenettet
2. Klikk på **lag-rullegardinmenyen** øverst til høyre i visningsvinduet
3. Velg et lag — bildet oppdateres umiddelbart

Nedtrekksmenyen viser **JPG, RAW (Original), RAW (Target), RAW (Reflectance)** først, i den rekkefølgen, og viser alt annet etter dem i den rekkefølgen produktene ble registrert.

### Lagpreferanse når du navigerer

Ved å trykke **←**/**→** går du til neste bilde og systemet prøver å holde deg på samme lag:

1. **Eksakt treff først** – hvis det neste bildet har et lag med samme navn, får du det. Dette er det som holder deg på `RAW (NDVI Index)` mens du går gjennom et helt sett
2. **Deretter et treff etter type** — et indekslag søker etter et hvilket som helst indekslag, en LUT etter en hvilken som helst LUT, reflektans etter reflektans, mål etter mål, original etter original, base etter base
3. **Deretter, kun for eksportlag** — navnet beholdes selv om laglisten ikke har hentet det inn ennå, fordi filen allerede finnes på disken. Dette er det som lar deg gjennomgå produkter mens en kjøring fortsatt skriver dem
4. **Ellers** — det første tilgjengelige laget, som normalt er basisbildet

Sidecar-filene `.daq` og `.csv` i prosjektet hoppes over ved navigering med piltastene, slik at man aldri havner på en lyssensoropptak når man blar gjennom bildene.

Zoom og panorering overføres også mellom bildene, noe som gjør det enkelt å sammenligne før- og etterbilder av samme feltposisjon.

***

## Forståelse av pikselverdier per lag

[Panelet «Cursor Values»](opening-an-image-full-screen.md#cursor-values) viser den faktiske verdien per kanal under markøren, i den enheten laget er lagret i. Kolonnene endres avhengig av laget:

| Lag | Rapportert enhet | Merknader |
| --- | --- | --- |
| Base (JPG / PNG

/ TIFF

-forhåndsvisning) | DN, 0–255 | Visningsverdier, gammakorrigert på RGB

. Kun visuell inspeksjon |
| RAW (Original) | DN | Rå digitale tall fra sensoren. Histogramaksen angir dybden: 255 (8-bit), 4095 (12-bit) eller 65535 (16-bit) |
| RAW (Debayered) | DN | Lineær, ingen strekking av visningen |
| RAW (Forhåndsvisning) / Hvitbalanse | DN | Visningsprodukt — strukket eller gammakorrigert. Ikke beregnet for måling |
| RAW (Stråling) | **W/m²/sr/nm** | Float32 fysisk stråling. Ingen DN-kolonne |
| RAW (Refleksjon) | DN **og %** | Prosent beregnet med filens egen skala – se nedenfor |
| Indeks / LUT / sandkasseeksport | Indeksverdi eller RGB

-komponenter | En enkeltkanals indeksfil rapporterer indeksverdien; en fargemappet LUT-fil rapporterer Red

/Green

/Blue

-komponenter |

### Reflektans: skalaen er per fil

{% hint style="warning" %}
**«Del med 65 535» er kun riktig for Survey3

.** LATTICE-refleksjonsgrad lagres i en annen skala, og å blande de to divisorene er den vanligste måten å få refleksjonsverdier som er nøyaktig halvparten av hva de burde være.
{% endhint %}

| Kilde | DN som tilsvarer reflektans 1,0 | Identifisert av |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | XMP-taggen `Chloros:PixelScale=32768` er innstemplet i hver LATTICE-refleksjonseksport. Den doble sikkerhetsmarginen betyr at ρ over 1,0 kan representeres i stedet for å bli avskåret |
| **Survey3**| **65535** | Ingen Chloros

XMP-skala-tag — Survey3

-kalibrering skriver ρ × dtype-max og klipper ved 1,0 |

For GIS og skripting: les `Chloros:PixelScale` fra filen og del med dette tallet. Hvis taggen mangler, er filen i Survey3

-skala (65535). Visningsprogrammet, indeks-/LUT-sandkassen og indekseksporten beregner alle skalaen på samme måte, så tallet du ser ved markøren er det tallet som ble brukt i indeksberegningen.

Formatspesifikk lagring på toppen av denne skalaen:

* **TIFF

(32-bit, prosent)** lagrer DN / 65535 som et flytende tall
* **PNG

(8-bit)**og**JPG (8-bit)** lagrer DN × 255 / 65535
* Et **8-bits DN-produkt fra en 8-bits opptak**, eller en reflektansfil skrevet før versjon 1.3.0, blir avskåret til 0–255 og har ingen skalamerking. Panelet viser kun DN for disse filene, uten prosentkolonne

### Indeksverdier

| Indeksfamilie | Typisk område | Måleverdi |
| --- | --- | --- |
| Normaliserte differanser (NDVI

, GNDVI

, NDRE

, ENDVI…) | −1 til +1 | Frisk vegetasjon vanligvis 0,4–0,9; bar jord nær 0; vann negativt |
| Jordjustert (SAVI

, OSAVI

, MSAVI2…) | omtrent −1 til +1,5 | Lignende måleverdi som NDVI

, men med jordbakgrunnen undertrykt |
| Forholdstall (GRVI

, GCI

, MSR, CIRE…) | ubegrenset over | Forholdstallene vokser uten begrensning når nevnerbåndet går mot null |
| EVI

/ LAI

| 0 til ~1, 0 til ~3,5 | Skyer og andre mettede piksler skyver begge utenfor måleområdet — maskér dem først |

Se [Formler for multispektrale indekser](../project-settings/multispectral-index-formulas.md) for den eksakte formelen bak hver forhåndsinnstilling.

***

## Vanlige arbeidsflyter

### Før/etter-sammenligning

1. Velg **RAW (Original)** og legg merke til vignetteringen og de ukalibrerte verdiene
2. Bytt til **RAW (Reflektans)**

3. Sammenlign — vignettering fjernet, verdier kalibrert. Zoom og panorering holdes fast, slik at du ser på samme område

### Gjennomgå én indeks over et helt sett

1. Åpne det første behandlede bildet og velg indekslaget
2. Trykk **→** gjentatte ganger — indekslaget følger deg fra bilde til bilde
3. Følg med på histogrammet i sidefeltet mens du går gjennom: et bilde der fordelingen hopper, er verdt å se nærmere på

### Verifiser kalibreringsmål

1. Velg **RAW (Mål)** på et målbilde
2. Bekreft at målet er tydelig synlig og oppdaget
3. Gå til neste målbilde — mållaget følger etter

### Sjekk nøyaktigheten av refleksjonsverdiene

1. Velg **RAW (Refleksjon)**

2. Les**%**-kolonnen i panelet «Cursor Values» — den er allerede skalert riktig for den filen
3. Kontroller mot kjente materialer i bildet: sunn vegetasjon har høy NIR

-verdi og lav rødverdi; et kalibreringsmål bør vise en verdi som ligger nær den publiserte reflektansen

***

## Feilsøking

### Et lag jeg forventet, finnes ikke i rullegardinmenyen

**Mulige årsaker**

* Bildet ble aldri behandlet — bare baselaget og `RAW (Original)` finnes
* Eksportbryteren for produktet er ikke merket av i prosjektinnstillingene
* Produktet gjelder ikke for det aktuelle kameraet (radiance og reflektans på en RGB

-master; alle indekser på et enkeltbånds M3M-monokamera)
* Reflektanskalibreringen hadde ingenting å jobbe med — ingen `.daq`-dekning i nedadgående retning og ingen QA-godkjent mål i bildet — så bildet falt tilbake til «Vignette Corrected» eller «Sensor Response»

**Hva du bør gjøre**

1. Sjekk kjøringsloggen: Chloros

oppgir når et forespurt eksportprodukt ikke kunne leveres og hvorfor
2. Sjekk eksportbryterne for hvert produkt i [Prosjektinnstillinger](../project-settings/project-settings.md)
3. Bekreft at produktmappen finnes i prosjektets utdatastruktur
4. Kjør prosessen på nytt med produktet aktivert

### Laglisten ser ut til å være utdatert

Chloros

skanner prosjektets produktmapper på nytt mens en kjøring pågår og retter manglende lagregistreringer ut fra det som faktisk finnes på disken, så et lag som er ferdig eksportert, vises vanligvis av seg selv i en avlesning. Å bytte bort fra bildet og tilbake tvinger frem en ny oppløsning.

### Refleksjonsverdiene ser ut til å være halvparten av det de burde være

Du deler nesten helt sikkert en LATTICE-fil med 65535. Bruk `Chloros:PixelScale` (32768), eller se i **%**-kolonnen, der dette allerede er anvendt.

### Indekslaget finnes, men bildet er tomt

Indeksen trenger bånd som laget ditt ikke har – for eksempel en indeks som leser en tredje kanal, brukt på en én- eller tokanalsfil. Bytt til et flerbåndslag (refleksjonsgrad eller debayert), eller velg en indeks som passer til kameraets filter.

***

## Neste trinn

* [**Åpne et bilde i fullskjerm**](opening-an-image-full-screen.md) — markøropplesning, histogram og GSD-kontroll
* [**Indeks/LUT-sandkasse**](index-lut-sandbox.md) — interaktiv indeksvisualisering og eksport
* [**Multispektrale indeksformler**](../project-settings/multispectral-index-formulas.md) — indeksreferansen
* [**Fullføre behandlingen**](../processing-images-gui/finishing-the-processing.md) — utdatamappestrukturen som disse lagene peker mot
