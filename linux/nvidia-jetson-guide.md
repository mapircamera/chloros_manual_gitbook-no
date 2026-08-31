# NVIDIA Jetson-veiledning

Chloros på NVIDIA Jetson muliggjør multispektral bildebehandling i utkanten — ute i felten, på droner og i avsidesliggende installasjoner. Chloros 1.2.0 gjenkjenner Jetson-modellen din ved oppstart og optimaliserer behandlingsstrategien for den maskinvaren den finner. **Det er ikke nødvendig med manuell innstilling.**

***

## Støttede Jetson-modeller

| Modell                | RAM            | Behandlingsstrategi                                     | Anbefalt bruk                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 GB delt | `GPU_PARALLEL` (2 arbeidere)                              | Maksimal ytelse, store datasett                      |
| **Jetson Orin NX**   | 8–16 GB delt  | `GPU_PARALLEL` (2 arbeidere, 16 GB) / `GPU_SINGLE` (8 GB)   | Primær anbefaling for bruk i luften og i felten |
| **Jetson Orin Nano** | 8 GB delt     | `GPU_SINGLE` (1 arbeider, sekvensiell)                     | Edge-databehandling på innstegsnivå                                 |

{% hint style="info" %}
arm64-pakken Linux krever **JetPack 6**, som er tilgjengelig på Jetson Orin-familien. Eldre modeller (Nano, TX2, Xavier NX) kan ikke kjøre JetPack 6 og støttes ikke av den nåværende pakken.
{% endhint %}

***

## Krav

* **JetPack 6.x** (siste versjon anbefales)
* **NVIDIA CUDA** (inkludert i JetPack)
* **Betalt Chloros+-abonnement** — Copper-nivå eller høyere (kreves for all tilgang til CLI/SDK; håndheves på serversiden)

## Installasjon

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.2.0"

# Install Python SDK (optional) — the bundled wheel always matches this build
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl

