# NVIDIA Jetson-veiledning

Chloros på NVIDIA Jetson muliggjør multispektral bildebehandling i utkanten — ute i felten, på UAV-er og i avsidesliggende installasjoner. Chloros 1.3.0 gjenkjenner Jetson-modellen din ved oppstart og optimaliserer behandlingsstrategien for den maskinvaren den finner. **Det kreves ingen manuell innstilling.**
{% hint style="info" %}
**Jetson-pakken 1.3.0 kjører på Jetsons GPU.** `chloros_1.3.0_arm64_jp6.deb` er utviklet for JetPack 6 og bruker JetPack 6s CUDA-versjon av PyTorch, slik at behandlingen – inkludert den teksturbevisst debayeren – kjører på Orins GPU. Pakken inneholder ikke CUDA i seg selv: den bruker CUDA- og cuDNN-bibliotekene som JetPack 6 installerer (`sudo apt install nvidia-jetpack` på et bilde som mangler disse). Der det ikke er noen CUDA-GPU tilgjengelig, kjører behandlingen på CPU-en, og Texture Aware bruker Standard-debayeren.
{% endhint %}

***

## Støttede Jetson-modeller

| Modell                | RAM            | Behandlingsstrategi                                     | Anbefalt bruk                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 GB delt | `GPU_PARALLEL` (2 arbeidere)                              | Maksimal ytelse, store datasett                      |
| **Jetson Orin NX**   | 8–16 GB delt  | `GPU_PARALLEL` (2 arbeidere, 16 GB) / `GPU_SINGLE` (8 GB)   | Primær anbefaling for feltutplassering              |
| **Jetson Orin Nano** | 8 GB delt     | `GPU_SINGLE` (1 arbeider, sekvensiell)                     | Edge-databehandling på innstegsnivå                                 |

{% hint style="info" %}
Linux-pakken for arm64 krever **JetPack 6**, som er tilgjengelig på Jetson Orin-familien. Eldre modeller (Nano, TX2, Xavier NX) kan ikke kjøre JetPack 6 og støttes ikke av den nåværende pakken.
{% endhint %}

***

## Krav

* **JetPack 6.x** (siste versjon anbefales)
* **NVIDIA CUDA og cuDNN** fra JetPack 6 (`sudo apt install nvidia-jetpack` hvis bildet ditt mangler disse)
* **Betalt Chloros+-abonnement** — Copper-nivå eller høyere (kreves for all tilgang til CLI/SDK; håndheves på serversiden)

## Installasjon

```bash
# Install the arm64 .deb package (JetPack 6) -- apt, not dpkg -i, so the
# Python SDK's requirements (python3-numpy, python3-requests) come too
sudo apt install ./chloros_1.3.0_arm64_jp6.deb

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.3.0"
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # the .deb installs the SDK

# Run system diagnostics
chloros-cli selftest
```

