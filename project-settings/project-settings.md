# Prosjektinnstillinger

Sidepanelet «Prosjektinnstillinger» <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

i Chloros lar deg konfigurere alle aspekter ved bildebehandling, gjenkjenning av kalibreringsmål, beregning av multispektrale indekser og eksportalternativer for prosjektet ditt. Disse innstillingene lagres sammen med prosjektet ditt og kan lagres som maler for gjenbruk på tvers av flere prosjekter.

## Åpne prosjektinnstillinger

Slik åpner du prosjektinnstillingene:

1. Åpne et prosjekt i Chloros


2. Klikk på fanen **Prosjektinnstillinger** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

i venstre sidepanel
3. Innstillingspanelet viser alle tilgjengelige konfigurasjonsalternativer sortert etter kategori

<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->



{% hint style="info" %}
**Innstillinger som er avhengige av andre innstillinger, er nedtonet.** Når en overordnet innstilling gjør en annen innstilling umulig (for eksempel hvis du fjerner merket for *Reflektanskalibrering / hvitbalanse*, blir *Eksporter reflektans* umulig), deaktiveres den avhengige kontrollen, og verktøytipset angir hvilken innstilling som må endres.
{% endhint %}

***

## Skjerm

### Oppløsning på miniatyrbilder

* **Type**: Nedtrekksmeny
* **Alternativer**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Standard**: Standard (512 px)
* **Beskrivelse**: Oppløsning (lengste kant, i piksler) som miniatyrbildene i bilde-rutenettet vises med. Høyere verdier ser skarpere ut når man zoomer inn, men laster tregere og bruker mer minne. Full oppløsning gir den opprinnelige bildestørrelsen.
* **Merk**: Kun for visning — dette påvirker aldri behandlingen eller eksporterte filer.

***

## Målgjenkjenning

Disse innstillingene styrer hvordan Chloros oppdager og behandler kalibreringsmål i bildene dine. Begge er kun aktive når **Refleksjonskalibrering / hvitbalanse** er aktivert (ellers er de nedtonet, fordi målgjenkjenningen hoppes over helt).

### Minste kalibreringsprøveområde (px)

* **Type**: Tall
* **Område**: 0 til 10 000 piksler
* **Standard**: 25 piksler
* **Beskrivelse**: Angir det minste arealet (i piksler) som kreves for at et oppdaget område skal regnes som en gyldig kalibreringsmålprøve. Lavere verdier vil oppdage mindre mål, men kan øke antallet falske positive resultater. Høyere verdier krever større og tydeligere målområder for gjenkjenning.
* **Når du bør justere**:
  * Øk verdien hvis du får falske deteksjoner på små bildeartefakter
  * Reduser verdien hvis kalibreringsmålene dine fremstår som små i bildene og ikke blir oppdaget

### Minste målgruppering (0–100)

* **Type**: Tall
* **Område**: 0 til 100
* **Standard**: 60
* **Beskrivelse**: Styrer klyngetærskelen for gruppering av områder med lignende farger ved deteksjon av kalibreringsmål. Høyere verdier krever at flere lignende farger grupperes sammen, noe som resulterer i en mer konservativ måldeteksjon. Lavere verdier tillater større fargevariasjon innenfor en målgruppe.
* **Når du bør justere**:
  * Øk verdien hvis kalibreringsmål blir delt opp i flere deteksjoner
  * Reduser verdien hvis kalibreringsmål med fargevariasjon ikke blir fullstendig detektert

***

## Behandling

Disse innstillingene styrer hvordan Chloros behandler og kalibrerer bildene dine.

### Vignettkorrigering

* **Type**: Avkryssingsrute
* **Standard**: Aktivert (avkrysset)
* **Beskrivelse**: Bruker vignettkorreksjon for å kompensere for mørkere kanter på bildene. Vignettering er et vanlig optisk fenomen der hjørnene og kantene på et bilde fremstår mørkere enn midten på grunn av objektivets egenskaper.
* **Bivirkning**: Denne bryteren velger også hvilket *ukalibrert reserveprodukt* en kjøring skriver ut (se nedenfor).

### Refleksjonskalibrering / hvitbalanse

* **Type**: Avkryssingsrute
* **Standard**: Aktivert (avkrysset)
* **Beskrivelse**: Aktiverer reflektanskalisering — fra oppdagede kalibreringsmål i bildet og/eller nedadrettede data fra DAQ-lyssensoren, avhengig av kameraet og hva som er tilgjengelig. Dette normaliserer reflektansverdiene i datasettet ditt og sikrer konsistente målinger uavhengig av lysforhold.
* **Når deaktivert**: Målgjenkjenning hoppes helt over, og**ingen reflektansprodukt kan produseres av noe kamera** — verken Survey3-målstyrt eller LATTICE DAQ-styrt. De avhengige innstillingene (*Eksporter reflektans*, *Minimumsintervall for rekalibrering* og terskelverdiene for målgjenkjenning) er nedtonet.

