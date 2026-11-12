# Behandlingsrørledning

Chloros 1.3.0 bruker en behandlingsrørledning med fire tråder som fungerer som et trinnvis samlebånd. Hver tråd håndterer en egen fase i arbeidsflyten, slik at flere bilder kan være under behandling i ulike faser samtidig.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Rørledningsarkitektur

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Hvert bilde går gjennom alle fire trådene i rekkefølge. Med Chloros' flertrådede behandling opptar flere bilder forskjellige tråder samtidig – mens tråd 3 behandler ett bilde, kan tråd 1 oppdage det neste, tråd 2 kalibrere et annet, og tråd 4 skrive et ferdig bilde til disken.

Fremdriften rapporteres per tråd og strømmes via Server-Sent Events (backenden publiserer dem på `/api/events`). I CLIs live-fremdriftsvisning er de fire trinnene merket **Detektering, Analyse, Behandling, Eksport**.

***

## Tråddetaljer

### Tråd 1: Deteksjon

**Formål**: Laste inn bilder og oppdage kalibreringsmål.

* Leser inn bildefiler fra disken — Survey3-parene `.raw`+`.jpg`, LATTICE `.tif`/`.tiff`-opptak og `.dng`
* Henter ut EXIF-metadata (GPS, kameramodell, tidsstempler, eksponering)
* Oppdager kalibreringsmål: ArUco-merkede målgeometrier for LATTICE-opptak, og den klassiske paneldetektoren for Survey3-kalibreringsmålbilder
* Utdata: bildedata + metadata + resultater fra måldeteksjon

Hovedsakelig en I/O- og CPU-avhengig tråd.

### Tråd 2: Kalibrering

**Formål**: Beregne kalibreringsparametere fra oppdagede mål.

* Beregner reflektanskalibreringskoeffisienter fra målbilder
* Beregner vignettkorreksjonsparametere
* Bestemmer kalibreringskurver per bånd
* Utdata: kalibreringsparametere for hvert bilde

En CPU-avhengig beregningstråd. Ved en Survey3-kjøring eller en blandet kjøring venter tråd 3 på denne når reflektanskalibrering er aktivert, slik at koeffisientene er klare før noe bilde behandles. Ved en ren LATTICE-kjøring kjører tråd 3 under deteksjonen: hvert bilde starter så snart dets målanker er endelig fastsatt. Et bilde tar det nærmeste målet i tid på hver side, så den venter til hvert bilde fra kameraet som ligger nærmere i tid enn det nåværende nærmeste målet, er skannet. Resultatene er de samme, bare tidligere.

### Tråd 3: Behandling (GPU)

**Formål**: Anvende korreksjoner og beregne vegetasjonsindekser.**Dette er den mest beregningsintensive tråden.*** **Debayering**: konverterer RAW-Bayer-data til flerkanalsbilder
  * Standard (rask, middels kvalitet) — standard, `--debayer standard`
  * Teksturbevisst (langsom, høyeste kvalitet) — kun Chloros+, `--debayer texture-aware`, bruker en AI/ML-støyreduksjonsmodell
  * LATTICE mono (M3M)-opptak er enkeltbånds: demosaik- og hvitbalansetrinnene hoppes over for disse (med en loggmelding på én linje), mens eventuelle M3C/Bayer-bilder i samme kjøring fortsatt gjennomgår disse trinnene
* **Vignettkorreksjon**: bruker objektivvignettkorreksjon på hele bildet
* **Refleksjonskalibrering**: bruker kalibreringskoeffisienter for å konvertere til refleksjonsverdier
* **Indeksberegning**: beregner vegetasjonsindekser (NDVI, NDRE, GNDVI, …)
* Utdata: bearbeidede bildedata klare for eksport

Denne tråden drar størst nytte av GPU-akselerasjon, og det er denne tråden [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) justerer.

### Tråd 4: Eksport

**Formål**: Skrive behandlede bilder til disk.