# Run system diagnostics
chloros-cli selftest
```

For generelle detaljer om Linux-installasjon, filplasseringer og feilsøking, se [Linux-installasjon](linux-installation.md).

{% hint style="info" %}
**Plasser utpakkingsmappen på et raskt lagringsmedium.** De kompilerte binærfilene pakker seg selv ut til en midlertidig mappe ved hver oppstart — noe som er svært tregt fra et SD-kort. Chloros bruker `/mnt/ssd/tmp` automatisk hvis den finnes; ellers må du angi `TMPDIR` til en sti på NVMe-enheten din (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Dynamisk beregnings tilpasning på Jetson

### Slik fungerer det

Ved oppstart profilerer Chloros systemet ditt:

1. **Identifiserer Jetson-modellen** via `/proc/device-tree/model`
2. **Leser tilgjengelig delt GPU-/CPU-minne** (Jetson bruker samlet minne)
3. **Velger en behandlingsstrategi** (`GPU_PARALLEL`, `GPU_SINGLE` eller `CPU_PARALLEL`)
4. **Angir antall arbeidere, pipelinetype og minnetildeling** automatisk

Valget avhenger av **totalt delt RAM**, ikke av modellnavnet:

* **Under 12 GB totalt RAM**(alle 8 GB Jetson-enheter): `GPU_SINGLE` med**1 arbeider — bevisst sekvensiell behandling**. Det er for lite minne til flere arbeidere samtidig, så bildene behandles ett om gangen. På Jetson-enheter med**8 GB eller mindre** hopper tråd 3 helt over arbeiderpoolen og utfører arbeidet for hvert enkelt bilde i selve prosessen.
* **12 GB eller mer**(Orin NX 16 GB, AGX Orin): det samlede minnet kvalifiserer for `GPU_PARALLEL`, men antall arbeidere er**begrenset til 2 på Jetson** — GPU-en, arbeiderprosessenes RAM og deres CUDA-kontekster per arbeider henter alle fra den samme delte puljen, så flere arbeidere øker risikoen for feil på grunn av minnebrist.

Du kan overstyre det automatiske valget med miljøvariabelen `CHLOROS_STRATEGY` – se [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Oppførsel per modell

| Jetson-modell                | Strategi       | Arbeidsprosesser | Utførelse                                      |
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

Texture Aware-debayeren kjører inferens i et nevralt nettverk på GPU-en, noe som kan utløse **overstrømsadvarsler**på Jetson-modeller med lav effekt (10–15 W-klassen) ved full GPU-klokkehastighet. Før «Texture Aware»-behandling på en**Jetson Nano eller Orin Nano**sjekker Chloros GPU-ens maksimale frekvens og begrenser den til**510 MHz** (510000000) hvis den for øyeblikket er høyere:

* Hvis CLI kan skrive til sysfs-noden for GPU-frekvensen, **blir begrensningen satt automatisk**, og en bekreftelse vises.
* Hvis ikke (krever root-rettigheter), viser CLI den nøyaktige `sudo`-kommandoen for å sette begrensningen manuelt, venter et øyeblikk slik at du kan lese den, og fortsetter deretter — behandlingen kjører fortsatt, men det kan vises advarsler om overstrøm.

For å sette begrensningen selv før behandlingen:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Modeller med høyere effekt (Orin NX 25W, AGX Orin 60W) kjører med full GPU-hastighet; ingen begrensning settes. Standard debayer utløser aldri begrensningen på noen modeller.

{% hint style="info" %}
**«Texture Aware» på Jetson kjører alltid ett bilde om gangen.** Hver arbeider trenger sin egen CUDA-kontekst (~1 GB) pluss sin egen kopi av støyreduksjonsmodellen, noe det samlede minnet ikke har kapasitet til – derfor er Texture Aware-banen på Jetson festet til én enkelt arbeider med serialisert GPU-tilgang. Forvent at Texture Aware vil være betydelig tregere enn Standard på alle Jetson-enheter.
{% endhint %}

***

## Termisk styring

Jetson-enheter har begrenset termisk spillerom, spesielt i lukkede eller luftbårne installasjoner. Chloros overvåker SoC-temperaturen og reduserer batchstørrelsene automatisk:

| Temperatur         | Tiltak                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normal drift — full prosesseringshastighet          |
| **70 °C** (Advarsel) | Batchstørrelsen reduseres gradvis (100 % → 50 % mellom 70 °C og 80 °C) |
| **80 °C** (Kritisk) | Aggressiv begrensning (50 % → 0 % mellom 80 °C og 90 °C) |
| **90 °C** (Avstengning) | Stopp GPU-behandlingen helt — avkjøling nødvendig |

{% hint style="warning" %}
**Sørg for tilstrekkelig ventilasjon og kjøling** for kontinuerlig prosessering, spesielt i lukkede feltkabinetter eller luftbårne systemer. Termisk begrensning vil redusere prosessorkapasiteten for å beskytte maskinvaren.
{% endhint %}

***

## Minnehåndtering

Jetson-enheter bruker **enhetlig minne** — GPU-en og CPU-en deler det samme fysiske RAM-minnet. Det oppgitte VRAM-minnet (f.eks. ~15,3 GB på en Orin NX 16 GB) er ikke dedikert GPU-minne; det er det samme RAM-minnet som operativsystemet og alle andre prosesser bruker.

### Advarsel om swap og anbefalinger

Før bearbeiding på Jetson teller CLI antall RAW-bilder i inndatamappen din (`.tif`, `.tiff`, `.raw`, `.dng` — JPG-forhåndsvisninger telles ikke med), anslår det maksimale minnebehovet for kjøringen og **gir en advarsel før oppstart** dersom RAM + swap sannsynligvis ikke vil være tilstrekkelig. Advarselen har overskriften `LOW MEMORY WARNING - Jetson Detected`, viser antall bilder, RAM, gjeldende swap-plass og estimert toppbehov, før den gir de eksakte kommandoene `fallocate` / `chmod` / `mkswap` / `swapon` tilpasset prosjektets størrelse (aldri mindre enn 8 GB). Den pauser noen sekunder slik at meldingen ikke går tapt i rullehistorikken, deretter fortsetter behandlingen.**Minneanslag brukt av advarselen:**

| Debayer-modus | Grunnleggende | Per bilde |
| --- | --- | --- |
| Standard | ~1,5 GB | ~10 MB |
| Teksturbevisst | ~2,5 GB (modell + Python-kjøringstid) | ~15 MB |

Advarselen utløses når den estimerte toppverdien overstiger RAM + swap minus en sikkerhetsmargin på 1 GB, og den teller kun **filbasert** swap — et oppsett som kun bruker zram vil fortsatt bli markert.

Slik legger du til swap manuelt (eksempel: 8 GB):



<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->

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
```### Håndtering av OOM (Out of Memory)

