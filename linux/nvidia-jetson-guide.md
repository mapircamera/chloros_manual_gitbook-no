# NVIDIA Jetson-veiledning

Chloros på NVIDIA Jetson muliggjør multispektral bildebehandling i utkanten – ute i felten, på UAV-er og i avsidesliggende installasjoner. Chloros gjenkjenner automatisk Jetson-modellen din og optimaliserer behandlingsstrategien for maskinvaren din.

***

## Støttede Jetson-modeller

| Modell                | RAM            | Behandlingsstrategi                                   | Anbefalt bruk                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 GB delt | `GPU_PARALLEL` (4 arbeidere)                            | Maksimal ytelse, store datasett                      |
| **Jetson Orin NX**   | 8–16 GB delt  | `GPU_PARALLEL` (3 arbeidere, 16 GB) / `GPU_SINGLE` (8 GB) | Primær anbefaling for bruk i luften og i felten |
| **Jetson Orin Nano** | 8 GB delt     | `GPU_SINGLE` (1 arbeider)                               | Edge-databehandling på inngangsnivå                                 |
| **Jetson Nano**      | 4–8 GB delt   | `GPU_SINGLE` (1 arbeider)                               | Inngangsnivå, begrenset minne                          |

{% hint style="info" %}
**Eldre Jetson-modeller** (TX2, TX1, Xavier NX) støttes kanskje ikke. Ytelsen vil variere avhengig av tilgjengelig GPU-minne og CUDA-funksjonalitet.
{% endhint %}

***

## Krav

* **JetPack 6.x** (siste versjon anbefales)
* **NVIDIA CUDA** (inkludert i JetPack)
* **Chloros+ lisens** (kreves for tilgang til CLI/SDK)

## Installasjon

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

For generelle detaljer om installasjon av Linux, se [Linux-installasjon](linux-installation.md).

***

## Dynamisk beregnings tilpasning på Jetson

Chloros oppdager automatisk Jetson-modellen din og velger den optimale behandlingsstrategien. **Ingen manuell justering er nødvendig.**

### Slik fungerer det

Ved oppstart profilerer Chloros systemet ditt:

1. **Oppdager Jetson-modellen** via `/proc/device-tree/model`
2. **Leser tilgjengelig GPU/delt minne**

3.**Velger en behandlingsstrategi** (`GPU_PARALLEL`, `GPU_SINGLE` eller `CPU_PARALLEL`)
4. **Angir antall arbeidere, rørledningstype og minnetildeling** automatisk

### Oppførsel per modell

| Jetson-modell                | Strategi       | Arbeidere | Rørledning                       | Samkjøring |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8 GB**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (minneeffektiv) | Serialisert  |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | Serialisert  |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | Serialisert  |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 3       | `fused_gpu` (full GPU-bane)    | Samtidig  |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | Samtidig  |

{% hint style="success" %}
**Jetson Orin NX 16 GB** er det perfekte valget for edge-distribusjon — den bruker `GPU_PARALLEL`-strategien med 3 samtidige arbeidere, og leverer ekte parallell GPU-behandling i en kompakt formfaktor.
{% endhint %}

Den viktigste forskjellen mellom plattformene er **minne**. En Jetson Nano med 8 GB delt minne må behandle bilder ett om gangen ved hjelp av en minneeffektiv tiled-tilnærming, mens en Orin NX med 16 GB kan kjøre 3 bilder gjennom GPU-en samtidig ved hjelp av den fusjonerte pipelinen med høyere gjennomstrømning.

For den komplette referansen for beregningsadaptasjon, se [Dynamisk beregningsadaptasjon](../processing-architecture/dynamic-compute-adaptation.md).

***

## Termisk styring

Jetson-enheter har begrenset termisk spillerom, spesielt i lukkede eller luftbårne installasjoner. Chloros inkluderer automatisk termisk overvåking og regulering:

