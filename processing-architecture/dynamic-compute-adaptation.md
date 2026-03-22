# Dynamisk tilpasning av databehandling

Chloros 1.1.0 introduserer intelligent maskinvaredeteksjon og automatisk valg av behandlingsstrategi. Behandlingsmotoren tilpasser seg maskinvaren din – fra en Jetson Nano til en arbeidsstasjon med flere GPU-er – uten manuell konfigurering.

***

## Slik fungerer det

Når Chloros starter, profilerer den automatisk systemet ditt:

1. **Gjenkjenner operativsystemet** — Windows eller Linux
2. **Identifiserer CPU-kjerner og totalt RAM**

3.**Oppdager tilstedeværelse av GPU** — NVIDIA CUDA-kapasitet, VRAM, modell
4. **Identifiserer Jetson-modell** (hvis aktuelt) — via `/proc/device-tree/model`
5. **Sjekker termiske sensorer** (Jetson) — for temperaturbevisst prosessering
6. **Velger den optimale beregningsstrategien** — basert på all oppdaget maskinvare
7. **Konfigurerer antall arbeidere, rørledningstype og minnetildeling** automatisk

Resultatet lagres i hurtigbufferen, slik at påfølgende kjøringer starter raskere. Hvis maskinvaren endres (f.eks. hvis en GPU legges til), profilerer Chloros på nytt ved neste oppstart.

***

## Beregningsstrategier

Chloros velger en av tre beregningsstrategier basert på maskinvaren din:

| Strategi | GPU påkrevd | Arbeidere | Rørledning | Best egnet for |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`** | Ja (12 GB+ VRAM eller 16 GB+ delt) | 3–4 | `fused_gpu` | Stasjonære GPU-er med 12 GB+, Jetson Orin NX 16 GB, AGX Orin |
| **`GPU_SINGLE`** | Ja (&lt; 12 GB VRAM) | 1–3 | `tiled_gpu` | GPU-er på innstegsnivå, Jetson Nano, Orin Nano |
| **`CPU_PARALLEL`** | Nei | kjerner - 1 | `cpu_fallback` | Systemer uten NVIDIA-GPU |

### Pipelinetyper

* **`fused_gpu`** — Full GPU-behandlingsbane. Alle debayer-, korreksjons- og indeksoperasjoner kjøres på GPU-en i ett enkelt sammensmeltet pass. Høyest gjennomstrømning, men krever mer VRAM.
* **`tiled_gpu`** — Minneeffektiv GPU-bane. Behandler bilder i fliser for å passe innenfor begrenset GPU-minne. Lavere gjennomstrømning, men fungerer på enheter med begrenset minne.
* **`cpu_fallback`** — Behandling kun på CPU ved hjelp av multitrådet parallellitet. Brukes når ingen NVIDIA-GPU er tilgjengelig.***

## Plattformsspesifikk oppførsel

| Plattform | Strategi | Arbeidere | Rørledning | Merknader |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (serialisert) | Minneeffektiv modus, behandler ett bilde om gangen |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` (samtidig) | Anbefalt kant-enhet — ekte parallell GPU-behandling |
| **Jetson AGX Orin 64 GB** | `GPU_PARALLEL` | 4 | `fused_gpu` (samtidig) | Maksimal ytelse i kantnettverk |
| **Stasjonær PC med 8 GB GPU** | `GPU_SINGLE` | 3 | `tiled_gpu` | God stasjonær ytelse med minneeffektive fliser |
| **Stasjonær PC med 12 GB+ GPU** | `GPU_PARALLEL` | 3–4 | `fused_gpu` | Optimal stasjonær ytelse |
| **System kun med CPU** | `CPU_PARALLEL` | kjerner - 1 | `cpu_fallback` | Ingen GPU nødvendig, bruker ThreadPool |

{% hint style="info" %}
**Jetson unified memory**: Jetson-enheter deler GPU- og CPU-minne. En Jetson Orin NX 16 GB rapporterer ~15,3 GB VRAM, men dette er det samme fysiske RAM-minnet som brukes av operativsystemet og CPU-prosessene. Chloros tar hensyn til dette når terskler for minnetildeling settes.
{% endhint %}

***

## Dynamisk GPU-minnetildeling

Chloros bruker en [4-tråds behandlingspipeline](processing-pipeline.md):

