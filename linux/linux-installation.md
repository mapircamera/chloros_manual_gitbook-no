# Linux-installasjon

Chloros distribueres for Linux som `.deb`-pakker som installerer CLI og backend-serveren. Python SDK er en egen pip-pakke (også inkludert i `.deb` som et versjonsmatchet wheel).

Pakkenes filnavn inneholder versjon og arkitektur: `chloros_1.2.0_amd64.deb` for x86_64 og `chloros_1.2.0_arm64_jp6.deb` for JetPack 6 Jetson-bygg. Erstatt med filen du faktisk lastet ned i kommandoene nedenfor.

***

## Linux amd64 (x86_64)

### Systemkrav

| Krav | Minimum | Anbefalt |
| --- | --- | --- |
| **Distribusjon** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Prosessor** | x86_64 (Intel/AMD) | Intel Core i7 eller bedre |
| **Minne (RAM)** | 8 GB | 16 GB eller mer |
| **Grafikkort** | Ingen (behandles av CPU) | NVIDIA GPU med 4 GB+ VRAM (12 GB+ låser opp `GPU_PARALLEL`, 7 GB+ holder Texture Aware utenfor banen for enkeltbilder) |
| **Lagringsplass** | 2 GB ledig plass | SSD med 10 GB eller mer ledig plass |
| **Python** | Python 3.7+ (for SDK) | Python 3.10+ |

> **Ubuntu 20.04 og Debian 11 støttes ikke.** Avhengighetslisten til `.deb` er
> avledet fra det Chloros-backenden faktisk linker mot, og det inkluderer
> `libc6 (>= 2.34)`. Både Focal og Bullseye leveres med glibc 2.31, så `apt` avviser
> installasjonen direkte i stedet for å la den mislykkes senere under kjøring.

### Installasjon

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
`dpkg -i` løser ikke avhengighetene. Hvis det rapporteres om manglende pakker, fullfører `sudo apt-get install -f` (eller `sudo apt --fix-broken install`) installasjonen — dette er den normale prosessen, ikke en feil.
{% endhint %}

Kontroller installasjonen:



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```***

## Linux arm64 (NVIDIA Jetson)

### Systemkrav

| Krav | Minimum | Anbefalt |
| --- | --- | --- |
| **Plattform** | NVIDIA Jetson med JetPack 6 | Jetson Orin NX 16 GB eller AGX Orin |
| **JetPack** | JetPack 6.x | Siste versjon av JetPack 6 |
| **Minne (RAM)** | 8 GB (delt mellom GPU og CPU) | 16 GB+ delt (12 GB+ er terskelen for parallelle GPU-arbeidsprosesser) |
| **Lagringsplass** | 2 GB ledig plass | NVMe SSD med 10 GB+ ledig |
| **Python** | Python 3.7+ (for SDK) | Python 3,10+ |

### Installasjon

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Samme oppsett som amd64 `.deb`, med en CUDA-versjon tilpasset Jetson Orin / Orin NX / Orin Nano. For informasjon om Jetsons minne, termiske egenskaper og oppførsel ved feltutplassering, se [NVIDIA Jetson-veiledningen](nvidia-jetson-guide.md).

***

## Python SDK-installasjon (alle Linux)

SDK er en ren-Python HTTP-klient for backenden, så den samme pakken fungerer på både amd64 og arm64. To kilder:**Fra PyPI** — den publiserte stabile utgaven:

```bash
pip install chloros-sdk
```

**Fra den medfølgende wheel-filen** — garantert kompatibel med CLI/backend-en du nettopp har installert (bruk denne når din `.deb` er nyere enn PyPI):

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**PEP 668-distribusjoner** (Ubuntu 23.10+, Debian 12+) tillater ikke systemomfattende pip-installasjoner. Bruk `pip install --user …`, et virtuelt miljø eller `sudo pip install --break-system-packages …`. Pakkeinstallatøren installerer aldri SDK automatisk i systemets Python — det valget er opp til deg.
{% endhint %}

Valgfrie tillegg:

| Tillegg | Kommando | Legger til |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` for strømming av fremdrift i sanntid |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` for BLE (DAQ-M)-overføring |

Kontroller SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
`.deb` installerer Chloros, CLI og backend. Python og SDK kommuniserer med dette backend-systemet via et lokalt HTTP og API (`http://127.0.0.1:5000`) og starter den automatisk når det er nødvendig. Bruk alltid den bokstavelige IPv4-adressen i stedet for `localhost` — `localhost` kan løses til `::1` og ta omtrent to sekunder per forespørsel.
{% endhint %}

