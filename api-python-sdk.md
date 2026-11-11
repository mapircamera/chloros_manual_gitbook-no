# API: Python SDK

{% hint style="info" %}
**Leter du etter den komplette API-en?** Denne siden er en praktisk veiledning. Alle offentlige klasser, metoder, eksakte signaturer og eksempler som kan kopieres og limes inn, finnes i [SDK-referansen](reference/sdk-reference.md), som er optimalisert for AI-assistenter.**Arbeider du med en AI-assistent?** Lim inn denne URL-en i chatten, slik at den får tilgang til den fullstendige, oppdaterte Chloros-versjonen 1.3.0 av API:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Hver side i denne håndboken er tilgjengelig som rå markdown under sluggen med små bokstaver + `.md`, og hele håndboken er indeksert på `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` på PyPI) styrer alt det skrivebordsappene kan gjøre fra Python: batch-bildebehandling, live LATTICE-kamera- og matrisestyring, DAQ-lyssensorsesjoner og automatisering av lagrede prosjekter. Det er et tynt lag over den samme lokale backenden som GUI og CLI bruker (HTTP på `127.0.0.1:5000`), så oppførselen er identisk på tvers av alle tre grensesnittene.

## Installasjon

SDK-et følger med Chloros-skrivebordspakken, som også inneholder behandlingsbackenden og maskinvareruntimene, og versjonen samsvarer alltid med denne.