* **Tråd 1** (Deteksjon) — Bildelasting, EXIF-parsing, måldeteksjon
* **Tråd 2** (Kalibrering) — Beregning av refleksjonskalibrering
* **Tråd 3** (Behandling) — GPU-debayer, vignettkorrigering, indeksberegning
* **Tråd 4** (Eksport) — Filskriving, innbygging av metadata

Når tidligere tråder i prosesskjeden fullfører arbeidet sitt (f.eks. når alle bilder er oppdaget), frigjøres deres GPU-minnetildeling og **omfordeles til de gjenværende aktive trådene**. Dette betyr at tråd 3 (det GPU-intensive stadiet) får stadig mer minne etter hvert som pipelinen går fremover, noe som forbedrer gjennomstrømningen for det mest beregningsintensive arbeidet.

### Tildelingsstadier

| Stadium | Aktive tråder | GPU-minnefordeling |
| --- | --- | --- |
| **Tidlig** | 1, 2, 3, 4 | Fordelt på alle tråder |
| **Midt-tidlig** | 2, 3, 4 | Tråd 1s minne omfordelt |
| **Midt-sen** | 3, 4 | Tråd 1+2s minne går til 3+4 |
| **Sen** | 3 eller 4 | Maksimalt minne for gjenværende tråd |***

## Teksturbevisst behandling

Den teksturbevisste debayer-metoden (kun Chloros+) bruker betydelig mer GPU-minne enn standardmetoden på grunn av AI/ML-støyreduksjonsmodellen:

* Systemer med **&lt; 7 GB VRAM** tvinges inn i en synkron behandlingssløyfe for teksturbevisst modus (ett bilde om gangen)
* Systemer med **7 GB+ VRAM** kan behandle teksturbevisst samtidig, men med redusert antall arbeidere sammenlignet med standard***

## Termisk styring (Jetson)

Jetson-enheter har termiske begrensninger, spesielt i lukkede eller luftbårne installasjoner. Chloros overvåker GPU- og CPU-temperaturer og justerer behandlingen automatisk:

| Temperatur | Respons |
| --- | --- |
| **&lt; 70 °C** | Normal drift — full hastighet |
| **70 °C** (Advarsel) | Reduser batchstørrelse |
| **80 °C** (Kritisk) | Aggressiv begrensning — reduser samtidigheten og antall arbeidere |
| **90 °C** (Avstengning) | Stopp GPU-behandlingen helt |

Temperaturovervåking bruker `tegrastats` på Jetson-plattformer. På stasjonære systemer med tilstrekkelig kjøling utløses termisk begrensning sjelden.

***

## Håndtering av minnepress

Chloros overvåker systemminnepresset under behandling:

* **Minneterskel**: 85 % utnyttelse utløser konservativ oppførsel
* **OOM-reduksjon**: Hvis det oppstår en minnefeil, reduseres tildelingen med 25 % (0,75x multiplikator)
* **Pipeline-fallback**: Under alvorlig minnepress faller pipelinen automatisk tilbake fra `fused_gpu` til `tiled_gpu`
* **Anbefalinger om swap**: På Jetson varsler Chloros deg hvis swap-plassen er utilstrekkelig for størrelsen på datasettet ditt***

## Overvåking av beregnings tilpasning

### CLI-statusutdata

Når behandlingen starter, viser CLI den oppdagede maskinvareprofilen:

```

Chloros CLI 1.1.0
Platform: Linux aarch64 (Jetson Orin NX 16GB)
Strategy: GPU_PARALLEL | Workers: 3 | Pipeline: fused_gpu
CUDA: Available | GPU Memory: 15.3 GB (shared)
```

### Systemdiagnostikk

Kjør `chloros-cli selftest` for å se en fullstendig maskinvareprofil og verifisere beregningskapasiteten:

```bash
chloros-cli selftest
```

Dette sjekker CUDA-tilgjengelighet, GPU-minne, støyreduksjonsmodeller og backend-tilkobling.

***

## Neste trinn

* [Behandlingsrørledning](processing-pipeline.md) — Forståelse av 4-tråds rørledningsarkitekturen
* [NVIDIA Jetson-veiledning](../linux/nvidia-jetson-guide.md) — Jetson-spesifikk distribusjon og optimalisering
* [CLI : Kommandolinje](../CLI.md) — Fullstendig CLI-referanse
