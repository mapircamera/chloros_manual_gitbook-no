# Opptaksinnstillinger og -moduser

Opptaket i fanen «Kameraer» styres av én rød **Capture All**-knapp og ett**Capture Settings**-vindu som bestemmer hva knappen skal utføre: hvilke kameraer som skal være med, hvilke eksporttyper hvert kamera skal lagre, og om lukkeren skal utløses én gang, kontinuerlig eller med et bestemt intervall. Denne siden dokumenterer hele prosessen – konfigurasjon, selve opptaket, hvor filene lagres på disken og hvordan de senere kan bearbeides til kalibrerte produkter. Selve kontrollene for kamera og oppstillinger finnes under [Kamerainnstillinger](camera-settings.md).

{% hint style="info" %}
**Opptak krever at et prosjekt er åpent.** «Capture All» og tannhjulet for «Capture Settings» er deaktivert inntil et prosjekt er åpent («Opprett eller åpne et prosjekt for å lagre opptak»). Hvert opptak lagres under prosjektmappen i `captures/`.
{% endhint %}

## Ruten for opptaksinnstillinger

Åpne den med **tannhjulet ved siden av «Ta opp alt»**i kameralisten i sidefeltet, eller med**knappen «Åpne opptaksinnstillinger…»** nederst i hvilken som helst innstillingsrute for et enkelt kamera. Overskriften lyder «Opptakinnstillinger» med en ← tilbake-knapp.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->

Valgene dine her – inkluderte kameraer, avkryssingsruter per type og opptaksmodus – lagres **per prosjekt** og gjenopprettes når du åpner det på nytt.

### Opptaksmoduser

Tre modusknapper øverst i ruten:

| Modus | Hva den gjør | Underinnstillinger (standard) |
| --- | --- | --- |
| **Enkelt** *(standard)* | Én opptaksserie med alle valgte kameraer. | — |
| **Kontinuerlig**| Opptak etter hverandre inntil en stoppbetingelse inntreffer. | Stopp ved**Antall opptak** (standard 1) *eller* **Opptaksvarighet** (standard 10 s; enheter: sekunder / minutter / timer / dager). |
| **Intervall**(tidsforløp) | Bildeserier etter en tidsinnstilling. |**Bilder / intervall**(standard 1) ·**Hvert**N enhet (standard 5 s) ·**I** N enhet (standard 1 m). |

I kontinuerlig eller intervallmodus blir «Ta alle»-knappen til en **Stopp (N)**-knapp mens opptaket pågår, og teller opptakene etter hvert som de blir tatt.

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->

### Velge kameraer og eksporttyper

Hjelpeteksten i ruten oppsummerer det: velg hvilke kameraer og eksporttyper «Capture All» skal produsere — alt er aktivert som standard, og valgene lagres sammen med dette prosjektet.

* **Velg alle / Velg ingen**-knappene endrer avmerkingsboksen for hvert kamera samtidig.
* **Veksle mellom masseeksporttyper**(to rader med knapper):**All Raw / All Debayered / All Preview / All Radiance / All Reflectance / All Index**. Hver av dem har tre fargestadier: grønn ✓ = aktivert for alle kameraer som støtter den, gul – = aktivert for noen, grå = ingen. En bryter er deaktivert når ingen tilkoblede kameraer støtter den typen. Alle blir grå når «Fastest Capture» er aktivert.
* **Rader per kamera**: en avkryssingsboks for inkludering, pluss en utvidbar (▸/▾) liste over de eksporttypene som gjelder for det aktuelle kameraet, med individuelle avkryssingsbokser. Raden viser et antall som er aktivert, for eksempel «4/6».

### Eksporttyper og hvilke kameraer som støtter dem

Det finnes seks eksporttyper: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Bare de aktuelle vises i raden for hvert kamera:

