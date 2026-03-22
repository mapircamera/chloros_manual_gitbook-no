# Installasjon av Linux

Chloros distribueres for Linux som `.deb`-pakker som installerer CLI og backend. Python SDK installeres separat via pip.

***

## Linux amd64 (x86_64)

### Systemkrav

| Krav | Minimum | Anbefalt |
| --- | --- | --- |
| **Distribusjon** | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+ |
| **Prosessor** | x86_64 (Intel/AMD) | Intel Core i7 eller bedre |
| **Minne (RAM)** | 8 GB | 16 GB eller mer |
| **Grafikkort** | Ingen (CPU-behandling) | NVIDIA GPU med 4 GB+ VRAM |
| **Lagring** | 2 GB ledig plass | SSD med 10 GB+ ledig plass |
| **Python** | Python 3.7+ (for SDK) | Python 3.10+ |

### Installasjon

Last ned `.deb`-pakken og installer:

```bash
sudo dpkg -i chloros-amd64.deb
```

Kontroller installasjonen:

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### Systemkrav

| Krav | Minimum | Anbefalt |
| --- | --- | --- |
| **Plattform** | NVIDIA Jetson med JetPack 6 | Jetson Orin NX 16 GB eller AGX Orin |
| **JetPack** | JetPack 6.x | Siste JetPack 6 |
| **Minne (RAM)** | 8 GB (delt GPU/CPU) | 16 GB+ delt |
| **Lagring** | 2 GB ledig plass | NVMe SSD med 10 GB+ ledig |
| **Python** | Python 3.7+ (for SDK) | Python 3.10+ |

### Installasjon

Last ned JetPack 6 `.deb`-pakken og installer:

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

Kontroller installasjonen:

```bash
chloros-cli --version
```

For detaljert Jetson-oppsett, inkludert termisk styring og feltutrulling, se [NVIDIA Jetson-veiledningen](nvidia-jetson-guide.md).

***

## Python SDK Installasjon (Alle Linux)

Python SDK installeres separat via pip og fungerer på både amd64 og arm64:

```bash
pip install chloros-sdk
```

For å inkludere valgfri støtte for strømming av fremdrift:

```bash
pip install chloros-sdk[progress]
```

Bekreft SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
`.deb`-pakken installerer Chloros, CLI og backend. Python SDK er en separat pip-pakke som kommuniserer med backend via en lokal HTTP API.
{% endhint %}

***

## Konfigurasjonskataloger

Chloros på Linux følger [XDG Base Directory Specification](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html):

| Formål | Linux Bane | Windows Tilsvarende |
| --- | --- | --- |
| **Konfigurasjon** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **Data / Prosjekter** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **Cache / påloggingsinformasjon** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## Plasseringer for kjørbare backend-filer

`.deb`-pakken installerer backenden til en standardplassering. CLI og SDK oppdager automatisk banen til backend:

| Installasjonsmetode | Backend-bane |
| --- | --- |
| `.deb`-pakken | `/usr/lib/chloros/chloros-backend` |
| Manuell / tilpasset | `/opt/mapir/chloros/backend/chloros-backend` |

Du kan overstyre backend-banen med flagget `--backend-exe` CLI eller konstruktorparameteren `backend_exe` SDK.

***

## Første gangs oppsett

### 1. Aktiver lisensen din

En Chloros+-lisens er nødvendig for tilgang til CLI og SDK:

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. Sjekk lisensstatusen din

```bash
chloros-cli status
```

### 3. Behandle ditt første datasett

```bash
chloros-cli process ~/datasets/flight001
```

### 4. Kjør systemdiagnostikk

Kontroller at systemet ditt er konfigurert riktig:

```bash
chloros-cli selftest
```

Dette kjører 7 diagnostiske kontroller, inkludert versjon, oppstart av backend, API tilkobling og CUDA/GPU-tilgjengelighet.

***

## Eksempler på Bash-skript

### Behandle flere datasett

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### Behandle med tilpassede innstillinger

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### Automatisert behandling med Cron

Legg til i crontab (`crontab -e`) for å behandle nye datasett automatisk:

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK Eksempel

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Feilsøking

### CLI ikke funnet etter installasjon

Hvis `chloros-cli` ikke blir funnet etter installasjon av `.deb`-pakken:

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### Tilgang nektet

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### Backend kunne ikke startes

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

### CUDA ikke oppdaget

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### Manglende delte biblioteker

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## Oppdatering av Chloros på Linux

Bruk den innebygde oppdateringskommandoen til å søke etter og installere oppdateringer:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## Neste trinn

* [NVIDIA Jetson-veiledning](nvidia-jetson-guide.md) — Jetson-spesifikk optimalisering og distribusjon
* [CLI : Kommandolinje](../CLI.md) — Fullstendig CLI-kommandoreferanse
* [API : Python SDK](../api-python-sdk.md) — Fullstendig SDK-referanse
* [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md) — Hvordan Chloros tilpasser seg maskinvaren din