`.deb` installerer Python-SDK-et for systemet `python3`, som samsvarer med denne versjonen. Bare et virtualenv trenger det medfølgende wheel-filen installert i seg: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` (helt offline). For generelle detaljer om Linux-installasjon, filplasseringer og feilsøking, se [Linux-installasjon](linux-installation.md).

{% hint style="info" %}
**Ingen utpakking ved oppstart lenger.** Fra og med versjon 1.3.0 installeres backend og CLI, ukomprimert, under `/usr/lib/chloros/`, slik at de ikke lenger pakker seg ut til en midlertidig katalog hver gang de startes. Midlertidige filer lagres fortsatt automatisk i `/mnt/ssd/tmp` når denne mappen finnes; ellers kan du angi `TMPDIR` til en sti på NVMe-enheten din (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Dynamisk beregnings tilpasning på Jetson

### Slik fungerer det

Ved oppstart lager Chloros en profil av systemet ditt:

1. **Oppdager Jetson-modellen** via `/proc/device-tree/model`
2. **Leser tilgjengelig delt GPU-/CPU-minne** (Jetson bruker samlet minne)
3. **Spør PyTorch om det kan bruke CUDA** — med JetPack 6s CUDA og cuDNN installert, kan det det
4. **Velger en behandlingsstrategi** (`GPU_PARALLEL`, `GPU_SINGLE` eller `CPU_PARALLEL`) og antall arbeidere automatisk

Valget avhenger av **totalt delt RAM**, ikke av modellnavnet:

* **Under 12 GB totalt RAM**(alle Jetson-enheter med 8 GB):**1 arbeider — bevisst sekvensiell behandling**. Minnekapasiteten er for begrenset for samtidige arbeidere, så bildene behandles ett om gangen. På Jetson-enheter med**8 GB eller mindre** hopper tråd 3 helt over arbeiderpoolen og kjører arbeidet per bilde i selve prosessen. Backend-loggen kaller denne strategien `GPU_SINGLE`.
* **12 GB eller mer**(Orin NX 16 GB, AGX Orin): det samlede minnet kvalifiserer for `GPU_PARALLEL`, men antall arbeidere er**begrenset til 2 på Jetson** — GPU-en, arbeiderprosessenes RAM og deres CUDA-kontekster per arbeider trekker alle fra den samme delte puljen, så flere arbeidere øker risikoen for feil på grunn av minnebrist.
* **Ingen brukbar CUDA** (JetPacks CUDA-biblioteker mangler): `CPU_PARALLEL` ved 12 GB eller mer, ett bilde om gangen nedenfor.

Du kan overstyre det automatiske valget med miljøvariabelen `CHLOROS_STRATEGY` — se [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Oppførsel per modell

| Jetson-modell                | Strategi       | Arbeidere | Utførelse                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | Sekvensiell sløyfe i prosessen (`tiled_gpu` ved minnepress) |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 1       | Sekvensiell sløyfe i prosessen                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Samtidige arbeidsprosesser, `fused_gpu`-bane  |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2       | Samtidige arbeidsprosesser, `fused_gpu`-bane  |

Den viktigste forskjellen mellom plattformene er **minne**. En Jetson med 8 GB må behandle bilder ett om gangen ved hjelp av en minneeffektiv tiled-tilnærming når belastningen er høy, mens en Orin med 16 GB eller mer kan kjøre to bilder gjennom GPU-en samtidig ved hjelp av den fusjonerte pipelinen med høyere gjennomstrømning.

### GPU-budsjett per modell

Hver Jetson-modell har også en maskinvareprofil som setter en grense for hvor mye av den delte minnepoolen prosesseringen kan benytte, og som skalerer batchstørrelsene:

| Modell | Tak for GPU-budsjett | Multiplikator for batchstørrelse | Reservert for system/skjerm |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70 % | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75 % | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80 % | ×1,5 | 4,0 GB |

Detektert RAM justerer profilen: en Jetson som rapporterer **16 GB eller mer** får batchmultiplikatoren økt med ×1,2. Grunnleggende batchstørrelse før multiplikatorer er 8 bilder.

For fullstendig referanse om beregnings tilpasning, se [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md).

***

## GPU-frekvensbegrensning for Texture Aware på Nano og Orin Nano

Texture Aware-debayeren kjører inferens i et nevralt nettverk på GPU-en, noe som kan utløse **overstrømsadvarsler**på Jetson-modeller med lav effekt (10–15 W-klassen) ved full GPU-klokkehastighet. Når du sender `--debayer texture-aware` på en**Jetson Nano eller Orin Nano**, sjekker CLI-en GPU-ens maksimale frekvens og begrenser den til **510 MHz** (510000000) hvis den for øyeblikket er høyere:

* Hvis CLI-en kan skrive til sysfs-noden for GPU-frekvensen, **blir begrensningen satt automatisk**, og en bekreftelse vises på skjermen.
* Hvis ikke (krever root-rettigheter), viser CLI-en den eksakte `sudo`-kommandoen for å sette begrensningen manuelt, venter et øyeblikk slik at du kan lese den, og fortsetter deretter — behandlingen kjører fortsatt, men det kan vises advarsler om overstrøm.

For å sette begrensningen selv før behandlingen:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Modeller med høyere effekt (Orin NX 25W, AGX Orin 60W) kjører med full GPU-hastighet; ingen begrensning settes. Standard debayer utløser aldri begrensningen på noen modeller.

{% hint style="info" %}
**«Texture Aware» på Jetson kjører alltid ett bilde om gangen.** Hver arbeider trenger sin egen CUDA-kontekst (~1 GB) pluss sin egen kopi av støyreduksjonsmodellen, noe det samlede minnet ikke har kapasitet til — derfor er Texture Aware-banen på Jetson festet til én enkelt arbeider med serialisert GPU-tilgang. Forvent at Texture Aware vil være betydelig tregere enn Standard på alle Jetson-enheter. Texture Aware gjelder for Survey3-bilder: LATTICE-opptak får alltid «Standard»-debayer. Uten en brukbar CUDA-GPU bruker kjøringen «Standard», og backenden logger én `[DEBAYER] … needs a CUDA GPU …`-linje som forklarer hvorfor (vist med `chloros-cli --verbose`).
{% endhint %}

***

## Termisk styring

Jetson-enheter har begrenset termisk margin, spesielt i lukkede eller luftbårne installasjoner. Chloros overvåker SoC-temperaturen og reduserer batchstørrelsene automatisk:

| Temperatur         | Tiltak                                            |
| ------------------- | ------------------------------------------------- |
| **< 70 °C**          | Normal drift — full prosesseringshastighet          |
| **70 °C** (Advarsel)  | Batchstørrelsen reduseres gradvis (100 % → 50 % mellom 70 °C og 80 °C) |
| **80 °C** (Kritisk) | Aggressiv begrensning (50 % → 0 % mellom 80 °C og 90 °C) |
| **90 °C** (Avstengning) | Stopp GPU-behandlingen fullstendig — avkjøling nødvendig |

{% hint style="warning" %}
**Sørg for tilstrekkelig ventilasjon og kjøling** ved kontinuerlig databehandling, spesielt i lukkede feltkabinetter eller luftbårne systemer. Termisk begrensning vil redusere prosessorkapasiteten for å beskytte maskinvaren.
{% endhint %}

***

## Minnehåndtering

Jetson-enheter bruker **enhetlig minne** — GPU-en og CPU-en deler det samme fysiske RAM-minnet. Det oppgitte VRAM-minnet (f.eks. ~15,3 GB på en Orin NX 16 GB) er ikke dedikert GPU-minne; det er det samme RAM-minnet som operativsystemet og alle andre prosesser bruker.

### Advarsel og anbefalinger om swap

Før behandling på Jetson teller CLI-en RAW-bildene i inndatamappen din (`.tif`, `.tiff`, `.raw`, `.dng` — JPG-forhåndsvisninger telles ikke med), estimerer det maksimale minnebehovet for kjøringen og **gir en advarsel før oppstart** hvis RAM + swap sannsynligvis ikke er tilstrekkelig. Advarselen har overskriften `LOW MEMORY WARNING - Jetson Detected`, viser antall bilder, RAM, gjeldende swap og estimert toppbehov, og gir deretter de eksakte kommandoene `fallocate` / `chmod` / `mkswap` / `swapon` tilpasset størrelsen på prosjektet ditt (aldri mindre enn 8 GB). Den pauser noen sekunder slik at meldingen ikke går tapt i rullehistorikken, deretter fortsetter behandlingen.**Minneestimater brukt av advarselen:**

| Debayer-modus | Grunn | Per bilde |
| --- | --- | --- |
| Standard | ~1,5 GB | ~10 MB |
| Teksturbevisst | ~2,5 GB (modell + Python-kjøringsmiljø) | ~15 MB |

Advarselen utløses når det estimerte toppforbruket overstiger RAM + swap minus en sikkerhetsmargin på 1 GB, og den teller kun **filbasert** swap — et oppsett som kun bruker zram vil fortsatt bli flagget.

For å legge til swap manuelt (eksempel: 8 GB):

```bash
# Check current memory and swap
free -h

