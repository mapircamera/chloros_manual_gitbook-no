# CLI : Kommandolinje

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** gir kraftig kommandolinjetilgang til bildebehandlingsmotoren Chloros, noe som muliggjør automatisering, skripting og headless drift for bildebehandlingsarbeidsflytene dine.

### Viktige funksjoner

* 🚀 **Automatisering** – Skriptbasert batchbehandling av flere datasett
* 🔗 **Integrasjon** – Integrer i eksisterende arbeidsflyter og rørledninger
* 💻 **Headless-drift** – Kjør uten GUI
* 🌍 **Flerspråklig** – Støtte for 38 språk
* ⚡ **Parallellbehandling** – [Dynamisk beregnings tilpasning](processing-architecture/dynamic-compute-adaptation.md) optimaliserer automatisk for maskinvaren din

### Krav

| Krav          | Detaljer                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Operativsystem** | Windows 10/11 (64-bit), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Lisens**          | Chloros+ ([betalt abonnement kreves](https://cloud.mapir.camera/pricing)) |
| **Minne**           | Minst 8 GB RAM (16 GB anbefales)                                  |
| **Internett**         | Kreves for lisensaktivering                                     |
| **Diskplass**       | Varierer etter prosjektstørrelse                                              |

{% hint style="warning" %}
**Lisenskrav**: CLI krever et betalt Chloros+-abonnement. Standardabonnementer (gratis) har ikke tilgang til CLI. Gå til [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) for å oppgradere.
{% endhint %}

## Hurtigstart

### Installasjon

#### Windows

CLI er automatisk inkludert i Chloros-installasjonsprogrammet:

1. Last ned og kjør **Chloros Installer.exe**

2. Fullfør installasjonsveiviseren
3. CLI installert til: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
Installasjonsprogrammet legger automatisk til `chloros-cli` i systemets PATH. Start terminalen på nytt etter installasjonen.
{% endhint %}

#### Linux

Installer `.deb`-pakken for din arkitektur:

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

For detaljert informasjon om oppsett av Linux, se [Linux Installasjon](linux/linux-installation.md).

### Første gangs oppsett

Før du bruker CLI, må du aktivere Chloros+-lisensen din:

**Windows:**

```powershell
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process ~/images/dataset001
```

### Grunnleggende bruk

Behandle en mappe med standardinnstillinger:

**Windows:**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
chloros-cli process ~/images/dataset001
```

***

## Kommandoreferanse

### Generell syntaks

```
chloros-cli [global-options] <command> [command-options]
```

***

## Kommandoer

### `process` - Behandle bilder

Behandle bilder i en mappe med kalibrering.

**Syntaks:**

```bash
chloros-cli process <input-folder> [options]
```

**Eksempler:**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### Kommandoalternativer

| Alternativ                | Type    | Standard        | Beskrivelse                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Bane    | _Påkrevd_     | Mappe som inneholder RAW/JPG-multispektrale bilder                                         |
| `-o, --output`        | Bane    | Samme som inndata  | Utdatamappe for behandlede bilder                                                     |
| `-n, --project-name`  | Streng  | Autogenerert | Egendefinert prosjektnavn                                                                    |
| `--vignette`          | Flagg    | Aktivert        | Aktiver vignettkorrigering                                                             |
| `--no-vignette`       | Flagg    | -              | Deaktiver vignettkorrigering                                                            |
| `--reflectance`       | Flagg    | Aktivert        | Aktiver refleksjonskalibrering                                                         |
| `--no-reflectance`    | Flagg    | -              | Deaktiver refleksjonskalibrering                                                        |
| `--ppk`               | Flagg    | Deaktivert       | Bruk PPK-korreksjoner fra .daq-lyssensordata                                      |
| `--format`            | Valg  | TIFF (16-bit)  | Utdataformat: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Heltall | Auto           | Minste målstørrelse i piksler for deteksjon av kalibreringspanel                          |
| `--target-clustering` | Heltall | Auto           | Terskel for målgruppering (0–100)                                                    |
| `--debayer`           | Valg  | `standard`     | Debayer-metode: `standard` eller `texture-aware` (kun Chloros+)                          |
| `--target`, `--targets` | Flagg  | Deaktivert       | Søk kun etter kalibreringsmål i en «target»- eller «targets»-undermappe (fremskynder behandlingen) |
| `--indices`           | Liste    | Ingen           | Vegetasjonsindekser som skal beregnes (f.eks. `--indices NDVI NDRE GNDVI`)                    |
| `--exposure-pin-1`    | Streng  | Ingen           | Lås eksponering for kameramodell (Pin 1)                                                 |
| `--exposure-pin-2`    | Streng  | Ingen           | Lås eksponering for kameramodell (Pin 2)                                                 |
| `--recal-interval`    | Heltall | Auto           | Rekalibreringsintervall i sekunder                                                      |
| `--timezone-offset`   | Heltall | 0              | Tidssoneforskjell i timer                                                               |

***

### `login` - Autentiser konto

Logg inn med dine Chloros+-påloggingsopplysninger for å aktivere CLI-behandling.

**Syntaks:**

```bash
chloros-cli login <email> <password>
```

**Eksempel:**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Spesialtegn**: Bruk enkle anførselstegn rundt passord som inneholder tegn som `$`, `!` eller mellomrom.
{% endhint %}

**Utdata:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Slett påloggingsinformasjon

Slett lagret påloggingsinformasjon og logg ut av kontoen din.

**Syntaks:**

```bash
chloros-cli logout
```

**Eksempel:**

```bash
chloros-cli logout
```

**Utdata:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**SDK-brukere**: Python SDK tilbyr også en programmatisk `logout()`-metode for å slette påloggingsinformasjon i Python-skript. Se [Python SDK dokumentasjonen](api-python-sdk.md#logout) for detaljer.
{% endhint %}

***

### `status` - Sjekk lisensstatus

Vis gjeldende lisens- og autentiseringsstatus.

**Syntaks:**

```bash
chloros-cli status
```

**Eksempel:**

```bash
chloros-cli status
```

**Utdata:**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` – Sjekk eksportfremdrift

Overvåk eksportfremdriften for tråd 4 under eller etter behandlingen.

**Syntaks:**

```bash
chloros-cli export-status
```

**Eksempel:**

```bash
chloros-cli export-status
```

**Bruksområde:** Kall denne kommandoen mens behandlingen pågår for å sjekke eksportfremdriften.***

### `language` – Administrer grensesnittspråk

Vis eller endre grensesnittspråket for CLI.

**Syntaks:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**Eksempler:**

```bash
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### Støttede språk (38 totalt)

| Kode    | Språk              | Navn på originalspråket      |
| ------- | --------------------- | ---------------- |
| `en`    | Engelsk               | English          |
| `es`    | Spansk               | Español          |
| `pt`    | Portugisisk            | Português        |
| `fr`    | Fransk                | Français         |
| `de`    | Tysk                | Deutsch          |
| `it`    | Italiensk               | Italiano         |
| `ja`    | Japansk              | 日本語              |
| `ko`    | Koreansk                | 한국어              |
| `zh`    | Kinesisk (forenklet)  | 简体中文             |
| `zh-TW` | Kinesisk (tradisjonell) | 繁體中文             |
| `ru`    | Russisk               | Русский          |
| `nl`    | Nederlandsk                | Nederlands       |
| `ar`    | Arabisk                | العربية          |
| `pl`    | Polsk                | Polski           |
| `tr`    | Tyrkisk               | Türkçe           |
| `hi`    | Hindi                 | हिंदी            |
| `id`    | Indonesisk            | Bahasa Indonesia |
| `vi`    | Vietnamesisk            | Tiếng Việt       |
| `th`    | Thai                  | ไทย              |
| `sv`    | Svensk               | Svenska          |
| `da`    | Dansk                | Dansk            |
| `no`    | Norsk             | Norsk            |
| `fi`    | Finsk               | Suomi            |
| `el`    | Gresk                 | Ελληνικά         |
| `cs`    | Tsjekkisk                | Čeština          |
| `hu`    | Ungarsk             | Magyar           |
| `ro`    | Rumensk              | Română           |
| `uk`    | Ukrainsk             | Українська       |
| `pt-BR` | Brasiliansk portugisisk  | Português Brasileiro |
| `zh-HK` | Kantonesisk             | 粵語             |
| `ms`    | Malay                 | Bahasa Melayu    |
| `sk`    | Slovakisk                | Slovenčina       |
| `bg`    | Bulgarsk             | Български        |
| `hr`    | Kroatisk              | Hrvatski         |
| `lt`    | Litauisk            | Lietuvių         |
| `lv`    | Lettisk               | Latviešu         |
| `et`    | Estisk              | Eesti            |
| `sl`    | Slovensk             | Slovenščina      |

{% hint style="success" %}
**Automatisk lagring**: Språkinnstillingen din lagres i `~/.chloros/cli_language.json` og beholdes gjennom alle økter.
{% endhint %}

***

### `set-project-folder` - Angi standard prosjektmappe

Endre plasseringen til standard prosjektmappen (delt med GUI på Windows).

**Syntaks:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Eksempler:**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` - Vis prosjektmappe

Vis gjeldende standardprosjektmappe.

**Syntaks:**

```bash
chloros-cli get-project-folder
```

**Eksempel:**

```bash
chloros-cli get-project-folder
```

**Utdata:**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` - Tilbakestill til standard

Tilbakestill prosjektmappen til standardplasseringen.

**Syntaks:**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` - Kjør systemdiagnostikk

Kjør 7 diagnostiske kontroller for å verifisere systemkonfigurasjonen.

**Syntaks:**

```bash
chloros-cli selftest
```

**Utførte diagnostiske tester:**

1. Versjonssjekk
2. Porttilgjengelighet (5000)
3. Oppstart av backend
4. API tilkoblingstest
5. Systeminformasjon og GPU-deteksjon
6. Verifisering av støyreduksjonsmodeller
7. Sjekk av CUDA-tilgjengelighet

{% hint style="info" %}
**Nyttig for feilsøking**: Kjør `selftest` etter installasjon for å verifisere at systemet er konfigurert riktig, spesielt på Linux/Jetson hvor GPU- og CUDA-oppsettet kan trenge verifisering.
{% endhint %}

***

### `update` – Sjekk etter oppdateringer (kun Linux)

Sjekk etter og installer CLI-oppdateringer på Linux-systemer.

**Syntaks:**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| Alternativ    | Beskrivelse                        |
| --------- | ---------------------------------- |
| `--check` | Bare søk etter oppdateringer, ikke installer |

{% hint style="info" %}
Denne kommandoen er kun tilgjengelig på Linux. På Windows leveres oppdateringer via installasjonsprogrammet.
{% endhint %}

***

## Globale alternativer

Disse alternativene gjelder for alle kommandoer:

| Alternativ            | Type    | Standard       | Beskrivelse                                      |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe`   | Sti    | Oppdages automatisk | Sti til kjørbar backend-fil                       |
| `--port`          | Heltall | 5000          | Portnummer for backend API                          |
| `--restart`       | Flagg    | -             | Tving omstart av backend (avslutter eksisterende prosesser) |
| `--version`       | Flagg    | -             | Vis versjonsinformasjon og avslutt                |
| `--help`          | Flagg    | -             | Vis hjelpeinformasjon og avslutt                   |

{% hint style="info" %}
**Automatisk gjenkjenning av backend**: Banen `--backend-exe` gjenkjennes automatisk per plattform:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (manuell)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**Eksempel med globale alternativer:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux:**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## Veiledning for behandlingsinnstillinger

### Parallellbehandling og dynamisk beregnings tilpasning

Chloros 1.1.0 inkluderer [Dynamisk beregnings tilpasning](processing-architecture/dynamic-compute-adaptation.md) — behandlingsmotoren **oppdager automatisk maskinvaren din** og velger den optimale strategien:

| Plattform | Strategi | Arbeidere | Rørledning | Merknader |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` | Minneeffektiv, serialisert |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` | Samtidig GPU-behandling |
| **Stasjonær PC med 8 GB GPU** | `GPU_SINGLE` | 3 | `tiled_gpu` | God stasjonær ytelse |
| **Stasjonær PC med 12 GB+ GPU** | `GPU_PARALLEL` | 3–4 | `fused_gpu` | Optimal stasjonær ytelse |
| **System kun med CPU** | `CPU_PARALLEL` | kjerner - 1 | `cpu_fallback` | Ingen GPU nødvendig |

{% hint style="success" %}
**Ingen manuell konfigurasjon nødvendig!** Chloros oppdager automatisk CPU, GPU, RAM og (på Jetson) termiske sensorer, og konfigurerer deretter den optimale prosesseringspipeline automatisk.
{% endhint %}

### Debayer-metoder

| Metode | CLI-flagg | Kvalitet | Hastighet | Lisens |
| --- | --- | --- | --- | --- |
| **Standard (rask, middels kvalitet)** | `--debayer standard` | God | Rask | Gratis / Chloros+ |
| **Teksturbevisst (Langsom, høyeste kvalitet)** | `--debayer texture-aware` | Høyeste | Langsom | Kun Chloros+ |

Standardmetoden for debayering er **Standard**. Metoden**Teksturbevisst** bruker en AI/ML-støyreduksjonsmodell for å oppnå utdata av høyeste kvalitet, men krever en Chloros+-lisens og en NVIDIA-GPU.

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### Vignettkorrigering

**Hva det gjør:** Korrigerer lysfall ved bildekantene (mørkere hjørner som ofte forekommer i kamerabilder).

* **Aktivert som standard** – De fleste brukere bør holde dette aktivert
* Bruk `--no-vignette` for å deaktivere

{% hint style="success" %}
**Anbefaling**: Aktiver alltid vignettkorreksjon for å sikre jevn lysstyrke over hele bildet.
{% endhint %}

### Refleksjonskalibrering

Konverterer rå sensorverdier til standardiserte refleksjonsprosentandeler ved hjelp av kalibreringspaneler.

* **Aktivert som standard** – Avgjørende for vegetasjonsanalyse
* Krever kalibreringsmålpaneler i bildene
* Bruk `--no-reflectance` for å deaktivere

{% hint style="info" %}
**Krav**: Sørg for at kalibreringspanelene er riktig eksponert og synlige i bildene dine for nøyaktig reflektanskonvertering.
{% endhint %}

### PPK-korreksjoner

**Hva det gjør:** Bruker etterbehandlede kinematiske korreksjoner ved hjelp av DAQ-A-SD-loggdata for forbedret GPS-nøyaktighet.

* **Deaktivert som standard**
* Bruk `--ppk` for å aktivere
* Krever .daq-filer i prosjektmappen fra MAPIR DAQ-A-SD-lyssensor.

### Utdataformater

<table><thead><tr><th width="197">Format</th><th width="130.20001220703125">Bittdybde</th><th width="116.5999755859375">Filstørrelse</th><th>Best egnet for</th></tr></thead><tbody><tr><td><strong>TIFF (16-bit)</strong> ⭐</td><td>16-biters heltall</td><td>Stor</td><td>GIS-analyse, fotogrammetri (anbefalt)</td></tr><tr><td><strong>TIFF (32-bit, prosent)</strong></td><td>32-biters flytende</td><td>Svært stor</td><td>Vitenskapelig analyse, forskning</td></tr><tr><td><strong>PNG (8-bit)</strong></td><td>8-biters heltall</td><td>Middels</td><td>Visuell inspeksjon, deling på nettet</td></tr><tr><td><strong>JPG (8-bit)</strong></td><td>8-biters heltall</td><td>Liten</td><td>Hurtig forhåndsvisning, komprimert utdata</td></tr></tbody></table>***

## Automatisering og skripting

### PowerShell-batchbehandling (Windows)

Behandle flere datasettmapper automatisk på Windows:

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### Windows Batch-skript (Windows)

Enkel sløyfe for batchbehandling på Windows:

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### Bash-batchbehandling (Linux)

Behandle flere datasettmapper på Linux:

```bash
#!/bin/bash
# process_all_datasets.sh

for dataset in ~/datasets/2026/*/; do
    name=$(basename "$dataset")
    echo "Processing $name..."

    chloros-cli process "$dataset" \
        --vignette \
        --reflectance

    if [ $? -eq 0 ]; then
        echo "✓ $name complete"
    else
        echo "✗ $name failed"
    fi
done

echo "All datasets processed!"
```

### Python automatiseringsskript (plattformuavhengig)

Avansert automatisering med feilhåndtering (fungerer på Windows og Linux):

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    # Adjust path for your platform
    # Windows: Path('C:/Datasets/2025')
    # Linux:   Path.home() / 'datasets' / '2025'
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## Behandlingsarbeidsflyt

### Standard arbeidsflyt

1. **Inndata**: Mappe som inneholder RAW/JPG-bildepar
2. **Oppdagelse**: CLI skanner automatisk etter støttede bildefiler
3. **Behandling**: Parallellmodus skaleres etter antall CPU-kjerner (Chloros+)
4. **Utdata**: Oppretter undermapper for kameramodeller med behandlede bilder

### Eksempel på utdatastruktur

```

MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### Estimert behandlingstid

Typisk behandlingstid for 100 bilder (12 MP hver):

| Plattform | Modus | Estimert tid | Merknader |
| --- | --- | --- | --- |
| **Stasjonær PC med 12 GB+ GPU** | `GPU_PARALLEL` | 5–10 min | Raskeste alternativ |
| **Stasjonær PC med 8 GB GPU** | `GPU_SINGLE` | 10–15 min | God ytelse |
| **Jetson Orin 16 GB** | `GPU_PARALLEL` | 15–25 min | Edge-beregning |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 30–60 min | Begrenset minne |
| **Kun CPU** | `CPU_PARALLEL` | 20–40 min | Ingen GPU nødvendig |

{% hint style="info" %}
**Ytelsestips**: Behandlingstiden varierer avhengig av antall bilder, oppløsning, debayer-metode og maskinvare. Texture Aware-debayer tar betydelig lengre tid enn Standard. Se [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) for detaljer.
{% endhint %}

***

## Feilsøking

### CLI ikke funnet

**Windows-feil:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Windows Løsninger:**

1. Kontroller installasjonsstedet:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Bruk full sti hvis ikke i PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Legg til i PATH manuelt:
   * Åpne Systemegenskaper → Miljøvariabler
   * Rediger PATH-variabelen
   * Legg til: `C:\Program Files\Chloros\resources\cli`
   * Start terminalen på nytt

**Linux Feil:**

```
chloros-cli: command not found
```

**Linux Løsninger:**

1. Kontroller installasjonen:

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. Last inn skallet på nytt:

```bash
source ~/.bashrc
```

3. Kontroller tillatelser:

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### Backend kunne ikke startes**Feil:**

```

Backend failed to start within 30 seconds
```

**Løsninger:**

1. Sjekk om backend allerede kjører (lukk den først)
2. Sjekk at brannmuren ikke blokkerer (Windows) eller sjekk porttilgjengelighet (Linux: `lsof -i :5000`)
3. Prøv en annen port:

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. Tving omstart av backend:

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. På Linux, sjekk om backend-kjørbar fil finnes:

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### Lisens-/autentiseringsproblemer**Feil:**

```

Chloros+ license required for CLI access
```

**Løsninger:**

1. Kontroller at du har et aktivt Chloros+-abonnement
2. Logg inn med påloggingsinformasjonen din:

```bash
chloros-cli login user@example.com 'password'
```

3. Sjekk lisensstatus:

```bash
chloros-cli status
```

4. Kontakt support: info@mapir.camera

***

### Ingen bilder funnet**Feil:**

```

No images found in the specified folder
```

**Løsninger:**

1. Kontroller at mappen inneholder støttede formater (.RAW, .TIF, .JPG)
2. Kontroller at mappestien er riktig (bruk anførselstegn for stier med mellomrom)
3. Sørg for at du har lesetillatelse for mappen
4. Kontroller at filtypene er riktige

***

### Behandlingen stopper opp eller henger seg opp**Løsninger:**

1. Sjekk tilgjengelig diskplass (sørg for at det er nok til utdata)
2. Lukk andre programmer for å frigjøre minne
3. Reduser antall bilder (behandle i batcher)

***

### Porten er allerede i bruk**Feil:**

```

Port 5000 is already in use
```

**Løsninger:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

**Linux:**

```bash
# Find what's using port 5000
lsof -i :5000

# Use a different port
chloros-cli --port 5001 process ~/datasets/field_a
```

***

## FAQ

### Spørsmål: Trenger jeg en lisens for CLI?

**Svar:**Ja! CLI krever en betalt**Chloros+-lisens**.

* ❌ Standard (gratis) abonnement: CLI deaktivert
* ✅ Chloros+ (betalt) abonnement: CLI fullt aktivert

Abonner på: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### Spørsmål: Kan jeg bruke CLI på en server uten GUI?**Svar:** Ja! CLI kjører helt uten grafisk grensesnitt. Dette er den primære brukssituasjonen på Linux.**Windows-server:**
* Windows Server 2016 eller nyere
* Visual C++ Redistributable installert

**Linux-server:**
* Ubuntu 20.04+ / Debian 11+ (amd64) eller JetPack 6 (arm64)
* Installer via `.deb`-pakken

**Begge plattformene:**
* Minst 8 GB RAM (16 GB anbefalt)
* Engangsaktivering av lisens: `chloros-cli login user@example.com 'password'`

***

### Spørsmål: Hvor lagres de behandlede bildene?**Svar:**Som standard lagres behandlede bilder i**samme mappe som inndata** i undermapper for kameramodeller (f.eks. `Survey3N_RGN/`).

Bruk alternativet `-o` for å angi en annen utdatamappe:

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### Spørsmål: Kan jeg behandle flere mapper samtidig?**A:** Ikke direkte med én kommando, men du kan bruke skripting for å behandle mapper sekvensielt. Se avsnittet [Automatisering og skripting](CLI.md#automation--scripting).***

### Spørsmål: Hvordan lagrer jeg CLI-utdata i en loggfil?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

**Linux Bash:**

```bash
chloros-cli process ~/datasets/field_a 2>&1 | tee processing.log
```

***

### Spørsmål: Hva skjer hvis jeg trykker Ctrl+C under behandlingen?**Svar:** CLI vil:

1. Avslutte behandlingen på en ordnet måte
2. Slå av backend-systemet
3. Avslutte med kode 130

Delvis behandlede bilder kan forbli i utdatamappen.

***

### Spørsmål: Kan jeg automatisere CLI-behandlingen?**Svar:** Absolutt! CLI er designet for automatisering. Se [Automatisering og skripting](CLI.md#automation--scripting) for eksempler på PowerShell (Windows), Batch (Windows), Bash (Linux) og Python (plattformuavhengig).***

### Spørsmål: Hvordan sjekker jeg CLI-versjonen?**Svar:**

```bash
chloros-cli --version
```

**Utdata:**

```

Chloros CLI 1.1.0
```

***

## Hjelp

### Hjelp på kommandolinjen

Vis hjelpeinformasjon direkte i CLI:

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Supportkanaler

* **E-post**: info@mapir.camera
* **Nettsted**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Priser**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## Fullstendige eksempler

### Eksempel 1: Grunnleggende behandling

Behandling med standardinnstillinger (vignett, reflektans):

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### Eksempel 2: Vitenskapelig utdata av høy kvalitet

32-biters flytende TIFF:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "TIFF (32-bit, Percent)" \
  --vignette \
  --reflectance
```

***

### Eksempel 3: Rask forhåndsvisning

8-bit PNG uten kalibrering for rask gjennomgang:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "PNG (8-bit)" \
  --no-vignette \
  --no-reflectance
```

***

### Eksempel 4: PPK-korrigert behandling

Bruk PPK-korreksjoner med reflektans:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --ppk \
  --reflectance
```

***

### Eksempel 5: Tilpasset utdataplassering

Behandle til en annen plassering med spesifikt format:

**Windows:**

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

**Linux:**

```bash
chloros-cli process ~/input/raw_images \
  -o ~/output/processed \
  --format "TIFF (16-bit)"
```

***

### Eksempel 6: Autentiseringsflyt

Fullført autentiseringsflyt (samme på alle plattformer):

```bash
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
# Windows: chloros-cli process "C:\Datasets\Field_A"
# Linux:   chloros-cli process ~/datasets/field_a
chloros-cli process ~/datasets/field_a

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Eksempel 7: Bruk av flere språk

Endre grensesnittspråk (samme på alle plattformer):

```bash
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
# Windows: chloros-cli process "C:\Vuelos\Campo_A"
# Linux:   chloros-cli process ~/vuelos/campo_a
chloros-cli process ~/vuelos/campo_a

# Change back to English
chloros-cli language en
```
