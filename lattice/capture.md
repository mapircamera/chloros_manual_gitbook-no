# Opptakinnstillinger og -moduser

Opptak i fanen «Kameraer» styres av én rød **Capture All**-knapp og ett**Capture Settings**-vindu som bestemmer hva denne knappen skal utføre: hvilke kameraer som skal delta, hvilke eksporttyper hvert kamera skal lagre, og om lukkeren skal utløses én gang, kontinuerlig eller med et bestemt intervall. Denne siden dokumenterer hele prosessen – konfigurasjon, selve opptaket, hvor filene lagres på disken og hvordan de senere kan bearbeides til kalibrerte produkter. Selve kontrollene for kamera og oppstillinger finnes under [Kamerainnstillinger](camera-settings.md).

{% hint style="info" %}
**Opptak krever at et prosjekt er åpent.** «Capture All» og tannhjulet for opptaksinnstillinger er deaktivert inntil et prosjekt er åpent («Opprett eller åpne et prosjekt for å lagre opptak»). Hvert opptak lagres under prosjektmappen i `captures/`.
{% endhint %}

## Ruten «Opptakinnstillinger»

Åpne den med **tannhjulet ved siden av «Ta opp alt»**i kameralisten i sidepanelet, eller med**knappen «Åpne opptakinnstillinger…»** nederst i en hvilken som helst innstillingsrute for et enkelt kamera. Overskriften lyder «Opptakinnstillinger» med en ←-tilbake-knapp.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->



Valgene dine her – inkluderte kameraer, avmerkingsbokser per type og opptaksmodus – lagres **per prosjekt** og gjenopprettes når du åpner det på nytt.

### Opptaksmoduser

Tre modusknapper øverst i ruten:

| Modus | Hva den gjør | Underinstillinger (standard) |
| --- | --- | --- |
| **Enkelt** *(standard)* | Én opptak på tvers av alle valgte kameraer. | — |
| **Kontinuerlig**| Opptak etter hverandre inntil en stoppbetingelse. | Stopp ved**antall opptak** (standard 1) *eller* **opptaksvarighet** (standard 10 s; enheter: sekunder / minutter / timer / dager). |
| **Intervall**(tidsforløp) | Serier av opptak etter en tidsinnstilling. |**Opptak / intervall**(standard 1) ·**Hvert**N enhet (standard 5 s) ·**I** N enhet (standard 1 m). |