# Create a swap file
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->


### Håndtering av OOM (Out of Memory)

Under behandlingen overvåker Chloros GPU-minnet og reduserer ytelsen på en kontrollert måte i stedet for å krasje:

1. Når utnyttelsen av GPU-minnet overstiger **85 %**, reduseres batchstørrelsene forebyggende
2. Hvis det likevel oppstår en «out-of-memory»-hendelse, **halveres** batchstørrelsen, og halveres igjen ved hver påfølgende OOM; hver påfølgende vellykkede batch reduserer denne straffen med ett trinn
3. Under vedvarende belastning går pipelinen tilbake fra `fused_gpu` til den minneeffektive `tiled_gpu`-banen, og til CPU-behandling som en siste utvei

Siden GPU-en deler Jetsons RAM, setter RAM pluss swap fortsatt en grense for kjøringen — swap-advarselen ovenfor er den kontrollen man må være oppmerksom på.

***

## Utplassering i felt

### Strømforbruk

| Jetson-modell     | Typisk strømforbruk | Merknader                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7–15 W              | DC-barrelkontakt          |
| Jetson Orin NX   | 10–25 W             | DC-barrelkontakt          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD eller barrelkontakt |

Planlegg strømforbruket for kontinuerlig prosessering — det høyeste strømforbruket oppstår under GPU-intensiv tråd 3 (prosessering).

