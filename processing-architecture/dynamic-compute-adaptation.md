# Dynamisk tilpasning av databehandling

Chloros 1.2.0 benytter maskinvaredeteksjon og automatisk valg av behandlingsstrategi. Behandlingsmotoren tilpasser seg maskinvaren din — fra en Jetson Orin Nano til en arbeidsstasjon med flere GPU-er — uten behov for manuell konfigurering.

***

## Slik fungerer det

Når Chloros starter, profilerer det systemet ditt:

1. **Gjenkjenner operativsystemet** — Windows eller Linux
2. **Identifiserer CPU-kjerner og totalt RAM**

3.**Oppdager tilstedeværelse av GPU** — NVIDIA CUDA-støtte, VRAM, modell
4. **Identifiserer Jetson-modell** (hvis aktuelt) — via `/proc/device-tree/model`
5. **Sjekker temperatursensorer** (Jetson) — for temperaturbevisst behandling
6. **Velger beregningsstrategi** — basert på all oppdaget maskinvare
7. **Konfigurerer antall arbeidere, pipelinetype og minnetildeling** automatisk

Den oppdagede profilen lagres i hurtigbufferen for økten, både i minnet og på disken, slik at senere kjøringer starter raskere:

| Plattform | Profil i hurtigbuffer |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json` (prioriterer `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Slett denne filen for å tvinge frem en ny gjenkjenning — nyttig etter at du har lagt til en GPU eller mer RAM. Chloros gjenkjenner også automatisk på nytt når cachen ble skrevet av en inkompatibel eldre versjon.

***

## Beregningsstrategier

Chloros velger en av tre beregningsstrategier basert på maskinvaren din:

| Strategi | Velges når | Arbeidere | Utfører | Rørledning |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| CUDA-GPU som rapporterer**12 GB+ VRAM**(på Jetson unified memory, krever også 12 GB+ totalt delt RAM) | `min(4, VRAM ÷ 4GB)`, minimum 2 —**begrenset til 2 på Jetson** | `ProcessPoolExecutor` (spawn) | `fused_gpu` |
| **`GPU_SINGLE`**| CUDA-GPU med**2–12 GB VRAM**| 3 (I/O-overlapping; GPU-tilgang serialisert av en semafor).**1 (sekvensiell) på Jetson-enheter med under 12 GB RAM** | `ProcessPoolExecutor` (spawn); sekvensiell i samme prosess på Jetson-enheter med lite RAM | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Ingen CUDA-GPU, eller under 2 GB VRAM | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Eksempler på hvordan `GPU_PARALLEL`-arbeiderformelen fungerer: 12 GB VRAM → 3 arbeidere, 16 GB+ → 4 arbeidere, alle Jetson-modeller → 2 arbeidere.

Parallellitet implementeres med Pythons standard `concurrent.futures`: GPU-strategier bruker en `ProcessPoolExecutor` med **spawn**-startmetoden (hver arbeider er en egen prosess med sin egen CUDA-kontekst — `fork` ville kopiere en allerede initialisert CUDA-tilstand og ødelegge underprosessene), og CPU-strategien bruker en `ThreadPoolExecutor`. Chloros bruker ikke noe distribuert rammeverk fra tredjepart (som Ray).

### Rørledningstyper

* **`fused_gpu`** — Full GPU-behandlingsbane. Debayer-, korreksjons- og indeksoperasjoner kjøres på GPU-en i ett enkelt sammensmeltet gjennomløp. Høyest gjennomstrømning, krever mest VRAM.
* **`tiled_gpu`** — Minneeffektiv GPU-bane. Behandler bilder i fliser for å passe innenfor begrenset GPU-minne. Lavere gjennomstrømning, men fungerer på enheter med begrenset minne.
* **`cpu_fallback`** — Behandling utelukkende på CPU ved hjelp av multitrådet parallellitet. Brukes når ingen NVIDIA-GPU er tilgjengelig, og som siste utvei når begge GPU-prosesseringsveiene mislykkes.

