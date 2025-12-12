# CLI : Kommandolinje

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** gir kraftig kommandolinjetilgang til Chloros bildebehandlingsmotor, noe som muliggjør automatisering, skripting og headless-drift for bildebehandlingsarbeidsflytene dine.

### Viktige funksjoner

* 🚀 **Automatisering** – Skriptbasert batchbehandling av flere datasett
* 🔗 **Integrasjon** – Integrer i eksisterende arbeidsflyter og rørledninger
* 💻 **Headless drift** – Kjør uten GUI
* 🌍 **Fler språk** – Støtte for 38 språk
* ⚡ **Parallell behandling** – Skaleres dynamisk til CPU-en din (opptil 16 parallelle arbeidere)

### Krav

| Krav          | Detaljer                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Operativsystem** | Windows 10/11 (64-bit)                                              |
| **Lisens**          | Chloros+ ([betalt abonnement kreves](https://cloud.mapir.camera/pricing)) |
| **Minne**           | Minimum 8 GB RAM (16 GB anbefales)                                  |
| **Internett**         | Kreves for lisensaktivering                                     |
| **Diskplass**       | Varierer etter prosjektstørrelse                                              |

{% hint style=&quot;warning&quot; %}
**Lisenskrav**: CLI krever et betalt Chloros+-abonnement. Standard (gratis) abonnementer har ikke tilgang til CLI. Besøk [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) for å oppgradere.
{% endhint %}

## Hurtigstart

### Installasjon

CLI er automatisk inkludert i Chloros-installasjonsprogrammet:

1. Last ned og kjør **Chloros Installer.exe**
2. Fullfør installasjonsveiviseren
3. CLI installert til: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style=&quot;success&quot; %}
Installasjonsprogrammet legger automatisk til `chloros-cli` i systemets PATH. Start terminalen på nytt etter installasjonen.
{% endhint %}

### Første gangs oppsett

Før du bruker CLI, må du aktivere Chloros+-lisensen din:

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### Grunnleggende bruk

Behandle en mappe med standardinnstillinger:

```powershell
chloros-cli process "C:\Images\Dataset001"
```

***

## Kommandoreferanse

### Generell syntaks

```
chloros-cli [global-options] <command> [command-options]
```

***

## Kommandoer

### `process` – Behandle bilder

Behandle bilder i en mappe med kalibrering.

**Syntaks:**

```bash
chloros-cli process <input-folder> [options]
```

**Eksempel:**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### Behandlingskommandoalternativer

| Alternativ                | Type    | Standard        | Beskrivelse                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Sti    | _Påkrevd_     | Mappe som inneholder RAW/JPG multispektrale bilder                                         |
| `-o, --output`        | Sti    | Samme som inngang  | Utgangsmappe for behandlede bilder                                                     |
| `-n, --project-name`  | Streng  | Autogenerert | Tilpasset prosjektnavn                                                                    |
| `--vignette`          | Flagg    | Aktivert        | Aktiver vignettkorreksjon                                                             |
| `--no-vignette`       | Flagg    | -              | Deaktiver vignettkorreksjon                                                            |
| `--reflectance`       | Flagg    | Aktivert        | Aktiver refleksjonskalibrering                                                         |
| `--no-reflectance`    | Flagg    | -              | Deaktiver refleksjonskalibrering                                                        |
| `--ppk`               | Flagg    | Deaktivert       | Bruk PPK-korreksjoner fra .daq lyssensordata                                      |
| `--format`            | Valg  | TIFF (16-bit)  | Utdataformat: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Heltall | Auto           | Minimum målstørrelse i piksler for kalibreringspaneldeteksjon                          |
| `--target-clustering` | Heltall | Auto           | Terskel for målklynging (0-100)                                                    |
| `--exposure-pin-1`    | Streng  | Ingen           | Lås eksponering for kameramodell (Pin 1)                                                 |
| `--exposure-pin-2`    | Streng  | Ingen           | Lås eksponering for kameramodell (Pin 2)                                                 |
| `--recal-interval`    | Heltall | Auto           | Rekalibreringsintervall i sekunder                                                      |
| `--timezone-offset`   | Heltall | 0              | Tidssoneforskjell i timer                                                               |

***

### `login` – Autentiser konto

Logg inn med Chloros+-påloggingsinformasjonen din for å aktivere CLI-behandling.

**Syntaks:**

```bash
chloros-cli login <email> <password>
```

**Eksempel:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Spesialtegn**: Bruk enkelt anførselstegn rundt passord som inneholder tegn som `$`, `!` eller mellomrom.
{% endhint %}

**Utdata:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Slett påloggingsinformasjon

Slett lagret påloggingsinformasjon og logg ut av kontoen din.

**Syntaks:**

```bash
chloros-cli logout
```

**Eksempel:**

```powershell
chloros-cli logout
```

**Utdata:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

***

### `status` - Sjekk lisensstatus

Vis gjeldende lisens- og autentiseringsstatus.

**Syntaks:**

```bash
chloros-cli status
```

**Eksempel:**

```powershell
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

```powershell
chloros-cli export-status
```

**Bruksområde:** Kall denne kommandoen mens behandlingen pågår for å sjekke eksportfremdriften.

***

### `language` – Administrer grensesnittspråk

Vis eller endre grensesnittspråket CLI.

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

```powershell
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

| Kode    | Språk              | Opprinnelig navn      |
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
| `nl`    | Nederlandsk                 | Nederlands       |
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
| `cs`    | Tsjekkisk                 | Čeština          |
| `hu`    | Ungarsk             | Magyar           |
| `ro`    | Rumensk              | Română           |
| `uk`    | Ukrainsk             | Українська       |
| `pt-BR` | Brasiliansk portugisisk  | Português Brasileiro |
| `zh-HK` | Kantonesisk             | 粵語             |
| `ms`    | Malayisk                 | Bahasa Melayu    |
| `sk`    | Slovakisk                | Slovenčina       |
| `bg`    | Bulgarsk             | Български        |
| `hr`    | Kroatisk              | Hrvatski         |
| `lt`    | Litauisk            | Lietuvių         |
| `lv`    | Lettisk               | Latviešu         |
| `et`    | Estisk              | Eesti            |
| `sl`    | Slovensk             | Slovenščina      |

{% hint style=&quot;success&quot; %}
**Automatisk lagring**: Språkinnstillingen din lagres i `~/.chloros/cli_language.json` og beholdes gjennom alle økter.
{% endhint %}

***

### `set-project-folder` - Angi standard prosjektmappe

Endre standardplasseringen for prosjektmappen (deles med GUI).

**Syntaks:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Eksempel:**

```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```

***

### `get-project-folder` – Vis prosjektmappe

Vis gjeldende standardplassering for prosjektmappen.

**Syntaks:**

```bash
chloros-cli get-project-folder
```

**Eksempel:**

```powershell
chloros-cli get-project-folder
```

**Utdata:**

```
ℹ Current project folder: C:\Projects\2025
```

***

### `reset-project-folder` – Tilbakestill til standard

Tilbakestill prosjektmappen til standardplasseringen.

**Syntaks:**

```bash
chloros-cli reset-project-folder
```

***

## Globale alternativer

Disse alternativene gjelder for alle kommandoer:

| Alternativ          | Type    | Standard       | Beskrivelse                                      |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | Sti    | Automatisk oppdaget | Sti til kjørbar backend                       |
| `--port`        | Heltall | 5000          | Backend API portnummer                          |
| `--restart`     | Flagg    | -             | Tving omstart av backend (avslutter eksisterende prosesser) |
| `--version`     | Flagg    | -             | Vis versjonsinformasjon og avslutt                |
| `--help`        | Flagg    | -             | Vis hjelpeinformasjon og avslutt                   |

**Eksempel med globale alternativer:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## Veiledning for behandlingsinnstillinger

### Parallell behandling

Chloros+ CLI **skalerer automatisk** parallell behandling for å tilpasse seg datamaskinens kapasitet:

**Slik fungerer det:**

* Oppdager CPU-kjernene og RAM-minnet
* Tildeler arbeidere: **2× CPU-kjerner** (bruker hyperthreading)
* **Maksimum: 16 parallelle arbeidere** (for stabilitet)

**Systemnivåer:**

| Systemtype   | CPU        | RAM      | Arbeidere  | Ytelse     |
| ------------- | ---------- | -------- | -------- | --------------- |
| **High-End**  | 16+ kjerner  | 32+ GB   | Opptil 16 | Maksimal hastighet   |
| **Mid-Range** | 8-15 kjerner | 16-31 GB | 8-16     | Utmerket hastighet |
| **Low-End**   | 4-7 kjerner  | 8-15 GB  | 4-8      | God hastighet      |

{% hint style=&quot;success&quot; %}
**Automatisk optimalisering**: CLI oppdager automatisk systemspesifikasjonene dine og konfigurerer optimal parallellbehandling. Ingen manuell konfigurasjon nødvendig!
{% endhint %}

### Debayer-metoder

CLI bruker **Høy kvalitet (raskere)** som standard og anbefalt debayer-algoritme:

| Metode                      | Kvalitet | Hastighet | Beskrivelse                                 |
| --------------------------- | ------- | ----- | ------------------------------------------- |
| **Høy kvalitet (raskere)** ⭐ | ⭐⭐⭐⭐    | ⚡⚡⚡   | Kantbevisst algoritme (standard, anbefalt) |

### Vignettkorreksjon

**Hva den gjør:** Korrigerer lysfall ved bildets kanter (mørkere hjørner som er vanlige i kamerabilder).

* **Aktivert som standard** – De fleste brukere bør holde denne funksjonen aktivert
* Bruk `--no-vignette` for å deaktivere

{% hint style=&quot;success&quot; %}
**Anbefaling**: Aktiver alltid vignettkorreksjon for å sikre jevn lysstyrke i hele bildet.
{% endhint %}

### Refleksjonskalibrering

Konverterer rå sensorverdier til standardiserte refleksjonsprosentandeler ved hjelp av kalibreringspaneler.

* **Aktivert som standard** – Viktig for vegetasjonsanalyse.
* Krever kalibreringsmålpaneler i bildene.
* Bruk `--no-reflectance` for å deaktivere.

{% hint style=&quot;info&quot; %}
**Krav**: Sørg for at kalibreringspanelene er riktig eksponert og synlige i bildene dine for nøyaktig refleksjonskonvertering.
{% endhint %}

### PPK-korreksjoner

**Hva det gjør:** Bruker etterbehandlede kinematiske korreksjoner ved hjelp av DAQ-A-SD-loggdata for forbedret GPS-nøyaktighet.

* **Deaktivert som standard**
* Bruk `--ppk` for å aktivere
* Krever .daq-filer i prosjektmappen fra MAPIR DAQ-A-SD lyssensor.

### Utdataformater

<table><thead><tr><th width="197">Format</th><th width="130.20001220703125">Bittdybde</th><th width="116.5999755859375">Filstørrelse</th><th>Best egnet for</th></tr></thead><tbody><tr><td><strong>TIFF (16-bit)</strong> ⭐</td><td>16-biters heltall</td><td>Stor</td><td>GIS-analyse, fotogrammetri (anbefalt)</td></tr><tr><td><strong>TIFF (32-bit, prosent)</strong></td><td>32-biters flytende</td><td>Svært stor</td><td>Vitenskapelig analyse, forskning</td></tr><tr><td><strong>PNG (8-bit)</strong></td><td>8-biters heltall</td><td>Middels</td><td>Visuell inspeksjon, deling på nettet</td></tr><tr><td><strong>JPG (8-bit)</strong></td><td>8-biters heltall</td><td>Liten</td><td>Rask forhåndsvisning, komprimert utdata</td></tr></tbody></table>***

## Automatisering og skripting

### PowerShell-batchbehandling

Behandle flere datasettmapper automatisk:

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

### Windows-batchskript

Enkel sløyfe for batchbehandling:

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

### Python automatiseringsskript

Avansert automatisering med feilhåndtering:

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
3. **Behandling**: Parallellmodus skaleres til CPU-kjernene dine (Chloros+)
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

| Modus              | Tid      | Maskinvare                                     |
| ----------------- | --------- | -------------------------------------------- |
| **Parallell modus** | 5–10 min  | i7/Ryzen 7, 16 GB RAM, SSD (opptil 16 arbeidere) |
| **Parallell modus** | 10–15 min | i5/Ryzen 5, 8 GB RAM, HDD (opptil 8 arbeidere)   |

{% hint style=&quot;info&quot; %}
**Ytelsestips**: Behandlingstiden varierer avhengig av antall bilder, oppløsning og datamaskinens spesifikasjoner.
{% endhint %}

***

## Feilsøking

### CLI ikke funnet

**Feil:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Løsninger:**

1. Kontroller installasjonsstedet:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Bruk full sti hvis ikke i PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Legg til PATH manuelt:
   * Åpne Systemegenskaper → Miljøvariabler
   * Rediger PATH-variabelen
   * Legg til: `C:\Program Files\Chloros\resources\cli`
   * Start terminalen på nytt

***

### Backend kunne ikke startes

**Feil:**

```
Backend failed to start within 30 seconds
```

**Løsninger:**

1. Kontroller om backend allerede kjører (lukk den først)
2. Kontroller at Windows brannmur ikke blokkerer
3. Prøv en annen port:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. Tving omstart av backend:

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### Lisens-/autentiseringsproblemer

**Feil:**

```
Chloros+ license required for CLI access
```

**Løsninger:**

1. Kontroller at du har et aktivt Chloros+-abonnement.
2. Logg inn med påloggingsinformasjonen din:

```powershell
chloros-cli login user@example.com 'password'
```

3. Kontroller lisensstatus:

```powershell
chloros-cli status
```

4. Kontakt kundestøtte: info@mapir.camera

***

### Ingen bilder funnet

**Feil:**

```
No images found in the specified folder
```

**Løsninger:**

1. Kontroller at mappen inneholder støttede formater (.RAW, .TIF, .JPG)
2. Kontroller at mappestien er riktig (bruk anførselstegn for stier med mellomrom)
3. Kontroller at du har lesetillatelse for mappen
4. Kontroller at filtypen er riktig

***

### Behandlingen stopper eller henger seg opp

**Løsninger:**

1. Kontroller ledig diskplass (sørg for at det er nok til utdata)
2. Lukk andre programmer for å frigjøre minne
3. Reduser antall bilder (behandle i grupper)

***

### Porten er allerede i bruk

**Feil:**

```
Port 5000 is already in use
```

**Løsning:**

Angi en annen port:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

***

## Vanlige spørsmål

### Spørsmål: Trenger jeg en lisens for CLI?

**Svar:** Ja! CLI krever en betalt **Chloros+ lisens**.

* ❌ Standard (gratis) plan: CLI deaktivert
* ✅ Chloros+ (betalte) planer: CLI fullt aktivert

Abonner på: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### Spørsmål: Kan jeg bruke CLI på en server uten GUI?

**Svar:** Ja! CLI kjører helt uten skjerm. Krav:

* Windows Server 2016 eller nyere
* Visual C++ Redistributable installert
* Tilstrekkelig RAM (minst 8 GB, 16 GB anbefales)
* Engangsaktivering av GUI-lisens på en hvilken som helst maskin

***

### Spørsmål: Hvor lagres behandlede bilder?

**Svar:** Som standard lagres behandlede bilder i **samme mappe som inndata** i undermapper for kameramodeller (f.eks. `Survey3N_RGN/`).

Bruk alternativet `-o` for å angi en annen utdatamappe:

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```

***

### Spørsmål: Kan jeg behandle flere mapper samtidig?

**Svar:** Ikke direkte med én kommando, men du kan bruke skript for å behandle mapper sekvensielt. Se avsnittet [Automatisering og skripting](CLI.md#automation--scripting).

***

### Spørsmål: Hvordan lagrer jeg CLI-utdata i en loggfil?

**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

***

### Spørsmål: Hva skjer hvis jeg trykker Ctrl+C under behandlingen?

**Svar:** CLI vil:

1. Stoppe behandlingen på en ordentlig måte
2. Slå av backenden
3. Avslutte med kode 130

Delvis behandlede bilder kan forbli i utdatamappen.

***

### Spørsmål: Kan jeg automatisere CLI-behandlingen?

**Svar:** Absolutt! CLI er designet for automatisering. Se [Automatisering og skripting](CLI.md#automation--scripting) for eksempler på PowerShell, Batch og Python.

***

### Spørsmål: Hvordan sjekker jeg CLI-versjonen?

**Svar:**

```powershell
chloros-cli --version
```

**Utdata:**

```
Chloros CLI 1.0.2
```

***

## Få hjelp

### Kommandolinjehjelp

Vis hjelpeinformasjon direkte i CLI:

```powershell
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
* **Priser**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

## Komplette eksempler

### Eksempel 1: Grunnleggende behandling

Behandling med standardinnstillinger (vignett, refleksjonsgrad):

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### Eksempel 2: Vitenskapelig utdata av høy kvalitet

32-biters flytende TIFF:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### Eksempel 3: Rask forhåndsvisning

8-biters PNG uten kalibrering for rask gjennomgang:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### Eksempel 4: PPK-korrigert behandling

Bruk PPK-korrigeringer med reflektans:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### Eksempel 5: Tilpasset utdataposisjon

Behandle til en annen stasjon med spesifikt format:

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### Eksempel 6: Autentiseringsarbeidsflyt

Fullfør autentiseringsflyten:

```powershell
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
chloros-cli process "C:\Datasets\Field_A"

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Eksempel 7: Flerspråklig bruk

Endre grensesnittspråk:

```powershell
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
chloros-cli process "C:\Vuelos\Campo_A"

# Change back to English
chloros-cli language en
```