### Anbefalinger for lagring

* **NVMe SSD** anbefales på det sterkeste for arm64-distribusjoner
* SD-kort er for trege for prosessering — bruk dem kun som oppstartsmedium
* Planlegg for 2–3 ganger størrelsen på råbildedataene for bearbeidet utdata

### Headless-drift via SSH


Chloros CLI er ideelt for headless Jetson-distribusjoner:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Alltid på-backend for LATTICE / DAQ-E-tidssynkronisering

Hvis Jetson-enheten din styrer LATTICE-kameraer eller DAQ-E-lyssensorer uten skjerm, må du aktivere backend-systemd-tjenesten **som din bruker**, slik at PTP-grandmasteren kjører kontinuerlig og SDK-skriptene finner en aktiv backend (enheten er installert, men ikke aktivert som standard):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
chloros-cli time-sync status
```

Backenden kjører da som deg: opptakene den skriver, er dine, og den bruker din `chloros-cli login`. Se [Linux-installasjon](linux-installation.md#always-on-ptp-for-headless-hosts) for detaljer, inkludert root-`chloros-backend.service` og hvordan pakken gjør PTP-portene 319/320 bindbare uten root-rettigheter.

### Automatisert behandling med systemd

Opprett en systemd-tjeneste for automatisert behandling:

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

`chloros-cli process` avsluttes med en verdi ulik null når en kjøring som har forespurt produkter ikke skriver noen bilder, slik at systemds feilstatus er meningsfull for overvåking.

Kombiner med en systemd-timer for planlagt behandling:

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## Eksempler på arbeidsflyter

### Grunnleggende Jetson-behandling

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI GNDVI OSAVI
```

### Python-SDK på Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### Batchbehandling av flere flyvninger

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI OSAVI
    echo "Completed $name"
done
```

***

## Anbefalte Jetson-systemer for feltbruk

For felt- og luftbårne installasjoner bør du vurdere følgende Jetson Orin NX 16 GB-bærerkortalternativer:

* **Luftbåren/drone**: Systemer med vibrasjonsklassifisering (MIL-STD), lav vekt (under 300 g) og passiv kjøling
* **Robust feltbruk**: Vanntette kabinetter med IP67/IP69K-klassifisering og PoE GigE-kamera-tilkobling
* **Minimal/budsjett**: Utviklersett med tilleggskabinett

Kontakt [MAPIR-support](https://www.mapir.camera/community/contact) for spesifikke maskinvareanbefalinger til ditt bruksscenario.

***

## Neste trinn

* [Linux-installasjon](linux-installation.md) — Generelle detaljer om Linux-installasjon
* [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md) — Fullstendig referanse for beregningsstrategier
* [Behandlingsrørledning](../processing-architecture/processing-pipeline.md) — Forståelse av 4-tråds rørledningen
* [CLI: Kommandolinje](../CLI.md) — Veiledning for CLI

* [API: Python SDK](../api-python-sdk.md) — Veiledning for SDK

* [Referanse for CLI](../reference/cli-reference.md) og [Referanse for SDK](../reference/sdk-reference.md) — Utførlige oversikter over kommandoer og API for versjon 1.3.0