Reservekjeden under kjøring er alltid `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Manuell overstyring av strategi

Sett miljøvariabelen `CHLOROS_STRATEGY` for å tvinge frem en bestemt strategi — en ekspertløsning for når automatisk gjenkjenning velger noe som ikke passer for din situasjon (for eksempel for å holde GPU-en ledig for annet arbeid):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

Variabelen samsvarer uten å skille mellom store og små bokstaver; alt som ikke er et av de tre navnene, ignoreres, og automatisk gjenkjenning fortsetter som normalt. Ved en overstyring velger Chloros fortsatt antall arbeidere for deg:

| Overstyring | Antall arbeidere brukt |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

Det anbefales å angi dette per kommando i stedet for permanent, slik at normale kjøringer fortsetter å tilpasse seg automatisk.

***

## Plattformsspesifikk oppførsel

| Plattform | Strategi | Arbeidere | Rørledning | Merknader |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekvensiell) | Minneeffektiv modus, ett bilde om gangen |
| **Jetson Orin NX 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekvensiell) | Delte RAM-minne under 12 GB tvinger frem sekvensiell behandling |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (samtidig) | Anbefalt kant-enhet — Jetson-begrenset til 2 arbeidere |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (samtidig) | Maksimal ytelse i kantnettverket (også begrenset av Jetson til 2 arbeidere) |
| **Stasjonær PC med 8 GB GPU** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 arbeidere overlapper I/O mens en semafor serialiserer GPU-tilgang |
| **Stasjonær PC med 12 GB+ GPU** | `GPU_PARALLEL` | 3–4 | `fused_gpu` (samtidige) | Optimal stasjonær ytelse: 12 GB → 3 arbeidere, 16 GB+ → 4 |
| **System kun med CPU** | `CPU_PARALLEL` | fysiske kjerner − 1 (min. 2) | `cpu_fallback` | Ingen GPU nødvendig, bruker en trådpool |

{% hint style="info" %}
**Jetsons samlede minne**: Jetson-enheter deler GPU- og CPU-minne. En Jetson Orin NX 16 GB rapporterer ca. 15,3 GB VRAM, men dette er det samme fysiske RAM-minnet som brukes av operativsystemet og CPU-prosessene. Det er derfor Jetson-enheter med 16 GB+ kvalifiserer for `GPU_PARALLEL` på samme måte som en stasjonær GPU med 12 GB+, men er likevel begrenset til 2 arbeidere — GPU-en, arbeiderprosessene og deres CUDA-kontekster per arbeider trekker alle fra den samme delte puljen.
{% endhint %}

### GPU-budsjett etter VRAM (diskrete GPU-er)

På x86_64-verter med en diskret NVIDIA-GPU bestemmer det oppdagede VRAM-minnet også hvor mye av kortets prosessorkapasitet som kan benyttes og hvor store batcher kan bli:

| Oppdaget VRAM | Tak for GPU-budsjett | Multiplikator for batchstørrelse |
| --- | --- | --- |
| **8 GB+** | 90 % | ×2,0 |
| **6–8 GB** | 85 % | ×1,75 |
| **3,5–6 GB** | 80 % | ×1,5 |
| **2–3,5 GB** | 75 % | ×1,25 |
| **Under 2 GB** | 70 % | ×1,0 |

Diskrete GPU-er reserverer bare 0,5 GB til systemet, fordi de ikke deler system-RAM. Jetson-profiler reserverer langt mer og har lavere tak — se [NVIDIA Jetson-veiledningen](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Dynamisk tildeling av GPU-minne

Chloros bruker en [4-tråds prosesseringspipeline](processing-pipeline.md):

* **Tråd 1** (Deteksjon) — Innlasting av bilder, EXIF-parsing, måldeteksjon
* **Tråd 2** (Kalibrering) — Beregning av refleksjonskalibrering
* **Tråd 3** (Behandling) — GPU-debayer, vignettkorrigering, indeksberegning
* **Tråd 4** (Eksport) — Filskriving, innbygging av metadata

Trådene 1, 2 og 4 bruker lite GPU-ressurser; tråd 3 er den som bruker mest. Når tidligere tråder i behandlingskjeden fullføres, blir deres GPU-ressurser **omfordelt til de gjenværende aktive trådene**, slik at tråd 3 får stadig mer minne etter hvert som behandlingen skrider frem.

### Tildelingsfaser

| Fase | Aktive tråder | Fordeling av GPU-minne |
| --- | --- | --- |
| **Tidlig** | 1, 2, 3, 4 | Fordelt på alle tråder, mesteparten til tråd 3 |
| **Midt-tidlig** | 2, 3, 4 | Tråd 1s andel omfordelt |
| **Midt-sen** | 3, 4 | Andelene til trådene 1 og 2 går til 3 og 4 |
| **Sen** | 3 eller 4 | Den siste aktive tråden får sin maksimale tildeling |

To regler styrer tallene:

* En tråd som er den **eneste** aktive, tildeles profilens maksimale tildeling.
* Når mer enn én *tung* GPU-oppgave er aktiv, deles hver tung oppgaves grunnleggende tildeling mellom dem (aldri under det konfigurerte minimumet).

Verdien som faktisk brukes ved kjøring, er den **laveste** av tildelingen i plattformprofilen og den aktuelle anbefalingen fra GPU-minneovervåkeren, så et travlt kort vinner alltid over en optimistisk profil.***

## Teksturbevisst behandling

Den teksturbevisst debayeren (**kun Chloros+** — `--debayer texture-aware`) kjører en AI/ML-støyreduksjonsmodell som trenger omtrent 1,75 GB VRAM i FP16 per kopi, så den bruker langt mer GPU-minne enn standardmetoden:

* Systemer med **mindre enn 7 GB VRAM**behandler teksturbevisst i en**synkron sløyfe, ett bilde om gangen** — flere modellkopier får ikke plass, og en arbeiderpool ville bare skape konkurranse
* Systemer med **7 GB eller mer VRAM** kan behandle Texture Aware samtidig, men med et redusert antall arbeidere sammenlignet med Standard
* På **Jetson**, er Texture Aware alltid knyttet til én enkelt arbeider, og på modeller med lavt strømforbruk (Nano, Orin Nano) pålegges det også automatisk en GPU-frekvensbegrensning — se [NVIDIA Jetson-veiledningen](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)***

## Termisk styring (Jetson)

Jetson-enheter har termiske begrensninger, spesielt i lukkede eller luftbårne installasjoner. Chloros overvåker Jetsons innebygde temperatursensorer og skalerer batchstørrelsene automatisk:

| Temperatur | Respons |
| --- | --- |
| **&lt; 70 °C** | Normal drift — full hastighet |
| **70 °C** (Advarsel) | Batchstørrelsen reduseres gradvis (100 % → 50 % mellom 70 °C og 80 °C) |
| **80 °C** (Kritisk) | Aggressiv ytelsesbegrensning (50 % → 0 % mellom 80 °C og 90 °C) |
| **90 °C** (Nedstengning) | Stopp GPU-behandlingen fullstendig |

På stasjonære systemer med tilstrekkelig kjøling utløses termisk begrensning sjelden.

***

## Håndtering av minnepress

Chloros overvåker GPU-minnet kontinuerlig under behandlingen og reagerer på tre nivåer.

**Batchstørrelse.** En batch starter med 8 bilder ganger plattformmultiplikatoren fra tabellene ovenfor. Chloros sjekker deretter ledig VRAM, reserverer 20 % av dette til PyTorchs egen overhead og antar omtrent 100 MB GPU-minne per 12 MP-bilde – batchen er den minste av de to, enten den minnebaserte grensen eller plattformgrunnlaget. Den faller aldri under 1.**Forebyggende reduksjon.**Over**85 % VRAM-utnyttelse** reduseres batchstørrelsene før noe svikter.**Trimming av tildeling per tråd.** Etter hvert som den aktuelle utnyttelsen stiger, reduseres GPU-budsjettet for hver tråd: ×0,75 ved utnyttelse over 80 %, ×0,5 ved utnyttelse over 90 %. Overvåkingsnivåene er 70 % (konservativt), 85 % (normal driftsgrense) og 95 % (OOM-risiko).**OOM-tilbaketrekning og gjenoppretting.** Hvis det likevel oppstår en «out-of-memory»-hendelse:

* blir batchstørrelsen **halvert**, og halvert igjen ved hver påfølgende OOM — hver påfølgende vellykkede batch reduserer denne straffen med ett trinn
* blir tildelingene til aktive tråder kuttet til 70 % av sin nåværende verdi, og tildelingsmekanismen bytter til sin konservative strategi, før den igjen blir mindre restriktiv etter en serie vellykkede tildelinger
* under sterkt press går pipelinen tilbake fra `fused_gpu` til `tiled_gpu`, og til `cpu_fallback` som en siste utvei

**Verts-RAM (Jetson).** Før behandlingen estimerer CLI toppverdien for vertsminnet ut fra antall bilder og debayer-modus, og gir en advarsel hvis RAM pluss filbasert swap sannsynligvis ikke er tilstrekkelig, samtidig som den skriver ut de nøyaktige kommandoene for å legge til swap — se [NVIDIA Jetson-veiledningen](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).***

## Overvåking av beregnings tilpasning

### Systemdiagnostikk

`chloros-cli selftest` er den raskeste måten å bekrefte hva beregningslaget ser:

```bash
chloros-cli selftest
```

De syv sjekkene dekker versjon, porttilgjengelighet, oppstart av backend, `/api/test`, systeminformasjon, tilstedeværelse av støyreduseringsmodell og klarhet for CUDA og støyredusering. Sjekk 5 viser maskinvarelinjen direkte:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

Sjekk 7 viser `CUDA: <bool>, Denoiser: <bool>` – begge må være sanne for at Texture Aware i det hele tatt skal kunne brukes.

### Backend-logger

Strategien og antall arbeidere velges inne i backend ved starten av hvert løp — det finnes ingen CLI-banner som kunngjør dem. Når noe oppfører seg uventet (en GPU-bane som faller tilbake, en OOM, en støyreduksjon som ikke vil laste inn), er det i backend-loggen for den aktuelle økten dette vises:

| Plattform | Loggplassering |
| --- | --- |
| **Linux / Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (én fil per oppstart) |
| **Linux, CLI-startet backend** | også `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Live fremdrift

Under en kjøring viser CLI live fremdrift per tråd (detektering, analyse, behandling, eksport) strømmet via Server-Sent Events — den praktiske indikasjonen på om tråd 3 er flaskehalsen. Se [Behandlingsrørledningen](processing-pipeline.md).

***

## Neste trinn

* [Behandlingsrørledningen](processing-pipeline.md) — Forståelse av rørledningsarkitekturen med 4 tråder
* [NVIDIA Jetson-veiledning](../linux/nvidia-jetson-guide.md) — Jetson-spesifikk distribusjon og optimalisering
* [CLI : Kommandolinje](../CLI.md) — Veiledningen til CLI
* [CLI-referanse](../reference/cli-reference.md) — Utfyllende kommandoliste for versjon 1.2.0