* Skriver utdatafiler i det valgte formatet — `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Legger inn metadata i utdatafilene (GPS, tidsstempler, behandlingsparametere) — inkludert `Chloros:*`-behandlingsmerkene, som fra og med versjon 1.3.0 skrives i alle behandlingsmoduser, både for Survey3 og LATTICE
* Organiserer utdata under prosjektmappen som `<camera>/<format>/<Product>_Images/` — for eksempel `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Eksporterte filer beholder kildefilens navn; mappen identifiserer produktet.**
* For LATTICE-opptak kan ett kildebilde deles opp i flere produkter (Debayered, Preview, Radiance, Reflectance, Index), hvert i sin egen produktmappe. Indeksprodukter beregnes ut fra refleksjonsproduktet som kjøringen skrev (når det ikke finnes noe, fra et annet produkt som kjøringen skrev, for eksempel radiance). Radiance- og reflektansprodukter fra multispektrale M3C-kameraer kan også få en klampemaske per piksel i `<format>/clamp_masks/` (se [Utdataformater for bilder](../output-image-formats.md#clamp-masks))
* Utdata: endelige filer på disken

Hovedsakelig en I/O-avhengig tråd — SSD-lagring gir merkbar forbedring.

***

## Under panseret: Utførere

Innenfor tråd 3 parallelliseres arbeidet per bilde som følger:

* **GPU-strategier**(`GPU_SINGLE`, `GPU_PARALLEL`) bruker en pool av**arbeidsprosesser**, hver en egen OS-prosess med sin egen CUDA-kontekst. Chloros starter hver arbeider ved å kjøre sitt eget installerte backend-program på nytt – ikke gjennom Pythons `multiprocessing`, da oppstarten av arbeiderne ikke fungerer inne i den pakkede applikasjonen
* **`CPU_PARALLEL`** bruker en trådpool (Pythons `ThreadPoolExecutor`) — NumPy og OpenCV frigjør GIL, så tråder er tilstrekkelig
* Jetson-enheter med 8 GB eller mindre delt RAM hopper helt over eksekutoren og behandler sekvensielt i prosessen
* Texture Aware på en GPU med under 7 GB VRAM kjører også sekvensielt — støyreduksjonsmodellen får ikke plass mer enn én gang

Chloros bruker ikke noe distribuert rammeverk fra tredjepart (som for eksempel Ray). Se [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) for hvordan strategien og antall arbeidere velges.

***

## Sekvensiell vs. pipelinert behandling

### Fri modus (sekvensiell)

I gratisversjonen av Chloros behandles bildene **ett om gangen**, sekvensielt gjennom alle fire trinnene:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

GUI-en viser en forenklet fremdriftslinje i fri modus; de sekvensielle fasene vises som **Målgjenkjenning**og deretter**Behandling**.

### Chloros+-modus (pipeline)

Med en Chloros+-lisens arbeider alle fire tråder **samtidig** på forskjellige bilder:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

Fremdriftslinjen i GUI-et viser de fire trinnene; hold musepekeren over den for å se fremdriften per tråd. I CLI vises de samme fire trinnene i sanntid som **Detecting, Analyzing, Processing, Exporting**.

{% hint style="info" %}
**Én betegnelse, to navn.** CLI-grensesnittet kaller trinn 3 for _Behandling_. Backendens fremdriftsfeed i premium-modus – den som fremdriftslinjen i GUI-grensesnittet viser – merker det samme trinnet som _Kalibrering_. Det er den samme tråden som utfører det samme arbeidet (Tråd 3: debayer, korreksjoner, indekser).
{% endhint %}

{% hint style="success" %}
**Pipelined-behandling med Chloros+** kan være 3–5 ganger raskere enn sekvensiell behandling, avhengig av maskinvaren din og størrelsen på datasettet. Hastighetsøkningen er størst på systemer med raske GPU-er og SSD-er.
{% endhint %}

***

## Tråd 4: Eksportfremdrift

Eksporttråden har sin egen fremdriftssporing, som du kan avlese separat:

**Kommandolinje (CLI):**

```bash
chloros-cli export-status
```

**SDK:**

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Behandlingen er fullført når tråd 4 når 100 %.

{% hint style="info" %}
**En kjøring som ikke skriver noen bilder, regnes som mislykket.**Ved vellykket gjennomføring rapporterer `chloros-cli process` hvor mange bildeprodukter den har skrevet (`Image products written: N`). Hvis det ble forespurt produkter og**ingen**ble skrevet — kun `project.json` og `calibration_data.json` — skriver CLI-en ut `Processing finished but wrote no image products.` og**avsluttes med en verdi ulik null**, og oppgir prosjektmappen og de vanlige årsakene (inndatamappen ble ikke gjenkjent som en opptak – sjekk oppsettet og `--input-level` – eller at alle forespurte produkter var uanvendelige for disse kameraene). Skript kan stole på avslutningskoden.
{% endhint %}

***

## Forholdet til dynamisk beregnings tilpasning

[Dynamisk beregnings tilpasning](dynamic-compute-adaptation.md) påvirker først og fremst **Tråd 3 (Behandling)**:

* **`GPU_PARALLEL`**: Tråd 3 kjører flere bilder gjennom GPU-en samtidig ved hjelp av `fused_gpu`-rørledningen
* **`GPU_SINGLE`**: Tråd 3 serialiserer GPU-tilgang med en semafor mens arbeidsprosesser overlapper I/O, ved hjelp av `fused_gpu` eller den minneeffektive `tiled_gpu`-rørledningen
* **`CPU_PARALLEL`**: Tråd 3 bruker CPU-basert behandling med flertrådet parallellitet

Tråd 3s GPU-minnetildeling øker også etter hvert som tråd 1 og 2 fullfører oppgavene sine — se [Dynamisk GPU-minnetildeling](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Neste trinn

* [Dynamisk beregnings tilpasning](dynamic-compute-adaptation.md) — Hvordan Chloros velger den optimale strategien for din maskinvare
* [NVIDIA Jetson-veiledning](../linux/nvidia-jetson-guide.md) — Plattformsspesifikk pipeline-atferd på Jetson
* [Overvåking av behandlingen](../processing-images-gui/monitoring-the-processing.md) — Overvåking av fremdrift via grafisk brukergrensesnitt
* [Referanse for CLI](../reference/cli-reference.md) — `process`, `export-status`, avslutningskoder og utdataoppsett
