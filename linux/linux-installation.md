# Installasjon på Linux

Chloros distribueres for Linux som `.deb`-pakker som installerer CLI, backend-serveren og Python-SDK-et (for systemet `python3`, pluss et versjonskompatibelt «wheel» for virtualenvs).

Pakkenavnene inneholder versjon og arkitektur: `chloros_1.3.0_amd64.deb` for x86_64, og `chloros_1.3.0_arm64_jp6.deb` for arm64 (NVIDIA Jetson, JetPack 6). Erstatt med filen du faktisk lastet ned i kommandoene nedenfor.

{% hint style="info" %}
**Nytt i 1.3.0: programmene leveres ukomprimert.** Backenden og CLI-en installeres som mapper, `/usr/lib/chloros/backend/` og `/usr/lib/chloros/cli/`, i stedet for enkeltstående selvutpakkende filer. De pakker seg ikke lenger ut i en midlertidig mappe ved hver oppstart, så de starter mye raskere (på x86_64 brukte backend-delen tidligere omtrent 100 sekunder på å pakke seg ut ved hver oppstart) og trenger ikke lenger flere GB ledig plass i `/tmp`. Amd64-versjonen `.deb` er også omtrent halvparten så stor som før.
{% endhint %}

***

## Linux amd64 (x86_64)

### Systemkrav

| Krav | Minimum | Anbefalt |
| --- | --- | --- |
| **Distribusjon** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Prosessor** | x86_64 (Intel/AMD) | Intel Core i7 eller bedre |
| **Minne (RAM)** | 8 GB | 16 GB eller mer |
| **Grafikkort** | Ingen (CPU-behandling) | NVIDIA-GPU med 4 GB+ VRAM (12 GB+ låser opp `GPU_PARALLEL`, 7 GB+ holder Texture Aware utenfor single-image-banen) |
| **Lagringsplass** | 2 GB ledig plass | SSD med 10 GB+ ledig plass |
| **Python** | Python 3.7+ (for SDK) | Python 3.10+ |

> **Ubuntu 20.04 og Debian 11 støttes ikke.** Avhengighetslisten til `.deb` er
> avledet fra det Chloros-backenden faktisk kobler seg mot, og det inkluderer en
> `libc6` som er nyere enn glibc 2.31 som følger med i Focal og Bullseye, så `apt` avviser
> installasjonen direkte i stedet for å la den mislykkes senere under kjøring.

### Installasjon

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**Bruk `apt install ./…`, ikke `dpkg -i`.** `apt` løser pakkens avhengigheter og installerer også det den anbefaler: kravene til Python SDK, `python3-numpy` og `python3-requests`. `dpkg -i` installerer ingen av dem. CLI-en og backenden er selvstendige og kjører uten dem, men `import chloros_sdk` trenger dem; installasjonsutdataene oppgir eventuelle manglende komponenter. Behold `./`, slik at `apt` installerer filen i stedet for å lete etter en pakke med det navnet. Pakken deklarerer også `libgl1`, som den medfølgende OpenCV-pakken laster inn, og som kan mangle i et minimalt serverbilde.
{% endhint %}

Kontroller installasjonen:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### Systemkrav

| Krav | Minimum | Anbefalt |
| --- | --- | --- |
| **Plattform** | NVIDIA Jetson med JetPack 6 | Jetson Orin NX 16 GB eller AGX Orin |
| **JetPack** | JetPack 6.x | Siste versjon av JetPack 6 |
| **Minne (RAM)** | 8 GB (delt mellom GPU og CPU) | 16 GB+ delt (12 GB+ er terskelen for parallellbehandling) |
| **Lagringsplass** | 2 GB ledig plass | NVMe SSD med 10 GB+ ledig |
| **Python** | Python 3.7+ (for SDK) | Python 3.10+ |

### Installasjon

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Samme oppsett som amd64 `.deb` (inkludert Python SDK, mottaksbuffere og backend-tjenester), for Jetson Orin / Orin NX / Orin Nano på **JetPack 6** (L4T R36). PyTorch-versjonen er JetPack 6s CUDA-versjon for Orin, så GPU-behandling, inkludert Texture Aware-debayer, kjører på Jetsons GPU. Pakken inneholder ikke CUDA i seg selv: den bruker CUDA- og cuDNN-bibliotekene som JetPack 6 installerer (`sudo apt install nvidia-jetpack` på et bilde som mangler dem). Der det ikke er noen CUDA-GPU tilgjengelig, kjører behandlingen på CPU-en, og `--debayer texture-aware` bruker Standard-debayeren (backenden logger én `[DEBAYER] … needs a CUDA GPU …`-linje som forklarer hvorfor). For informasjon om Jetsons minne, termiske egenskaper og oppførsel ved feltutplassering, se [NVIDIA Jetson-veiledningen](nvidia-jetson-guide.md).***