| Eksporttype | Innhold | RGB (FRGB) | Bayer-multispektral (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Bayer-mosaikk (mono: enkeltbånd) direkte fra sensoren | ✓ | ✓ | ✓ |
| **Debayered** | Lineær demosaikk (mono: 1-kanals gråtoner) | ✓ | ✓ | ✓ |
| **Forhåndsvisning** | Full visningskjede (hvitbalanse + gamma i henhold til kameraets profil; multispektral: strekking av falske farger) | ✓ | ✓ | ✓ |
| **Strålingsintensitet** | float32 W/m²/sr/nm via hele den radiometriske kjeden | — (tilbys ikke) | ✓ | ✓ |
| **Refleksjonsgrad** | uint16 ρ (32768 = 1,0) | — (tilbys ikke) | ✓ — vises kun når kameraet har en DAQ-lyssensor (sin egen eller arvet fra matrisen) | det samme som multispektral |
| **Indeks** | Vegetasjonsindeks (LUT) gjengivelse | — | ✓ — krever et aktivert, ikke-tomt indeksuttrykk på kameraet, og tilbys ikke til medlemmer av kombinerte matriser (matrisen har én delt indeks) | — (en indeks krever ≥2 bånd; se [Monokameraer og vegetasjonsindekser](mono-indices.md)) |

Stråling og refleksjon tilbys aldri for RGB-kameraer — stråling per Bayer-piksel er ikke meningsfull for en bredbånds fotometrisk sensor.

### Raskeste opptak

Vippebryteren **⚡ Raskeste opptak — kun RAW**(oransje når den er på) overstyrer alle eksportvalg til**kun RAW** — pluss en gratis sammensatt fil med kombinert indeks for matriser — slik at bildet lagres så raskt som mulig: beregningene av strålingsintensitet, refleksjonsgrad og visning hoppes helt over ved opptakstidspunktet.

{% hint style="info" %}
**Et `.daq` lagres fortsatt.** Når en lyssensor er tilordnet, skriver «Raskeste opptak» fortsatt DAQ-målingen av nedadgående lys ved siden av råbildene — slik at strålings-, refleksjons- og indeksprodukter alle kan bygges senere ved omprosessering (se [Omprosessering av opptak](#re-processing-captures-into-calibrated-products)). Fastest Capture påvirker heller ikke valgene du har gjort i avmerkingsboksene: slå den av, så kommer de tilbake.
{% endhint %}

### Kontroller per matrise

Hver tilkoblet matrise får sitt eget gruppekort i ruten:

* **Avkrysningsruten «Inkluder»** (tre tilstander på tvers av medlemmene) og matrisens navn med visningsmodus: «(kombinert | separat)».
* **Justert**-avkryssingsrute (standard**på**): forvrenger eksport av medlemmer til matrisens justeringsprofil, slik at eksportene er pikselregistrert på tvers av kameraene. Raw-data forblir uforvrengt, men inneholder transformasjonen i metadataene. (Selve profilen beregnes i [panelet for arrayinnstillinger](camera-settings.md#alignment-co-registration-combined-only).)
* Kameraradene for medlemmene er nestet inne i kortet.

Arraykortet inneholder også to opptakere. Tenk på dem som **overvåking vs. analyse**:

| Opptaker | Grad | Hva den tar opp |
| --- | --- | --- |
| **● Ta opp indeksvideo / ■ Stopp opptak** *(kun kombinerte matriser)* | **Overvåking** | Den kombinerte indekskompositten i sanntid til video med 10 fps — 8-bit, forhåndsvisningsoppløsning, LUT innebygd. Krever et åpent prosjekt og en strømmende livevisning. Viser bilder + forløpt tid under opptak. |
| **⦿ Ta opp rå burst / ■ Stopp rå burst** *(alle matriser)* | **Analyse**| Rå Bayer-bilder med live-opptaksfrekvens (uten behandling) pluss et manifest per bilde og `.daq`-målinger, til `captures/bursts/`. Etter en serie vises en**Lag video**-knapp: den bearbeider serien offline til kalibrert video — kombinert indeks og/eller stråling / refleksjon / indeks per kamera — pluss valgfrie TIFF-filer. Opprettelsen av den kombinerte indeksen starter automatisk når du stopper serien.

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

|## «Capture All»-prosessen

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

Trykk på **Capture All** i kameralisten i sidefeltet:

1. Hvert inkludert, synlig kamera som ikke er satt på pause, tar opp med de valgte eksporttypene. **Kameraoppstillinger utløses som én synkronisert utløser** (en enkelt synkronisert gruppe på tvers av alle medlemmer — se [Multi-Camera Arrays](arrays.md)); frittstående kameraer tar opp individuelt.
2. Skjulte (øye) eller pausede kameraer hoppes over. Et array er kun fullstendig blokkert når *alle* medlemmene er skjult eller pauset.
3. Når en lyssensor er tilordnet, lagres den tilsvarende DAQ-målingen av nedadrettet lysstråling som en `.daq`-fil sammen med bildene – selv for opptak som kun består av rådata – slik at radiometriske produkter alltid kan utledes senere.
4. Knappen viser fremdrift i sanntid – «Tar opp… ferdig/totalt» – og blir til **Stopp (N)** i kontinuerlig/intervallmodus. Hvert opptakselement har en tidsavbrudd på 300 sekunder.
5. Når gjennomgangen er fullført, vises en resultatsmelding som sier **«Lagret N filer»**eller**«Lagret N, F mislyktes»**, pluss «(S skjult/pauset/hoppet over)» når kameraer ble hoppet over.

## Hvor opptakene lagres

Opptakene lagres under det åpne prosjektet i `<project>/captures/`. Hver eksporttype havner i sin **egen undermappe**, slik at et opptak på flere nivåer aldri blander typer:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── *.daq          the downwelling reading matched to the capture
```

* `<ts>` er opptakets tidsstempel og `<serial>` er kameraets serienummer. Frittstående opptak får navnet `capture_<ts>_SN<serial>_<level>`; serieopptak fra én synkronisert utløser får navnet `sync_<ts>_SN<serial>_<level>` og **deler ett tidsstempel på tvers av alle kameraene i gruppen** (nivåsuffikset fjernes når et kamera kun lagrer ett enkelt nivå).
* **En asymmetri å være klar over:** visningsnivået lagres i en mappe med navnet `preview/`, mens filene beholder `_display` i navnet — mappen og suffikset avviker kun for dette nivået.
* Ukjente nivåer faller tilbake til en mappe med sitt eget navn; hvis det ikke kan opprettes en undermappe, skrives filen til rotmappen for opptak i stedet for å gå tapt.
* Opptakene i TIFF-format komprimeres som standard uten tap (DEFLATE) og inneholder alle metadataene om kalibrering og behandling **inne i filens XMP** — opptakene er selvbeskrivende, uten andre sidecar-filer enn `.daq`-filen.

Dette er samme oppsett som `chloros-cli lattice capture` / `array-capture` skriver til sin `-o`-katalog — dokumentert i [CLI-referansen § Hvordan en opptak-mappe ser ut](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, a .daq file at the root, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Ombehandling av opptak til kalibrerte produkter

Innsamlede råbilder pluss den lagrede `.daq` er alt behandlingsrørledningen trenger — dette er grunnen til at «Fastest Capture» er trygt å bruke i reelt arbeid.

* **GUI**: legg til opptaksmappen i et prosjekt ([Legge til filer i et prosjekt](../processing-images-gui/adding-files-to-a-project.md)) og behandle som vanlig.
* **CLI**: pek `process` mot**rotmappen for opptakene**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` importerer normalt bare mappen du angir, men når den mappen ikke inneholder bilder og har undermapper, går den automatisk ned i underkatalogene — slik at undermappene på det aktuelle nivået og filene i roten `.daq` hentes inn på én gang. Hvert opptak importeres som et **enkelt bilde** med de andre nivåene knyttet til det som visningsmoduser, ikke som ett bilde per nivå.

Det fungerer også å navngi en undermappe på et nivå direkte (f.eks. `…/captures/raw/`), men da blir rotfilene `.daq` liggende igjen — kopier dem ved siden av når du beregner et radiometrisk produkt på nytt fra `raw/`, ellers har tidsstempelmatchingen ingenting å sammenligne seg med.

{% hint style="warning" %}
**Behandlingen starter alltid fra `raw`.**Innenfor hver opptaksserie er råbildet kilden til behandlingskjeden; `debayered`, `radiance`, `reflectance` og `preview` følger med som visningsmoduser, men føres aldri tilbake gjennom prosesskjeden — å bearbeide et avledet produkt på nytt ville innebære å påføre vignett, farge og strålingsberegninger som allerede er innbakt i pikslene på nytt, så Chloros avviser dette i stedet for å utføre dobbel bearbeiding. `index/`- og `composite/`-renderingene blir aldri behandlet i det hele tatt (de er utdata, ikke opptak). En «captures»-mappe som er lagret**uten** råimport, vises normalt, men `process` hopper over den og opplyser om dette; `--input-level {raw,debayered,processed}` er den bevisste «nødutgangen» som tvinger frem et inngangspunkt. Se [CLI-referansen](../reference/cli-reference.md#what-a-captures-folder-looks-like) for de nøyaktige hoppemeldingene.
{% endhint %}

To andre oppførsler det er verdt å kjenne til når man skriver skript for ombehandling:

* En `chloros-cli process`-kjøring som ba om produkter, men som **ikke skrev ut noen bildeprodukter, mislykkes med en tydelig feilmelding og avsluttes med en verdi som ikke er null** — du vil aldri få en stille, tom kjøring. Vellykkede kjøringer rapporterer antall produkter. (En bevisst kjøring med kun metadata regnes fortsatt som vellykket.)
* Reimporterte, bearbeidede eksporter tar aldri en opptakets rådata-plass — den opprinnelige rådataen forblir alltid kilden i rørledningen.

## CLI-ekvivalenter

Alt på denne siden kan kjøres uten grafisk grensesnitt. Opptakmodusene i GUI-et tilsvarer direkte `chloros-cli lattice array-capture`:

| GUI | CLI |
| --- | --- |
| Enkelt | `chloros-cli lattice array-capture` |
| Kontinuerlig | `array-capture --continuous [--count N] [--duration S]` |
| Intervall | `array-capture --interval S [--duration S]` |
| Raskeste opptak | `array-capture --fastest` |
| Justert avkryssingsrute | `--aligned / --no-aligned` |
| Avkrysningsruter for eksporttype | `--processing LEVEL` eller `--levels L1,L2,…` (standard `all`) |
| Ta opp indeksvideo | `chloros-cli lattice array-record` |
| Ta opp rå bildeserie / Lag video | `chloros-cli lattice array-burst` / `array-build-video` |

Fullstendige flaggtabeller, alternativet for smart-AE-stabilisert opptak (`--smart`) og modellen for vedvarende hastighet finnes i [CLI Referanse § Opptaksmoduser, opptakere og offline-bearbeiding](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).
