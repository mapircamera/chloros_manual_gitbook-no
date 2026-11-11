# Dynamisk tilpasning av beregningskapasitet:

Chloros
1.3.0 benytter maskinvaredeteksjon og automatisk valg av behandlingsstrategi. Behandlingsmotoren tilpasser seg maskinvaren din – fra en Jetson Orin Nano til en arbeidsstasjon med flere GPU-er – uten behov for manuell konfigurering.

***

## Slik fungerer det

Når Chloros
starter, profilerer det systemet ditt:

1. **Gjenkjenner operativsystemet** – Windows
eller Linux
2. **Identifiserer CPU-kjerner og totalt RAM**

3.**Gjenkjenner tilstedeværelse av GPU** – NVIDIA CUDA-støtte, VRAM, modell
4. **Identifiserer Jetson-modell** (hvis aktuelt) — via `/proc/device-tree/model`
5. **Sjekker temperatursensorer** (Jetson) — for temperaturbevisst prosessering
6. **Velger beregningsstrategi** — basert på all oppdaget maskinvare
7. **Konfigurerer antall arbeidere, pipelinetype og minnetildeling** automatisk

Den oppdagede profilen lagres i hurtigbufferen for økten, både i minnet og på disken, slik at senere kjøringer starter raskere:

| Plattform | Profil i hurtigbuffer |
| --- | --- |
| **Linux
/ Jetson** | `~/.config/chloros/system_config.json` (prioriterer `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Slett denne filen for å tvinge frem en ny gjenkjenning — nyttig etter at du har lagt til en GPU eller mer RAM. Chloros
utfører også automatisk en ny gjenkjenning når cachen ble skrevet av en inkompatibel eldre versjon.

***

## Beregningsstrategier

Chloros
velger én av tre beregningsstrategier basert på maskinvaren din:

| Strategi | Velges når | Arbeidere | Utfører | Rørledning |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| CUDA-GPU som rapporterer**12 GB+ VRAM**(på Jetson unified memory, krever også minst 12 GB delt RAM totalt) | `min(4, ⌊(VRAM − 0.5GB) × 0.9 ÷ 3.5GB⌋)`, minimum 2 —**begrenset til 2 på Jetson** | Arbeidsprosesser (separate CUDA-kontekster) | `fused_gpu` |
| **`GPU_SINGLE`**| CUDA-GPU med**2–12 GB VRAM**| 3 (I/O-overlapping; GPU-tilgang serialisert av en semafor).**1 (sekvensiell) på Jetson-enheter med under 12 GB RAM** | Arbeidsprosesser (separate CUDA-kontekster); sekvensiell prosessinternt på Jetson-enheter med lite RAM | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Ingen CUDA-GPU, eller under 2 GB VRAM | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Eksempler på hvordan `GPU_PARALLEL`-arbeiderformelen fungerer: 12 GB VRAM → 2 arbeidere, 16 GB → 3 arbeidere, 20 GB+ → 4 arbeidere. Hver arbeider tildeles 3,5 GB, det beregnede behovet for en teksturbevisst oppgave, innenfor 90 % av det gjenværende VRAM-minnet etter en reserve på 0,5 GB.

{% hint style="info" %}
**GPU-strategiene krever en PyTorch-versjon som kan bruke CUDA.** En maskin der PyTorch ikke har noen brukbar CUDA-enhet, får `CPU_PARALLEL`, uansett hvilken GPU den har. Jetson-pakken 1.3.0 bruker JetPack 6s CUDA-versjon av PyTorch, så en Jetson kvalifiserer seg så snart JetPacks CUDA- og cuDNN-biblioteker er installert. En Jetson med under 12 GB RAM behandler alltid ett bilde om gangen, i prosessen (logget som `GPU_SINGLE`).
{% endhint %}

GPU-strategier kjører en gruppe arbeiderprosesser, hver som en egen OS-prosess med sin egen CUDA-kontekst, som Chloros
starter ved å kjøre sitt eget installerte backend-program på nytt (ikke via Python
s `multiprocessing`, der oppstarten av arbeiderprosesser ikke fungerer inne i den pakkede applikasjonen). CPU-strategien bruker en trådpool (Python
s `ThreadPoolExecutor`). Chloros
bruker ikke noe distribuert rammeverk fra tredjepart (som for eksempel Ray).

### Rørledningstyper

* **`fused_gpu`** — Full GPU-behandlingsbane. Debayer-, korreksjons- og indeksoperasjoner kjøres på GPU-en i én samlet gjennomgang. Høyest gjennomstrømning, krever mest VRAM.
* **`tiled_gpu`** — Minneeffektiv GPU-bane. Behandler bilder i fliser for å passe innenfor begrenset GPU-minne. Lavere gjennomstrømning, men fungerer på enheter med begrenset minne.
* **`cpu_fallback`** — Behandling utelukkende på CPU ved hjelp av multitrådet parallellitet. Brukes når ingen NVIDIA-GPU er tilgjengelig, og som siste utvei når begge GPU-banene mislykkes.

Reservekjeden under kjøring er alltid `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Manuell overstyring av strategi

Sett miljøvariabelen `CHLOROS_STRATEGY` for å tvinge frem en bestemt strategi — en ekspertløsning for når automatisk gjenkjenning velger noe som ikke passer for din situasjon (for eksempel for å holde GPU-en ledig til annet arbeid):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

Variabelen samsvarer uten å skille mellom store og små bokstaver; alt som ikke er et av de tre navnene, ignoreres, og automatisk gjenkjenning fortsetter som normalt. Ved en overstyring velger Chloros
fortsatt antall arbeidere for deg:

| Overstyring | Antall arbeidere brukt |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

En overstyring av `GPU_SINGLE` eller `GPU_PARALLEL` på en maskin uten brukbar CUDA-enhet avvises med en `[STRATEGY]`-linje i loggen, og kjøringen bruker `CPU_PARALLEL`.

Det anbefales å angi dette per kommando fremfor permanent, slik at normale kjøringer fortsetter å tilpasse seg automatisk.

***

## Plattformspesifikk oppførsel

| Plattform | Strategi | Arbeidere | Rørledning | Merknader |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekvensiell) | Minneeffektiv modus, ett bilde om gangen |
| **Jetson Orin NX 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekvensiell) | Delte RAM-minne under 12 GB tvinger frem sekvensiell behandling |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (samtidig) | Anbefalt kant-enhet — Jetson-begrenset til 2 arbeidere |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (samtidig) | Maksimal ytelse i kantnettverket (også begrenset til 2 arbeidere på Jetson) |
| **Stasjonær PC med 8 GB GPU** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 arbeidere overlapper I/O mens en semafor serialiserer GPU-tilgang |
| **Stasjonær PC med 12 GB+ GPU** | `GPU_PARALLEL` | 2–4 | `fused_gpu` (samtidige) | Optimal stasjonær ytelse: 12 GB → 2 arbeidere, 16 GB → 3, 20 GB+ → 4 |
| **System kun med CPU** | `CPU_PARALLEL` | fysiske kjerner − 1 (min. 2) | `cpu_fallback` | Ingen GPU nødvendig, bruker en trådpool |