## Installasjon av Python-SDK (alle Linux-systemer)**`.deb` installerer SDK-pakken for deg.** Begge pakkene (amd64 og Jetson) installerer den for systemet `python3`, så `import chloros_sdk` fungerer umiddelbart etter installasjonen, uten pip og uten nettverk. Pakker (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` og prosjektlasterne) legges inn i `/usr/lib/python3/dist-packages`, som eies av pakken: en oppgradering erstatter dem, og når du fjerner pakken, fjernes også disse. Versjonen samsvarer alltid med den CLI-en og backenden du har installert.

SDK-ets egne krav, `python3-numpy` og `python3-requests`, er **anbefalt, ikke påkrevd** av pakken: `sudo apt install ./chloros_<version>_<arch>.deb` installerer dem, mens `dpkg -i` ikke gjør det. Installasjonsloggen viser hvilken SDK-versjon `python3` importerer, eventuelle manglende krav, samt eventuelle `chloros_sdk` andre steder i `python3`s søkevei som overskygger pakkens kopi (for eksempel en som ble installert tidligere med `pip install --user`).**I en virtualenv**, som ikke har tilgang til systemets pakker, installerer du det medfølgende wheel-filen i den (helt offline):

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**Fra PyPI** — kun for en maskin uten installert Chloros-pakke, der SDK-et driver en backend på en annen maskin (se [Remote-Backend Mode](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

Valgfrie tillegg, for en pip-installasjon (et virtualenv eller PyPI):

| Tillegg | Kommando | Legger til |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` for strømming av fremdrift i sanntid |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` for BLE (DAQ-M)-transport |

Bekreft SDK-et:

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Python-SDK-et kommuniserer med backenden som `.deb` installerer via et lokalt HTTP-API (`http://127.0.0.1:5000`). `ChlorosLocal` og tilkoblingshjelperne starter den automatisk når det er nødvendig; `connect_all()` i et lagret prosjekt gjør det aldri, så på en maskin uten skjerm må du kjøre backenden som en [tjeneste](#always-on-ptp-for-headless-hosts). Bruk alltid den bokstavelige IPv4-adressen i stedet for `localhost` — `localhost` kan løses til `::1` og ta omtrent to sekunder per forespørsel.
{% endhint %}

***

## Første gangs oppsett

### 1. Logg inn

Tilgang til CLI og SDK krever et betalt Chloros+-abonnement (**Copper** eller høyere), noe som håndheves på serversiden: en bruker som er logget ut får `401 AUTH_REQUIRED`, og en bruker med gratisabonnement (Iron) får `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Påloggingsopplysningene lagres i `~/.chloros/user_session.json`.

{% hint style="info" %}
**Oppgraderinger beholder påloggingsopplysningene dine; fjerning sletter dem.** Hvis du installerer en nyere versjon av `.deb` over en eksisterende installasjon (inkludert via `chloros-cli update`), beholdes `~/.chloros/user_session.json` og den lagrede lisensen, slik at headless CLI-/SDK-jobber fortsetter å fungere. Hvis du fjerner pakken (`sudo apt remove chloros`), slettes den lagrede påloggingen og lisensbufferen for alle brukere på maskinen, så en ny installasjon krever `chloros-cli login` på nytt.
{% endhint %}

### 2. Sjekk lisensstatusen din

```bash
chloros-cli status
```

`chloros-cli status` fungerer på alle nivåer (inkludert gratis), så du kan alltid se hvorfor tilgang er eller ikke er tilgjengelig.

### 3. Kjør systemdiagnostikk

```bash
chloros-cli selftest
```

Syv kontroller kjøres i rekkefølge, og kommandoen avsluttes med en verdi ulik null hvis noen av dem mislykkes:

| # | Kontroll | Hva den bekrefter |
| --- | --- | --- |
| 1 | **Versjon** | CLI-en rapporterer sin versjon (`v1.3.0`). |
| 2 | **Port tilgjengelig** | Port 5000 er ledig, *eller* har allerede blitt besvart av en velfungerende Chloros-backend (det regnes som bestått). |
| 3 | **Oppstart av backend** | Backend-binærfilen starter. |
| 4 | **Test av API (`/api/test`)** | Backenden svarer med `status: ok`. |
| 5 | **Systeminformasjon** | Skriver ut `GPU: <name>, CUDA: <bool>, PyTorch: <version>` fra `/api/system-info`. |
| 6 | **Støyreduksjonsmodeller** | Finner `*.pth.enc`-modeller (på Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + støyreduksjon**| Texture Aware er faktisk brukbar — krever CUDA**og** minst én modellfil. |

Kjøringen avsluttes med `N/7 checks passed`, som viser eventuelle feil etter navn.

### 4. Behandle ditt første datasett

```bash
chloros-cli process ~/datasets/flight001
```

***

## Filer og kataloger

### Per bruker

Chloros lagrer påloggingsopplysningene og CLI-konfigurasjonen i én plattformuavhengig katalog, **`~/.chloros/`** (på Windows, `%USERPROFILE%\.chloros\`). To Linux-spesifikke cacher følger derimot XDG-konvensjonene — disse bruker `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` når de er angitt.

| Sti | Formål |
| --- | --- |
| `~/.chloros/user_session.json` | Cache for påloggingsøkt skrevet av `chloros-cli login` (bevares gjennom pakkeoppgraderinger; tømmes når pakken fjernes) |
| `~/.chloros/working_directory.txt` | Overstyring av standard prosjektmappe (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Språkinnstilling for CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Språkinnstilling som deles med Windows-brukergrensesnittet — en `language` her har prioritet over `cli_language.json` |
| `~/.chloros/update_cache.json` | En times cache for oppdateringssjekken ved oppstart av Linux/Jetson |
| `~/.chloros/backend.log` | Backend-logg når backenden ble startet av CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Buffrede LATTICE-kalibreringspakker per kamera, sortert etter serienummer og pakke-hash |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Valgfrie brukeroverskrivninger for DAQ-kapasitetskorreksjonsprofiler |
| `~/.config/chloros/system_config.json` | Buffret maskinvareprofil fra Dynamic Compute Adaptation — slett den for å tvinge frem en ny maskinvaredeteksjon |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Backend-serverlogger, én fil per oppstart |
| `~/Documents/MAPIR/Chloros Projects/` | Standard prosjektmappe når ingen overstyring er angitt |

### Systemomfattende

| Bane | Formål |
| --- | --- |
| `/usr/bin/chloros-cli` | Wrapper-skript — angir `LD_LIBRARY_PATH` for de medfølgende native bibliotekene (og `TMPDIR` til `/mnt/ssd/tmp` når den mappen finnes), og kjører deretter den egentlige binærfilen |
| `/usr/bin/chloros-backend` | Wrapper-skript — det samme for backenden, som det merker som en installert produksjonsversjon |
| `/usr/lib/chloros/cli/`, `/usr/lib/chloros/backend/` | Den kompilerte CLI-en og backenden, installert i utpakket tilstand (nytt i 1.3.0); programmene er `cli/chloros-cli` og `backend/chloros-backend` |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | To-linjers startskript som starter de egentlige programmene ovenfor — oppbevares på disse stiene fordi SDK, CLI og eldre skript leter etter dem her |
| `/usr/lib/chloros/arena_runtime/` | Arena SDK-kjøringsmiljø som kreves av LATTICE-kameraer |
| `/usr/lib/chloros/models/*.pth.enc` | Krypterte støyreduksjonsmodeller som brukes av Texture Aware-debayeren |
| `/usr/lib/python3/dist-packages/chloros_sdk/` (og `lattice_sdk`, `lattice_cam`, `arena_api`, prosjektlasterne) | Python-SDK-et, installert for systemet `python3` og eid av pakken |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python-SDK-wheel som samsvarer nøyaktig med denne versjonen, for en virtualenv |
| `/usr/lib/chloros/exiftool` | Medfølgende ExifTool-starter (symlinket til `/usr/local/bin/exiftool` bare hvis det ikke finnes noe system-exiftool). Fra og med versjon 1.3.0 leveres ExifTool-skriptet med sitt eget Perl-bibliotek i `/usr/lib/chloros/exiftool-dist/`, slik at det kjører uten systempakken `libimage-exiftool-perl` |
| `/etc/chloros/update.conf` | Konfigurasjon for oppdateringskanal som leses av `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Angir `net.ipv4.ip_unprivileged_port_start = 319` slik at backenden kan binde PTP-portene uten root-rettigheter |
| `/etc/sysctl.d/60-chloros-gige.conf` | Øker kjernens mottaksbuffere for sokler (`net.core.rmem_max`, `net.core.rmem_default`) til 32 MB for LATTICE-kamerastrømmer, og bruker dem ved installasjon (en verdi som allerede er satt høyere, forblir uendret). Med kjernens standardinnstilling blir kamerapakker droppet i kjernen, og et array mister bilder og bildefrekvens; `chloros-cli lattice network` rapporterer når buffere er lave. Se [Linux-verter — mottaksbuffere for sokler](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Peker den dynamiske lasteren mot `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Gir den påloggede brukeren tilgang til DAQ-U USB-seriellbroen (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | Opt-in-tjeneste for alltid-på-backend som kjører backenden som den brukeren du angir (installert, **ikke aktivert**) — den du må aktivere, se [Always-On PTP](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | Den samme tjenesten som kjører backend som **root**(installert,**ikke aktivert**), beholdes slik at en maskin som har aktivert den fortsetter å kjøre |
| `/var/log/chloros/` | Den detaljerte loggen til root-tjenesten, lesbar av gruppen `adm` (slettes når pakken fjernes) |
| `/usr/share/applications/chloros-cli.desktop` | «Chloros CLI»-oppføring i applikasjonsmenyen som åpner en terminal |

**Brannmur (nytt i 1.3.0).** Hvis `ufw` eller `firewalld` er aktivt ved installasjon, åpner pakken UDP-porter `5002` (DAQ-E-data), `5353` (mDNS-oppdagelse), `319` og `320` (PTP), og fjerner disse reglene når pakken fjernes. Hvis en brannmur er installert, men ikke aktiv, legger den ikke til noe og viser hvilke porter som skal tillates dersom du aktiverer den senere. Den åpner ingen TCP-porter: en DAQ-E-firmwareoppdatering, der sensoren kobler seg tilbake til denne datamaskinen via TCP for å hente bildet, mislykkes med feilmeldingen `… device never connected back on TCP <port> …` dersom en aktiv brannmur blokkerer denne tilkoblingen.

## Plassering av kjørbar backend-fil

CLI-en og SDK-en oppdager backenden automatisk:

| Komponent | Sti |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` (et startprogram for `/usr/lib/chloros/backend/chloros-backend`) |

Overstyr banen til backend med CLI-flagget `--backend-exe` eller SDK-konstruktorparameteren `backend_exe`, og porten med `--port` (standard `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` peker på kommandofamiliene **`lattice`**,**`project`**og**`daq pool-*`**-kommandofamiliene til en ekstern backend. Kjerne-kommandoene (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) ignorerer den bevisst og retter seg alltid mot `http://127.0.0.1:<port>`.
{% endhint %}

***

## LATTICE-kameraer og DAQ-lyssensorer på Linux

Alle kommandofamiliene i live-hardware-pakken fungerer på Linux (amd64 og Jetson):

* **`chloros-cli lattice`** — oppdager, kobler til, konfigurerer og tar opp fra LATTICE-kameraer og synkroniserte matriser. `.deb` pakker sammen Arena SDK-kjøretiden de trenger og registrerer den hos den dynamiske lasteren.
* **`chloros-cli daq pool-*`** — koble til DAQ-U/M/E-lyssensorer via backend-poolen, strøm kalibrerte spektra, ta opp `.daq`-filer og bytt cap-profiler, og for en DAQ-E: endre nettverksinnstillingene, start den på nytt og oppdater fastvaren. `chloros-cli daq discover` søker etter sensorer du kan koble til, og `chloros-cli daq --help` viser en liste over `pool-*`-underkommandoer.
* **`chloros-cli project`** — kjør et lagret prosjekt (med kameraer, sensorer og behandlingsinnstillinger) uten skjerm.
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

`--sensor-id` er en forutsetning for `pool-latest`, `pool-stream`, `pool-record` og `pool-set-cap`; `pool-list` viser ID-ene som for øyeblikket er i puljen.

{% hint style="info" %}
**`--eth-host` hopper over oppdagelsen.** `pool-connect --eth` finner en DAQ-E via mDNS, med en ARP-reserve som fungerer fra en tom ARP-cache; ved å sende sensorens IP-adresse eller vertsnavn med `--eth-host` kobles man direkte til den.
{% endhint %}

**DAQ-U-serielle tillatelser** håndteres av den installerte udev-regelen (`uaccess` + gruppen `dialout`). Hvis en sensor som allerede var koblet til forblir utilgjengelig, må du laste inn reglene på nytt eller koble den til på nytt:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Se [CLI-referansen](../CLI.md) for det fullstendige kommandosettet.

### Alltid aktiv PTP for verter uten skjerm

Pakken installerer to systemd-tjenester for backenden, men ingen av dem er aktivert. På en Jetson uten skjerm eller en server som skal holde PTP-tidssynkroniseringen i gang kontinuerlig for DAQ-E-sensorer og LATTICE-kameraer, eller der SDK-skript kobler til lagrede prosjekter (`connect_all()` starter aldri en backend), må du aktivere tjenesten **som din bruker**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` kjører backenden som den brukeren: opptakene den skriver tilhører denne brukeren, den detaljerte loggen finnes i brukerens `~/.cache/chloros/logs`, og den bruker brukerens `chloros-cli login` (`~/.chloros`), så logg inn én gang som den brukeren først. Når du installerer pakken, vises denne kommandoen med ditt brukernavn fylt inn.

Uten en tjeneste kjører backenden (og med den PTP-grandmasteren) bare så lenge noe har startet den – en `chloros-cli`-kommando eller et SDK-skript.

Enheten knytter backenden til miljøinnstillingene `127.0.0.1:5000` (miljøinnstillingene `CHLOROS_HOST` / `CHLOROS_PORT` inne i enheten; overstyr med `sudo systemctl edit chloros-backend@.service`) og starter den på nytt ved feil etter 5 sekunder.

**Root-tjenesten.**`chloros-backend.service` kjører backend-en som**root**. Den er fortsatt installert slik at en maskin som har aktivert den, fortsetter å kjøre, men alt som en root-backend skriver tilhører root, så en bruker kan ikke slette sine egne opptak; loggen ligger i `/var/log/chloros` og kan leses av gruppen `adm`. For å bytte til din bruker:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**Backenden til en tjeneste er delt og stoppes aldri av en klient.** `chloros-cli` og SDK-skript bruker den slik den er: `process` kjører på den og laster inn sitt eget prosjekt i stedet for det forrige. Der CLI-en ellers ville ha startet backenden på nytt (`--restart`, eller en `process` der importen ikke fant noe), navngir den enheten, skriver ut `sudo systemctl restart <unit>` og avslutter `1`, fordi en backend som stoppes på den måten, vil forbli nede. Start den på nytt selv med `sudo systemctl restart chloros-backend@$USER.service`. Backenden til en tjeneste stopper aldri av seg selv når den er inaktiv, og `chloros-cli backend stop` avviser den og oppgir kommandoen `systemctl stop`.**Hvordan PTP får sine porter.** PTP bruker UDP 319/320, begge under den normale grensen på 1024 for privilegerte porter. Pakken `postinst` skriver `/etc/sysctl.d/60-chloros-ptp.conf` med `net.ipv4.ip_unprivileged_port_start = 319`, noe som lar backenden binde dem mens den kjører som din bruker. Fra og med versjon 1.3.0 setter pakken ingen filrettigheter (`setcap`) på backenden, og `libcap2-bin` er ikke lenger en avhengighet: på den utpakkede backenden ville en filrettighet hindre den i å starte, så ikke legg til en selv.***

## Eksempler på Bash-skripting

{% hint style="info" %}
**Skriptvennlige avslutningskoder.**`chloros-cli process` avslutter `0` ved suksess og**gir en verdi ulik null ved feil — inkludert en kjøring som ba om bildeprodukter, men ikke skrev noen** (det skriver ut `Processing finished but wrote no image products.` og oppgir prosjektmappen og de vanlige årsakene). Vellykkede kjøringer rapporterer hvor mange bildeprodukter som ble skrevet (`Image products written: N`, pluss `(plus M clamp masks)` når masker ble skrevet). Avslutningskoder: `0` suksess, `1` feil, `2` argumentfeil, `130` avbrutt.
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
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

De gyldige `--format`-verdiene er nøyaktig fire, og de inneholder mellomrom — sett dem alltid i anførselstegn:

| `--format`-verdi | Utdatamappe |
| --- | --- |
| `TIFF (16-bit)` *(standard)* | `tiff16` |
| `TIFF (8-bit)` | `tiff8` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` godtar `standard` (standard) eller `texture-aware` (Chloros+).

### Automatisert behandling med Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Eksempel på Python-SDK

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Feilsøking

### CLI-verktøyet finnes ikke etter installasjon

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
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### Backend starter ikke etter at filrettigheter er lagt til

Ikke kjør `setcap` på backenden. Fra og med versjon 1.3.0 angir pakken ingen filfunksjoner, og PTPs porter 319/320 kommer fra innstillingen `sysctl` beskrevet under [Hvordan PTP får sine porter](#always-on-ptp-for-headless-hosts). En filfunksjon setter den utpakkede backenden i en sikker kjøringsmodus der den ikke kan finne sine egne biblioteker, slik at den ikke starter. Hvis du har lagt til en, må du fjerne den:

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
```

### PTP starter ikke / kan ikke binde port 319

Bekreft at grensen for ikke-privilegerte porter er senket, og bruk den på nytt for den aktuelle oppstarten hvis den ikke var det:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Sjekk deretter grandmasteren:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### `lattice`-kommandoer mislykkes med «LATTICE-kameradrivere ikke funnet»

Backenden klarte ikke å laste inn LATTICE-kamerakjøringsmiljøet som den er installert med. Alle `lattice`-kommandoer kjøres på backenden; CLI-en i seg selv inneholder ingen kameradrivere. Installer Chloros på nytt (`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`): `.deb` installerer runtime-modulen ved siden av backend-modulen, i `/usr/lib/chloros/arena_runtime`, og registrerer den hos den dynamiske lasteren. For å sjekke denne registreringen:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
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

Backend-loggene for den mislykkede oppstarten finnes i `~/.cache/chloros/logs/`. Hvis port 5000 tilhører en backend-tjeneste (`systemctl status chloros-backend@$USER.service`), må du starte tjenesten på nytt i stedet for å avslutte prosessen.

### CUDA ikke oppdaget

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

`chloros-cli selftest` rapporterer det samme i én linje: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Manglende delte biblioteker

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### Treg oppstart på systemer med SD-kort

Fra og med versjon 1.3.0 installeres backend og CLI uten å pakkes ut, slik at de ikke lenger pakker seg ut til en midlertidig mappe ved hver oppstart. Midlertidige filer lagres automatisk i `/mnt/ssd/tmp` hvis den mappen finnes; ellers kan du peke dem mot et raskt filsystem:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Oppdatering av Chloros på Linux

Kommandoen `update` er kun tilgjengelig på Linux/Jetson. Den sjekker versjonen som er publisert i oppdateringskanalen konfigurert i `/etc/chloros/update.conf` og tilbyr å laste ned og installere den tilsvarende `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

På Linux/Jetson utfører CLI-en også en ikke-blokkerende oppdateringssjekk ved hver oppstart (resultatet lagres i cachen i én time i `~/.chloros/update_cache.json`) og viser `Update available: vX.Y.Z` når en nyere versjon finnes. Innstillingene, prosjektene og påloggingen din beholdes etter en oppdatering.

Oppgradering av pakken stopper alle backend-tjenester mens den installeres, slik at et skript som kjører mister tilgangen til kameraer og sensorer, og deretter starter hver enkelt av dem som er aktivert på nytt (et skript som er startet manuelt, men aldri aktivert, forblir stoppet, akkurat som etter en omstart).

## Avinstallering

```bash
sudo apt remove chloros
```

Avinstallering stopper og deaktiverer alle backend-tjenester, gjenoppretter standardgrensen for ikke-privilegerte porter (1024), fjerner innstillingsfilen for mottaksbufferen (de aktive verdiene beholdes inntil omstart), symlinken til det medfølgende exiftool, Arena-lasterkonfigurasjonen, brannmurreglene pakken la til og Python-SDK-en den installerte, samt tømmer bufret påloggingsinformasjon. Prosjektene dine og `~/.chloros/`-datafilene blir ikke berørt. `sudo apt purge chloros` sletter også `/var/log/chloros`.

***

## Neste trinn

* [NVIDIA Jetson-veiledning](nvidia-jetson-guide.md) — Jetson-spesifikk optimalisering og distribusjon
* [CLI: Kommandolinje](../CLI.md) — CLI-veiledningen
* [API: Python SDK](../api-python-sdk.md) — SDK-veiledningen
* [Referanse for CLI](../reference/cli-reference.md) og [Referanse for SDK](../reference/sdk-reference.md) — uttømmende oversikter over kommandoer og API for versjon 1.3.0
* [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md) — hvordan Chloros tilpasser seg maskinvaren din
