# Chloros Python SDK Referanse

**Versjon:**

1.2.0**Generert:**

29.07.2026 kl. 19:19 ·**Revidert:**

30.08.2026**Pakke:** `chloros-sdk` (PyPI)**Målgruppe:** Optimalisert for bruk av store språkmodeller (LLM); lesbar for mennesker.**Omfang:** Alle offentlige klasser, funksjoner, og hjelpefunksjon som er eksponert av `import chloros_sdk`, med eksempler som kan kopieres og limes inn, og som dekker bildebehandling, styring av enkeltkamera, synkroniserte matriser, DAQ-sensorer og prosjektautomatisering.

Hvis du bare trenger høydepunktene, kan du hoppe til:
- [Installasjon og hurtigstart](#installation)
- [Smart-Connect for LATTICE-kameraer](#smart-connect-for-lattice-cameras)
- [DAQ-sensorøkter](#daq-sensor-sessions)
- [Prosjektautomatisering](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## Arkitektur på 60 sekunder

SDK er et tynt Python-lag over Chloros-backenden (den samme Flask-serveren som brukes av skrivebords-GUI-en og CLI bruker). For automatisering importerer du `chloros_sdk` og kaller metoder på høyt nivå; bak kulissene blir hvert kall en HTTP-forespørsel til den lokale backenden på port 5000 — `http://127.0.0.1:5000/api/...` (bevisst ikke `localhost`, som først løses til `::1` på Windows og koster ca. 2 sekunder per forespørsel mot en backend som kun støtter IPv4). Backenden eier maskinvarepoolen — kameraer, DAQ-sensorer, justeringsprofiler, rammebuffere — slik at SDK-skript kan eksistere side om side med GUI-en uten å måtte konkurrere om serielle porter eller nettverkskortets båndbredde.

Det er tre grensesnitt du vil bruke:

1. **`ChlorosLocal` + frie funksjoner** (`process_folder`, `process_lattice_capture`) – Bildbehandlingsrørledning. Kjør en hel mappe gjennom kalibrering / debayer / indekseksport fra ett Python-kall.
2. **Smart-connect-håndtak** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Åpne en vedvarende backend-økt for live-maskinvare. Samme «smart-prep»-flyt som i GUI: nettverkssonde, automatisk valg av lag, PTP, AE-seeding, GPIO-triggerkonfigurasjon.
3. **`ChlorosProject` / `open_project`** — Last inn et lagret prosjekt (mappe med `cameras.json` + `sensors.json` + `project.json`), koble til alt på en gang, og utfør opptak med navngitte håndtak.

Overflater 1 og 2 **starter automatisk en lokal backend** hvis det ikke allerede er en som lytter (den samme medfølgende binærfilen som GUI/CLI starter) — så et enkelt skript fungerer fra et nytt skall uten at du må starte en backend først. Overfør `auto_start_backend=False` for å velge bort dette (f.eks. når du peker mot en ekstern backend, som aldri startes). Se [Automatisk start av backend](#backend-auto-start). Surface 3 oppfører seg annerledes: `open_project()` tar ikke imot noen `auto_start_backend`-parametere, og `connect_all()` starter aldri en backend — den søker etter `http://127.0.0.1:5000` én gang, og hvis ingenting svarer, faller den stille tilbake til direkte (backend-fri) `lattice_sdk`-enhetskontroll. Bare `proj.process()` og `stream(..., overlays=True)` oppretter en `ChlorosLocal()` på en «lazy»-måte (som har automatisk oppstart).

Alle tre er autorisasjonsavhengige: kjør `chloros-cli login` én gang på maskinen, eller logg inn via skrivebordsgrensesnittet. SDK-kall uten en gyldig økt utløser feilmeldingen `ChlorosAuthenticationError`.

Krav:
- Python 3.7+ (som angitt i pakken; utviklet/testet på 3.10)
- Chloros Desktop installert lokalt (backend-binærfilen følger med i installasjonsprogrammet)
- Aktiv Chloros+-pålogging. SDK/CLI-nivået er **Copper**eller høyere (Copper / Bronze / Silver / Gold); det gratis**Iron**-nivået har ingen SDK/CLI-tilgang. Dette håndheves**på serversiden**: hver SDK/CLI-merket forespørsel må inneholde både en aktiv økt og et betalt abonnement, ellers returnerer backenden `403` sammen med `error_code: PLAN_UPGRADE_REQUIRED` (vist som `ChlorosLicenseError` av `ChlorosLocal`, og som `ChlorosConnectError` av `connect_*`-hjelperne). En utlogget bruker får i stedet `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) i stedet – de to er forskjellige fordi en ny kjøring av `chloros-cli login` løser den første, men ikke den andre.
- Offlinebruk støttes innenfor planens nådeperiode: nivået hentes fra servervalideringsbufferen (5 minutter) eller den signerte, maskinbundne lisensbufferen (30 dager for månedlige abonnementer, til abonnementets utløp for årlige). Når denne fristen utløper, går abonnementet over til gratisversjonen, og SDK/CLI-tilgangen stoppes inntil maskinen kan opprette kontakt med serveren én gang. `chloros-cli status` (`GET /api/license-status`) forblir tilgjengelig på gratisnivået, slik at årsaken er synlig — det er den eneste SDK/CLI-ruten som er unntatt fra nivåbegrensningen.
- Windows 10/11 64-bit, **Ubuntu 22.04 LTS eller nyere**, eller Jetson (JetPack 6). Ubuntu 20.04 støttes**ikke**: avhengighetene til `.deb` er avledet fra det backenden kobler seg mot, inkludert `libc6 (>= 2.34)`, og Focal leverer glibc 2.31.

---

## Installasjon

Python SDK er et tynt Python-lag over Chloros-backenden. For alt utover noen få DAQ-baserte arbeidsflyter, må du ha **Chloros-skrivebordspakken installert lokalt** (Windows-installasjonsprogrammet eller Linux `.deb`) — deter det som gir backend-binærfilen, Arena SDK-kjøretiden for LATTICE-kameraer og kalibreringspakkene.

Siste nedlastinger: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Trinn 1 — Installer plattformpakken Chloros

#### Windows (.exe)

1. Last ned `Chloros-Setup-x.y.z.exe` fra nedlastingssiden.
2. Kjør installasjonsprogrammet og følg veiviseren. Standard installasjonssti er `C:\Program Files\MAPIR\Chloros\`.
3. Start Chloros minst én gang og logg inn med din Chloros+-konto.

#### Linux amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### Trinn 2 — Installer Python SDK

**Chloros-installasjonsprogrammet leveres med et tilhørende SDK-wheel.** Hvert Windows-installasjonsprogram og Linux .deb-pakke plasserer en `chloros_sdk-X.Y.Z-py3-none-any.whl` på disken som stemmer nøyaktig overens med GUI- / CLI / backend-versjonen. Du trenger ikke å følge med på PyPI for å holde deg oppdatert.

#### Windows

Installasjonsprogrammet kjører automatisk `pip install` mot det medfølgende wheel-filen ved hjelp av systemets Python (`py.exe`-lanseringsprogram foretrekkes, faller tilbake til `python -m pip`). Ingen handling er nødvendig — `import chloros_sdk` fungerer i ditt Python-miljø etter en vellykket installasjon. Hvis det ikke finnes noen Python på maskinen, hopper installasjonsprogrammet stille over dette trinnet, og GUI + CLI fortsetter å fungere.

#### Linux (.deb)

.deb-filen plasserer wheel-filen i `/usr/lib/chloros/sdk/`. `postinst` skriver ut den eksakte kommandoen — PEP 668-distribusjoner avviser globale pip-skrivninger som standard, så vi installerer ikke automatisk:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

For Jetson-distribusjoner med luftspalte foregår dette helt offline – hjulet ligger allerede på disken.

#### Offentlig PyPI

For verter som kun bruker pip (ingen Chloros-desktop-pakke installert; arbeidsflyter med kun ekstern backend eller kun DAQ):

```bash
pip install chloros-sdk
```

PyPI oppdateres ved installasjonsbygg for utgivelsesversjoner, slik at det publiserte wheel-filen samsvarer med den nyeste stabile utgivelsen. Utviklerbygg (f.eks. `1.1.4.dev1`) leveres kun via det medfølgende installasjons-wheelet.

#### Bekreft

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Chloros+ abonnement kreves.** Alle SDK-kall krever en aktiv Chloros+-pålogging. Kjør `chloros-cli login user@example.com 'YourPassword'` én gang per maskin; påloggingsopplysningene lagres i `~/.chloros/`.

### Trenger jeg skrivebordspakken?

Pip-pakken alene er **ikke** tilstrekkelig for de fleste arbeidsflyter. Her er hva hver SDK-overflate trenger:

| SDK-overflate | Trenger Desktop-pakken? | Hvorfor |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Ja** | Starter backend-binærfilen automatisk på `/usr/lib/chloros/chloros-backend` (Linux) eller `C:\Program Files\MAPIR\Chloros\…` (Windows). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Ja**(lokalt)**/ Nei**(eksternt) | Rene HTTP-klienter via backenden. Lokal backend → skrivebordspakke kreves. Ekstern backend → `backend_url=`**gjennom en tunnel** (se Fjern-backend-modus — medfølgende backender binder kun til loopback). |
| `ChlorosProject` / `open_project` | **Ja** | Driver lagrede prosjekter gjennom backenden. |
| Direkte LATTICE-klasser (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Ja** | Krever Arena SDK-kjøretid som følger med i skrivebordspakken. `CAMERA_AVAILABLE` er ellers `False` ved import. |
| Direkte DAQ-klasser (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **Nei** | Ren Python via pyserial/bleak/zeroconf. Et miljø som kun bruker pip kan styre DAQ-er fra ende til ende. |

### Fjern-backend-modus (pip-only-vert, via tunnel)

> **Den medfølgende backenden er ikke tilgjengelig over LAN.** Produksjonsversjoner
> binder kun loopback (begge loopback-familiene) og avviser kategorisk den
> eneste ikke-loopback-modusen (`CHLOROS_CLOUD_MODE`), så
> `backend_url="http://<lan-ip>:5000"` **kan ikke fungere mot en installert
> Chloros** — dette mønsteret har kun fungert mot en source/dev
> backend. For å kjøre en backend på en annen maskin, må du selv videresende dens loopback-
> port og peke SDK mot tunnelen:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

Headless-/CI-/robotikk-verter kan beholde én maskin med full skrivebordsinstallasjon som «Chloros-server» og `pip install chloros-sdk` alle andre steder — men transporten mellom dem er den brukerdefinerte tunnelen ovenfor, ikke en direkte LAN-forbindelse URL.

> **Kjent begrensning — `ChlorosLocal` støtter ikke kun pip.** `ChlorosLocal(backend_url=BACKEND)` løser for øyeblikket en lokal backend-binærfil i sin konstruktør *før* den undersøker URL, og utløser feilmeldingen `ChlorosBackendError` («Chloros-backend ikke funnet…») når ingen skrivebordspakke er installert — selv med en tilgjengelig ekstern backend. Kun smart-connect-grensesnittet ovenfor (`connect_camera` / `connect_array` / `connect_daq_sensor`, pluss `analyze_array_network` og hjelpefunksjonene `list_*` / `discover_*`) fungerer fra en pip-only-vert.

### Arbeidsflyt kun for DAQ (pip-only-vert)

Hvis du kun trenger DAQ-sensorer og ikke bruker LATTICE-kameraer eller bildebehandling, er pip-pakken selvstendig:

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

Ingen backend, ingen .deb, ingen Chloros+-pålogging nødvendig for direkte DAQ-arbeid med maskinvare.

---

## Hurtigstart

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## API-indeks på øverste nivå

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## Bildebehandling — `ChlorosLocal`

Hovedklassen for behandlingsrørledningen. Starter backenden ved første bruk, oppretter og konfigurerer prosjekter, overvåker fremdriften og returnerer sammendrag etter kjøring.

### Konstruktør

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Metoder

| Metode | Beskrivelse |
| --- | --- |
| `create_project(project_name, camera=None)` | Opprett et nytt prosjekt (eventuelt med en kameramal som `"Survey3N_RGN"`). |
| `import_images(folder_path, recursive=False)` | Importer RAW/TIF/JPG/DNG-bilder **og `.daq`-opptak fra lyssensor**. Returnerer `count` (bilder) og `scan_count` (opptak). Gir kun en advarsel hvis mappen ikke inneholder noen av delene. |
| `export_light_sensor(daq=True, csv=True)` | Skriv kalibrerte `.daq` + `.csv` for hver lys-sensoropptak i prosjektet, inn i `<project>/Light Sensor/`. Se [Lyssensoropptak](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Angi behandlingsparametere. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Kjør pipelinen. Returnerer `{"status": "complete", "async": False}`, pluss en `summary`-nøkkel når backenden leverer en — se [Oppsummering og tips etter kjøring](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | Kontroller backend-tilstanden. |
| `logout()` | Tøm bufret påloggingsinformasjon. |
| `shutdown_backend()` | Avslutt backenden (hvis den ble startet av SDK). |
| `discover_cameras()` | Oppdag LATTICE-kameraer **via denne instansens backend** (`/api/camera/discover`). Returnerer en liste med ordbøker (`serial`, `model`, `ip`, …) — samme form som GUI/CLI ser. Tom liste hvis ingen ble funnet eller backend ikke kan nås. |
| `camera_capture(output_dir, format="tiff", **settings)` | Ta et enkelt bilde**gjennom backenden**(startes automatisk av dette håndtaket), slik at det får samme forberedelse som GUI/CLI (12-bit standard, gjenbruk av pool, innebygde kalibreringsmetadata). Bestem målet med `serial=` eller `device_index=`; send `exposure`/`gain`/`pixel_format`/`preset` som `**settings`. Returnerer den gamle metadatadiktet (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Genererer forhåndsvisningsrammer med overleggssammensetning fra et samlet kamera — en enkel MJPEG-klient via backendens `/api/camera/<serial>/stream-annotated`-rute (zebra / rutenett / siktemerk / histogram / peaking / spot tegnet på serversiden). `decode=True` genererer BGR-matriser; `False` genererer rå JPEG-bytes. Kan også nås per prosjekt som `ChlorosProject.stream(overlays=True)`. |

Brukes som kontekstbehandler for garantert opprydding:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Lyssensoropptak — kalibrert `.daq` + `.csv`

En DAQ-U / DAQ-M / DAQ-E kan registreres **uten** kalibreringspakken. Det er
det de offentlige [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
opptakene (`record_daq.py`) gjør som standard: de skriver rå sensortall og stempler
filen, slik at Chloros henter sensorens fabrikkalibrering **via seriell** — først fra lokal cache,
deretter fra MAPIR Cloud — og bruker den ved import.

Chloros skriver resultatet tilbake som to produkter per opptak, under
`<project>/Light Sensor/`:

| Produkt | Hva det er |
| --- | --- |
| `<name>_calibrated.daq` | Det gjenbehandlingsbare arkivet — samme skjema som et live-opptak, men nå med angivelse av pakken som produserte det. Å importere det på nytt **kalibrerer**det**ikke** en gang til. |
| `<name>_calibrated.csv` | Spektral irradians i W/m²/nm på sensorens eget bølgelengderaster, én rad per måling, pluss fotometriske kolonner (total effekt, fotopisk/skotopisk lux, PPFD og dens oppdeling i blått/grønt/rødt, toppbølgelengde). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Kun sensorer uten pakke (DAQ-A).** Rå spektrale sensortellinger — *ikke* irradians. Se nedenfor. |

`process()` utfører denne eksporten som et av trinnene. Det krever **ikke** bilder:
en lyssensor som flys alene er en førsteklasses arbeidsflyt, og et slikt prosjekt har null
bilder per definisjon.

**DAQ-A-opptak eksporteres som rå tellinger.** DAQ-A-familien er eldre enn det serienummerbaserte
bundlesystemet og har ingen bundle å hente — den kalibreres i stedet i felten mot et
refleksjonsmål, og det er grunnen til at den aldri har trengt en. Disse opptakene eksporteres
under en `_raw`-stamme i stedet for `_calibrated`: et annet filnavn i stedet for et flagg
inne i filen, fordi opplysningen må overleve å bli videresendt via e-post som et rent filnavn. `.csv`-overskriften
angir `raw spectral sensor counts (NOT irradiance)` og advarer om at
verdiene er sammenlignbare **innenfor** filen — akkurat det målbasert kalibrering bruker
dem til — og ikke på tvers av sensorer. De effektavhengige fotometriske kolonnene (total effekt,
fotopisk/skotopisk lux, PPFD) returneres som **NULL** i stedet for å bli integrert fra tellinger.

En DAQ-U / DAQ-M / DAQ-E hvis pakke ganske enkelt ikke kunne hentes, blir fortsatt **hoppet over**,
ikke skrevet i råformat: der finnes pakken, og «koble til på nytt og behandle på nytt» er et reelt råd.

Eldre **v1.01 / v1.02** opptak (en DAQ-A-SD skriver disse) har ingen epoke per avlesning,
bare filens skrivetid. Bildet↔nedstrømnings-matcher avviser dem fortsatt — å matche en
ramme mot en skrivetid ville være feil uten at man ser det — men eksportøren leser dem, og
CSV skriver ut `clock=daq_created_on`, slik at produktet angir hvilken klokke det bruker.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

En opptak hvis kalibreringspakke ikke kan hentes (offline, eller en sensor uten
kalibrering i filen) rapporteres under `skipped` **med årsaken**. Den skrives aldri
ut som en «kalibrert» fil som inneholder rå tall — koble til internett og
kjør på nytt, så fullføres eksporten.

### Tilbakemeldinger om fremdrift

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Oppsummering og tips etter kjøring

Når prosessen er fullført, henter `process()` `GET /api/processing-summary` og legger ved innholdet som `result["summary"]`. Hentingen skjer etter beste evne og blokkerer aldri en vellykket retur — hvis sammendraget ikke er tilgjengelig, faller `process()` tilbake til den enkle `{"status": "complete", "async": False}`-formen. Hver oppføring i `summary["hints"]` — hele setninger med foreslått utbedring, f.eks. hvorfor en kjøring ga null utdata — sendes også ut på nytt som en Python `UserWarning`, slik at kjøringer uten utdata er selvdiagnostiserende selv om du aldri sjekker ordboken:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` er den maskinlesbare delen:

| Nøkkel | Hva den teller |
| --- | --- |
| `models` | Kameragrupper i kjøringen. |
| `images_in_groups` | Kildebilder på tvers av disse gruppene. |
| `targets_found` | Detekterte reflektansmål. |
| `images_calibrated` | Bilder som kjøringen kalibrerte. |
| `exported_files` | **Bildeproduktfiler som kjøringen skrev.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | Opptak fra lyssensoren, bevisst telt separat — de stammer fra et annet trinn og finnes også for kjøringer uten bilder i det hele tatt, så å inkludere dem ville få en ren DAQ-kjøring til å se ut som om det hadde eksportert bilder. |

Ved siden av disse: `summary["output_dirs"]` (hver katalog som ble skrevet til),
`summary["light_sensor_export"]`, `summary["stopped"]` (sant når brukeren avbrøt
kjøringen, slik at delvise tellinger ikke tolkes som en fullført kjøring med for lav produksjon), og
`summary["groups"]` (oppdelingen per gruppe).

`exported_files` registreres av rørledningen **mens den skriver**, ikke skannes fra
prosjektets bildeobjekter i etterkant. Parallell- og GPU-strategiene bygger sine egne bildeobjekter
(i arbeider-underprosesser for GPU-banene), så den gamle skanningen rapporterte
`0 file(s) written` for hvert slikt løp og sendte deretter ut «zero-exports»-hintet — på løp
der alt hadde fungert. Hvis du skriver et skript basert på dette tallet, vil en vellykket parallellkjøring nå
rapportere et tall som ikke er null.

Hopp i lyssensoren rapporterer årsaken som leseren faktisk fastslo for hver fil – et
uleselig skjema, en manglende pakke, en skrivefeil – **deduplisert**, slik at tjue filer
som ble hoppet over av én årsak, leses som én årsak i stedet for tjue repetisjoner av den.

> **`process()` utløses ikke når en kjøring ikke produserer noen bilder.** Dette er det eneste stedet hvor SDK og
> CLI bevisst skiller seg fra hverandre: `chloros-cli process` behandler «produkter ble forespurt, ingen ble
> skrevet» som en feil og avsluttes med en verdi ulik null, mens SDK avsluttes normalt og rapporterer
> tilstanden gjennom `summary` / hints. Hvis pipelinen din skal stoppe ved en tom kjøring, må du sjekke den
> selv — undersøk `summary` (eller tell filene i prosjektmappen) i stedet for å stole på
> fraværet av et unntak. De vanligste årsakene er en inndatamappe som ikke ble gjenkjent som en
> opptakskilde, og produkter som ble hoppet over fordi de ikke var relevante for de tilstedeværende kameraene (f.eks. stråling fra kameraer som kun bruker RGB
>).

### Hjelpefunksjoner

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### Støttede verdier

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Radiometrisk utdata (LATTICE multispektral pipeline)

LATTICE multispektral (M3C/M3M) eksportnivået i `process`-pipelineen — `reflectance` (standard), `radiance`, `sensor-response` eller `all` (hver gjeldende modus per bilde) — tilordnes prosjektets behandlingsinnstilling **«Radiometrisk utdata»**. `configure()` har et eget nøkkelord:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Den avanserte nødutgangen — å skrive prosjektets `"Radiometric output"`-nøkkel gjennom `custom_settings` — fungerer fortsatt, men husk at den erstatter hele innstillingsblokken (se advarselen nedenfor):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (standardverdien) deler kameraets strålingsintensitet med **DAQ-nedstrålingen med samsvarende tidsstempel**, som beregnes automatisk fra en registrert `.daq` (DAQ-U/M/E)**eller en DAQ-M-spesifikk `.csv`**som finnes sammen med bildene; eventuelle kalibreringspakker per kamera eller DAQ som mangler lokalt,**hentes automatisk fra AWS** ved første bruk. CLI viser dette som produktbrytere per type på `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.

> `custom_settings` **erstatter** hele blokken med beregnede innstillinger (den omgår `configure()`s andre nøkkelord og validering, slik den er utformet). Når du bruker den, må du inkludere alle `Project Settings`-nøkler du er interessert i, som i eksemplet ovenfor.

---

## Smart-Connect for LATTICE-kameraer

Vedvarende backend-økter for live-maskinvare. Samme endepunkter som GUI-en bruker, slik at oppførselen er identisk på tvers av SDK / CLI / GUI.

### Enkelt kamera — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

#### `connect_camera()`-signatur

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

#### `CameraSession`-metoder

| Metode | Beskrivelse |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Leser GenICam-noder; returnerer `{nodes, errors, enums, device}`. |
| `set_settings(**kwargs)` | Skriver noder etter vennlig navn (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | Tar opp et **enkelt** bilde. Returnerer en liste med ett element bestående av ordbøker med bildemetadata. (Serietakning/opptak av flere bilder ble fjernet — kall `capture()` i en løkke hvis du trenger en serie.) |
| `disconnect()` | Frigjør fra puljen. Ingen effekt hvis vi er koblet til en allerede åpen økt. |

`capture()` eksportkontroller (samme modell som arrayet + GUI):

- `processing` / `levels` — `processing="all"` lagrer alle relevante eksporttyper; `levels=["raw","radiance"]` lagrer bare disse (overstyrer `processing`). Utelat begge for bakendens standardinnstilling.
- `force_daq=True` — lagre den tildelte DAQ/DLS-avlesningen som en `.daq`-sidecar selv ved en ren rådata-opptak, slik at rammen senere kan bearbeides til reflektans/indeks. Ingen handling hvis ingen DAQ er koblet til.

### Synkronisert matrise — `ArraySession` (Smart-Prep)

`connect_array` er **det anbefalte utgangspunktet** for oppsett med flere kameraer. Den kjører hele GUI-prosessen for smart-prep i bakgrunnen:

1. **Nettverksanalyse** (`/api/camera/array/recommend`) — finner den største rammestørrelsen som passer til sim-emit-nivået uten å miste rammer.
2. **Automatisk valg av nivå** — `sim-capture-sim-emit` hvis nettverket tåler det; ellers `sim-capture-ftd-stagger` eller `slip-emit-and-capture`.
3. **Automatisk reduksjon**— reduserer rammestørrelsen / øker binning uten varsel når kabelen ikke kan opprettholde den forespurte oppløsningen.**Dette sikkerhetsnettet dekker ikke samlet overabonnement**: for mange kameraer for kabelen kan ikke løses ved å redusere rammestørrelsen — se [Overabonnement](#over-subscription-the-per-cam-floor).
4. **PTP aktivert** som standard — tidsstempler på tvers av kameraer er sammenlignbare med en nøyaktighet på mikrosekunder.
5. **Automatisk valg av pikselformat per kamera** — RGB-kameraer → `BayerRG8`, multispec → `BayerRG12`.
6. **AE-seeding** — tar øyeblikksbilder av hvert kameras gjeldende AE-tilstand, slik at tilkoblingen ikke tilbakestiller eksponeringen underveis.
7. **GPIO-utløserkonfigurasjon** — `connect_array` aktiverer hvert kamera (`TriggerMode=On`, `TriggerSource=Line2`) slik at masterens puls styrer slavene via M8-kabelen. Dette er et trinn som kun gjelder for matriser: et enkelt kamera åpnes med `LatticeCamera` free-kjører i stedet.

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### `connect_array()`-signatur

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`force_tier`-verdier:
- `"sim-capture-sim-emit"` — ekte samtidig (alle kameraer utløses på samme klokkekant).
- `"sim-capture-ftd-stagger"` — fleksibel tidsforskyvning (kameraene sender på litt forskjøvede tidspunkter, slik at pakkene serialiseres på ledningen).
- `"slip-emit-and-capture"` — sekvensiell innfanging per kam (ingen tidssynkronisering; eneste alternativ når ingen rammestørrelse passer til sim).

`wire_ceiling_mbps` overstyrer **vertens vedvarende ledningsbudsjett** i MB/s — det enkelte
tallet som hele array-tildelingen er avhengig av. La det stå på `None` for å bruke den automatisk oppdagede
verdien. Senk den når arrayet rapporterer GVSP-korrupte rammer: den automatiske verdien er avledet
fra NIC-kortets oppgitte koblingshastighet, som overvurderer USB-adaptere, smale PCIe-baner og
trafikkbelastede delte nettverk — og denne overvurderingen viser seg som korrupte rammer i stedet for som en
synlig treg forbindelse. Verdien lagres i prosjektets array-opptaksblokk, så en
gjenåpning eller en senere `connect_array` gjenoppretter den som enhver annen array-innstilling.
Se [Array Health](#array-health--which-subsystem-is-losing-frames).

#### Overabonnement (minimumsgrensen per kamera)

Sim-emit-pacing tildeler hvert kamera en andel av det kollisjonssikre ledningsbudsjettet, med en minimumsgrense på **8 MB/s per kamera**(`per_cam_floor_bps`). Når `N × floor` overskrider det kollisjonssikre taket,**overtegner arrayet nettverkskapasiteten**— feilmodusen er GVSP-pakketap, ikke lavere bildefrekvens — og det finnes ingen løsning knyttet til bildestørrelse:**binning og ROI reduserer antall byte per ramme, ikke antall byte per sekund**som den samlede sjekken sammenligner. Praktiske tak for full oppløsning på en 1 GbE-vert:**6 kameraer @ 1500 MTU, 9 med jumbo-rammer** (`max_cams_collision_safe` i analysesvaret angir taket for din kabel). Løsninger: færre kameraer, jumbo-rammer fra ende til ende, eller et raskere nettverkskort.

- Svarene fra `analyze_array_network()` og `/api/camera/array/connect`-svarene inneholder `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` og `per_cam_floor_bps`. Når `oversubscribed` er sant, **setter projeksjonen fps-feltene til null** (`achievable_fps_max` / `fps_bright` / `fps_dark`) i stedet for å rapportere en misvisende hastighet som er treg, men som fungerer.
- `POST /api/camera/array/connect` aksepterer en `pin_resolution`-kroppsparameter (**kun HTTP — ikke en SDK kwarg**; `connect_array` eksponerer den ikke). Pinning fjerner sikkerhetsnettet for binning-walk-down, slik at en overtegnede tilkobling med `pin_resolution` satt blir**kategorisk avvist** med en feilmelding som angir alle mulige løsninger. Uten pinning fortsetter tilkoblingen med walk-down, men gir en advarsel om at reduksjon ikke kan fjerne aggregatet.
- Utvei for testarbeid: sett `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` i backend-miljøet for å nedgradere avslaget til en tydelig advarsel — du kobler til uansett og aksepterer pakketapet.

#### Array-tilstand — hvilket delsystem mister rammer

`GET /api/camera/array/<array_id>/capability` bærer en aktiv `health`-blokk på et
tilkoblet array, som revurderes i et rullende **10-sekunders** vindu. Den deler rammetapet
opp i de to årsakene som krever motsatte løsninger, i stedet for én «ufullstendig» rate som
ikke spesifiserer noen av dem:

| Felt | Hva det betyr | Hvilket undersystem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (per seriell) | Rammen **kom frem og var strukturelt feil**— GVSP-pakketap. |**Nettverk**: ledningskapasitet, pacing, NIC RX-ring, MTU |
| `never_arrived_rate_pct` (per serienummer) | Rammen **kom aldri i det hele tatt**— kameraet utløste ikke, eller ingenting forlot det. |**Utløser / synkronisering**: M8-kabel, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Dårligste kamerafrekvens for hver. | — |
| `per_cam_rate_pct` | Samlet andel ufullstendige bilder per kamera (begge årsakene samlet). | — |
| `stable_for_seconds` | Hvor lenge hvert kamera har ligget under 0,01 %. | — |

I tillegg til `health` viser den samme posten tallet som hele tildelingen henger på:

| Felt | Hva det betyr |
| --- | --- |
| `wire_ceiling_mbps` | Vertens gjeldende vedvarende båndbredde, MB/s. |
| `wire_ceiling_source` | Hvor tallet kommer fra, i ord — f.eks. `USB-capped 200 MB/s (was theoretical 1062; …)` eller `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` når `wire_ceiling_mbps=` angir det. |
| `nic_is_usb` | `true` for en USB-Ethernet-adapter. |

Det finnes ingen SDK-wrapper for dette endepunktet — les den direkte:

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Lesing:** `gvsp_corrupt_rate_pct` ulik null med `never_arrived_rate_pct` på 0 betyr
at utløsning og kabelsynkronisering er perfekte, og at 100 % av tapet ligger på nettverksbanen — senk
`wire_ceiling_mbps` og koble til på nytt. Det motsatte mønsteret peker i stedet på synkroniseringskabelen eller
utløserlinjen.

> **`target_fps` er ikke avgjørende for korrupte rammer.** GevSCPD-takten skrives inn én gang ved
> tilkobling, så å senke utløserfrekvensen endrer arbeidscyklussen, men ikke
> burst-frekvensen for samtidig utsending. En målt reduksjon av etterspørselen med 5× ga ingen forbedring, mens
> å senke ledningstaket fra 240 til 200 MB/s reduserte andelen korrupte rammer for samme rigg fra 10,4 % til
> 0,00 %.

> **Automatisk reduksjon midt i strømmen er ikke tilgjengelig på TRI032S-firmware.** Et array i drift kan ikke
> løse dette selv; koble fra og koble til på nytt, slik at valgverktøyet for tilkoblingstid planlegger på nytt ut fra
> det nye taket.

En **USB-Ethernet-adapter er begrenset til 200 MB/s** av sonden, uavhengig av hva som står på
typenavnet: effektivitetstabellen som omregner en koblingshastighet til en vedvarende hastighet, er
avledet fra PCIe, og et USB-nettverkskort oppgir sin Ethernet-koblingshastighet samtidig som det er begrenset av
USB-bussen og driveren. Begrensningen er absolutt, ikke en brøkdel — en USB 1 GbE-adapter
oppnår ~80 MB/s og påvirkes ikke.

#### `ArraySession`-metoder

| Metode | Beskrivelse |
| --- | --- |
| `status(timeout=10.0)` | Live `{fps, ptp, frame_count, last_error, …}`. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | Én synkronisert opptaksgruppe. Returnerer en `CaptureResult` (liste over rammediktsjoner + `.skipped`). Eksportkontroller nedenfor. |
| `capture(..., smart=True)` | **Smart opptak** — venter til AE har stabilisert seg på alle kameraer, og utløser deretter. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | Raskeste opptak: kun rådata + den tildelte DAQ-avlesningen (+ den frie kombinerte indeksen). Speiler «Fastest Capture»-knappen i brukergrensesnittet. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Enkelt / Kontinuerlig / Intervall i én avgrenset sløyfe. Returnerer `list[CaptureResult]`.**Krever `count` og/eller `duration_s`** slik at den avsluttes (SDK har ikke Ctrl+C). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | Start opptak av live-visningen av den kombinerte indeksen til video/GIF → `RecorderHandle`. Én sammensatt opptaker per matrise. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | Start en rå-Bayer-serie med høy bildefrekvens → `RecorderHandle`. Behandle på nytt offline med `build_video()`. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Behandle en lagret rå bildeserie offline til kalibrerte videoer. Blokkerer inntil ferdig (`wait=True`) og returnerer `{outputs, errors, combined}`. |
| `build_video_status(job_id, timeout=15.0)` | Avleser status for en offline-byggjobb: `{running, result, error, burst_dir}`. |
| `disconnect()` | Frigjør hele matrisen. |

`capture()` eksportkontroller (samme endepunkt som GUI/CLI bruker):

- `processing` / `levels` — `processing="all"` (eller `levels=["raw","radiance",…]`) lagrer alle relevante eksporttyper per kamera; en enkelt `processing`-verdi lagrer bare det aktuelle nivået.
- `aligned=True` — forvrenger alle medlemmers ikke-rå eksport til matrisens [justeringsprofil](#array-alignment) (samregistrert); rådata forblir uforvrengt, men bærer transformasjonen i metadata. Faller tilbake til ujustert (med en advarsel som vises i resultatets `alignment`) hvis matrisen ikke har noen profil.
- `render_index=False` — hopp over vegetasjons-indeksoverlegg; standardinnstillingen gjengir det der det er konfigurert.
- `force_daq=True` — lagre den tildelte DAQ/DLS-avlesningen som en `.daq`-sidecar selv når ingen valgt nivå trenger den.

**TIFF-komprimering (kun HTTP-knott):**`ArraySession.capture()` sender ingen `compression`-nøkkel, så standardinnstillingen for backenden gjelder — `POST /api/camera/array/capture` leser en `compression`-body-parameter, `"deflate"` som standard (tapsfri zlib L1 + horisontal prediktor, ~4,1 MB per ramme i full oppløsning). `"none"` skriver ukomprimert (~6,3 MB/ramme) med en**~5 ganger raskere skriving** — begge er tapsfrie og leses identisk ved import. SDK eksponerer ingen kwarg for dette; utveien er `chloros-cli lattice array-capture --compression none` eller rå HTTP. DEFLATE holder også Python GIL, så komprimerte skrivinger kan ikke parallelliseres på tvers av skrivertrådene per kamera — vedvarende opptak med 8 kameraer i full oppløsning ved sensorhastighet krever `compression: "none"`. Detaljer: [CLI Referanse → array-opptak](cli-reference.md).**Overridelser for eksport per medlem (HTTP-kun):**det samme endepunktet aksepterer også `exclude_serials` (liste – fjern medlemmer fra det lagrede settet; matrisen utløses fortsatt som én synkronisert gruppe, og ekskluderte medlemmer returneres i `excluded`), `serial_levels` (`{serial: [level tokens]}` overstyringer på kameranivå) og `serial_index` (`{serial: bool}` overstyringer av indeksoverlegg per kamera). Dette er GUI-paritetsparametere og**ennå ikke SDK kwargs**; medlemmer som mangler i kartene faller tilbake til de array-omfattende `levels` / `render_index`.

##### Inspisere hoppede kammer — `CaptureResult.skipped`

`ArraySession.capture()` returnerer en `CaptureResult`, som er en `list`-underklasse: iterere den, indeksere den, `len()` den — alle eksisterende mønstre fungerer fortsatt. Ny kode kan inspisere `.skipped`-attributtet for å se hvilke kammer som ble ekskludert og hvorfor. Det vanligste tilfellet er RGB-kammer i en blandet filtermatrise når du ber om `processing="radiance"` eller `"reflectance"` — per-Bayer-stråling er meningsløs for en bredbåndssensor, så backenden hopper over disse kameraene i stedet for å produsere meningsløse data.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Årsakstokener følger mønsteret `<level>-not-applicable-to-rgb-cam` (én oppføring per hoppet over nivå, hver med `level`). De refleksjonsspesifikke hoppene er `reflectance-skipped-no-fresh-dls` (ingen ny nedadrettet måling tilgjengelig), `reflectance-skipped-bound-daq-unavailable (…)` (den tilknyttede DAQ-enheten kunne ikke nås) og `dls-uncalibrated-band-<nm>` – båndet ligger for det meste utenfor DAQ-lyssensorens radiometrisk kalibrerte område (~374–974 nm), så den absolutte DAQ-baserte reflektansinndelingen avvises, og bildet nedgraderes tydelig til sensorrespons. Blant de leverbare SKU-ene er det kun F988 som utløser dette; det kameraets støttede arbeidsflyt er reflektanspanel-arbeidsflyten.

`processing`-nivåer:

| Nivå | Utgang |
| --- | --- |
| `"raw"` | Enkanals Bayer (monokameraer: enkeltbåndet) direkte fra sensoren. |
| `"debayered"` *(SDK standard)* | 3-kanals BGR via bilineær demosaik (monokameraer: 1-kanals gråtoner). |
| `"radiance"` | float32 W/m²/sr/nm via hele den radiometriske kjeden. Kun multispektralt — RGB-kameraer hoppes over. |
| `"reflectance"` | uint16 0..32768 (Pix4D-klar); krever en live DAQ-parring for absolutt referanse. Kun multispektralt. |
| `"display"` | Full kjede som samsvarer med forhåndsvisningen i brukergrensesnittet (CCM + WB + gamma i henhold til kameraets profil). |
| `"all"` | **Én fil per aktuelt nivå** for hvert kamera (samsvarer med GUI-innstillingen «Capture All» / CLI-standard). Den returnerte `CaptureResult` inneholder deretter én rammedikt per `(cam, level)`, med nivået i hver dikt; nivåer som ikke er relevante vises i `.skipped`. DAQ-målingen som brukes for ethvert refleksjonsbilde, lagres som en `.daq`-sidecar. |

> **Merk — standardinnstillingen avviker fra CLI.** `ArraySession.capture()` har som standardverdi `processing="debayered"`; kommandoen `chloros-cli lattice array-capture` har som standardverdi `processing="all"`. Overfør `processing="all"` eksplisitt fra SDK for å speile CLI/GUI-lagring på flere nivåer.

### Opptakmoduser og opptakerenheter

Matriseoverflaten speiler GUI-opptakspanelet: Enkeltbilde / Kontinuerlig / Intervall / Raskeste lukker-moduser, pluss to opptakerenheter (live-komposittvideo og rå bildeserie → offline etterbehandling).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**er kontinuerlig/intervall-sløyfen til SDK. Siden det ikke finnes noen `Ctrl+C` for å avbryte den fra et skript,**må** du overføre `count` og/eller `duration_s` (den stopper når en av disse nås). `interval_s` måles fra starten av hvert gjennomløp (i samsvar med GUI-en). Gjenværende kwargs sendes direkte videre til `capture()`.
- **`record`** er *overvåkingsnivå*: den fanger opp den kombinerte indeksen i sanntid slik den vises, så den kombinerte strømmen må være åpen for at bilder skal kunne lagres. Én komposittopptaker per matrise (gir feilmelding hvis en allerede kjører).
- **`burst` → `build_video`** er *analyseklasse*: `burst` skriver rå bilder + et manifest per bilde + én `.daq` per unik DLS-avlesning under `<output>/bursts/<base>/` med grab-sløyfens fulle hastighet (ingen kjede, ingen exiftool, ingen live-visning). `build_video` tidssammenstiller hvert bilde med nærmeste `.daq` og kjører importpipelinens radianse-/reflektans-/indekskjede på nytt. `products` er en liste over `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (standard: den kombinerte indeksen). `burst().stop()` starter også automatisk en «best-effort»-kombinert-indeksoppbygging, som returneres som `build_job` i stopp-resultatet.

#### `RecorderHandle`

Returneres av `ArraySession.record()` og `ArraySession.burst()`. Bruk den som en kontekstbehandler for automatisk stopp ved utgang av omfanget, eller styr den manuelt.

| Medlem | Beskrivelse |
| --- | --- |
| `job_id` | Backend-jobb-ID (str). |
| `kind` | `"composite"` (fra `record`) eller `"raw"` (fra `burst`). |
| `start_stats` | Diktet som returneres av `start`-kallet. |
| `result` | `None` under kjøring; den endelige stopp--resultat-diktet når det er stoppet. |
| `stats(timeout=10.0)` | Live jobbstatistikk (antall skrevne rammer, realisert fps, forløpt tid). |
| `stop(timeout=60.0)` | Stopp opptakeren; returnerer og lagrer det endelige resultatet i hurtigbufferen. Idempotent (et nytt kall returnerer det bufret resultatet). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### Koble til et alleredetilkoblet array — `attach_array`

Hvis arrayet allerede er oppe (GUI-en har åpnet det, eller en tidligere SDK-økt har kalt `connect_array`), bruk `attach_array` for å hente et håndtak til den i stedet for å koble til på nytt. `connect_array` gir alltid feilmeldingen «Kameraet  <sn>er allerede i arrayet<id>» i en slik situasjon, fordi POST-forespørselen `/array/connect` for et medlem i-poolen er ikke idempotent; `attach_array` leser `/api/camera/array/list` og matcher enten etter array_id eller serienummer.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Mønster: Skript som SDK, som deles med skrivebords-GUI-en, bør prøve `attach_array` først og falle tilbake til `connect_array` hvis det ikke er noen matrise i puljen ennå.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Viktig — når kontekstbehandleren avsluttes, blir forbindelsen brutt.**`ArraySession.disconnect()` sender alltid en POST-forespørsel til `/array/disconnect`; det finnes ingen «attached-not-owned»-beskyttelse slik det er for `CameraSession` / `DAQSensorSession`. Hvis du deler ressurser med GUI-en og ikke ønsker å rive ned arrayet ved utgang fra omfanget,**må du ikke bruke `with`-blokken** — oppbevar håndtaket i en vanlig variabel og hopp over den eksplisitte `disconnect()`:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### Hjelpeverktøy for nettverksanalyse

Nyttig før du åpner matrisen — viser om de foreslåtte innstillingene vil passe:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

`status` er en av `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (ellers `error`). `auto_capped_fps` betyr at den forespurte oppløsningen kun passer til RX-ringen ved en begrenset utløserfrekvens — behold oppløsningen og gå videre fra `target_fps=result["recommended"]["recommended_target_fps"]` til `connect_array` (se [Eksempel 6](#6-capability-probe-before-connecting-a-4-cam-array)).

**Hvordan lese projeksjonen** (samme modell som GUI-panelet «Array Settings»):

- **Burst (`frame_bytes_total`) summeres per kamera i hvert kameras faktiske pikselformat.**Mono**M3M**-kameraer streamer Mono12 (2 B/px) uavhengig av hvilken `pixel_format`-verdi du angir, så et bilde i full oppløsning fra et oppsett med 4 kameraer er**~25 MB** med tre mono-kameraer, ikke de ~12,6 MB som en antagelse om at alt er 8-bit gir. Backenden bestemmer hvert kameras format ut fra modellen.
- **Admittance (`burst_fits_nic_ring`) tar hensyn til drenering**, ikke hele-burst-vs-ring: sim-emit passer når verten tømmer RX-ringen raskere enn kameraene fyller den. En 10G-vert + 1 GbE-kameraer**slipper inn** full oppløsning selv når burstet overstiger ringen; en 1 GbE-vert blokkerer (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` er et konservativt tak for seriell henting** — `max(readout+emit, N×emit)` med utsending per kamera begrenset til 1 GbE-kameralink, uavhengig av eksponering. F.eks. ~2,8 fps for et 4-kamera-array med full oppløsning og 12-bit (stemmer overens med kjøretidens målte ~2,7–3,0). Full modell: [CLI Referanse → Array fps &amp; burst-modell](cli-reference.md#array-fps--burst-model).
- **Overtegnelse (`oversubscribed: true`) betyr at N × minimumsgrensen per kamera overstiger det-sikre taket** — fps-feltene (`achievable_fps_max` / `fps_bright` / `fps_dark`) viser 0, og automatisk krymping/binning kan ikke løse problemet (disse reduserer antall byte per ramme, ikke antall byte per sekund). Løsninger er færre kameraer, jumbo-rammer eller et raskere nettverkskort; `max_cams_collision_safe` rapporterer taket (6 kameraer med full oppløsning på 1 GbE @ 1500 MTU, 9 med jumbo). Svaret inneholder også `aggregate_demand_bps`, `collision_safe_ceiling_bps` og `per_cam_floor_bps` (8 MB/s). Se [Over-Subscription](#over-subscription-the-per-cam-floor).

### Oppdagelse og oppføring

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

LATTICE-matriser kjører kontinuerlig AE i bakgrunnen så snart de er tilkoblet, men det tar et øyeblikk før et nylig innrettet motiv konvergerer. **Smart-capture** er den praktiske løsningen: den avleser eksponeringen på hvert kamera, venter til matrisen er stabil over hele vinduet, og utløser deretter opptaket. Dette tilsvarer GUI-funksjonen: «smart»-opptaksknappen i skrivebordsappen kaller det samme backend-endepunktet.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Når du bruker `ChlorosProject` (neste avsnitt), får du flere innstillingsmuligheter:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Smart-AE-innstillingen er konservativ som standard. Stram inn `exposure_tolerance_pct` for krevende radiometrisk arbeid; løsne for raskt skiftende scener der du bare vil ha «nær nok».

---

## DAQ-sensorsesjoner

Permanent backend-pool for spektralsensorer (DAQ-U via USB, DAQ-M via BLE, DAQ-E via Ethernet). Speiler kameraets overflate: smart-detect, gjenbruk av pool, idempotent tilkobling.

### Smart-Detect (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Prioritet: Ethernet → BLE → USB. Send et hvilket som helst eksplisitt hint for å låse transporten.

### Låst transport

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### `DAQSensorSession`-metoder

| Metode | Beskrivelse |
| --- | --- |
| `status(timeout=10.0)` | Oppsummering av pooloppføring (strømming/opptakstilstand, bølgelengdeområde, kalibrerings-SHA, integrasjonstid, frame_avg, AE-tilstand). |
| `latest(n=1, timeout=10.0)` | Returnerer opptil N av de nyeste spektrumrammene. |
| `stream_start()` / `stream_stop()` | Gjenopptar / pause av strømming (håndtaket forblir åpent). |
| `record_start(output_dir=None, device_name=None)` | Start opptak av en .daq-fil. Returnerer filstien. Avvises for DAQ-U/M uten et AWS-kalibreringspakke (DAQ-E er unntatt). |
| `record_stop()` | Stopp opptak. Returnerer `{path, rows}`. |
| `disconnect()` | Frigjør fra puljen. Ingen effekt for tilknyttede håndtak som ikke eies. |

> **Kap-korreksjonsprofiler (`cap_id`) er ikke en SDK-kontroll.** `connect_daq_sensor()` / `DAQSensorSession` eksponerer ingen `cap_id`-parameter eller `set_cap`-metode. Velg en flåtekapasitetskorreksjonsprofil via CLI (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) eller backendens `/api/daq` HTTP-rutene (`/api/daq/connect` og `/api/daq/<id>/cap-id` godtar `cap_id`).

### Oppdagelse — å finne en adresse å koble seg til

`discover_daq_sensors()` skanner USB / BLE / ETH etter sensorer du *kan* åpne. Det er DAQ-motstykket til `discover_lattice_cameras()`, og den eneste måten å finne **DAQ-M-enhetens BLE-MAC** på — en DAQ-E har et vertsnavn og en DAQ-U en COM-port, men en MAC-adresse er verken trykt på enheten eller oppført av operativsystemet.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Felt | Beskrivelse |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM-port / BLE MAC / vertsnavn — sendes til `connect_daq_sensor` som `port=` / `mac=` / `eth_host=`. |
| `display` | Lesbar etikett. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E`, eller `None` for en port som skanningen ikke kan identifisere (serielle USB-adaptere kan ikke skilles fra hverandre uten en sonde, så ukjente porter vises frem i stedet for å skjules). |
| `extra` | Detaljer per transporttype (BLE-annonsert navn, USB-produsent, DAQ-E ip/fw/…). Tomme verdier utelates. |

| Parameter | Standard | Beskrivelse |
| --- | --- | --- |
| `transports` | alle tre | Sekvens (eller CSV-streng) som begrenser skanningen. Verdt å oppgi når du vet hva du vil ha — BLE er den tregeste delen. |
| `scan_timeout` | 5 | Skanningsvindu per transport i sekunder; backenden begrenser til 1–20. |
| `timeout` | 60,0 | HTTP-tak for hele anropet (som andre steder i SDK). |
| `auto_start_backend` | `True` | Opprett en lokal backend hvis ingen kjører. Opprettes aldri for en ekstern `backend_url`. |

> **Sensorer som allerede er åpne i puljen, vises ikke.** En tilkoblet BLE-periferienhet slutter å annonsere, og en åpen COM-port kanikke undersøkes, så oppdagingsfunksjonen viser hva som er *tilgjengelig for tilkobling*. Det er forventet et tomt resultat rett etter at du har koblet til noe — bruk `list_daq_sensors()` for det du allerede har. Transportprotokoller der skanningen ikke kan kjøres (ingen bleak / zeroconf installert) hoppes over i stedet for å gi feilmelding, slik at en maskin uten Bluetooth fortsatt får svarene for USB og ETH.

### Liste

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Sambruk med GUI / CLI

Hvis GUI-en allerede har en sensor åpen, returnerer et kall til `connect_daq_sensor(port="COM3")` fra Python et håndtak merket `already_connected=True`. Sesjonens `disconnect()` er da en no-op, slik at SDK-skriptet ditt ikke river sensoren ut fra GUI-en når scope avsluttes.

### Direkte maskinvareklasser (uten backend)

`daq_sdk` er re-eksportert av `chloros_sdk`, slik at du også kan styre sensorer fra ende til ende i prosessen uten backend:

> **Tilgjengelighet:**`daq_sdk` leveres med Chloros-desktopinstallasjonen,**ikke** med PyPI-pakken — `pip install chloros-sdk` gir deg `lattice_sdk`, men utelater `chloros_sdk.DAQ_AVAILABLE == False`. Sjekk dette flagget før du bruker disse klassene; på en vertsmaskin som kun bruker pip, bør du i stedet kjøre bruker du i stedet sensoren via [`connect_daq_sensor()`](#daq-sensor-sessions), som ikke trenger lokale transportbiblioteker.

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

Velg helst smart-connect-banen (`connect_daq_sensor`) når du ønsker delt eierskap med GUI; bruk de direkte klassene for skript uten grafisk grensesnitt som eier sensoren eksklusivt.

---

## Prosjektautomatisering — `ChlorosProject`

Et lagret Chloros-prosjekt er en mappe som inneholder `cameras.json` + `sensors.json` + `project.json`. `open_project` laster inn manifestet, og `connect_all` kobler alle lagrede enheter til nettet med de lagrede innstillingene — samme maskinvaretilstand som GUI-en ville ha produsert.

### Minimalt eksempel

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
proj.capture_all("./out")

proj.disconnect_all()
```

Eller som en kontekstbehandler:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### `ChlorosProject`-metoder

| Metode | Beskrivelse |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | Oppdager og kobler til alle lagrede enheter. Returnerer en tilkoblingsrapport per klasse. Bruker en kjørende backend når en lytter på `127.0.0.1:5000`; ellers faller den stille tilbake til direkte (backend-fri) `lattice_sdk`-enhetskontroll — den oppretter aldri en backend. |
| `disconnect_all()` | Avslutt alt. |
| `capture_all(output_dir=".")` | Én ramme fra hvert kamera + matrise + spektrum fra hver sensor. |
| `stream(camera, overlays=False, fps=10.0)` | Generator som gir BGR `numpy`-bilder fra et navngitt kamera (eller en matrise). `overlays=False` er en direkte `lattice_sdk`-hentesløyfe (matriser gir `{serial: frame}`-ordbøker). `overlays=True` rutes gjennom `ChlorosLocal.camera_stream()` → backendenss `/api/camera/<serial>/stream-annotated` MJPEG-strøm, der kameraets lagrede `ui.overlay`-blokk sendes videre som spørringsparametere. Krever backend-modus og et **frittstående kamera**: et kamera i direkte modus utløser `RuntimeError` (backenden kan ikke hente et kamera som denne prosessen eier) og en matrise utløser `NotImplementedError` (overlegger sammensatt bilde per kamera — strømmer et element etter navn). Tilsvarende engangsoperasjon: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Kjør justering på alle matriser som for øyeblikket er tilkoblet. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Kjør kalibrerings-/indekspipeline på prosjektets bilder (omfatter `ChlorosLocal.process`; disse fire er de **eneste** aksepterte kwargs — `indices=` osv. utløser `TypeError`; angir indekser via `ChlorosLocal.configure()`). Konstruerer en `ChlorosLocal()` på en lat måte, som automatisk starter en backend. |

Attributter:
- `proj.cameras` — `Dict[str, CameraHandle]` indeksert etter navn OG serienummer.
- `proj.arrays` — `Dict[str, ArrayHandle]` indeksert etter navn OG array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` indeksert etter navn OG slot_id.
- `proj.config` — `project.json["config"]` ordbok.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Behandlingsnivåer.** `capture()`, `grab()`, og `frame_stream()` bruker alle det samme `processing`
tokenet, og kjeden er kumulativ — hvert nivå kjører alt som ligger over det:

| Nivå | Utgang | Merknader |
| --- | --- | --- |
| `raw` | 1-kanals Bayer, sensor-innfødt | Ingen demosaik. Overlegg er ikke tilgjengelige på dette nivået. |
| `debayered` | 3-kanals BGR (**standard**) | Bilineær demosaik. Det eneste nivået som fungerer uten backend-modus. |
| `radiance` | float32, W/m²/sr/nm | Full radiometrisk kjede: demosaik + 3×3 unmix (multispektral) + DSNU + flat-field + NIST-skala, med eksponering × forsterkning trukket fra slik at verdiene er absolutte. |
| `reflectance` | uint16, 32768 = 1,0 | Strålingsintensitet dividert med nedadgående irradians (ρ = π·L/E). Krever en DLS/DAQ-avlesning — se merknaden nedenfor. |
| `display` | 8-bit sRGB-lignende | GUI-ekvivalent gjengivelse: CCM + hvitbalanse + gamma via kameraets aktive fargeprofil. |

Alt annet enn `debayered` krever backend-modus; et kamera i direkte modus utløser
`NotImplementedError`. `reflectance` trenger en brukbar nedadrettet avlesning — rammeendepunktet trekker
den samlede DAQ-en inn i kameraets DLS-spor automatisk, men uten en tilknyttet DAQ avviser kjeden
reflektansutgangen og merker nedgraderingen tydelig i de returnerte metadataene, i stedet for å stille
levere tilbake et dårligere produkt.

> **Refleksjons-DN-skala – ikke hardkod den.** LATTICE-refleksjon bruker `32768` = ρ 1.0 og merker
> XMP `Chloros:PixelScale=32768`; Survey3-reflektans bruker `65535` = ρ 1,0 og har ingen
> `Chloros:*`-tagger. Les av taggen og del med den. Den er definert i uint16-domene, så den forblir
> `32768` for alle formater som skaleres om (16-biters TIFF, 8-biters PNG/JPG, 32-biters prosent) — normaliser
> først den lagrede datatypen tilbake til uint16 (×257 fra 8-bit, ×65535 fra float). Det eneste unntaket:
> en opptak fra en 8-bit-kilde skrevet som 8-bit TIFF blir *klippet*, ikke omskalert, så ingen skala beskriver
> den — Chloros utelater `PixelScale` og MicaSense-tuplen helt i dette tilfellet. Behandle en manglende
> tag i en LATTICE-refleksjonsfil som «ingen gyldig skala», ikke som en standard.

> **EXIF overføres til eksporten.** `process()` kopierer kildeopptakets GPS-blokk
> **og ExifIFD** til hvert produkt, så eksportene inneholder `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` og `CameraSerialNumber` samt
> georeferansen. `FocalLength` er det Pix4D bruker til å beregne bakkeprøveavstanden ut fra – uten det
> faller rekonstruksjonen tilbake til en helt feil skala (i et målt tilfelle ble et område på 411 m
> forvandlet til et på 47,8 km). Kopien er bevisst ikke `-all:all`: IFD0s strukturelle tagger ødelegger
> LATTICE-utdataene, og `ExifImageWidth`/`Height` er utelatt fordi de beskriver kildeopptaket
> snarere enn det eksporterte rasteret.

Underflagg for opptakstrinnet (gjelder de radiometriske nivåene — `radiance`, `reflectance`, `display`):

| Flagg | Standard | Betydning |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + flat-field + 3x3 unmix + NIST-radiometrisk skala. |
| `apply_white_balance` | `True` | WB LUT. DLS-kompatibel når en DAQ er koblet til kameraet. |
| `apply_index` | `False` | Evaluering av vegetasjonsindeks. |
| `index_expression` | `None` | Overstyringsformel. Ikke-tom → aktiverer indeksen automatisk. |
| `annotated` | `False` | Overlegg av GUI-dekorasjoner (zebra/rutenett/peaking). Ikke tilgjengelig for `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Returtypen er `CapturePathMap`, ikke `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` er `Dict[str, Union[str, List[str]]]`: et enkeltnivå
> `processing` gir hver serie én bane, mens en flernivå-variant (`"all"`, eller en
> eksplisitt `levels`-liste) gir den den **ordnede listen** av alle produktene som er lagret for det
> kameraet. En live sammensatt kompositt, hvis en ble strømmet, kommer inn under den ekstra
> `"combined"`-nøkkelen i stedet for under en serienummer. Kode som forutsetter `str` bryter sammen i
> listeformen uten at noen typekontroll protesterer — kommentaren sa `Dict[str, str]`
> en stund etter at listeformen ble lansert, og det er grunnen til at aliaset finnes. Normaliser
> når du ønsker den flate formen:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### Justering av matriser

`ArrayHandle` eksponerer hele justeringsflaten. Profiler er som standard kun sesjonsbaserte — kall `export_alignment()` eksplisitt for å lagre dem.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Justering ved tilkobling

`connect_all(align=...)` kan automatisk justere alle matriser ved tilkobling:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Bruker `project.json["config"]["auto_align_on_connect"]` som standard hvis ikke annet er angitt.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## Direkte maskinvare (uten backend)

Når du ønsker null avhengighet av backenden (CI, headless-roboter, innebygd), importerer du `lattice_sdk` og `daq_sdk` direkte — begge reeksporteres av `chloros_sdk`. Vær oppmerksom på `CAMERA_AVAILABLE` / `DAQ_AVAILABLE`: `lattice_sdk` finnes i PyPI-pakken (men krever at Arena SDK-kjøretiden er til stede), mens `daq_sdk` kun leveres med skrivebordsinstallasjonen.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Forhåndsinnstillinger og utløseren

Tre av de fire forhåndsinnstillingene **free-run**: kameraet eksponerer kontinuerlig, og et
`capture()` returnerer neste bilde. `triggered` er unntaket — det aktiverer
kameraet for en maskinvarekant på linje 2, så det tar ikke opp noe før en slik kant oppstår.

| Forhåndsinnstilling | Utløser | Bruk den når |
| --- | --- | --- |
| `default` | fri drift | generell bruk |
| `high_speed` | fri drift | 8-bit, 60 fps-begrensning, kort eksponering |
| `high_quality` | fri drift | 12-bit, ingen fps-begrensning — det vanlige valget for stillbilder |
| `triggered` | **klar, linje 2** | kameraet er koblet til en M8-synkroniseringskabel, og noe annet utløser det |

Hvis du velger `triggered` (eller selv angir `trigger_mode="On"`) uten at noe
styrer linje 2, vil hver `capture()` vil gå i timeout — helt riktig, siden du ba
kameraet om å vente. SDK forklarer dette når det skjer; se
[SC_ERR_TIMEOUT under opptak](#direct-hardware-backend-free).

> **Merk — «GVSP probe» / `SC_ERR_TIMEOUT -1011`-meldinger ved tilkobling er ikke feil.**&gt; Ved tilkobling prøver SDK å forhandle om**jumbo-rammer** (9000-byte GVSP-pakker) for høyere gjennomstrømning. På en direkte punkt-til-punkt-NIC-forbindelse (f.eks. en link-local `169.254.x.x`-adresse) kan nettverket vanligvis ikke håndtere jumbo-rammer, så denne sonden går ut på tid og logger linjer som:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Dette er den **planlagte reserveplanen**: SDK går automatisk tilbake til standardpakker på 1500 byte, og kameraet fortsetter å koble seg til som vanlig (`[chunk-enable …]`-linjene som følger, er en del av den normale tilkoblingssekvensen). Opptaket fungerer fortsatt.
>
> Du kan hoppe over denne testen, men **det er ikke bare en logg-demper – den slår av jumbo-rammer.** Kameraet svarer kun på «Don&#x27;t-Fragment»-pinger opp til 1500 byte, uansett hvor godt nettverket ditt er, så ping-testen alene kan aldri oppdage jumbo-rammer; denne sonden er det eneste som kan det. Deaktiver den, så kjører kameraet standardpakker på 1500 byte for alltid, uansett nettverk:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Det er kun verdt det på et nettverk du *vet* ikke kan håndtere jumbo, der det sparer omtrent ett sekund i tilkoblingstid per kamera. Siden dette er en reell avveining og ikke bare en kosmetisk endring, angir SDK nå dette når du bruker den:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **La den være i fred med mindre du har en grunn.** Hvis den er aktivert, måler hver tilkobling på nytt hvilket nettverk du faktisk har: koble til en switch som støtter jumbo-pakker, så vil neste tilkobling automatisk ta i bruk jumbo-pakker, uten at du trenger å konfigurere noe eller starte på nytt.
>
> Hvis du *ønsker* jumbo-gjennomstrømning, aktiver jumbo fra ende til ende (NIC MTU 9000 + en switch som slipper dem gjennom), eller lås det med `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` når du vet at lenken støtter det — men foretrekker en kommandobasert innstilling med `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` fremfor å sette det permanent, siden en låst størrelse hopper over sonderingen og slutter å tilpasse seg nettverket foran seg. **Hver** enhet i banen må kunne videreformidle jumbo-pakker — inkludert eventuelle PoE-splittere eller -injektorer, som er den vanligste årsaken til at et ellers jumbo-kompatibelt oppsett ikke klarer å håndtere dem.

> **`SC_ERR_TIMEOUT -1011` under `capture()` / `grab*()` er et annet problem – det er en reell feil.**&gt; Merknaden ovenfor gjelder kun `-1011` som er logget av**connect-time probe**. Den samme feilen som oppstår ved en**capture**, betyr at kameraet ble koblet til uten problemer, men ikke sender noen bilder:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Det som avslører dette, er et kamera der *kontroll*kanalen fungerer som den skal – oppdagelsen fungerer, innstillingene og `[chunk-enable …]`-skrivingene lykkes alle – mens *hver* ramme går ut på tid.
>
> **Den vanligste årsaken er at kameraet er innstilt på en maskinvareutløser.** Med `trigger_mode="On"` og `trigger_source="Line2"` sender kameraet ikke noe som helst før det kommer en elektrisk flanke på M8-synkroniseringskabelen. Hvis du ikke har noen kabel som driver den linjen, vil hver innspilling vente i all evighet. Kameraet er ikke ødelagt og nettverket fungerer som det skal — det gjør akkurat det det ble bedt om.
>
> `CameraSettings()` og forhåndsinnstillingene `default` / `high_speed` / `high_quality` kjører i frittløp, og en opptaksserie som utløses mens det er aktivert, forklarer seg selv i stedet for å skrive ut bare `-1011`. `PRESETS["triggered"]` aktiverer Line2, slik det er ment.
>
> For å tvinge et hvilket som helst kamera til fri-kjøring:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Hvis det fortsatt går ut med `trigger_mode="Off"`, leverer kameraet virkelig ikke data — send oss loggen og `ip link show`.

#### Fargeprofiler (RGB live-forhåndsvisning) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` velger skjermfargeprofilen for **live-forhåndsvisning** på RGB-kameraer (multispek-kameraer ignorerer innstillingen):

| Profil | Betydning |
| --- | --- |
| `raw` | Omgå den radiometriske kjeden fullstendig. |
| `linear` | DSNU + flat + WB, ingen CCM, ingen gamma. |
| `natural` | Lineær + målt CCM + sRGB-gamma, kun med «billig finish» (kromatisk utjevning + desaturering av høylys) — den realistiske standardinnstillingen. |
| `enhanced` | `natural` pluss full «hub-parity»-finish (defringe, vibrance, CLAHE lokal kontrast). Rikere utseende med omtrent **dobbel kostnad per bilde**, og dermed lavere LIVE-bildefrekvens. |
| `custom_temp` | `natural`, men hvitbalanse fastsatt til `custom_cct_k` Kelvin (DLS ignoreres; begrenset til 2000–10 000 K på backend-siden). |

Profilen er en **kun live-forhåndsvisning**-hastighets-/utseende-regulator: lagrede opptak får alltid den fulle, rike finishen uavhengig av valgt profil, så å velge `natural` for å spare inn rammetid reduserer ikke kvaliteten på det som lagres på disken. En ukjent profil øker `ValueError`; når en chloros-backend er tilgjengelig, blir endringen også sendt til den via POST, slik at neste forhåndsvisningsbilde gjenspeiler den (brukere av direkte-SDK uten backend får fortsatt endringen i innstillingene).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Mono (M3M)-kameraer og `Calibration`

Et mono **M3M**-kamera (`M3M-<lens>-F<wavelength>`) er enkeltbånds: ett gråtonenivå, ingen Bayer-mosaikk, ingen 3×3 spektral-crosstalk-matrise. `Calibration` gjenkjenner det og eksponerer et `is_mono`-flagg. Reflektans gjelder fortsatt som et radiometrisk kart per bånd (avblandingen er identitetsmatrisen), men flerbåndsberegninger på et enkelt kamera gir meningsfulle resultater i stedet for meningsløse:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

For å lage et vegetasjonsindeks fra monokromt utstyr, kombinerer du flere M3M-kameraer med ulike bølgelengder til en justert flerbåndsstabel (se [Arrayjustering](#array-alignment)) og beregne indeksen på tvers av denne stakken i stedet for på ett enkelt kamera.

DAQ direkte-modus:

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **`apply_sensor_settings` godkjente nøkler**— nøyaktig `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E; utfaset til fordel for `cap_id`), `filter_model` (DAQ-M), og `cap_id` (alle DAQ-typer; `None`/`""`/`"none"` = ren sensor, uten kapasitanskorreksjon). Ukjente nøkler**ignoreres uten varsel** — f.eks. `{"integration_time": 64}` gjør ingenting (det må være `integration_time_ms`). Returnerer `{"applied": [...], "errors": {...}}` og utløser aldri unntak.

`chloros_sdk`-eksporterer kun kjernen som er brukt ovenfor. Den fullstendige offentlige `daq_sdk` API (22 navn) legger til følgende — importer dem direkte fra `daq_sdk`:

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

---

## Unntak

Fang opp baseklassen for å håndtere «alt som gikk galt med Chloros»:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` og `ChlorosConfigurationError` eksporteres på øverste nivå sammen med resten; de kan også importeres fra `chloros_sdk.exceptions` som vist.

Hierarki:

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Eksempler fra start til slutt

### 1. Behandle en mappe med en tilpasset fremdriftslinje

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Live LATTICE-matrise → Refleksjonsgrad + DAQ-referanse

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Prosjektdrevet innsamlingskampanje

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Bildestrøm fra flere kameraer → NumPy-rørledning

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Skript for opptak uten grafisk grensesnitt direkte fra maskinvare (uten backend)

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Funksjonstest før tilkobling av et opptakssystem med 4 kameraer

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Tilsvarende opptakskonfigurasjon (ren Python)

Oppskrifts-DSL-en til CLI har en direkte Python-ekvivalent:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Automatisk start av backend

Smart-connect-inngangspunktene — `connect_camera`, `connect_array`, `connect_daq_sensor`, og `discover_lattice_cameras` — er enkle HTTP-klienter som forutsetter at en backend lytter på `127.0.0.1:5000` (standardverdien for smart-connect-grensesnittet URL). Når GUI-en eller CLI allerede kjører, er det en som gjør det. Fra et rent skript er det kanskje ikke — så disse funksjonene **starter den medfølgende backend-binæren automatisk** (uten vindu, på samme måte som `ChlorosLocal` gjør) før deres første påkalling, og venter deretter i opptil `backend_startup_timeout` på at den skal starte opp.

Regler:

- **Det startes kun en lokal URL.** En `backend_url` som peker mot `localhost` / `127.0.0.1` / `[::1]` er kvalifisert; alle andre verter antas å være andres maskiner og blir aldri opprettet.
- **Backenden forblir i gang for gjenbruk** (samme som CLI) — det skjer ingen implisitt avslutning når skriptet ditt avsluttes. Når du kjører skriptet på nytt, gjenbrukes den aktive backenden.
- **Velg bort med `auto_start_backend=False`** på et hvilket som helst av disse anropene (f.eks. når du har pekt på en ekstern backend, eller du administrerer backendens livssyklus selv).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Hvis den medfølgende binærfilen ikke kan finnes eller startes, utløser det påfølgende HTTP-kallet en handlingsbar, **plattform****`ChlorosConnectError`** i stedet for en ren feilmelding om avvist tilkobling – ved Windows viser den deg til skrivebordsappen eller en `chloros-cli`-kommando; ved Linux (uten GUI) leder den deg til en `chloros-cli`-kommando eller `.deb`.

---

## Miljø og overskrifter

SDK merker hvert backend-HTTP-kall med `X-Chloros-Client: sdk`. Backenden bruker lisensreglene for SDK/CLI (pålogging **og** et betalt Chloros+-abonnement kreves) i stedet for GUI-løsningen for gratisnivået. Dette settes automatisk ved import – du trenger ikke å gjøre noe.

`http://localhost` og `http://127.0.0.1` oppdages som den lokale backenden. Anrop til andre verter (f.eks. din egen analysetjeneste) forblir uendret.

Overstyr backenden URL ved å sende `backend_url=` (eller `api_url=` på `ChlorosLocal`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(En `backend_url` som ikke er loopback, når kun en source/dev-backend — medfølgende backender binder kun loopback; se «Remote-Backend Mode» for tunnelmønsteret.)

---

## Versjonering og kompatibilitet

- SDK-versjonen eksponeres som `chloros_sdk.__version__`.
- SDK-pinnene er avhengige av den medfølgende backend-versjonen. Å blande en eldre SDK med en nyere backend fungerer vanligvis (fremoverkompatible endepunkter), men å blande en nyere SDK med en eldre backend kan føre til `404`-feil på nye endepunkter — oppgrader skrivebordsappen slik at den samsvarer.
- Smart-Connect-grensesnittet (`connect_camera` / `connect_array` / `connect_daq_sensor`) og endepunktet for nettverksanalyse returnerer stabile JSON-skjemaer; nye felt er additive.

---

## Tips til feilsøking

- **`ChlorosAuthenticationError: Login required`** → Kjør `chloros-cli login EMAIL PASSWORD` én gang på denne maskinen, eller logg inn via Chloros-skrivebordsappen.
- **`ChlorosConnectError: No Chloros backend is running …`** → Smart-connect-anropene starter automatisk en lokal backend, så dette vises bare når den medfølgende binærfilen ikke kan finnes/startes (f.eks. en pip-only-vert uten skrivebordspakke). Meldingen er plattformavhengig: på Windows må du åpne skrivebordsappen eller kjøre en hvilken som helst `chloros-cli`-kommando; på Linux kjør en `chloros-cli`-kommando (det finnes ikke noe grafisk grensesnitt) eller installer `.deb`. For en ekstern backend, send `backend_url=` (og `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** ved import → `lattice_sdk` kunne ikke lastes inn (vanligvis er ikke Arena SDK-kjøretids-DLL-ene installert). Overflaten uten kamera fungerer fortsatt.
- **Array connect returnerer oppløsning lavere enn den opprinnelige**→ Backendens smart-prep-funksjon reduserer automatisk bildestørrelsen for å passe til ledningen. Bruk `analyze_array_network()` for å se hvorfor, og oppgrader deretter lenken, godta reduksjonen eller bruk `force_tier="slip-emit-and-capture"` for sekvensiell opptak. Sikkerhetsnettet for reduksjon**ikke** dekke samlet overabonnement (`oversubscribed: true`, fps-feltene 0): for mange kameraer for kabelen kan ikke løses ved binning/ROI – reduser antall kameraer, aktiver jumbo-rammer eller bytt til et raskere nettverkskort (se [Over-Subscription](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` rapporterer at NIC-mottakerringen er svært liten (~0,26 MB) / koble til porter med «FRAMES WILL DROP»** → Verts-NIC-ensmottaksring er på standardverdien (blir ofte tilbakestilt til 32 etter en oppdatering av nettverkskortdriveren). På et Realtek USB 10GbE-kort må du sette `ReceiveBufferLen=256` og `PendingReceives=64` (forhøyet), og start deretter backenden på nytt slik at den leser ringen på nytt. Fullstendig fremgangsmåte: [CLI Referanse → Konfigurering og justering av verts-NIC](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Verten henger seg opp ved omstart/avslutning, senere WMI-feil `Invalid class` / nettverkskortet kan ikke aktiveres** → Utdatert USB 10GbE-driver forårsaker `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Oppdater adapterdriveren til en aktuell versjon (≥ 2026) og bruk innstillingene for mottaksringen på nytt. Se [CLI-referanse → Konfigurering og justering av verts-NIC](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Refleksjonsmåling avvist** → En aktiv DAQ må være tilknyttet kameraet (eller matrisen) for refleksjonsmåling i absolutt skala. Tilknytt enten via brukergrensesnittet eller bruk `processing="radiance"` (W/m²/sr/nm), som ikke krever en tilkoblet sensor.
- **`smart=True`-opptaket tar lengre tid enn forventet** → AE-konvergens avhenger av scenens dynamikk; stram inn `exposure_tolerance_pct` eller forkort `stability_window_s` hvis du ønsker en raskere (mindre stabil) utløser.

---

## Se også

- [CLI-referanse](cli-reference.md) — hver CLI-underkommando tilsvarer et SDK-kall.
- [Veiledning for DAQ-sensorer](../daq/README.md) — sensorspesifikke regler for kabling, kalibrering og registrering.
- Online dokumentasjon: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>