Under behandlingen overvåker Chloros GPU-minnet og reduserer ytelsen gradvis i stedet for å krasje:

1. Når utnyttelsen av GPU-minnet overstiger **85 %**, reduseres batchstørrelsene forebyggende
2. Hvis det likevel oppstår en «out-of-memory»-hendelse, **halveres** batchstørrelsen, og halveres igjen ved hver påfølgende OOM; hver påfølgende vellykkede batch reduserer denne begrensningen med ett trinn
3. Under vedvarende belastning går pipelinen tilbake fra `fused_gpu` til den minneeffektive `tiled_gpu`-banen, og til CPU-behandling som en siste utvei

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

Chloros og CLI er ideelle for headless Jetson-distribusjoner:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Alltid påslått backend for LATTICE / DAQ-E tidssynkronisering

Hvis Jetson-enheten din styrer LATTICE-kameraer eller DAQ-E-lyssensorer uten skjerm, må du aktivere backend-systemd-tjenesten slik at PTP-grandmasteren kjører kontinuerlig (enheten er installert, men ikke aktivert som standard):

```bash
sudo systemctl enable --now chloros-backend.service
chloros-cli time-sync status
```

Se [Linux Installasjon](linux-installation.md#always-on-ptp-for-headless-hosts) for detaljer, inkludert hvordan pakken gjør PTP-portene 319/320 bindbare uten root-rettigheter.

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
    --indices NDVI NDRE GNDVI
```

### Python og SDK på Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
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
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Anbefalte Jetson-systemer for feltbruk

For bruk i felten og i luften bør du vurdere følgende Jetson Orin NX 16 GB-bærerkortalternativer:

* **Luftbåren/drone**: Systemer med vibrasjonsklassifisering (MIL-STD), lav vekt (under 300 g) og passiv kjøling
* **Robust feltbruk**: Vanntette kabinetter med IP67/IP69K-klassifisering og PoE GigE-kamera-tilkobling
* **Minimal/budsjett**: Utviklersett med tilleggskabinett

Kontakt [MAPIR Support](https://www.mapir.camera/community/contact) for spesifikke maskinvareanbefalinger for ditt bruksscenario.

***

## Neste trinn

* [Linux Installasjon](linux-installation.md) — Generelle detaljer om installasjon av Linux
* [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md) — Fullstendig referanse for beregningsstrategier
* [Behandlingsrørledning](../processing-architecture/processing-pipeline.md) — Forståelse av rørledningen med 4 tråder
* [CLI : Kommandolinje](../CLI.md) — Veiledningen for CLI
* [API : Python SDK](../api-python-sdk.md) — Veiledningen til SDK
* [CLI-referanse](../reference/cli-reference.md) og [SDK-referanse](../reference/sdk-reference.md) — Utførlig oversikt over kommandoer/API for versjon 1.2.0