{% hint style="info" %}
**Jetsons samlede minne**: Jetson-enheter deler GPU- og CPU-minne. En Jetson Orin NX med 16 GB rapporterer ca. 15,3 GB VRAM, men dette er det samme fysiske RAM-minnet som brukes av operativsystemet og CPU-prosessene. Det er derfor Jetson-enheter med 16 GB eller mer kvalifiserer for `GPU_PARALLEL` på samme måte som en stasjonær GPU med 12 GB eller mer, men likevel er begrenset til 2 arbeidere — GPU-en, arbeiderprosessene og deres CUDA-kontekster per arbeider trekker alle fra den samme delte puljen.
{% endhint %}

### GPU-budsjett etter VRAM (diskrete GPU-er)

På x86_64-verter med en diskret NVIDIA-GPU bestemmer det oppdagede VRAM-minnet også hvor mye av kortets prosessorkapasitet som kan benyttes og hvor store batcher kan bli:

| Registrert VRAM | Tak for GPU-budsjett | Multiplikator for batchstørrelse |
| --- | --- | --- |
| **8 GB+** | 90 % | ×2,0 |
| **6–8 GB** | 85 % | ×1,75 |
| **3,5–6 GB** | 80 % | ×1,5 |
| **2–3,5 GB** | 75 % | ×1,25 |
| **Under 2 GB** | 70 % | ×1,0 |