### Ukalibrerte reserveprodukter: Eksporter sensorrespons / Eksporter vignettkorrigert

* **Type**: To avmerkingsbokser
* **Standardinnstillinger**: Begge aktivert (avmerket)
* **Beskrivelse**: Når et bilde ikke kan refleksjonskalibreres (det ble ikke funnet noe kalibreringsmål, eller refleksjonskalibrering er slått av), lagres det i stedet som et *ukalibrert reserveprodukt*. **Det finnes nøyaktig ett av de to reserveproduktene per kjøring, for hver kameramodell**, valgt av *Vignettkorreksjon*-bryteren:
  * Vignettkorreksjon **på**→ `Vignette_Corrected_Images/` (styrt av**Eksporter vignettkorrigert**)
  * Vignettkorreksjon **av**→ `Sensor_Response_Images/` (styrt av**Eksporter sensorrespons**)
* Det reserveproduktet som ikke er i bruk, er nedtonet. Hvis du fjerner merket for det som er i bruk, vil den filen ikke bli skrevet i det hele tatt.

### LATTICE-eksportprodukter

For prosjekter som inneholder LATTICE-opptak, blir hvert importert LATTICE-bilde fordelt på alle aktiverte **og relevante**produkter i én enkelt behandlingsomgang. Fire avmerkingsbokser styrer fordelingen (alle er som standard**på**):

| Innstilling | Utdatamappe | Hva den eksporterer |
| --- | --- | --- |
| **Eksporter debayert** | `Debayered_Images/` | Det lineære, debayerte bildet: kalibreringspakkens mørkeforskyvning er fjernet, deretter en kantbevisst demosaik, uten linearitetskorreksjon, flatfelt eller unmix — den samme filen som et opptak lagrer på sitt debayered-nivå. Bevares i sensorens opprinnelige orientering (registrert i XMP). Gjelder for RGB- og multispektrale kameraer. |
| **Eksporter forhåndsvisning** | `Preview_Images/` | Forhåndsvisningen på skjermen. RGB = hvitbalanse (DAQ-lysstyrke når tilgjengelig, ellers gråverden) + gamma; multispektral = strekking med falske farger. |
| **Eksporter radianse** | `Radiance_Images/` | Float32 spektral radianse i W/m²/sr/nm. Kun multispektralt (M3C/M3M) — gjelder ikke for RGB-masterbilder. Skrives alltid som 32-biters TIFF, uavhengig av innstillingen for *Kalibrert bildeformat*. |
| **Eksporter reflektans**| `Reflectance_Calibrated_Images/` | Uint16-reflektans, skalert slik at**32768 = reflektans 1,0** (merket som XMP `Chloros:PixelScale`). Kun multispektralt, skrives når en samsvarende `.daq`-nedstrålingspost (eller et QA-godkjent mål i bildet) dekker bildet. |

