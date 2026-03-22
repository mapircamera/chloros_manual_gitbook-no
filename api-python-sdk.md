# API : Python SDK

**Chloros Python SDK** gir programmatisk tilgang til Chloros-bildebehandlingsmotoren, noe som muliggjør automatisering, tilpassede arbeidsflyter og sømløs integrering med dine Python-applikasjoner og forskningspipelines.

### Viktige funksjoner

* 🐍 **Native Python** – Ren, Pythonic API for bildebehandling
* 🔧 **Full API-tilgang** – Full kontroll over Chloros-behandling
* 🚀 **Automatisering** – Bygg tilpassede arbeidsflyter for batchbehandling
* 🔗 **Integrasjon** – Integrer Chloros i eksisterende Python-applikasjoner
* 📊 **Klar for forskning** – Perfekt for vitenskapelige analysepipelines
* ⚡ **Parallellbehandling** - Skalerer etter antall CPU-kjerner (Chloros+)

### Krav

| Krav          | Detaljer                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros installert** | Windows: Installasjonsprogram for skrivebordet; Linux: `.deb`-pakke                  |
| **Lisens**          | Chloros+ ([betalt abonnement kreves](https://cloud.mapir.camera/pricing)) |
| **Operativsystem** | Windows 10/11 (64-bit), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Python**           | Python 3.7 eller nyere                                                |
| **Minne**           | Minst 8 GB RAM (16 GB anbefalt)                                  |
| **Internett**         | Nødvendig for lisensaktivering                                     |

{% hint style="warning" %}
**Lisenskrav**: Python SDK krever et betalt Chloros+-abonnement for tilgang til API. Standardplaner (gratis) har ikke tilgang til API/SDK. Besøk [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) for å oppgradere.
{% endhint %}

## Hurtigstart

### Installasjon

Installer via pip:

```bash
pip install chloros-sdk
```

{% hint style="info" %}
**Første gangs oppsett**: Før du bruker SDK, må du aktivere Chloros+-lisensen din ved å åpne Chloros, Chloros (nettleser) eller Chloros CLI og logg inn med påloggingsinformasjonen din. Dette trenger bare å gjøres én gang. På Linux (uten grafisk brukergrensesnitt), bruk: `chloros-cli login user@example.com 'password'`
{% endhint %}

### Grunnleggende bruk

Behandle en mappe med bare noen få linjer:

```python
from chloros_sdk import process_folder

# One-line processing (Windows)
results = process_folder("C:\\DroneImages\\Flight001")

# One-line processing (Linux)
results = process_folder("/home/user/drone_images/flight001")
```

{% hint style="info" %}
**Plattformuavhengige baner**: Kodeeksemplene på denne siden bruker baner i Windows-stil (f.eks. `C:\\DroneImages\\Flight001`). På Linux bør du i stedet bruke Linux-baner (f.eks. `/home/user/drone_images/flight001` eller `~/drone_images/flight001`). SDK fungerer identisk på begge plattformene.
{% endhint %}

### Full kontroll

For avanserte arbeidsflyter:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")  # Windows
# chloros.import_images("/home/user/drone_images/flight001")  # Linux

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```

***

## Installasjonsveiledning

### Forutsetninger

Før du installerer SDK, må du sørge for at du har:

1. **Chloros installert** — Windows: Installasjonsprogram for skrivebordet ([nedlasting](download.md)); Linux: `.deb`-pakke ([Linux Installasjon](linux/linux-installation.md))
2. **Python 3.7+** installert ([python.org](https://www.python.org))
3. **Aktiv Chloros+ lisens** ([oppgradering](https://cloud.mapir.camera/pricing))

### Installer via pip

**Standardinstallasjon:**

```bash
pip install chloros-sdk
```

**Med støtte for fremdriftsovervåking:**

```bash
pip install chloros-sdk[progress]
```

**Utviklingsinstallasjon:**

```bash
pip install chloros-sdk[dev]
```

### Bekreft installasjon

Test at SDK er installert riktig:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Første gangs oppsett

### Lisensaktivering

SDK bruker samme lisens som Chloros, Chloros (nettleser) og Chloros CLI. Aktiver én gang via GUI eller CLI:**Windows:**Åpne**Chloros eller Chloros (Browser)** og logg inn på <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> , eller bruk CLI.**Linux:** Bruk CLI (ingen GUI tilgjengelig):

```bash
chloros-cli login user@example.com 'your_password'
```

Lisensen lagres lokalt og beholdes ved omstart.

{% hint style="success" %}
**Engangsoppsett**: Etter å ha logget inn via GUI eller CLI, bruker SDK automatisk den bufret lisensen. Ingen ytterligere autentisering nødvendig!
{% endhint %}

{% hint style="info" %}
**Logg ut**: SDK-brukere kan programmatisk slette lagrede påloggingsopplysninger ved hjelp av `logout()`-metoden. Se [logout()-metoden](#logout) i API-referansen.
{% endhint %}

### Test tilkobling

Kontroller at SDK kan koble seg til Chloros:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## API-referanse

### ChlorosLocal-klasse

Hovedklasse for lokal Chloros-bildebehandling.

#### Konstruktør

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**Parametere:**

| Parameter                 | Type | Standard                   | Beskrivelse                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL for lokal Chloros-backend          |
| `auto_start_backend`      | bool | `True`                    | Start backend automatisk om nødvendig |
| `backend_exe`             | str  | `None` (auto-detect)      | Sti til kjørbar backend-fil            |
| `timeout`                 | int  | `30`                      | Tidsavbrudd for forespørsel i sekunder            |
| `backend_startup_timeout` | int  | `60`                      | Tidsavbrudd for oppstart av backend (sekunder) |

**Eksempler:**

```python
# Default (auto-start backend, auto-detect path on Windows and Linux)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path (Windows)
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom backend path (Linux)
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")

# Custom timeout with longer startup (e.g., for Jetson)
chloros = ChlorosLocal(timeout=60, backend_startup_timeout=120)
```

{% hint style="info" %}
**Plattformuavhengig automatisk gjenkjenning**: SDK prøver automatisk den riktige bakendebanen for din plattform:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (manuell)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

***

### Metoder

#### `create_project(project_name, camera=None)`

Opprett et nytt Chloros-prosjekt.

**Parametere:**

| Parameter      | Type | Påkrevd | Beskrivelse                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Ja      | Navn på prosjektet                                     |
| `camera`       | str  | Nei       | Kameramal (f.eks. &quot;Survey3N\_RGN&quot;, «Survey3W\_OCN») |

**Returnerer:** `dict` – Svar på opprettelse av prosjekt**Eksempel:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Importer bilder fra en mappe.

**Parametere:**

| Parameter     | Type     | Påkrevd | Beskrivelse                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | Ja      | Sti til mappe med bilder         |
| `recursive`   | bool     | Nei       | Søk i undermapper (standard: False) |

**Returnerer:** `dict` - Importresultater med antall filer**Eksempel:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Konfigurer behandlingsinnstillinger.

**Parametere:**

| Parameter                 | Type | Standard                 | Beskrivelse                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | &quot;Standard (Rask, Middels kvalitet)&quot; | Debayer-metode            |
| `vignette_correction`     | bool | `True`                  | Aktiver vignettkorreksjon      |
| `reflectance_calibration` | bool | `True`                  | Aktiver reflektanskalibrering  |
| `indices`                 | list | `None`                  | Vegetasjonsindekser som skal beregnes |
| `export_format`           | str  | &quot;TIFF (16-bit)&quot;         | Utdataformat                   |
| `ppk`                     | bool | `False`                 | Aktiver PPK-korreksjoner          |
| `custom_settings`         | dict | `None`                  | Avanserte tilpassede innstillinger        |

**Eksportformater:**

* `"TIFF (16-bit)"` - Anbefalt for GIS/fotogrammetri
* `"TIFF (32-bit, Percent)"` - Vitenskapelig analyse
* `"PNG (8-bit)"` - Visuell inspeksjon
* `"JPG (8-bit)"` - Komprimert utdata

**Tilgjengelige indekser:**NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 og flere.**Eksempel:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

Behandle prosjektbildene.

**Parametere:**

| Parameter           | Type     | Standard      | Beskrivelse                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Behandlingsmodus: «parallel» eller «serial»   |
| `wait`              | bool     | `True`       | Vent på fullføring                       |
| `progress_callback` | callable | `None`       | Tilbakekallingsfunksjon for fremdrift (progress, msg) |
| `poll_interval`     | float    | `2.0`        | Pollingintervall for fremdrift (sekunder)   |

**Returnerer:** `dict` - Behandlingsresultater

{% hint style="warning" %}
**Parallellmodus**: Krever Chloros+ lisens. Skalerer automatisk til CPU-kjernene dine (opptil 16 arbeidere).
{% endhint %}

**Eksempel:**

```python
# Simple processing
results = chloros.process()

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

# Fire-and-forget (non-blocking)
chloros.process(wait=False)
```

***

#### `get_config()`

Hent gjeldende prosjektkonfigurasjon.

**Returnerer:** `dict` - Gjeldende prosjektkonfigurasjon**Eksempel:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Hent informasjon om backend-status, inkludert behandlingsfremdrift per tråd.

**Returnerer:** `dict` - Backend-status med følgende struktur:

```python
{
    "running": True,
    "url": "http://localhost:5000",
    "processing": {
        "percent": 75.0,
        "phase": "processing"
    },
    "export": {
        "percent": 50.0,
        "phase": "exporting",
        "active": True
    }
}
```

**Eksempel:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
print(f"Processing: {status['processing']['percent']}%")
print(f"Export: {status['export']['percent']}% - Active: {status['export']['active']}")
```

***

#### `shutdown_backend()`

Avslutt backend (hvis startet av SDK).

**Eksempel:**

```python
chloros.shutdown_backend()
```

***

#### `logout()`

Tømmer bufret påloggingsinformasjon fra det lokale systemet.

**Beskrivelse:**

Logg ut programmatisk ved å fjerne bufret påloggingsinformasjon. Dette er nyttig for:
* Å bytte mellom forskjellige Chloros+-kontoer
* Å tømme påloggingsinformasjon i automatiserte miljøer
* Sikkerhetsformål (f.eks. å fjerne påloggingsinformasjon før avinstallering)

**Returnerer:** `dict` - Resultat av utloggingsoperasjonen**Eksempel:**

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Clear cached credentials
result = chloros.logout()
print(f"Logout successful: {result}")

# After logout, login required via GUI/CLI/Browser before next SDK use
```

{% hint style="info" %}
**Ny autentisering kreves**: Etter å ha kalt `logout()`, må du logge inn på nytt via Chloros, Chloros (nettleser), eller Chloros CLI før du bruker SDK.
{% endhint %}

***

### Praktiske funksjoner

#### `process_folder(folder_path, **options)`

Enkel funksjon for å behandle en mappe.

**Parametere:**

| Parameter                 | Type     | Standard         | Beskrivelse                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Obligatorisk        | Sti til mappe med bilder     |
| `project_name`            | str      | Autogenerert  | Prosjektnavn                   |
| `camera`                  | str      | `None`          | Kameramal                |
| `indices`                 | liste     | `["NDVI"]`      | Indekser å beregne           |
| `vignette_correction`     | bool     | `True`          | Aktiver vignettkorreksjon     |
| `reflectance_calibration` | bool     | `True`          | Aktiver reflektanskalibrering |
| `export_format`           | str      | &quot;TIFF (16-bit)&quot; | Utdataformat                  |
| `mode`                    | str      | `"parallel"`    | Behandlingsmodus                |
| `progress_callback`       | callable | `None`          | Tilbakekalling for fremdrift              |

**Returnerer:** `dict` - Behandlingsresultater**Eksempel:**

```python
from chloros_sdk import process_folder

# Simple one-liner
results = process_folder("C:\\DroneImages\\Flight001")

# With custom settings
results = process_folder(
    "C:\\DroneImages\\Flight001",
    project_name="Field_A_Survey",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    mode="parallel"
)

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

results = process_folder(
    "C:\\DroneImages\\Flight001",
    progress_callback=show_progress
)
```

***

## Støtte for kontekstbehandlere

SDK støtter kontekstbehandlere for automatisk opprydding:

```python
from chloros_sdk import ChlorosLocal

# Auto-cleanup when done
with ChlorosLocal() as chloros:
    chloros.create_project("MyProject")
    chloros.import_images("C:\\Images")
    chloros.configure(indices=["NDVI"])
    chloros.process()
# Backend automatically shut down here
```

***

## Fullstendige eksempler

{% hint style="info" %}
**Linux-brukere**: Alle eksemplene nedenfor bruker Windows-baner. Erstatt `C:\\...`-baner med dine egne Linux-baner (f.eks. `/home/user/...` eller `~/...`). All SDK-funksjonalitet er identisk på tvers av plattformer.
{% endhint %}

### Eksempel 1: Grunnleggende behandling

Behandle en mappe med standardinnstillinger:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### Eksempel 2: Tilpasset arbeidsflyt

Full kontroll over behandlingsrørledningen:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project with camera template
chloros.create_project("Research_Plot_A", camera="Survey3N_RGN")

# Import images
import_results = chloros.import_images("C:\\Research\\PlotA")
print(f"Imported {len(import_results.get('files', []))} images")

# Configure advanced settings
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=False,
    export_format="TIFF (16-bit)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI"]
)

# Process with progress monitoring
def show_progress(progress, message):
    print(f"Progress: {progress}% - {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

print("Processing complete!")
```

***

### Eksempel 3: Batchbehandling av flere mapper

Behandle flere flydatasett:

```python
from chloros_sdk import ChlorosLocal
from pathlib import Path

# Initialize SDK once
chloros = ChlorosLocal()

# List of flight folders
flights = [
    "C:\\Datasets\\Flight_001",
    "C:\\Datasets\\Flight_002",
    "C:\\Datasets\\Flight_003"
]

for flight_path in flights:
    flight_name = Path(flight_path).name
    print(f"\n{'='*60}")
    print(f"Processing: {flight_name}")
    print('='*60)
    
    try:
        # Create project
        chloros.create_project(flight_name, camera="Survey3N_RGN")
        
        # Import images
        chloros.import_images(flight_path)
        
        # Configure
        chloros.configure(
            vignette_correction=True,
            reflectance_calibration=True,
            indices=["NDVI", "NDRE", "GNDVI"]
        )
        
        # Process
        chloros.process(mode="parallel", wait=True)
        
        print(f"✓ {flight_name} completed successfully")
    
    except Exception as e:
        print(f"✗ {flight_name} failed: {e}")

print("\n" + "="*60)
print("All flights processed!")
```

***

### Eksempel 4: Integrasjon i forskningsrørledning

Integrer Chloros med dataanalyse:

```python
from chloros_sdk import ChlorosLocal
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Chloros
chloros = ChlorosLocal()

# Field survey data
surveys = [
    {"name": "Plot_A", "folder": "C:\\Research\\PlotA", "biomass": 4500},
    {"name": "Plot_B", "folder": "C:\\Research\\PlotB", "biomass": 3800},
    {"name": "Plot_C", "folder": "C:\\Research\\PlotC", "biomass": 5200}
]

results = []

for survey in surveys:
    # Process with Chloros
    chloros.create_project(survey['name'])
    chloros.import_images(survey['folder'])
    chloros.configure(indices=["NDVI", "NDRE"])
    chloros.process(mode="parallel", wait=True)
    
    # Get results
    config = chloros.get_config()
    
    # Extract NDVI values (example - adjust based on your needs)
    # In real implementation, you would read the processed TIFF files
    
    results.append({
        'plot': survey['name'],
        'biomass': survey['biomass'],
        # Add your NDVI extraction here
    })

# Statistical analysis
df = pd.DataFrame(results)
print("\nResults:")
print(df)

# Create correlation plot
# plt.scatter(df['ndvi'], df['biomass'])
# plt.xlabel('NDVI')
# plt.ylabel('Biomass (kg/ha)')
# plt.title('NDVI vs Biomass Correlation')
# plt.show()
```

***

### Eksempel 5: Tilpasset fremdriftsovervåking

Avansert fremdriftssporing med loggføring:

```python
from chloros_sdk import ChlorosLocal
from datetime import datetime
import logging

# Setup logging
logging.basicConfig(
    filename=f'processing_{datetime.now():%Y%m%d_%H%M%S}.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

# Progress callback with logging
def log_progress(progress, message):
    log_msg = f"[{progress}%] {message}"
    logging.info(log_msg)
    print(log_msg)

# Process with logging
chloros = ChlorosLocal()
chloros.create_project("LoggedProcess")
chloros.import_images("C:\\DroneImages")
chloros.configure(indices=["NDVI", "NDRE"])

logging.info("Starting processing...")
chloros.process(
    mode="parallel",
    progress_callback=log_progress,
    wait=True
)
logging.info("Processing complete!")
```

***

### Eksempel 6: Feilhåndtering

Robust feilhåndtering for produksjonsbruk:

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import (
    ChlorosError,
    ChlorosBackendError,
    ChlorosLicenseError,
    ChlorosProcessingError
)

def process_safely(folder_path):
    """Process with comprehensive error handling"""
    try:
        with ChlorosLocal() as chloros:
            chloros.create_project("SafeProcess")
            chloros.import_images(folder_path)
            chloros.configure(indices=["NDVI"])
            chloros.process()
            
        return True, "Success"
    
    except ChlorosLicenseError as e:
        return False, f"License error: {e}. Upgrade to Chloros+ at cloud.mapir.camera/pricing"
    
    except ChlorosBackendError as e:
        return False, f"Backend error: {e}. Ensure Chloros is installed (Windows installer or Linux .deb package)."
    
    except ChlorosProcessingError as e:
        return False, f"Processing error: {e}"
    
    except FileNotFoundError as e:
        return False, f"Folder not found: {e}"
    
    except ChlorosError as e:
        return False, f"Chloros error: {e}"
    
    except Exception as e:
        return False, f"Unexpected error: {e}"

# Use the safe function
success, message = process_safely("C:\\DroneImages\\Flight001")
if success:
    print(f"✓ {message}")
else:
    print(f"✗ {message}")
```

***

### Eksempel 7: Kontoadministrasjon og utlogging

Administrer påloggingsinformasjon programmatisk:

```python
from chloros_sdk import ChlorosLocal

def switch_account():
    """Clear credentials to switch to a different account"""
    try:
        chloros = ChlorosLocal()
        
        # Clear current credentials
        result = chloros.logout()
        print("✓ Credentials cleared successfully")
        print("Please log in with new account via Chloros, Chloros (Browser), or CLI")
        
        return True
    
    except Exception as e:
        print(f"✗ Logout failed: {e}")
        return False

def secure_cleanup():
    """Remove credentials for security purposes"""
    try:
        chloros = ChlorosLocal()
        chloros.logout()
        print("✓ Credentials removed for security")
        
    except Exception as e:
        print(f"Warning: Cleanup error: {e}")

# Switch accounts
if switch_account():
    print("\nRe-authenticate via Chloros GUI/CLI/Browser before next SDK use")

# Or perform secure cleanup
# secure_cleanup()
```

***

### Eksempel 8: Kommandolinjeverktøy

Bygg et tilpasset CLI-verktøy med SDK:

```python
#!/usr/bin/env python
"""
Custom Chloros CLI Tool
Process multiple folders from command line
"""

import sys
import argparse
from pathlib import Path
from chloros_sdk import process_folder

def main():
    parser = argparse.ArgumentParser(description='Custom Chloros Processor')
    parser.add_argument('folders', nargs='+', help='Folders to process')
    parser.add_argument('--indices', nargs='+', default=['NDVI'],
                       help='Indices to calculate (default: NDVI)')
    parser.add_argument('--camera', default=None,
                       help='Camera template')
    parser.add_argument('--format', default='TIFF (16-bit)',
                       help='Export format')
    parser.add_argument('--logout', action='store_true',
                       help='Clear cached credentials before processing')
    
    args = parser.parse_args()
    
    # Handle logout if requested
    if args.logout:
        from chloros_sdk import ChlorosLocal
        chloros = ChlorosLocal()
        chloros.logout()
        print("Credentials cleared. Please re-login via Chloros GUI/CLI/Browser.")
        return 0
    
    successful = []
    failed = []
    
    for folder in args.folders:
        folder_path = Path(folder)
        
        if not folder_path.exists():
            print(f"✗ Skipping {folder}: not found")
            failed.append(folder)
            continue
        
        print(f"\nProcessing: {folder_path.name}...")
        
        try:
            process_folder(
                folder_path,
                camera=args.camera,
                indices=args.indices,
                export_format=args.format
            )
            print(f"✓ {folder_path.name} complete")
            successful.append(folder)
        
        except Exception as e:
            print(f"✗ {folder_path.name} failed: {e}")
            failed.append(folder)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"Summary: {len(successful)} successful, {len(failed)} failed")
    
    return 0 if not failed else 1

if __name__ == '__main__':
    sys.exit(main())
```

**Bruk:**

```bash
# Process multiple folders
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI

# Clear cached credentials
python my_processor.py --logout
```

***

## Unntakshåndtering

SDK tilbyr spesifikke unntaksklasser for ulike feiltyper:

### Unntakshierarki

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### Eksempler på unntak

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros is installed (Windows installer or Linux .deb package).")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## Avanserte emner

### Tilpasset backend-konfigurasjon

Bruk en tilpasset backend-plassering eller -konfigurasjon:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Ikke-blokkerende behandling

Start behandlingen og fortsett med andre oppgaver:

```python
# Start processing (non-blocking)
chloros.process(wait=False)

# Do other work here...
print("Processing started in background...")

# Check status later
import time
while True:
    status = chloros.get_config()
    if status.get('processing_complete'):
        break
    time.sleep(5)

print("Processing complete!")
```

### Minnehåndtering

For store datasett, behandle i batcher:

```python
from pathlib import Path

base_folder = Path("C:\\LargeDataset")
batch_size = 100

# Get all image files
images = list(base_folder.glob("*.RAW"))

# Process in batches
for i in range(0, len(images), batch_size):
    batch = images[i:i+batch_size]
    batch_folder = base_folder / f"batch_{i//batch_size}"
    
    # Create batch folder and move images
    # ... (implementation details)
    
    # Process batch
    process_folder(batch_folder)
```

***

## Feilsøking

### Backend starter ikke

**Problem:** SDK klarer ikke å starte backend**Løsninger:**

1. Kontroller at Chloros er installert:

```python
import os
import platform

# Auto-detect backend path
if platform.system() == "Windows":
    backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
else:
    backend_path = "/usr/lib/chloros/chloros-backend"

print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Sjekk brannmur (Windows) eller porttilgjengelighet (Linux: `lsof -i :5000`)
3. Prøv manuell bakgrunnssti:

```python
# Windows
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")

# Linux
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")
```

***

### Lisens ikke oppdaget**Problem:** SDK varsler om manglende lisens**Løsninger:**

1. Åpne Chloros, Chloros (nettleser) eller Chloros CLI og logg inn.
2. Kontroller at lisensen er lagret i hurtigbufferen:

```python
from pathlib import Path
import os
import platform

# Check cache location
if platform.system() == "Windows":
    cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
else:
    cache_path = Path.home() / '.cache' / 'chloros'

print(f"Cache exists: {cache_path.exists()}")
```

3. Hvis du opplever problemer med påloggingsinformasjon, tøm den lagrede påloggingsinformasjonen og logg inn på nytt:

```python
from chloros_sdk import ChlorosLocal

# Clear cached credentials
chloros = ChlorosLocal()
chloros.logout()

# Then login again via Chloros, Chloros (Browser), or Chloros CLI
```

4. Kontakt support: info@mapir.camera

***

### Importfeil**Problem:** `ModuleNotFoundError: No module named 'chloros_sdk'`**Løsninger:**

```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```

***

### Tidsavbrudd i behandlingen**Problem:** Tidsavbrudd i behandlingen**Løsninger:**

1. Øk tidsavbruddet:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Behandle mindre batcher
3. Sjekk ledig diskplass
4. Overvåk systemressurser

***

### Porten er allerede i bruk**Problem:** Backend-port 5000 opptatt**Løsninger:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Eller finn og lukk prosessen som forårsaker konflikten:

```powershell
# Windows PowerShell
Get-NetTCPConnection -LocalPort 5000
```

```bash
# Linux
lsof -i :5000
kill $(lsof -t -i :5000)
```

***

## Tips for ytelse

### Optimaliser behandlingshastigheten

1. **Bruk parallellmodus** (krever Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Reduser utskriftsoppløsningen** (hvis akseptabelt)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Deaktiver unødvendige indekser**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Behandle på SSD** (ikke HDD)***

### Minneoptimalisering

For store datasett:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### Bakgrunnsbehandling

Frigjør Python til andre oppgaver:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## Integrasjonseksempler

### Django-integrasjon

```python
# views.py
from django.http import JsonResponse
from chloros_sdk import process_folder

def process_images_view(request):
    if request.method == 'POST':
        folder_path = request.POST.get('folder_path')
        
        try:
            results = process_folder(folder_path)
            return JsonResponse({'success': True, 'results': results})
        except Exception as e:
            return JsonResponse({'success': False, 'error': str(e)})
```

### Flask API

```python
# app.py
from flask import Flask, request, jsonify
from chloros_sdk import process_folder

app = Flask(__name__)

@app.route('/api/process', methods=['POST'])
def process():
    data = request.get_json()
    folder_path = data.get('folder_path')
    
    try:
        results = process_folder(folder_path)
        return jsonify({'success': True, 'results': results})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run()
```

### Jupyter Notebook

```python
# notebook.ipynb
from chloros_sdk import ChlorosLocal
import matplotlib.pyplot as plt

# Initialize
chloros = ChlorosLocal()

# Process
chloros.create_project("JupyterTest")
chloros.import_images("C:\\Data")
chloros.configure(indices=["NDVI"])

# Progress in notebook
from IPython.display import clear_output

def notebook_progress(progress, message):
    clear_output(wait=True)
    print(f"Progress: {progress}%")
    print(message)

chloros.process(progress_callback=notebook_progress)

# Visualize results
# ... (your visualization code)
```

***

## FAQ

### Spørsmål: Krever SDK en internettforbindelse?

**Svar:** Kun for den første lisensaktiveringen. Etter innlogging via Chloros, Chloros (nettleser) eller Chloros CLI, blir lisensen lagret lokalt og fungerer offline i 30 dager.***

### Spørsmål: Kan jeg bruke SDK på en server uten grafisk brukergrensesnitt?**Svar:** Ja! SDK fungerer uten grafisk brukergrensesnitt på både Windows- og Linux-servere.**Linux (anbefalt for bruk uten grafisk brukergrensesnitt):**
* Installer via `.deb`-pakken
* Aktiver lisens: `chloros-cli login user@example.com 'password'`

**Windows-server:**
* Windows Server 2016 eller nyere
* Chloros installert (engangs)
* Lisens aktivert via CLI eller på en hvilken som helst maskin

***

### Spørsmål: Hva er forskjellen mellom Desktop, CLI og SDK?

| Funksjon         | Desktop GUI | CLI Kommandolinje | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Grensesnitt**   | Pek-og-klikk | Kommando          | Python API  |
| **Best egnet for**    | Visuelt arbeid | Skripting        | Integrasjon |
| **Automatisering**  | Begrenset     | God             | Utmerket   |
| **Fleksibilitet** | Grunnleggende       | God             | Maksimal     |
| **Lisens**     | Chloros+    | Chloros+         | Chloros+    |***

### Spørsmål: Kan jeg distribuere apper bygget med SDK?**Svar:** SDK-kode kan integreres i applikasjonene dine, men:

* Sluttbrukere må ha Chloros installert
* Sluttbrukere må ha aktive Chloros+-lisenser
* Kommersiell distribusjon krever OEM-lisensiering

Kontakt info@mapir.camera for spørsmål om OEM.

***

### Spørsmål: Hvordan oppdaterer jeg SDK?

```bash
pip install --upgrade chloros-sdk
```

***

### Spørsmål: Hvor lagres behandlede bilder?

Som standard i prosjektbanen:

```

Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### Spørsmål: Kan jeg behandle bilder fra Python-skript som kjører etter en tidsplan?**Svar:** Ja! Bruk operativsystemets tidsplanlegger med Python-skript:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("/data/flights/today")  # Linux
# results = process_folder("C:\\Flights\\Today")  # Windows
```

**Windows:** Planlegg via Oppgaveplanlegger for daglig kjøring.**Linux:** Planlegg via cron:

```cron
# Run at 2 AM daily
0 2 * ** /usr/bin/python3 /home/user/scheduled_processing.py >> /var/log/chloros.log 2>&1
```

***

### Spørsmål: Støtter SDK async/await?**Svar:** Den nåværende versjonen er synkron. For asynkron oppførsel, bruk `wait=False` eller kjør i en egen tråd:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

### Spørsmål: Hvordan bytter jeg mellom forskjellige Chloros+-kontoer?**Svar:** Bruk metoden `logout()` for å tømme bufret påloggingsinformasjon, og logg deretter på igjen med den nye kontoen:

```python
from chloros_sdk import ChlorosLocal

# Clear current credentials
chloros = ChlorosLocal()
chloros.logout()

# Re-login via Chloros, Chloros (Browser), or Chloros CLI with new account
```

Etter utlogging må du autentisere deg med den nye kontoen via GUI, nettleser eller CLI før du bruker SDK igjen.

***

## Få hjelp

### Dokumentasjon

* **API-referanse**: Denne siden

### Supportkanaler

* **E-post**: info@mapir.camera
* **Nettsted**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Priser**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Eksempelkode

Alle eksemplene som er oppført her, er testet og klare for produksjon. Kopier og tilpass dem til ditt bruksområde.

***

## Lisens**Proprietær programvare** – Copyright (c) 2025 MAPIR Inc.

SDK krever et aktivt Chloros+-abonnement. Uautorisert bruk, distribusjon eller modifisering er forbudt.
