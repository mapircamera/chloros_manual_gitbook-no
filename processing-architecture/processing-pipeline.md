# Behandlingsrørledning

Chloros 1.1.0 bruker en behandlingsrørledning med fire tråder som fungerer som et trinnvis samlebånd. Hver tråd håndterer en egen fase i behandlingsarbeidsflyten, slik at flere bilder kan behandles samtidig i ulike trinn.

***

## Rørledningsarkitektur

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Hvert bilde går gjennom alle fire tråder i rekkefølge. Med Chloros+ flertrådet behandling kan flere bilder være i forskjellige tråder samtidig – mens tråd 3 behandler ett bilde, kan tråd 1 oppdage det neste, tråd 2 kan kalibrere et annet, og tråd 4 kan skrive et tidligere behandlet bilde til disken.

***

## Tråddetaljer

### Tråd 1: Deteksjon

**Formål**: Laster inn bilder og oppdager kalibreringsmål.

* Leser bildefiler fra disken (RAW, JPG)
* Henter ut EXIF-metadata (GPS, kameramodell, tidsstempler, eksponering)
* Oppdager ArUco-kalibreringsmål i merkede målbilder
* Utdata: bildedata + metadata + resultater fra måloppdagelse

Dette er primært en I/O- og CPU-avhengig tråd.

### Tråd 2: Kalibrering

**Formål**: Beregne kalibreringsparametere fra oppdagede mål.

* Beregner reflektanskalibreringskoeffisienter fra målbilder
* Beregner vignettkorreksjonsparametere
* Bestemmer kalibreringskurver per bånd
* Utdata: kalibreringsparametere for hvert bilde

Dette er en CPU-avhengig beregningstråd.

### Tråd 3: Behandling (GPU)

**Formål**: Anvende korreksjoner og beregne vegetasjonsindekser.**Dette er den mest beregningsintensive tråden.*** **Debayering**: Konverterer RAW-data med Bayer-mønster til flerkanalsbilder
  * Standard (rask, middels kvalitet) — standard
  * Teksturbevisst (langsom, høyeste kvalitet) — kun Chloros+, bruker AI/ML-støyreduksjon
* **Vignettkorreksjon**: Anvender objektivvignettkorreksjon over hele bildet
* **Refleksjonskalibrering**: Bruker kalibreringskoeffisienter for å konvertere til refleksjonsverdier
* **Indeksberegning**: Beregner vegetasjonsindekser (NDVI, NDRE, GNDVI, etc.)
* Utdata: behandlede bildedata klare for eksport

Denne tråden drar mest nytte av GPU-akselerasjon. Systemet [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) optimaliserer primært denne trådens oppførsel.

### Tråd 4: Eksport

**Formål**: Skrive behandlede bilder til disk.

* Skriver utdatafiler i det valgte formatet (TIFF 16-bit, TIFF 32-bit %, PNG, JPG)
* Legger inn EXIF-metadata i utdatafilene (GPS, tidsstempler, behandlingsparametere)
* Organiserer utdataene i undermapper etter kameramodell
* Utdata: endelige filer på disken

Dette er primært en I/O-avhengig tråd. SSD-lagring forbedrer ytelsen til tråd 4 betydelig.

***

## Sekvensiell vs. pipelinert behandling

### Gratis modus (sekvensiell)

I gratisversjonen av Chloros behandles bildene **ett om gangen**, sekvensielt gjennom alle fire trinn:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

Fremdriftslinjen i brukergrensesnittet viser to trinn: Målregistrering og Behandling.

### Chloros+-modus (pipelinert)

Med en Chloros+-lisens opererer alle fire tråder **samtidig** på forskjellige bilder:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

Fremdriftslinjen i brukergrensesnittet viser 4 trinn: Deteksjon, analyse, kalibrering, eksport. Hold musepekeren over fremdriftslinjen for å se fremdriften per tråd.

{% hint style="success" %}
**Pipelined-behandling med Chloros+** kan være 3–5 ganger raskere enn sekvensiell behandling, avhengig av maskinvaren og størrelsen på datasettet. Hastighetsøkningen er størst på systemer med raske GPU-er og SSD-er.
{% endhint %}

***

## Fremdrift for eksport i tråd 4

I Chloros 1.1.0 har eksporttråden (tråd 4) sin egen dedikerte fremdriftssporing. Du kan overvåke eksportfremdriften separat:**CLI:**
```bash
chloros-cli export-status
```

**SDK:**
```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Behandlingen er fullført når tråd 4 når 100 %.

***

## Forholdet til dynamisk beregningsadaptasjon

[Dynamisk beregningsadaptasjon](dynamic-compute-adaptation.md)-systemet påvirker primært **tråd 3 (behandling)**:

* **`GPU_PARALLEL`**-strategi: Tråd 3 kjører flere bilder gjennom GPU-en samtidig ved hjelp av `fused_gpu`-rørledningen
* **`GPU_SINGLE`**-strategi: Tråd 3 behandler ett bilde om gangen ved hjelp av den minneeffektive `tiled_gpu`-rørledningen
* **`CPU_PARALLEL`**-strategi: Tråd 3 bruker CPU-basert behandling med flertrådet parallellitet

Tråd 3s GPU-minnetildeling endres også dynamisk etter hvert som tråd 1 og 2 fullføres — se [Dynamisk GPU-minnetildeling](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Neste trinn

* [Dynamisk beregnings tilpasning](dynamic-compute-adaptation.md) — Hvordan Chloros velger den optimale strategien for maskinvaren din
* [NVIDIA Jetson-veiledning](../linux/nvidia-jetson-guide.md) — Plattformspesifikk pipeline-atferd på Jetson
* [Overvåking av behandlingen](../processing-images-gui/monitoring-the-processing.md) — Overvåking av fremdrift via GUI