* RGB-hovedkameraer sender ut debayert + forhåndsvisning; strålingsintensitet/refleksjonsgrad hoppes over for disse da de ikke er relevante.
* Bitdybden for debayert/forhåndsvisning følger innstillingen *Kalibrert bildeformat*; strålingsintensitet er alltid float32.
* Survey3-behandlingen påvirkes ikke av disse fire bryterne.
* Radiance- og reflektansprodukter fra multispektrale M3C-kameraer kan også skrive en **klemmaske** per piksel i en `clamp_masks/`-mappe ved siden av produktmappene, og registrerer hvor båndseparasjonen har kuttet en negativ verdi til null. Se [Utgangsbildeformater](../output-image-formats.md#clamp-masks).

De samme fire bryterne finnes i headless-modus som `chloros-cli process --debayered / --preview / --radiance / --reflectance`. De erstattet det gamle CLI-flagget `--radiometric-output`, som ikke lenger eksisterer.

{% hint style="warning" %}
**Hvis alle aktuelle produkter deaktiveres, mislykkes kjøringen.** Fra og med versjon 1.2.0 rapporterer en behandlingskjøring som ble bedt om produkter, men som ikke skrev noen bildeprodukter, en feil, og CLI-en avsluttes med en verdi ulik null, i stedet for å rapportere en stille suksess. Loggen angir hvilket produkt den ikke kunne skrive og hvorfor. En bevisst kjøring kun med metadata (ingenting forespurt) regnes fortsatt som vellykket.
{% endhint %}

### Reflektanskilde (prosjektinnstilling, angitt via CLI)

Prosjektet lagrer også hvilken **reflektansreferanse** LATTICE-reflektansproduktet bruker. Det finnes ingen egen kontroll i innstillingspanelet; verdien lagres i prosjektkonfigurasjonen som `Processing → "Target reflectance source"` og angis med `chloros-cli process --reflectance-source {auto,target,daq}` (SDKs `configure()` har ikke noe nøkkelord for dette):

* **`auto`** (standard): et kalibreringsmål innenfor rammen som har bestått kvalitetssikring (QA), blir den absolutte referansen, mens systemet faller tilbake til DAQ-nedstrålingsdelingsverdien (ρ = πL/E) når det ikke finnes noe mål eller når kvalitetssikringen mislykkes. Et mål som består QA med en feilmelding (et panel som har falt ned, eller et bånd som er ute av drift) blir ikke automatisk foretrukket: kjeden med målet, målet+DAQ og kun DAQ blir prøvd i rekkefølge etter deres målte poengsum på utelatte paneler.
* **`target`**: streng målstyrt reflektans — ingen DAQ-erstatning.
* **`daq`**: DAQ-autoritativ reflektans; mål innenfor rammen brukes ikke som referanse.

Den lagrede verdien matches uten å skille mellom store og små bokstaver, og noen få stavemåter godtas som aliaser: `target`, `target_image`, `empirical` og `empirical_line` betyr alle **mål**; `daq`, `dls`, `light_sensor` og `sensor` betyr alle**daq**. Alt annet – inkludert en manglende nøkkel – blir til**auto**.**Målte** målingene per enhet blir slått opp etter måleenhetens serienummer/QR-kode, som `<serial>.csv`, på tre steder: katalogen angitt med `--target-reflectance-dir` (lagret som `Processing → "Target reflectance dir"`), prosjektets egen `target_reflectance/`-mappe, og banen i miljøvariabelen `CHLOROS_TARGET_REFLECTANCE_DIR`. Når det ikke finnes noen målt skanning for den enheten, brukes den nominelle publiserte kurven for målmodellen i stedet.

### Debayer-metode

* **Type**: Valg fra rullegardinmeny
* **Alternativer**:
  * Standard (rask, middels kvalitet)
  * Teksturbevisst (langsom, høyeste kvalitet) \[Chloros+]
* **Standard**: Standard (rask, middels kvalitet)
* **Beskrivelse**: Velger demosaicing-algoritmen som brukes til å konvertere rå sensordata fra Bayer-mønsteret til fullfargebilder. Metoden «Standard (rask, middels kvalitet)» gir en optimal balanse mellom behandlingshastighet og bildekvalitet. «Teksturbevisst (langsom, høyeste kvalitet)» \[Chloros+] bruker en kantbevisst debayer av høy kvalitet kombinert med en AI/ML-støyreduksjonsmodell som fjerner nesten all debayering-støy. «Teksturbevisst»-modellen krever GPU-minne (VRAM) for å kjøre. Vi anbefaler å bruke den når du har >4 GB VRAM tilgjengelig for raskere behandling.
* **Når raden i det hele tatt er en rullegardinmeny**: rullegardinmenyen med to alternativer vises bare når**begge** vilkårene er oppfylt — du er logget inn med et kvalifiserende Chloros+-abonnement, **og** prosjektet inneholder ingen LATTICE-opptak. Ellers vises raden som ren tekst med teksten `Standard (Fast, Medium Quality)` uten noe å velge mellom.
* **LATTICE-merknad**: Det finnes ingen LATTICE-trent «Texture Aware»-modell, og behandlingskjeden tvinger frem standard demosaik for LATTICE-rammer uavhengig av den lagrede verdien. Hvis du legger til en LATTICE-mappe i et prosjekt der «Texture Aware» allerede var valgt, skriver Chloros innstillingen tilbake til «Standard» i stedet for å beholde en foreldet verdi i `project.json`.

### Minste intervall for rekalibrering

* **Type**: Tall
* **Område**: 0 til 3 600 sekunder
* **Standard**: 0 sekunder
* **Beskrivelse**: Angir det minste tidsintervallet (i sekunder) mellom bruk av kalibreringsmål. Når verdien er satt til 0, vil Chloros bruke hvert eneste kalibreringsmål som oppdages. Når verdien er satt til et høyere tall, vil Chloros kun bruke kalibreringsmål som er adskilt med minst dette antallet sekunder, noe som reduserer behandlingstiden for datasett med hyppige registreringer av kalibreringsmål.
* **Når du bør justere**:
  * Sett til 0 for maksimal kalibreringsnøyaktighet når lysforholdene varierer
  * Øk verdien (f.eks. til 60–300 sekunder) for raskere behandling når lysforholdene er jevne og du har hyppige bilder av kalibreringsmål

### Tidssoneforskjell for lyssensor

* **Type**: Tall
* **Område**: -12 til +12 timer
* **Standard**: 0 timer
* **Beskrivelse**: Angir tidssoneforskyvningen (i timer fra UTC) for tidsstempler på lyssensordata, som brukes når lyssensorlogger skal samsvare med tidspunktet for bildeopptak. Nyere `.daq`-opptak har sin egen tidssoneangivelse, så dette er hovedsakelig nødvendig for eldre logger som er registrert i lokal tid.

### Bruk PPK-korreksjoner

* **Type**: Avkryssingsrute
* **Standard**: Deaktivert (ikke merket av)
* **Beskrivelse**: Aktiverer bruk av PPK-korreksjoner (Post-Processed Kinematic) fra MAPIR DAQ-opptakere som inneholder GPS (GNSS). Når dette er aktivert, vil Chloros bruke alle .daq-loggfiler som inneholder eksponeringspin-data i prosjektkatalogen din og anvende presise geolokasjonskorreksjoner på bildene dine.
* **Krav**: Det må finnes en .daq-loggfil med eksponeringspin-oppføringer i prosjektkatalogen din
* **Når skal du aktivere**: Det anbefales å alltid aktivere PPK-korreksjon hvis du har eksponeringsfeedback-oppføringer i .daq-loggfilen din.

### Eksponeringspinne 1

* **Type**: Valg fra rullegardinmeny
* **Synlighet**: Kun synlig når «Bruk PPK-korreksjoner» er aktivert OG eksponeringsdata er tilgjengelig for pin 1
* **Alternativer**:
  * Navn på kameramodeller som er oppdaget i prosjektet
  * «Ikke bruk» – Ignorer denne eksponeringspinnen
* **Standard**: Velges automatisk basert på prosjektkonfigurasjonen
* **Beskrivelse**: Tilordner et bestemt kamera til eksponeringspinne 1 for PPK-tidssynkronisering. Eksponeringspinnen registrerer det nøyaktige tidspunktet når kameraets lukker utløses, noe som er avgjørende for nøyaktig PPK-geolokalisering.
* **Oppførsel ved automatisk valg**:
  * Ett kamera + én pinne: Velger kameraet automatisk
  * Ett kamera + to pinner: Pinne 1 tilordnes automatisk til kameraet
  * Flere kameraer: Manuell valg kreves

### Eksponeringspinne 2

* **Type**: Valg fra rullegardinmeny
* **Synlighet**: Kun synlig når «Bruk PPK-korreksjoner» er aktivert OG eksponeringsdata er tilgjengelige for pinne 2
* **Alternativer**:
  * Kameramodellnavn som er oppdaget i prosjektet
  * «Ikke bruk» – Ignorer denne eksponeringspinnen
* **Standard**: Velges automatisk basert på prosjektkonfigurasjonen
* **Beskrivelse**: Tilordner et bestemt kamera til eksponeringspinne 2 for PPK-tidssynkronisering ved bruk av et oppsett med to kameraer.
* **Oppførsel ved automatisk valg**:
  * Ett kamera + én pinne: Pinne 2 settes automatisk til «Ikke bruk»
  * Ett kamera + to pinner: Pinne 2 settes automatisk til «Ikke bruk»
  * Flere kameraer: Manuell valg kreves
* **Merk**: Det samme kameraet kan ikke tilordnes både pinne 1 og pinne 2 samtidig.

***

## DAQ-lyssensor

Denne delen vises i Prosjektinnstillinger og viser alle DAQ-nedstrålingsfiler i prosjektet — `.daq`-opptak og DAQ-M `.csv`-nedstrålingslogger. Opptak laget i fanen «Lyssensorer» legges automatisk til i det åpne prosjektet.

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->



Hver rad viser filen, sensormodellen og diffusorhettekorreksjonen som faktisk gjelder for den aktuelle filen. Over radene finnes en enkelt prosjektomfattende kontroll:

### Overstyring av hette (alle filer)

* **Type**: Valg fra rullegardinmeny
* **Alternativer**: `Auto` pluss de kappe-korreksjonsprofilene som gjelder for sensortypene i prosjektet
* **Standard**: Auto
* **Lagret som**: `Processing → "DAQ cap id"` (standard `auto`)
* **Beskrivelse**: `Auto` bruker den registrerte hetten for hver fil (det antas at det er en Sunshine-hette der ingenting er registrert, men DAQ-U, DAQ-M og DAQ-E leveres uten hette, og Chloros kan ikke se hva som er montert, så behandlingen gir en advarsel om en antatt kappe: velg den kappen som var montert). Å velge en spesifikk kappe overstyrer **alle** nedstrømsfiler i prosjektet unntatt en DAQ-E-S, der diffusoren er en del av kalibreringen; et prosjekt som kun består av DAQ-E-S viser ingen rullegardinmeny: råopptak korrigeres med den, og opptak som allerede har en hette, blir referert på nytt (den innspilte korreksjonen blir tilbakestilt, og den valgte blir brukt).
* **Viktig**: Den valgte hetten må samsvare med den hetten som fysisk var montert under opptaket. Verken sensoren eller programvaren kan oppdage den fysiske hetten – en feilaktig hette-ID fører til feil korreksjon av spektrene.

Det er bevisst **én** prosjektomfattende innstilling i stedet for rullegardinmenyer per fil: innstillingen gjelder for alle nedadrettede kilder i prosjektet.

***

## Array-justering

Denne delen vises **bare** når minst ett bilde i prosjektet har den modul-til-modul-justeringstransformasjonen som LATTICE-matriser merker ved opptakstidspunktet (XMP `Chloros:Alignment*`-tagger). Den viser hvor mange bilder som har justeringstagger, hvilket kamera som er referansen (`REF`-merke), og en tabell med antall bilder per kamera.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Bruk array-justering

* **Type**: Avkryssingsrute
* **Standard**: Aktivert (avkrysset)
* **Lagret som**: `Processing → "Array alignment"`
* **Beskrivelse**: Forvrenger hvert bearbeidet produkt (debayered / forhåndsvisning / radiance / reflektans / indeks) til matrisens felles referansegeometri ved hjelp av transformasjonen som ble stemplet ved opptaket. Av = eksporter i opprinnelig geometri per sensor.

### Beskjær til felles overlapping

* **Type**: Avkryssingsrute (kun aktiv når *Bruk matrisejustering* er aktivert)
* **Standard**: Aktivert (avkrysset)
* **Lagret som**: `Processing → "Array alignment crop"`
* **Beskrivelse**: Beskjærer justerte eksporter til området som alle kameramodulene deler, slik at hvert bånd har samme fotavtrykk. Av beholder hele sensorområdet (svart utfylling utenfor kilden).

### Resampling

* **Type**: Nedtrekksmeny (kun aktiv når *Bruk matrisejustering* er slått på)
* **Alternativer**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Standard**: Bilineær
* **Lagret som**: `Processing → "Array alignment interpolation"`
* **Beskrivelse**: Interpolering brukt av justeringsforvrengningen. «Nærmeste» bevarer eksakte kildeverdier (ingen blanding mellom piksler) for streng radiometrisk analyse; «Bilineær» er best for kartlegging og visuell bruk.

De samme tre alternativene finnes uten foranliggende navn som `chloros-cli process --array-alignment`, `--array-alignment-crop` og `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Indeks

Disse innstillingene lar deg konfigurere multispektrale indekser for analyse og visualisering.

### Legg til indeks

* **Type**: Spesielt konfigurasjonspanel for indekser
* **Beskrivelse**: Åpner et interaktivt panel der du kan velge og konfigurere multispektrale vegetasjonsindekser (NDVI, NDRE, EVI osv.) som skal beregnes under bildebehandlingen. Du kan legge til flere indekser, hver med sine egne visualiseringsinnstillinger.
* **Tilgjengelige indekser**: Nedtrekksmenyen i brukergrensesnittet inneholder**27** forhåndsdefinerte multispektrale indeksformler (se [Multispektrale indeksformler](multispectral-index-formulas.md) for den fullstendige listen, inkludert hvilke navn som også godtas av CLI/SDK `--indices`-alternativet).
* **Funksjoner**:
  * Velg blant forhåndsdefinerte indeksformler
  * Dra kameraets filterkanaler inn i formelens båndplasser
  * Konfigurer fargegradienter for visualisering (LUT – Look-Up Tables)
  * Angi terskelverdier og klippemoduser
  * Lag egendefinerte indeksformler
* **Merk**: Indekser beregnes ikke for enkeltbånds LATTICE M3M-monokameraer — flerbåndsindekser er udefinerte på ett bånd. Survey3 og LATTICE M3C påvirkes ikke.

<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->



Hver indeks du legger til, gjengir formelen som matematikk, med en farget sirkel per båndplass: rød = Red, grønn = Green, blå = Blue, oransje = Orange, cyan = Cyan, lilla = NIR, magenta = RE. Dra en sirkel fra raden over formelen og slipp den på en plass for å knytte den til den; dobbeltklikk på en tilknyttet plass for å fjerne den. Indeksen beregnes bare én gang for hver plass formelen bruker som har en kanal.

### Egendefinerte formler (Chloros+-funksjon)

* **Type**: Matrise med definisjoner av egendefinerte formler
* **Tilgjengelighet**: Krever pålogging med et kvalifiserende Chloros+-abonnement.
* **Beskrivelse**: Lar deg opprette og lagre egendefinerte multispektrale indeksformler ved hjelp av båndmatematikk. Egendefinerte formler lagres sammen med prosjektinnstillingene dine og kan brukes på samme måte som innebygde indekser.
* **Slik oppretter du**:
  1. Åpne kalkulatoren for egendefinerte formler i konfigurasjonspanelet for indekser
  2. Skriv formelen ved å bruke **bånd-slot-symbolene**, ikke båndnavn
  3. Lagre formelen med et beskrivende navn – den vises da nederst i formelmenyen, og du drar kameraets kanalsirkler inn i slotene akkurat som med en innebygd forhåndsinnstilling
* **Formelsyntaks**:
  * Båndslots: `x`, `y`, `z`, `a`, `b`, `c` — seks posisjoner som du tilordner til virkelige kanaler ved å dra
  * Operatører: `+`, `-`, `*`, `/`, `^` og `()` for gruppering
  * Funksjoner: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Hvorfor symboler, ikke bandnavn**: En formel skrevet som `(y-x)/(y+x)` fungerer på alle kameraer, fordi tilordningen ved dra-og-slipp avgjør om `y` er 850 nm NIR for et RGN-filter eller 808 nm NIR for et OCN-filter. De innebygde forhåndsinnstillingene lagres på samme måte — se [Multispektrale indeksformler](multispectral-index-formulas.md) for den nøyaktige symbolformen til alle 27.
* **Hvor de fungerer**: egendefinerte formler lagres sammen med prosjektinnstillingene og kan brukes både i [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) og i behandlingen. De godtas**ikke** av CLI-/SDK-navnelisten `--indices`, som kun utvider de 22 innebygde forhåndsinnstillingene.

***

## Eksport

Disse innstillingene styrer formatet og kvaliteten på eksporterte, bearbeidede bilder.

### Kalibrert bildeformat

* **Type**: Valg fra rullegardinmeny
* **Alternativer**:
  * **TIFF (16-bit)** – Ukomprimert 16-biters TIFF-format
  * **TIFF (8-bit)** – Ukomprimert 8-bit TIFF-format
  * **TIFF (32-bit, prosent)** – 32-bit TIFF med flytende desimaler, der reflektansverdiene angis i prosent
  * **PNG (8-bit)** – Komprimert 8-bit PNG-format
  * **JPG (8-bit)** – Komprimert 8-biters JPEG-format
* **Standard**: TIFF (16-bit); et prosjekt som utelukkende består av 8-biters data viser TIFF (8-bit) og tilbyr kun 8-biters formater
* **Beskrivelse**: Velger filformatet for lagring av bearbeidede og kalibrerte bilder. Eksporterte filer plasseres i en undermappe for hvert format inne i hver kameramappe (`tiff16`, `tiff32`, `png8`, `jpg8`), med én `<Product>_Images/`-mappe per produkt. Eksporterte filer beholder kildens filnavn – det er mappen, ikke filnavnets suffiks, som identifiserer produktet.
* **Anbefalinger for format**:
  * **TIFF (16-bit)**: Anbefales for vitenskapelig analyse og profesjonelle arbeidsflyter. Bevarer maksimal datakvalitet uten komprimeringsartefakter. Best egnet for multispektral analyse og videre behandling i GIS-programvare.
  * **TIFF (32-bit, prosent)**: Best egnet for arbeidsflyter som krever reflektansverdier i prosent (0–100 %). Gir maksimal presisjon for radiometriske målinger.
  * **PNG (8-bit)**: Egnet for visning på nettet og generell visualisering. Mindre filstørrelser med tapsfri komprimering, men redusert dynamisk område.
  * **JPG (8-bit)**: Minste filstørrelser, best kun for forhåndsvisninger og visning på nettet. Bruker komprimering med tap, noe som ikke er egnet for vitenskapelig analyse.
* **Merk**: LATTICE-radiance eksporteres alltid som 32-bit float-TIFF, uavhengig av denne innstillingen.

***

## Lagre prosjektmal

Denne funksjonen lar deg lagre de gjeldende prosjektinnstillingene dine som en gjenbrukbar mal.

* **Type**: Tekstfelt + Lagre-knapp
* **Beskrivelse**: Skriv inn et beskrivende navn på innstillingsmalen din og klikk på lagre-ikonet. Malen vil lagre alle dine gjeldende prosjektinnstillinger (målgjenkjenning, behandlingsalternativer, indekser, og eksportformat) for enkel gjenbruk i fremtidige prosjekter. Maler lagres i mappen `Project Templates/` inne i prosjektets lagringsmappe, og kan også velges eller eksporteres fra hovedmenyen (*Velg mal* / *Lagre mal* / *Eksporter mal*).
* **Bruksområder**:
  * Opprett maler for ulike kamerasystemer (RGB, multispektralt, NIR)
  * Lagre standardkonfigurasjoner for bestemte avlingstyper eller analysearbeidsflyter
  * Del ensartede innstillinger på tvers av et team
* **Slik bruker du det**:
  1. Konfigurer alle ønskede prosjektinnstillinger
  2. Skriv inn et malnavn (f.eks. «RedEdge Survey3 NDVI Standard»)
  3. Klikk på lagringsikonet
  4. Malen kan nå lastes inn når du oppretter nye prosjekter

***

## Lagringsmappe for prosjekter

Denne innstillingen angir hvor nye prosjekter lagres som standard.

* **Type**: Visning av katalogbane + Rediger-knapp
* **Standard (Windows)**: `Documents\MAPIR\Chloros Projects` i Dokumenter-mappen din — vanligvis `C:\Users\[Username]\Documents\MAPIR\Chloros Projects`, eller der OneDrive eller en roamingprofil har flyttet Dokumenter
* **Standard (Linux)**: `~/Documents/MAPIR/Chloros Projects`
* **Beskrivelse**: Viser den gjeldende standardkatalogen der nye Chloros-prosjekter opprettes. Klikk på redigeringsikonet for å velge en annen katalog. Endringen lagres som en enkelt tekstlinje i `~/.chloros/working_directory.txt` — på Windows er dette `C:\Users\<Username>\.chloros\working_directory.txt`. Hvis den filen mangler, eller angir en sti som ikke lenger eksisterer, faller Chloros tilbake til standarden ovenfor. CLI-grensesnittet leser og skriver til den samme filen, så `chloros-cli` og GUI-grensesnittet er alltid enige om hvor prosjektene ligger.
* **Prosjektmaler** ligger i en `Project Templates/`-undermappe i denne katalogen.
* **Når du bør endre**:
  * Sett til et nettverksdrev for teamsamarbeid
  * Bytt til et drev med mer lagringsplass for store datasett
  * Organiser prosjekter etter år, kunde eller prosjekttype i forskjellige mapper
* **Merk**: Endring av denne innstillingen påvirker kun NYE prosjekter. Eksisterende prosjekter forblir på sine opprinnelige steder.
* **Windows-banelengde**: med mindre lange Windows-baner er aktivert (alternativet **Aktiver lange Windows-baner** i installasjonsprogrammet), avvises en prosjektmappe som er dypere enn ca. 130 tegn før behandling, fordi resultatene vil overskride Windows’ grense på 260 tegn for baner. Se [Starte behandlingen](../processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

***

## Lagring av innstillinger

Et Chloros-prosjekt er en **mappe**. Alle prosjektinnstillinger lagres i `project.json` inne i den; tilkoblet maskinvare lagres sammen med dem i `cameras.json` og `sensors.json`, slik at når du åpner et prosjekt på nytt, kobles kameraene og lyssensorene til igjen. Når du åpner et prosjekt på nytt, gjenopprettes alle innstillinger nøyaktig slik du forlot dem. Lagrede prosjekter kan også kjøres uten skjerm ved hjelp av `chloros-cli project` eller SDK-filene `open_project`.

### Innstillingshierarki

Innstillingene brukes i følgende rekkefølge:

1. **Systemstandarder** – Innebygde standarder definert av Chloros

2. **Malinnstillinger** – Hvis du laster inn en mal når du oppretter et prosjekt
3. **Lagrede prosjektinnstillinger** – Innstillinger lagret sammen med prosjektfilen
4. **Manuelle justeringer** – Eventuelle endringer du gjør i løpet av den aktuelle økten

### Innstillinger og bildebehandling

Behandlingsinnstillingene leses inn når en behandlingskjøring starter. Å endre en innstilling påvirker ikke produkter som allerede ligger på disken – kjør behandlingen på nytt for å bruke de nye innstillingene. Noen få innstillinger påvirker aldri behandlingen i det hele tatt:

* Oppløsning på miniatyrbilder (kun for visning)
* Lagre prosjektmal
* Lagre prosjektmappe

***

## Referanse for konfigurasjonsnøkler

For automatisering (CLI `--config`, SDK `configure` eller direkte lesing av `project.json`) er dette de eksakte nøklene under `Project Settings`:

| Nøkkelbane | Type | Standard |
| --- | --- | --- |
| `Display → Image Thumbnail Resolution` | `"512" \| "1024" \| "2048" \| "full"` | `"512"` |
| `Target Detection → Minimum calibration sample area (px)` | tall 0–10 000 | `25` |
| `Target Detection → Minimum Target Clustering (0-100)` | tall 0–100 | `60` |
| `Processing → Vignette correction` | bool | `true` |
| `Processing → Reflectance calibration / white balance` | bool | `true` |
| `Processing → Export sensor response` | bool | `true` |
| `Processing → Export vignette corrected` | bool | `true` |
| `Processing → Export debayered` | bool | `true` |
| `Processing → Export preview` | bool | `true` |
| `Processing → Export radiance` | bool | `true` |
| `Processing → Export reflectance` | bool | `true` |
| `Processing → Array alignment` | bool | `true` |
| `Processing → Array alignment crop` | bool | `true` |
| `Processing → Array alignment interpolation` | `"Bilinear" \| "Nearest" \| "Cubic"` | `"Bilinear"` |
| `Processing → Debayer method` | `"Standard (Fast, Medium Quality)" \| "Texture Aware (Slow, Highest Quality)"` | Standard |
| `Processing → Minimum recalibration interval` | tall 0-3600 | `0` |
| `Processing → Light sensor timezone offset` | tall -12..12 | `0` |
| `Processing → Apply PPK corrections` | bool | `false` |
| `Processing → DAQ cap id` | cap-profil-ID eller `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | liste over indekskonfigurasjoner | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (8-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

`Array alignment`-nøklene skrives første gang «Array Alignment»-delen gjengis, eller når et automatiseringskall setter dem. Så lenge de mangler, bruker pipelinen de samme verdiene som vist ovenfor (`true`, `true`, bilineær), så et prosjekt.json uten dem oppfører seg identisk med et som har dem.

### Nøkler lagret i `project.json` uten kontroll i innstillingspanelet

Disse ligger under det samme `Project Settings`-treet og leses av prosesseringen, men du finner ikke en widget for dem i sidefeltet:

| Nøkkelbane | Type | Standard | Angitt av |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK `input_level=`. Overstyrer hvordan LATTICE-inngangs-TIFF-filer tolkes; `auto` utleder dette fra hver fils `Chloros:ProcessingLevel` XMP-tag samt antall kanaler. Ignoreres for Survey3-opptak. Dette er bevisst ikke en GUI-innstilling — «auto» er riktig i alle normale tilfeller. |
| `Processing → Target reflectance dir` | stistreng | `""` | `chloros-cli process --target-reflectance-dir`, eller prosjektets mål-API |
| `Processing → Target reflectance config` | ordbok med nøkkel etter kameraets serienummer | `{}` | Registrering av et mål innenfor bildet (modus `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | banestreng | `""` | SDK `process_folder(daq_log_path=…)`. Peker på en `.daq`-opptak eller en mappe med slike |
| `Target Detection → Minimum calibration target squares` | tall | `4` | Eldre standard; ingen kontroll og ingen CLI-flagg |
| `UI → Grid thumbnail size` | tall | `160` | Bildegitterets egen glidebryter for miniatyrzoom |

To visningsinnstillinger lagres **på øverste nivå i `project.json`**, helt utenfor `Project Settings`, fordi de angår visningsstatus snarere enn behandlingsinnstillinger:

| Nøkkelbane | Type | Standard | Angitt av |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | heltall 1–256 | `1` | Bildetabbenes GSD (px)-kontroll — se [Åpne et bilde i fullskjerm](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Anbefalte fremgangsmåter

1. **Start med standardinnstillingene**: Standardinnstillingene fungerer godt for de fleste MAPIR-kamerasystemer og typiske arbeidsflyter.
2. **Lag maler**: Når du har optimalisert innstillingene for en bestemt arbeidsflyt eller et bestemt kamera, bør du lagre dem som en mal for å sikre konsistens på tvers av prosjekter.
3. **Test før fullstendig behandling**: Når du eksperimenterer med nye innstillinger, bør du teste dem på et lite utvalg av bilder før du behandler hele datasettet.
4. **Dokumenter innstillingene dine**: Bruk beskrivende malnavn som angir kamerasystem, behandlingstype og tiltenkt bruk (f.eks. «Survey3\_RGB\_NDVI\_Agriculture»).
5. **Valg av eksportformat**: Velg eksportformat basert på sluttbruken:
   * Vitenskapelig analyse → TIFF (16-bit eller 32-bit)
   * GIS-behandling → TIFF (16-bit)
   * Rask visualisering → PNG (8-bit)
   * Deling på nettet → JPG (8-bit)

***

For mer informasjon om multispektrale indekser i Chloros, se siden [Formler for multispektrale indekser](multispectral-index-formulas.md).