I kontinuerlig- eller intervallmodus blir «Ta alt»-knappen til en **Stopp (N)**-knapp mens opptaket pågår, og teller opptakene etter hvert som de blir tatt. En fullt opptatt harddisk stopper kjøringen (se [«Capture All»-flyten](#the-capture-all-flow)).**Kontinuerlig på matriser kjører inne i backenden (nytt i 1.3.0).** Når alle kameraene i listen tilhører et array (frittstående kameraer er skjult eller ekskludert), sender «Kontinuerlig» ikke lenger én forespørsel per opptak: Chloros-backenden kjører opptaksløkken selv, skriver én synkronisert gruppe mens den velger den neste, og knappen avleser antallet. Med frittstående kameraer i listen tar Chloros opptak pass for pass som før.

{% hint style="warning" %}
**Rask opptak på Windows: hold Chloros-backenden ekskludert fra Microsoft Defender.** Sanntidsskanning av hver fil som et opptak med høy hastighet skriver, koster omtrent 30 % av opptakshastigheten i MAPIR’ tester (hundrevis av filer i sekundet ved et lite ROI). Windows-installasjonsprogrammet tilbyr **«Ekskluder Chloros-backenden fra Microsoft Defender-skanning (mye raskere kameraopptak; anbefalt)»**, som er merket av som standard, og fjerner ekskluderingen ved avinstallering. Hvis du har fjernet merket, må du kjøre installasjonsprogrammet på nytt med det merket av, eller legge til et unntak for Defender-prosessen for `chloros-backend.exe` (i mappen `backend` i Chloros-installasjonsmappen). Når Defender bruker én eller flere CPU-kjerner under kontinuerlig opptak fra et array, registrerer Chloros-backenden dette én gang i loggen sin, med de samme to løsningene.
{% endhint %}

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->




### Velge kameraer og eksporttyper

Hjelpeteksten i ruten oppsummerer det: velg hvilke kameraer og eksporttyper «Capture All» skal produsere – alt er aktivert som standard, og valgene lagres sammen med dette prosjektet.

* **«Velg alle» / «Ikke velg noen»-knappene** slår av og på avmerkingsboksene for hvert kamera samtidig.
* **Bulk-brytere for eksporttyper**(to rader med knapper):**Alle Raw / Alle Debayered / Alle forhåndsvisning / Alle Radiance / Alle Reflectance / Alle Index**. Hver av dem har tre fargestadier: grønn ✓ = aktivert for alle kameraer som støtter det, gul – = aktivert for noen, grå = ingen. En bryter er deaktivert når ingen tilkoblede kameraer støtter den typen. Alle blir grå når «Raskeste opptak» er aktivert.
* **Rader per kamera**: en avkryssingsboks for inkludering, pluss en utvidbar (▸/▾) liste over eksporttyper som gjelder for det aktuelle kameraet, med individuelle avkryssingsbokser. Raden viser et antall som er aktivert, for eksempel «4/6».

### Eksporttyper og hvilke kameraer som støtter dem

Det finnes seks eksporttyper: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Bare de aktuelle vises i raden for hvert kamera:

| Eksporttype | Innhold | RGB (FRGB) | Bayer-multispektral (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Bayer-mosaikk (mono: det ene båndet) direkte fra sensoren | ✓ | ✓ | ✓ |
| **Debayered** | Lineær demosaik: kalibreringspakkens mørkeforskyvning (DSNU) fjernet fra rådataene, deretter en kantbevisst demosaik — ingen linearitetskorreksjon, flatfelt eller unmix. Lagret som en 3-kanals RGB-TIFF-fil (mono: 1-kanals gråtoner) | ✓ | ✓ | ✓ |
| **Forhåndsvisning** | Full visningskjede (hvitbalanse + gamma i henhold til kameraets profil, pluss RGB-kameraets «Highlights»-innstilling; multispektral: strekking av falske farger) | ✓ | ✓ | ✓ |
| **Strålingsintensitet** | float32 W/m²/sr/nm via den fullstendige radiometriske kjeden | — (tilbys ikke) | ✓ | ✓ |
| **Refleksjonsgrad** | uint16 ρ (32768 = 1,0) | — (tilbys ikke) | ✓ — vises kun når kameraet har en DAQ-lyssensor (egen eller arvet fra matrisen) | det samme som multispektral |
| **Indeks** | Vegetasjonsindeks (LUT) gjengivelse | — | ✓ — krever et aktivert, ikke-tomt indeksuttrykk på kameraet, og tilbys ikke til medlemmer av kombinerte matriser (matrisen har én felles indeks) | — (et indeks krever ≥2 bånd; se [Monokameraer og vegetasjonsindekser](mono-indices.md)) |

Stråling og reflektans tilbys aldri for RGB-kameraer — stråling per Bayer-piksel er ikke meningsfullt for en bredbåndsfotometrisk sensor.

### Raskeste opptak

Vippebryteren **⚡ Raskeste opptak — kun RAW**(oransje når den er på) overstyrer alle eksportvalg til**kun RAW** — pluss en gratis sammensatt indeks for matriser — slik at bildet lagres så raskt som mulig: beregningene for radianse/reflektans/visning hoppes helt over ved opptakstidspunktet.

{% hint style="info" %}
**En `.daq` lagres fortsatt.** Når en lyssensor er tilordnet, skriver «Raskeste opptak» fortsatt ned DAQ-målingen for nedadgående lys (i opptakets `daq/`-mappe) — slik at strålings-, refleksjons- og indeksprodukter alle kan bygges senere ved ombehandling (se [Ombehandling av opptak](#re-processing-captures-into-calibrated-products)). «Fastest Capture» påvirker heller ikke valgene du har gjort i avmerkingsboksene: slå den av, så kommer de tilbake.
{% endhint %}

### Kontroller per matrise

Hver tilkoblet matrise får sitt eget gruppekort i ruten:

* **Avkrysningsruten «Inkluder»** (tre tilstander på tvers av medlemmene) og matrisens navn med visningsmodus: «(kombinert | separat)».
* **Justert**-avkryssingsrute (standard**på**): forvrenger eksporten av medlemmene til matrisens justeringsprofil, slik at eksportene blir pikselregistrert på tvers av kameraene. Raw-data forblir uforvrengt, men bærer transformasjonen i metadataene. (Selve profilen beregnes i [panelet for arrayinnstillinger](camera-settings.md#alignment-co-registration).) Filene til arraymedlemmene lagres i hvert kameras opprinnelige orientering, og eventuelle innstillinger for speiling/rotasjon lagres i filen i stedet for å bli brukt.
* Kameraradene i matrisen er nestet inne i kortet.

Matrisekortet inneholder også to opptakere. Tenk på dem som **overvåking vs. analyse**:

| Opptaker | Grad | Hva den tar opp |
| --- | --- | --- |
| **● Ta opp indeksvideo / ■ Stopp opptak** *(kun kombinerte matriser)* | **Overvåking** | Den kombinerte indekskompositten i sanntid til video med 10 fps — 8-bit, forhåndsvisningsoppløsning, LUT innebygd. Krever et åpent prosjekt og en strømmende livevisning. Viser bilder + forløpt tid under opptak. |
| **⦿ Ta opp rå burst / ■ Stopp rå burst** *(alle matriser)* | **Analyse**| Rå Bayer-bilder med live-opptakshastighet (uten behandling) pluss et manifest per bilde og `.daq`-målinger, til `captures/bursts/`. Etter en serie vises en**Lag video**-knapp: den bearbeider serien offline til kalibrert video — kombinert indeks og/eller radianse / reflektans / indeks per kamera — pluss valgfrie TIFF-filer. Opprettelsen av den kombinerte indeksen starter automatisk når du stopper serien. |

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

## «Capture All»-prosessen

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->



Trykk på **Capture All** i kameralisten i sidepanelet:

1. Hvert inkludert, synlig kamera som ikke er satt på pause, tar opp med de valgte eksporttypene. **Kameraoppstillinger utløses som én synkronisert utløser** (en enkelt synkronisert gruppe på tvers av alle medlemmene — se [Multi-Camera Arrays](arrays.md)); frittstående kameraer tar opp individuelt.
2. Skjulte (øye) eller pausede kameraer hoppes over. En gruppe er kun fullstendig blokkert når *alle* medlemmene er skjult eller pauset.
3. Når en lyssensor er tilordnet, lagres den tilsvarende DAQ-målingen av nedadgående lys som en `.daq`-fil i opptakets `daq/`-mappe – selv for opptak som kun består av rådata – slik at radiometriske produkter alltid kan utledes senere.
4. Knappen viser fremdriften i sanntid – «Capturing… done/total» – og blir til **Stop (N)** i kontinuerlig/intervallmodus. Hvert opptakselement har en tidsavbrudd på 300 s.
5. Når gjennomgangen er fullført, viser et resultatvindu **«Lagret N filer»**eller**«Lagret N, F mislyktes»**, samt «(S skjult/pauset/hoppet over)» når kameraer ble hoppet over. Hvis harddisken blir full, stopper kjøringen og viser meldingen**«Stoppet: harddisken er full. Lagret N, F mislyktes»** — hver fil som ikke kunne skrives, regnes som mislykket.

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
├── clamp_masks/   radiance/…_clampmask.tif, reflectance/…, index/… — unmix clamp masks,
│                  one subfolder per product folder, only when a band was clamped
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── daq/           the downwelling reading(s) matched to each capture (.daq), each with a
                   .daq.cameras.json naming the cameras it describes
```

* `<ts>` er opptakets tidsstempel og `<serial>` er kameraets serienummer. Frittstående opptak får navnet `capture_<ts>_SN<serial>_<level>`; serieopptak fra én synkronisert utløser får navnet `sync_<ts>_SN<serial>_<level>` og **deler én tidsstempel på tvers av alle kameraene i gruppen** (nivåsuffikset fjernes når et kamera kun lagrer ett enkelt nivå).
* **En asymmetri å være klar over:** visningsnivået lagres i en mappe med navnet `preview/`, mens filene beholder `_display` i navnet — mappen og suffikset avviker kun for dette nivået.
* Ukjente nivåer lagres i en mappe med sitt eget navn; hvis det ikke er mulig å opprette en undermappe, skrives filen til roten for opptakene i stedet for å gå tapt.
* Opptakene i TIFF-format komprimeres som standard uten tap (DEFLATE) og inneholder alle metadataene for kalibrering og behandling **inne i filens XMP** — opptakene er selvbeskrivende, uten andre sidecar-filer enn `.daq`-avlesningen (og dens `.daq.cameras.json`).
* **Klemmasker (nytt i 1.3.0).**Der den 3×3 spektrale separasjonen måtte klemme et bånd ned til 0 på minst én piksel, får radianse-, reflektans- og indeksproduktene en maske per piksel, `clamp_masks/<product folder>/<product name>_clampmask.tif` (bit 0 = båndet i produktets R-kanal, bit 1 = G, bit 2 = B); produktets `Chloros:UnmixClampMask` XMP-tag peker til den. Maskene ligger**ved siden av** produktmappene, aldri inne i en, så en produktmappe som for eksempel `reflectance/` kan overføres til fotogrammetri-programvare slik den er. Chloros’ egen import hopper over maskene.
* **Orientering (1.3.0).** For et frittstående kamera roteres produktene «Preview», «Radiance», «Reflectance» og «Index» i henhold til innstillingen «Mirror/Rotation»; «Raw» og «Debayered» forblir slik sensoren leste dem. Alle filene i arrayet forblir i opprinnelig format. Hver fil lagrer innstillingen i sin XMP, slik at den brukes på samme måte ved ny behandling.
* **Hvilket kamera en `.daq` tilhører (1.3.0).** Et opptakets `.daq` blir ved behandling kun matchet mot bilder fra kameraene som dets `.daq.cameras.json` navngir. Et frittstående kamera uten egen lyssensor låner derfor ikke lenger avlesningen fra et annet kamera — knytt en sensor til det, eller registrer en i fanen «Lyssensorer». En lyssensorregistrering (og enhver `.daq` uten den filen, inkludert eldre opptak) betjener fortsatt hvert kamera.

Dette er den samme strukturen som `chloros-cli lattice capture` / `array-capture` skriver til sin `-o`-katalog — dokumentert i [CLI-referanse § Hvordan en opptakskatalog ser ut](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, the daq/ and clamp_masks/ folders, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Ombehandling av opptak til kalibrerte produkter

Innsamlede råbilder pluss den lagrede `.daq` er alt behandlingsrørledningen trenger – dette er grunnen til at «Fastest Capture» er trygt å bruke i reelt arbeid.

* **GUI**: legg til opptaksmappen i et prosjekt ([Legge til filer i et prosjekt](../processing-images-gui/adding-files-to-a-project.md)) og behandle som vanlig.
* **CLI**: pek `process` mot **opptaksrotmappen**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` importerer normalt bare mappen du angir, men når den mappen ikke inneholder bilder og har undermapper, går den automatisk ned i undermappene — slik at undermappene på det aktuelle nivået og mappen `daq/` hentes inn på én gang. Hver innlasting importeres som et **enkelt bilde** med de andre nivåene knyttet til som visningsmoduser, ikke som ett bilde per nivå.

Å navngi en undermappe på et nivå direkte (f.eks. `…/captures/raw/`) fungerer også, men etterlater mappen `daq/` – kopier den ved siden av når du-utleder et radiometrisk produkt fra `raw/`, ellers har tidsstempel-sammenligningen ingenting å sammenligne mot.

{% hint style="warning" %}
**Behandlingen starter alltid fra `raw`.** Innenfor hver opptaksserie er råbildet kilden til behandlingskjeden; `debayered`, `radiance`, `reflectance`, og `preview` følger med som visningsmoduser, men føres aldri tilbake gjennom rørledningen – å behandle et avledet produkt på nytt ville innebære å påføre vignett, farge og strålingsberegninger som allerede er innbakt i pikslene på nytt, så Chloros-programmet avviser dette i stedet for å utføre dobbel behandling. Renderingene `index/` og `composite/` blir aldri behandlet i det hele tatt (de er utdata, ikke opptak). En «captures»-mappe som er lagret **uten** råimport, vises normalt, men `process` hopper over den og opplyser om dette; `--input-level {raw,debayered,processed}` er den bevisste nødutgangen som tvinger frem et inngangspunkt. Se [CLI-referansen](../reference/cli-reference.md#what-a-captures-folder-looks-like) for de nøyaktige hoppemeldingene.
{% endhint %}

To andre oppførsler det er verdt å kjenne til når man skriver skript for nybehandling:

* En `chloros-cli process`-kjøring som ba om produkter, men som **ikke skrev noen bildeprodukter, mislykkes med høylydt feilmelding og avsluttes med et tall større enn null** — du vil aldri få en stille, tom kjøring. Vellykkede kjøringer rapporterer antall produkter. (En bevisst kjøring med kun metadata regnes fortsatt som vellykket.)
* Reimporterte, bearbeidede eksporter tar aldri en opptakets rådata-plass — den opprinnelige rådataen forblir alltid kilden i behandlingskjeden.

## CLI-ekvivalenter

Alt på denne siden kan kjøres uten grafisk grensesnitt. Opptaksmodusene i GUI-en tilsvarer direkte `chloros-cli lattice array-capture`:

| GUI | CLI |
| --- | --- |
| Enkelt | `chloros-cli lattice array-capture` |
| Kontinuerlig | `array-capture --continuous [--count N] [--duration S]` |
| Intervall | `array-capture --interval S [--duration S]` |
| Raskest opptak | `array-capture --fastest` |
| Justert avkryssingsrute | `--aligned / --no-aligned` |
| Avkrysningsruter for eksporttype | `--processing LEVEL` eller `--levels L1,L2,…` (standard `all`) |
| Ta opp indeksvideo | `chloros-cli lattice array-record` |
| Ta opp rå bildeserie / Lag video | `chloros-cli lattice array-burst` / `array-build-video` |

Fullstendige flaggtabeller, alternativet for smart-AE-fastsatt opptak (`--smart`) og modellen for vedvarende hastighet finnes i [CLI-referanse § Opptaksmoduser, opptakere og offline-bearbeiding](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).