***

## Første gangs oppsett

### 1. Logg inn

Tilgang til CLI og SDK krever et betalt Chloros+-abonnement (**Copper** eller høyere), som håndheves på serversiden: en utlogget bruker får `401 AUTH_REQUIRED`, og en bruker på gratisabonnementet (Iron) får `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Påloggingsopplysningene lagres i `~/.chloros/user_session.json`.

{% hint style="warning" %}
**Du må logge inn på nytt etter hver installasjon eller oppgradering.** Pakkens `prerm`-skript sletter bevisst `~/.chloros/user_session.json` og den lagrede lisensen for alle brukere på maskinen, slik at en ny versjon alltid validerer lisensen på nytt i stedet for å stole på en utdatert cache.
{% endhint %}

### 2. Sjekk lisensstatusen din

```bash
chloros-cli status
```

`chloros-cli status` fungerer på alle nivåer (inkludert gratisversjonen), slik at du alltid kan se hvorfor tilgang er eller ikke er tilgjengelig.

### 3. Kjør systemdiagnostikk

```bash
chloros-cli selftest
```

Syv kontroller kjøres i rekkefølge, og kommandoen avsluttes med en verdi ulik null hvis noen av dem mislykkes:

| # | Kontroll | Hva den bekrefter |
| --- | --- | --- |
| 1 | **Versjon** | CLI rapporterer sin versjon (`v1.2.0`). |
| 2 | **Port tilgjengelig** | Port 5000 er ledig, *eller* blir allerede besvart av en velfungerende Chloros-backend (det regnes som bestått). |
| 3 | **Oppstart av backend** | Backend-programmet starter. |
| 4 | **API-test (`/api/test`)** | Backenden svarer `status: ok`. |
| 5 | **Systeminformasjon** | Skriver ut `GPU: <name>, CUDA: <bool>, PyTorch: <version>` fra `/api/system-info`. |
| 6 | **Støyreduksjonsmodeller** | Finner `*.pth.enc`-modeller (på Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + støyreduksjon**| «Texture Aware» er faktisk brukbar — krever CUDA**og** minst én modellfil. |

Kjøringen avsluttes med `N/7 checks passed`, og viser eventuelle feil etter navn.

### 4. Behandle ditt første datasett

```bash
chloros-cli process ~/datasets/flight001
```

***

## Filer og kataloger

### Per bruker

Chloros lagrer påloggingsopplysningene og konfigurasjonen til CLI i én enkelt plattformuavhengig katalog, **`~/.chloros/`** (på Windows, `%USERPROFILE%\.chloros\`). To Linux-spesifikke hurtigbuffere følger derimot XDG-konvensjonene — disse respekterer `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` når de er angitt.

| Bane | Formål |
| --- | --- |
| `~/.chloros/user_session.json` | Påloggingssesjonsbuffer skrevet av `chloros-cli login` (tømmes ved hver pakkeinstallasjon/oppgradering) |
| `~/.chloros/working_directory.txt` | Overstyring av standard prosjektmappe (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | CLI språkinnstilling (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Språkinnstilling som deles med Windows-brukergrensesnittet — en `language` her har forrang fremfor `cli_language.json` |
| `~/.chloros/update_cache.json` | En times cache for oppdateringssjekken ved oppstart av Linux/Jetson |
| `~/.chloros/backend.log` | Backend-logg når backenden ble startet av CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Buffrede LATTICE-kalibreringspakker per kamera, sortert etter serienummer og bunt-hash |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Valgfrie brukeroverridelser for DAQ-kapasitetskorreksjonsprofiler |
| `~/.config/chloros/system_config.json` | Buffret maskinvareprofil fra Dynamic Compute Adaptation – slett den for å tvinge frem en ny maskinvaredeteksjon |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Backend-serverlogger, én fil per oppstart |
| `~/Chloros Projects/` | Standard prosjektmappe når ingen overstyring er angitt |

### Systemomfattende

| Sti | Formål |
| --- | --- |
| `/usr/bin/chloros-cli` | Wrapper-skript — angir `LD_LIBRARY_PATH` for de medfølgende native bibliotekene, og kjører deretter den egentlige binærfilen |
| `/usr/bin/chloros-backend` | Wrapper-skript — det samme, pluss `CHLOROS_PRODUCTION=1` slik at backendens autentiseringsport aldri kan deaktivere seg selv uten varsel |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | De kompilerte binærfilene |
| `/usr/lib/chloros/arena_runtime/` | Arena SDK-kjøretid som kreves av LATTICE-kameraer |
| `/usr/lib/chloros/models/*.pth.enc` | Krypterte støyreduksjonsmodeller brukt av Texture Aware-debayeren |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK-hjul som samsvarer nøyaktig med denne versjonen |
| `/usr/lib/chloros/exiftool` | Medfølgende exiftool (symlinket til `/usr/local/bin/exiftool` kun hvis det ikke finnes noe system-exiftool) |
| `/etc/chloros/update.conf` | Oppdater-kanalkonfigurasjon som leses av `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Konfigurerer `net.ipv4.ip_unprivileged_port_start = 319` slik at backenden kan binde PTP-portene uten root-rettigheter |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Peker den dynamiske lasteren mot `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Gir den påloggede brukeren tilgang til DAQ-U USB-seriellbroen (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend.service` | Aktiverer alltid-på-backend-tjeneste (installert, **ikke aktivert**) |
| `/usr/share/applications/chloros-cli.desktop` | «Chloros CLI»-oppføring i applikasjonsmenyen som åpner en terminal |

## Plassering av kjørbar backend-fil

CLI og SDK oppdager backenden automatisk:

| Komponent | Sti |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` |

Overstyr backend-banen med flagget `--backend-exe` CLI eller konstruktorparameteren `backend_exe` SDK, og porten med `--port` (standard `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` peker mot **`lattice`**,**`project`**og**`daq pool-*`**-kommandofamiliene på en ekstern backend. Kjerne-kommandoene (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) ignorerer den bevisst og retter seg alltid mot `http://127.0.0.1:<port>`.
{% endhint %}

***

## LATTICE-kameraer og DAQ-lyssensorer på Linux

Alle kommandofamiliene for live-hardware fungerer på Linux (amd64 og Jetson):

* **`chloros-cli lattice`** — oppdage, koble til, konfigurere og ta opp fra LATTICE-kameraer og synkroniserte matriser. `.deb` inneholder Arena SDK-kjøretiden de trenger, og registrerer den hos den dynamiske lasteren.
* **`chloros-cli daq pool-*`** — koble til DAQ-U/M/E-lyssensorer via backend-poolen, strøm kalibrerte spektra og ta opp `.daq`-filer. Den kompilerte CLI leveres kun med `pool-*`-familien: `pool-connect`, `pool-disconnect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`.
* **`chloros-cli project`** — kjør et lagret prosjekt (med tilhørende kameraer, sensorer og behandlingsinnstillinger) uten skjerm.
* **`chloros-cli time-sync`** — inspiser PTP-grandmasteren som Chloros-backenden kjører for LATTICE-kameraer og DAQ-E-sensorer.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` er en forutsetning for `pool-latest`, `pool-stream`, `pool-record` og `pool-set-cap`; `pool-list` viser ID-ene som for øyeblikket finnes i puljen.

{% hint style="info" %}
**Bruk helst `--eth-host` for den første DAQ-E-tilkoblingen på en maskin med flere nettverksgrensesnitt.** Automatisk oppdagelse søker gjennom mDNS og kan overse sensorens grensesnitt på grunn av en tom ARP-cache, så den første `pool-connect --eth` etter oppstart kan mislykkes selv om sensoren er helt i orden. Ved å oppgi sensorens IP-adresse eller vertsnavn hoppes oppdagelsen helt over.
{% endhint %}

**DAQ-U-serielle tillatelser** håndteres av den installerte udev-regelen (`uaccess` + gruppen `dialout`). Hvis en sensor som allerede var koblet til forblir utilgjengelig, må du laste inn reglene på nytt eller koble den til på nytt:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Se [CLI-referansen](../CLI.md) for det fullstendige kommandosettet.

### Alltid aktiv PTP for verter uten skjerm

Ved første installasjon genereres systemd-enheten `chloros-backend.service`, men den er **ikke aktivert**. På en Jetson eller server uten skjerm som skal holde PTP-tidssynkroniseringen i gang kontinuerlig for DAQ-E-sensorer og LATTICE-kameraer, må du aktivere den:

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

Uten denne kjører PTP bare mens Chloros-backenden er i gang — det vil si under en aktiv CLI/SDK-økt.

Enheten knytter backend-en til `127.0.0.1:5000` (miljøinnstillingene `CHLOROS_HOST` / `CHLOROS_PORT` inne i enheten; overstyr med `sudo systemctl edit chloros-backend.service`) og starter den på nytt ved feil etter 5 sekunder.

**Hvordan PTP får tak i portene sine.** PTP bruker UDP 319/320, begge under den normale grensen på 1024 for privilegerte porter. Pakken `postinst` skriver `/etc/sysctl.d/60-chloros-ptp.conf` med `net.ipv4.ip_unprivileged_port_start = 319`, noe som lar backend-programmet binde dem mens det kjører som din bruker. Den bruker også `setcap cap_net_bind_service,cap_net_raw=+ep` på backend-binærfilen som en ekstra sikkerhetsforanstaltning – det er derfor `libcap2-bin` er en deklarert avhengighet for pakken.***

## Eksempler på Bash-skript

{% hint style="info" %}
**Skriptvennlige avslutningskoder.**`chloros-cli process` avslutter med `0` ved suksess og**en verdi ulik null ved feil — inkludert en kjøring som forespurte bildeprodukter, men ikke skrev noen** (det skriver ut `Processing finished but wrote no image products.` og oppgir prosjektmappen og de vanlige årsakene). Vellykkede kjøringer rapporterer hvor mange bildeprodukter som ble skrevet ut (`Image products written: N`). Avslutningskoder: `0` suksess, `1` feil, `2` argumentfeil, `130` avbrutt.
{% endhint %}

### Behandle flere datasett

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Behandle med tilpassede innstillinger

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

De gyldige `--format`-verdiene er nøyaktig fire, og de inneholder mellomrom — sett dem alltid i anførselstegn:

| `--format`-verdi | Utdatamappe |
| --- | --- |
| `TIFF (16-bit)` *(standard)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` godtar `standard` (standard) eller `texture-aware` (Chloros+).

### Automatisert behandling med Cron

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

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Tilgang nektet

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### «setcap mislyktes» under installasjon

`.deb` bruker `cap_net_bind_service` på `/usr/lib/chloros/chloros-backend` slik at den kan binde PTP-portene 319/320 uten root-rettigheter. Hvis `libcap2-bin` manglet under installasjonen, hoppes påkallingen over. Installer den og installer pakken på nytt:

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### PTP starter ikke / kan ikke binde port 319

Bekreft at grensen for porter uten privilegier er senket, og bruk den på nytt for den aktuelle oppstarten hvis den ikke var det:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Sjekk deretter grandmasteren:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### «LATTICE-kameradrivere ikke funnet»

Arena SDK-kjøretiden blir ikke løst. Bekreft at loader-konfigurasjonen som pakken skriver, er til stede og oppdatert:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
ls /usr/lib/chloros/arena_runtime | head
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

Backend-loggene for den mislykkede oppstarten finnes i `~/.cache/chloros/logs/`.

### CUDA ble ikke oppdaget

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

`chloros-cli selftest` rapporterer det samme i én linje: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Manglende delte biblioteker

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### Treg oppstart på SD-kort-systemer

De kompilerte binærfilene pakker seg ut til en midlertidig katalog ved hver oppstart. Hvis `/mnt/ssd/tmp` finnes, bruker Chloros den automatisk; ellers må du angi `TMPDIR` til et raskt filsystem:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Oppdatering av Chloros på Linux

Kommandoen `update` er kun tilgjengelig på Linux/Jetson. Den sjekker versjonen som er publisert i oppdateringskanalen konfigurert på `/etc/chloros/update.conf` og tilbyr å laste ned og installere den tilsvarende `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

På Linux/Jetson utfører CLI også en ikke-blokkerende oppdateringssjekk ved hver oppstart (resultatet lagres i cachen i én time i `~/.chloros/update_cache.json`) og viser `Update available: vX.Y.Z` når en nyere versjon finnes. Innstillingene og prosjektene dine beholdes etter en oppdatering; du må logge inn på nytt etterpå.

## Avinstallering

```bash
sudo apt remove chloros
```

Avinstallering stopper `chloros-backend.service`, gjenoppretter standardgrensen for ikke-privilegerte porter (1024), fjerner symlinken til det medfølgende exiftool og Arena-lasterkonfigurasjonen, og sletter lagrede påloggingsopplysninger. Prosjektene dine og `~/.chloros/`-datafilene blir ikke berørt.

***

## Neste trinn

* [NVIDIA Jetson-veiledning](nvidia-jetson-guide.md) — Jetson-spesifikk optimalisering og distribusjon
* [CLI : Kommandolinje](../CLI.md) — veiledningen for CLI
* [API : Python SDK](../api-python-sdk.md) — veiledningen SDK
* [CLI-referanse](../reference/cli-reference.md) og [SDK-referanse](../reference/sdk-reference.md) — uttømmende oversikt over kommandoer/API for versjon 1.2.0
* [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md) — hvordan Chloros tilpasser seg maskinvaren din