**Trinn 1 — Installer Chloros.** Windows: kjør installasjonsprogrammet for skrivebordet (standardbane `C:\Program Files\Chloros\`) fra [Nedlastingssiden](download.md). Linux: installer pakken `.deb` med `sudo apt install ./chloros_1.3.0_<arch>.deb` ([Linux-installasjon](linux/linux-installation.md)).**Trinn 2 — Sjekk SDK** (Python 3.7+). Vanligvis er det ingenting å installere:

* **Windows:** installasjonsprogrammet installerer det medfølgende SDK-hjulet i systemets Python (hvis det ikke er installert noen Python, hopper det over dette trinnet).
* **Linux:** `.deb` installerer SDK-en for systemets `python3` i `/usr/lib/python3/dist-packages`. Kravene, `python3-numpy` og `python3-requests`, anbefales av pakken: `apt install` inkluderer dem, mens `dpkg -i` ikke gjør det.
* **Et virtualenv** ser ikke systemets pakker; installer den medfølgende wheel-filen i det: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` på Linux, eller `chloros_sdk-*.whl` i mappen `sdk` i Windows-installasjonsmappen (`C:\Program Files\Chloros\sdk\` som standard).

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # Windows: py -c "…"
```

På en maskin uten Chloros-pakken, der SDK-en driver en backend på en annen maskin (se [Hva en installasjon kun med pip gir](#what-a-pip-only-install-gets)), må du installere den fra PyPI: `pip install chloros-sdk`. PyPI oppdateres ved hver utgivelse, slik at den samsvarer med den nyeste stabile utgivelsen.

**Trinn 3 — Logg inn én gang per maskin:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Påloggingsopplysningene lagres i `~/.chloros/` (begge plattformene). På Windows kan du på samme måte logge inn via fanen «Bruker» <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> i skrivebordsappen. SDK-et krever et betalt Chloros+-abonnement — se [Lisenskrav](#license-requirement) nedenfor.

| Krav | Detaljer |
| --- | --- |
| **Chloros installert** | Windows: installasjonsprogram for skrivebordet; Linux: `.deb`-pakken (inneholder backend-binærfilen og SDK-et) |
| **Python** | 3.7 eller høyere (utviklet/testet på 3.10) |
| **Operativsystem** | Windows 10/11 64-bit, Ubuntu 22.04 LTS eller nyere, eller NVIDIA Jetson (JetPack 6) |
| **Lisens** | Aktiv Chloros+-pålogging, hvilket som helst betalt nivå (Copper eller høyere) |

## Seieren på 60 sekunder

Én kommando oppretter et prosjekt, importerer en mappe, konfigurerer behandlingen og kjører pipelinen – og starter backend-programmet automatisk hvis det ikke allerede kjører:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)
```

(På Linux bruker du Linux-stier: `/home/user/drone_images/flight001`. SDK-et fungerer på samme måte på begge plattformene.)

Skal du behandle en LATTICE-opptaksmappe? Bruk den LATTICE-vennlige wrapperen – den bruker de riktige standardinnstillingene (ingen deteksjon av panelmål, standard debayer):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` – full kontroll over pipelinen

For alt utover en enkeltlinje, bruk `ChlorosLocal`. Den bruker et backend som allerede kjører (skrivebordsappens), eller starter det installerte ved første bruk (`auto_start_backend=True`; på Windows fra mappen `backend` i Chloros-installasjonsmappen, `C:\Program Files\Chloros\backend\` som standard), oppretter og konfigurerer prosjekter, overvåker fremdriften og returnerer en oppsummering etter kjøringen.

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

{% hint style="info" %}
Behold standardverdien `http://127.0.0.1:5000` i stedet for å erstatte den med `localhost` — på Windows blir `localhost` først omgjort til `::1` og tar ca. 2 sekunder per forespørsel mot backenden som kun støtter IPv4.
{% endhint %}

Bruk den som en kontekstbehandler. Når du forlater blokken, avsluttes denne instansens bruk av backend-en, ikke selve backend-en: den fortsetter å kjøre for det neste skriptet og stopper av seg selv etter 30 minutter uten bruk. `shutdown_backend()` stopper denne instansen når den er startet.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` godtar følgende nøkkelord: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` og `custom_settings`. De viktigste verdiene:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (8-bit)"            # 8-bit TIFF; what all-8-bit images get for a deeper format
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

De LATTICE-spesifikke innstillingene (`input_level`, `radiometric_output`, `array_alignment*`-familien) er dokumentert med sine fullstendige verditabeller i [SDK-referansen](reference/sdk-reference.md#supported-values).

### Overvåke fremdrift

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Lesing av sammendraget etter kjøring — og oppdagelse av tomme kjøringer

Når kjøringen er fullført, legger `process()` ved backendens behandlingssammendrag som `result["summary"]`. Hver oppføring i `summary["hints"]` er en fullstendig setning som forklarer alt som er verdt å merke seg — for eksempel hvorfor en kjøring ikke ga noe resultat — og hvert hint sendes også ut på nytt som en Python-`UserWarning`, slik at tomme kjøringer diagnostiserer seg selv selv om du aldri sjekker ordboken:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` utløses ikke når en kjøring ikke produserer noen bilder.** Dette er det eneste stedet hvor SDK-et og CLI-et bevisst skiller seg fra hverandre: `chloros-cli process` behandler «produkter ble forespurt, ingen ble skrevet» som en feil og avsluttes med en verdi ulik null, mens SDK-et returnerer normalt og rapporterer tilstanden gjennom `summary` / hint. Hvis pipelinen din skal stoppe ved en tom kjøring, må du sjekke dette selv — undersøk `summary` (eller tell filene i prosjektmappen) i stedet for å stole på et unntak.
{% endhint %}

## Smart Connect — live-maskinvare

Tre hjelpefunksjoner åpner vedvarende økter i backendens maskinvarepool — den samme poolen som CLI, andre skript og GUI-ens kamerafaner bruker, slik at SDK-skript kan sameksistere med dem uten å konkurrere om serielle porter eller nettverksbåndbredde. Alle tre starter automatisk en lokal backend hvis ingen kjører. GUI-ens DAQ-panel er ikke en del av puljen: lukk en lyssensor i GUI-en før du åpner den fra et skript.

### Enkelt LATTICE-kamera — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

{% hint style="info" %}
**Relative baner ender der skriptet ditt kjører (endret i 1.3.0).** Med en lokal backend, en relativ `output_dir` (inkludert standard `"output"`) — og de andre stiene som backenden åpner (`burst_dir`, `profile_path`, `save_path`, `mask_path`) — blir løst i forhold til skriptets arbeidsmappe, slik at `cam.capture("output/")` skriver til `<current folder>/output/`. I versjon 1.2.0 løste en installert backend dem i sin egen installasjonsmappe og ga feilmeldingen «Tilgang nektet». Mot en ekstern backend sendes stiene slik de er angitt.
{% endhint %}

### Synkronisert matrise — `connect_array`

`connect_array` er det anbefalte inngangspunktet for rigger med flere kameraer. Den kjører den samme «smart-prep»-flyten som GUI-en: nettverksanalyse, automatisk valg av synkroniseringsnivå, PTP-tidssynkronisering, valg av pikselformat per kamera, AE-seeding og aktivering av GPIO-utløser. Den **første serienummeret er masteren** (den utløser maskinvareutløserpulsen); resten er slaver.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Legg til `smart=True` til en hvilken som helst array-opptak for å vente på at autoeksponeringen skal stabilisere seg på tvers av alle kameraene før utløsning.

Nytt i versjon 1.3.0: en `ArraySession` gir deg også synkroniserte bilder direkte, uten å skrive filer:

* `arr.grab()` returnerer ett bilde per kamera, alle fra én utløsning, som numpy-matriser (`{serial: array}`); `.group_id` er den delte utløseren, og `.errors` angir eventuelle kameraer som ikke er inkludert.
* `arr.frame_stream(fps=5.0)` gir ett slikt sett per tick, for live-behandling i din egen pipeline.
* `arr.continuous("output/")` kjører backendens egen kontinuerlige opptak (GUI-ens kontinuerlige modus) og returnerer en `ContinuousHandle` med `status()`, `wait()` og `stop()` – bruk dette i stedet for en opptakssløyfe når du trenger at hver utløser lagres med høy frekvens.

For opptaksmoduser (Enkelt / Kontinuerlig / Intervall / Raskest), opptakere, burst-til-video og matrisejustering, se [SDK-referansen](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### DAQ-lyssensor — `connect_daq_sensor`

Uten argumenter oppdager `connect_daq_sensor()` automatisk transportprotokollen (prioritet: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Hver ramme inneholder 135-punkts `spectrum` (W/m²/nm når kalibrert), et `is_saturated`-flagg og CIE `x`, `y`, `z`. For å feste en bestemt sensor eller transport, må du oppgi ett eksplisitt hint:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Merk at profiler for takkorreksjon (`cap_id`) **ikke** er en SDK-knott — velg dem i stedet via `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Lagrede prosjekter — `open_project`

Et lagret Chloros-prosjekt beholder den tilkoblede maskinvaren (`cameras.json` + `sensors.json` sammen med `project.json`), og `chloros_sdk.open_project(path)` kan koble til kameraene, matrisene og lyssensorene sine på en gang og styre opptakene etter enhetsnavn. Lyssensorer kobles til på nytt via en kjørende Chloros-backend, med sine lagrede innstillinger og opptakskapasitet. Se [Prosjektautomatisering](reference/sdk-reference.md#project-automation--chlorosproject) i referansen.

## Hva en installasjon med kun pip gir

Sjekk tilgjengelighetsflaggene på modulnivå før du bruker maskinvareflater:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

På en vert med **kun** `pip install chloros-sdk` og uten Chloros-skrivebordspakke:

* `ChlorosLocal`, `process_folder` og `process_lattice_capture` fungerer **ikke** — de trenger backend-binærfilen som følger med i skrivebordsinstallasjonsprogrammet.
* Smart-connect-hjelpeprogrammene (`connect_camera`, `connect_array`, `connect_daq_sensor`) er rene HTTP-klienter, så de fungerer mot en backend på en annen maskin — men de medfølgende backendene binder kun til loopback, så du må videresende porten selv (f.eks. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) og overføre `backend_url="http://127.0.0.1:5000"` sammen med `auto_start_backend=False`. Se [Fjern-backend-modus](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* LATTICE-klassene for direkte maskinvare (`LatticeCamera`, `CameraPool`, …) kan importeres, men krever Arena SDK-kjøretiden fra skrivebordspakken — uten den er `CAMERA_AVAILABLE` lik `False`.
* DAQ-sensorer drives kun gjennom en (tunnellert) backend — `connect_daq_sensor()`, eller et prosjekts `proj.sensors`; backenden åpner sensoren på sin egen maskin.

## AI-assistenter — `chloros-mcp`

SDK-et inkluderer `chloros-mcp`, en MCP-server som lar Claude Desktop, Claude Code, Cursor og andre MCP-klienter styre Chloros: behandle bilder og kjøre LATTICE-kameraer, matriser og DAQ-lyssensorer. Verktøyene dekker det denne SDK-en gjør. Den installeres sammen med SDK-en, så for Claude Code er oppsettet én linje:

```bash
claude mcp add chloros -- chloros-mcp
```

Andre klienter: [Bruk av Chloros med AI-assistenter](ai-assistants.md). Alternativer og alle verktøy: [MCP-server (AI-assistenter)](reference/sdk-reference.md#mcp-server-ai-assistants).

## Lisenskrav

Tilgang til SDK krever en aktiv Chloros+-pålogging på et hvilket som helst betalt nivå — **Copper eller høyere**(Copper / Bronze / Silver / Gold); det gratis Iron-abonnementet har ingen tilgang til SDK/CLI. Kontroll av dette skjer**på serversiden**: hver SDK-forespørsel må inneholde både en aktiv økt og et betalt abonnement, ellers returnerer backenden `403` / `PLAN_UPGRADE_REQUIRED` (generert som `ChlorosLicenseError` av `ChlorosLocal`, og som `ChlorosConnectError` av hjelpefunksjonene `connect_*`). En utlogget bruker får feilmeldingene `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) i stedet — å kjøre `chloros-cli login` på nytt løser det første tilfellet, men ikke det andre.

Offlinebruk fungerer innenfor planens nådeperiode: nivået hentes fra servervalideringsbufferen (5 minutter) eller den signerte, maskinbundne lisensbufferen (30 dager for månedlige planer; til abonnementets utløp for årlige). Når fristen utløper, endres abonnementet til gratisversjonen, og SDK-tilgangen stanses inntil maskinen har opprettet kontakt med serveren én gang. `chloros-cli status` forblir tilgjengelig på gratisnivået, slik at årsaken alltid er synlig. Se [Chloros+ pålogging](chloros+-login.md).

## Unntak

Fang opp baseklassen for å håndtere «alt som gikk galt i Chloros»:

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

Alle unntak i rørledningen (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) stammer fra `ChlorosError`. En liten felle: `ChlorosConnectError` — utløses kun av `connect_camera` / `connect_array` / `connect_daq_sensor` — stammer fra vanlig `Exception`, **ikke** fra `ChlorosError`, så `except ChlorosError` vil ikke fange den opp. Den fullstendige hierarkien finnes i [SDK-referansen](reference/sdk-reference.md#exceptions).

## Se også

* [SDK-referansen](reference/sdk-reference.md) — den komplette API-flaten, optimalisert for AI-assistenter.
* [CLI-referanse](reference/cli-reference.md) — hver CLI-underkommando speiler et SDK-kall.
* [Bruk av Chloros med AI-assistenter](ai-assistants.md) — oppsett og oppskrifter for kommandoprompter for `chloros-mcp`.
* [Last ned](download.md) — installasjonsprogrammer for Windows og Linux.