| Temperatur         | Tiltak                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normal drift — full prosesseringshastighet          |
| **70 °C** (Advarsel)  | Reduser batchstørrelsen automatisk                   |
| **80 °C** (Kritisk) | Aggressiv begrensning — lavere samtidighet         |
| **90 °C** (Avstengning) | Stopp GPU-prosessering helt — avkjøling nødvendig |

{% hint style="warning" %}
**Sørg for tilstrekkelig ventilasjon og kjøling** for kontinuerlig behandling, spesielt i lukkede feltkabinetter eller luftbårne systemer. Termisk begrensning vil redusere behandlingskapasiteten for å beskytte maskinvaren.
{% endhint %}

***

## Minnehåndtering

Jetson-enheter bruker **unified memory** — GPU-en og CPU-en deler det samme fysiske RAM-minnet. Dette betyr at den rapporterte VRAM-en (f.eks. 15,3 GB på Orin NX 16 GB) ikke er dedikert GPU-minne; den deles med operativsystemet og andre prosesser.

### Anbefalinger for swap

For store datasett eller Texture Aware-debayer-behandling kan Chloros anbefale å opprette swap-plass:

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Minneestimater per bilde:**

* Standard debayer: ~10 MB per bilde
* Texture Aware debayer: ~15 MB per bilde

Chloros beregner automatisk nødvendig minne basert på størrelsen på datasettet ditt og varsler deg hvis swap anbefales.

### OOM (Out of Memory) Fallback

Hvis det oppdages en situasjon med mangel på minne under behandlingen:

1. Chloros reduserer automatisk antall GPU-arbeidere
2. Faller tilbake fra `fused_gpu` til `tiled_gpu`-rørledningen (mer minneeffektiv)
3. Fortsetter behandlingen med redusert gjennomstrømning i stedet for å krasje

***

## Utplassering i felt

### Strømforhold

| Jetson-modell     | Typisk strømforbruk | Merknader                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5–10 W              | USB-C eller barrel-kontakt    |
| Jetson Orin Nano | 7–15 W              | DC-barrel-kontakt          |
| Jetson Orin NX   | 10–25 W             | DC-barrel-kontakt          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD eller barrel-kontakt |

Planlegg strømforbruket for kontinuerlig prosessering — toppstrømforbruket oppstår under GPU-intensiv tråd 3 (prosessering).

### Anbefalinger for lagring

* **NVMe SSD** anbefales på det sterkeste for arm64-distribusjoner
* SD-kort er for trege for prosessering — bruk dem kun som oppstartsmedium
* Planlegg for 2–3 ganger størrelsen på råbildedataene for bearbeidet utdata

### Headless drift via SSH

Chloros CLI er ideelt for headless Jetson-distribusjoner:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

### Automatisert prosessering med systemd

Opprett en systemd-tjeneste for automatisert prosessering:

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
    --format tiff-32 \
    --indices NDVI NDRE GNDVI
```

### Python SDK på Jetson

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
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Anbefalte Jetson-systemer for feltbruk

For felt- og luftbårne installasjoner bør du vurdere disse Jetson Orin NX 16 GB-bærerkortalternativene:

* **Luftbåren/drone**: Systemer med vibrasjonsklassifisering (MIL-STD), lettvekt (under 300 g), passiv kjøling
* **Robust felt**: IP67/IP69K vanntette kabinetter med PoE GigE-kamera-tilkobling
* **Minimal/budsjett**: Utviklersett med tilleggskabinett

Kontakt [MAPIR Support](https://www.mapir.camera/community/contact) for spesifikke maskinvareanbefalinger for ditt bruksscenario.

***

## Neste trinn

* [Linux Installasjon](linux-installation.md) — Generelle detaljer om Linux-installasjon
* [Dynamisk databehandlingstilpasning](../processing-architecture/dynamic-compute-adaptation.md) — Fullstendig referanse for databehandlingsstrategi
* [Behandlingsrørledning](../processing-architecture/processing-pipeline.md) — Forståelse av 4-tråds rørledningen
* [CLI : Kommandolinje](../CLI.md) — Fullstendig CLI-referanse
* [API : Python SDK](../api-python-sdk.md) — Fullstendig referanse for SDK