Diskrete GPU-er reserverer bare 0,5 GB til systemet, fordi de ikke deler system-RAM. Jetson-profiler reserverer langt mer og har lavere tak — se [NVIDIA Jetson-veiledningen](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Dynamisk tildeling av GPU-minne

Chloros
bruker en [4-tråds behandlingspipeline](processing-pipeline.md):

* **Tråd 1** (Deteksjon) — Innlasting av bilder, EXIF-parsing, måldeteksjon
* **Tråd 2** (Kalibrering) — Beregning av refleksjonskalibrering
* **Tråd 3** (Behandling) — GPU-debayer, vignettkorrigering, indeksberegning
* **Tråd 4** (Eksport) — Filskriving, innbygging av metadata

Trådene 1, 2 og 4 bruker lite GPU-kapasitet; Tråd 3 er den som krever mest. Når tidligere tråder i prosesskjeden er ferdige, blir deres GPU-ressurser **omfordelt til de gjenværende aktive trådene**, slik at tråd 3 får stadig mer minne etter hvert som kjøringen fortsetter.

### Tildelingsfaser

| Fase | Aktive tråder | Fordeling av GPU-minne |
| --- | --- | --- |
| **Tidlig** | 1, 2, 3, 4 | Fordelt på alle tråder, mesteparten til tråd 3 |
| **Midt-tidlig** | 2, 3, 4 | Tråd 1s andel omfordelt |
| **Midt-sen** | 3, 4 | Andelene til tråd 1 og 2 går til 3 og 4 |
| **Sen** | 3 eller 4 | Den siste aktive tråden får sin maksimale tildeling |

To regler styrer tallene:

* En tråd som er den **eneste** aktive, tildeles profilens maksimale tildeling.
* Når mer enn én *tung* GPU-oppgave er aktiv, deles hver tung oppgaves grunnleggende tildeling mellom dem (aldri under det konfigurerte minimumet).

Verdien som faktisk brukes under kjøring, er den **laveste** av tildelingen i plattformprofilen og den sanntidsanbefalingen fra GPU-minnemonitoren, så et travlt kort vinner alltid over en optimistisk profil.***

## Teksturbevisst behandling

Den teksturbevisst debayeren (**kun Chloros
+** — `--debayer texture-aware`) kjører en AI/ML-støyreduksjonsmodell som trenger omtrent 1,75 GB VRAM i FP16 per kopi, så den bruker langt mer GPU-minne enn standardmetoden:

* Systemer med **mindre enn 7 GB VRAM**behandler «Texture Aware» i en**synkron sløyfe, ett bilde om gangen** — flere modellkopier får ikke plass, og en arbeiderpool ville bare føre til konkurranse
* Systemer med **7 GB eller mer VRAM** kan behandle Texture Aware samtidig, men med et redusert antall arbeidere sammenlignet med Standard
* På **Jetson** er «Texture Aware» alltid knyttet til én enkelt arbeider
* Uten **brukbar CUDA-GPU** bruker kjøringen standard-debayeren, og backenden logger én `[DEBAYER] … needs a CUDA GPU …`-linje som forklarer hvorfor
* På Jetson-enheter med lav effekt (Nano, Orin Nano) setter CLI
en begrensning på GPU-frekvensen når du ber om «Texture Aware» — se [NVIDIA Jetson-veiledningen](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)

***

## Termisk styring (Jetson)

Jetson-enheter har termiske begrensninger, spesielt i lukkede eller luftbårne installasjoner. Chloros
overvåker Jetsons innebygde temperatursensorer og skalerer batchstørrelsene automatisk:

| Temperatur | Respons |
| --- | --- |
| **&lt; 70 °C** | Normal drift – full hastighet |
| **70 °C** (Advarsel) | Batchstørrelsen reduseres gradvis (100 % → 50 % mellom 70 °C og 80 °C) |
| **80 °C** (Kritisk) | Aggressiv ytelsesbegrensning (50 % → 0 % mellom 80 °C og 90 °C) |
| **90 °C** (Avstengning) | Stopp GPU-behandlingen fullstendig |

På stasjonære systemer med tilstrekkelig kjøling utløses termisk begrensning sjelden.

***

## Håndtering av minnepress

Chloros
overvåker GPU-minnet kontinuerlig under behandlingen og reagerer på tre nivåer.

**Batchstørrelse.** En batch starter med 8 bilder ganger plattformmultiplikatoren fra tabellene ovenfor. Chloros
sjekker deretter ledig VRAM, reserverer 20 % av dette til PyTorchs egen overhead og antar omtrent 100 MB GPU-minne per 12 MP-bilde – batchen er den minste av de to, enten den minnebaserte grensen eller plattformgrunnlaget. Den faller aldri under 1.

**Forebyggende reduksjon.**Ved**85 % VRAM-utnyttelse** eller høyere reduseres batchstørrelsene før noe svikter.**Trimming av tildeling per tråd.** Etter hvert som den aktuelle utnyttelsen stiger, reduseres GPU-budsjettet for hver tråd: ×0,75 ved utnyttelse over 80 %, ×0,5 ved utnyttelse over 90 %. Overvåkingsnivåene er 70 % (konservativt), 85 % (normal driftsgrense) og 95 % (OOM-risiko).**OOM-tilbaketrekning og gjenoppretting.** Hvis det likevel oppstår en minnefeil:

* blir batchstørrelsen **halvert**, og halvert igjen ved hver påfølgende OOM-hendelse — hver påfølgende vellykket batch reduserer denne straffen med ett trinn
* blir tildelingene til aktive tråder kuttet til 70 % av sin nåværende verdi, og tildelingsmekanismen bytter til sin konservative strategi, før den igjen blir mindre restriktiv etter en serie vellykkede tildelinger
* under sterkt press går pipelinen tilbake fra `fused_gpu` til `tiled_gpu`, og til `cpu_fallback` som en siste utvei

**Verts-RAM (Jetson).** Før behandlingen starter, estimerer CLI
-en toppverdien for vertsminnet ut fra antall bilder og debayer-modus, og gir en advarsel hvis RAM pluss filbasert swap sannsynligvis ikke vil være tilstrekkelig. Den viser da de nøyaktige kommandoene for å legge til swap — se [NVIDIA Jetson-veiledningen](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).

***

## Overvåking av beregnings tilpasning

### Systemdiagnostikk

`chloros-cli selftest` er den raskeste måten å bekrefte hva beregningslaget ser:

```bash
chloros-cli selftest
```

De 7 kontrollene dekker versjon, porttilgjengelighet, oppstart av backend, `/api/test`, systeminformasjon, tilstedeværelse av denoiser-modell og klarhet for CUDA + denoiser. Kontroll 5 viser maskinvarelinjen direkte:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

Sjekk 7 viser `CUDA: <bool>, Denoiser: <bool>` — begge må være sanne for at Texture Aware i det hele tatt skal kunne brukes.

### Backend-logger

Strategien og antall arbeidere velges inne i backend ved starten av hvert løp — det finnes ingen CLI
-banner som viser disse. Når noe oppfører seg uventet (en GPU-bane som faller tilbake, en OOM, en støyreduksjon som ikke vil laste inn), er det i backend-loggen for den aktuelle økten dette vises:

| Plattform | Loggplassering |
| --- | --- |
| **Linux
/ Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (én fil per oppstart) |
| **Linux
, backend startet via CLI** | også `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Live fremdrift

Under en kjøring viser CLI
-grensesnittet sanntids fremdrift per tråd (Detecting, Analyzing, Processing, Exporting) strømmet via Server-Sent Events — den praktiske måten å se om tråd 3 er flaskehalsen. Se [Behandlingsrørledning](processing-pipeline.md).

***

## Neste trinn

* [Behandlingsrørledning](processing-pipeline.md) — Forståelse av rørledningsarkitekturen med 4 tråder
* [NVIDIA Jetson-veiledning](../linux/nvidia-jetson-guide.md) — Jetson-spesifikk distribusjon og optimalisering
* [CLI
: Kommandolinje](../CLI.md) — Veiledning til CLI

* [CLI
-referanse](../reference/cli-reference.md) — Utførlig kommandoliste for versjon 1.3.0
