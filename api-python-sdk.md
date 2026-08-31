# API : Python SDK

{% hint style="info" %}
**Leter du etter den komplette API?** Denne siden er en praktisk veiledning. Alle offentlige klasser, metoder, eksakte signaturer og eksempler som kan kopieres og limes inn, finnes i [SDK-referansen](reference/sdk-reference.md), som er optimalisert for AI-assistenter.**Bruker du en AI-assistent?** Lim inn denne URL i chatten, slik at den får den fullstendige, oppdaterte Chloros 1.2.0 API:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Hver side i denne håndboken er tilgjengelig som rå markdown under sin slug med små bokstaver + `.md`, og hele håndboken er indeksert under `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` på PyPI) styrer alt skrivebordsappen kan gjøre fra Python: batchbehandling av bilder, live LATTICE-kamera- og matrisestyring, DAQ-lyssensor-økter og automatisering av lagrede prosjekter. Det er et tynt lag over den samme lokale backenden som GUI-en og CLI bruker (HTTP på `127.0.0.1:5000`), så oppførselen er identisk på alle tre plattformene.

## Installasjon

Installasjonen består av to trinn: først Chloros-skrivebordspakken (den inneholder behandlingsbackenden og maskinvareruntimene), deretter Python-pakken.

**Trinn 1 — Installer Chloros.** Windows: Kjør installasjonsprogrammet for skrivebordet (standardbane `C:\Program Files\MAPIR\Chloros\`) fra [Nedlastingssiden](download.md). Linux: installer `.deb`-pakken ([Linux-installasjon](linux/linux-installation.md)).**Trinn 2 — Installer SDK** (Python 3.7+):

```bash
pip install chloros-sdk
```

Du trenger kanskje ikke engang pip: hvert installasjonsprogram leveres med et tilhørende SDK-wheel. Windows-installasjonsprogrammet installerer det automatisk i systemet ditt Python; Linux `.deb` plasserer det i `/usr/lib/chloros/sdk/` og viser den nøyaktige `pip install --user`-kommandoen. PyPI oppdateres ved utgivelser, så `pip install chloros-sdk` samsvarer med den nyeste stabile utgaven.

**Trinn 3 — Logg inn én gang per maskin:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Påloggingsopplysningene lagres i `~/.chloros/` (begge plattformene). På Windows kan du på samme måte logge inn via fanen «Bruker<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">» i skrivebordsappen. SDK krever et betalt Chloros+-abonnement — se [Lisenskrav](#license-requirement) nedenfor.

| Krav | Detaljer |
| --- | --- |
| **Chloros installert** | Windows: installasjonsprogram for skrivebordet; Linux: `.deb`-pakke (inneholder backend-binærfilen) |
| **Python** | 3.7 eller nyere (utviklet/testet på 3.10) |
| **Operativsystem** | Windows 10/11 64-bit, Ubuntu 22.04 LTS eller nyere, eller NVIDIA Jetson (JetPack 6) |
| **Lisens** | Aktiv Chloros+-pålogging, hvilket som helst betalt nivå (Copper eller høyere) |

## 60 sekunders gevinst

Én kommando oppretter et prosjekt, importerer en mappe, konfigurerer behandlingen og kjører pipelinen – og starter backenden automatisk hvis den ikke allerede kjører:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(På Linux, bruk Linux-stier: `/home/user/drone_images/flight001`. SDK fungerer identisk på begge plattformene.)

Behandler du en LATTICE-opptaksmappe? Bruk den LATTICE-vennlige wrapperen — den bruker de riktige standardinnstillingene (ingen deteksjon av panelmål, standard debayer):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — full kontroll over behandlingskjeden

For alt som er mer enn en enkeltlinje, bruk `ChlorosLocal`. Den starter backenden ved første bruk (`auto_start_backend=True`), oppretter og konfigurerer prosjekter, overvåker fremdriften og returnerer en oppsummering etter kjøringen.

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
Behold standardverdien `http://127.0.0.1:5000` i stedet for å erstatte den med `localhost` — på Windows `localhost` løses først til `::1` og tar ca. 2 sekunder per forespørsel mot backenden som kun støtter IPv4.
{% endhint %}

Bruk den som en kontekstbehandler for garantert opprydding:

```python
import chloros_sdk

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

`configure()` godtar følgende nøkkelord: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` og `custom_settings`. De viktigste verdiene:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

De LATTICE-spesifikke knappene (`input_level`, `radiometric_output`, `array_alignment*`-familien) er dokumentert med sine fullstendige verditabeller i [SDK-referansen](reference/sdk-reference.md#supported-values).

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

Når kjøringen er fullført, legger `process()` ved backendens behandlingssammendrag som `result["summary"]`. Hver oppføring i `summary["hints"]` er en fullstendig setning som forklarer alt som er verdt å merke seg — for eksempel hvorfor en kjøring ga null utdata – og hvert hint sendes også ut på nytt som en Python `UserWarning`, slik at tomme kjøringer diagnostiserer seg selv selv om du aldri sjekker ordboken:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` utløses ikke når en kjøring ikke produserer noen bilder.** Dette er det eneste stedet hvor SDK og CLI bevisst skiller seg fra hverandre: `chloros-cli process` behandler «produkter ble forespurt, ingen ble skrevet» som en feil og avsluttes med en verdi ulik null, mens SDK avsluttes normalt og rapporterer tilstanden via `summary` / hints. Hvis pipelinen din skal stoppe ved en tom kjøring, må du sjekke dette selv — undersøk `summary` (eller tell filene i prosjektmappen) i stedet for å stole på et unntak.
{% endhint %}

## Smart Connect — live-maskinvare

Tre hjelpefunksjoner åpner vedvarende økter i backendens maskinvarepool – den samme poolen som GUI-en bruker, slik at SDK-skript kan eksistere side om side med skrivebordsappen uten å konkurrere om serielle porter eller nettverksbåndbredde. Alle tre starter automatisk en lokal backend hvis ingen kjører.

### Enkelt LATTICE-kamera — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### Synkronisert oppstilling — `connect_array`

`connect_array` er det anbefalte utgangspunktet for rigger med flere kameraer. Det kjører den samme «smart-prep»-flyten som GUI-en: nettverksanalyse, automatisk valg av synkroniseringsnivå, PTP-tidssynkronisering, valg av pikselformat per kamera, AE-seeding og aktivering av GPIO-utløser. Den **første serien er masteren** (den utløser maskinvaretriggerpulsen); resten er slaver.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Legg til `smart=True` til hvilken som helst array-opptak for å vente på at automatisk eksponering skal stabilisere seg på tvers av alle kameraene før utløsning. For opptaksmoduser (Enkelt / Kontinuerlig / Intervall / Raskest), opptakere, burst-til-video og matrisejustering, se [SDK-referansen](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### DAQ-lyssensor — `connect_daq_sensor`

Uten argumenter oppdager `connect_daq_sensor()` automatisk transporttypen (prioritet: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Hver ramme inneholder 135-punkts `spectrum` (W/m²/nm når kalibrert), et `is_saturated`-flagg og CIE `x`, `y`, `z`. For å spesifisere en bestemt sensor eller transport – det pålitelige valget på verter med flere nettverksgrensesnitt, der automatisk oppdagelse via Ethernet kan overse en velfungerende DAQ-E ved første forsøk – må du sende ett eksplisitt hint:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Merk at profiler for kapasitanskorreksjon (`cap_id`) **ikke** er en SDK-kontroll – velg dem i stedet via `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap` i stedet.

### Lagrede prosjekter — `open_project`

Et lagret Chloros-prosjekt beholder den tilkoblede maskinvaren (`cameras.json` + `sensors.json` sammen med `project.json`), og `chloros_sdk.open_project(path)` kan koble til alt på en gang og utføre opptak etter enhetsnavn. Se [Prosjektautomatisering](reference/sdk-reference.md#project-automation--chlorosproject) i referansen.

## Hva en installasjon kun med pip gir

Sjekk tilgjengelighetsflaggene på modulnivå før du bruker maskinvareflater:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

På en vert med **kun** `pip install chloros-sdk` og uten Chloros-skrivebordspakke:

* `ChlorosLocal`, `process_folder` og `process_lattice_capture` fungerer **ikke** — de trenger backend-binærfilen som følger med i skrivebordsinstallasjonsprogrammet.
* Smart-connect-hjelperne (`connect_camera`, `connect_array`, `connect_daq_sensor`) er rene HTTP-klienter, så de fungerer mot en backend på en annen maskin — men de medfølgende backendene binder kun til loopback, så du må videresende porten selv (f.eks. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) og sende `backend_url="http://127.0.0.1:5000"` sammen med `auto_start_backend=False`. Se [Fjern-backend-modus](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* De direkte maskinvarebaserte LATTICE-klassene (`LatticeCamera`, `CameraPool`, …) kan importeres, men trenger Arena SDK-kjøretiden fra skrivebordspakken – uten den er `CAMERA_AVAILABLE` lik `False`.
* `daq_sdk` (de direkte DAQ-klassene) følger med skrivebordsinstallasjonen, ikke PyPI-pakken, så `DAQ_AVAILABLE` er `False` på en vert som kun bruker pip — styr DAQ-sensorer via `connect_daq_sensor()` mot et (tunnelert) backend i stedet.

## Lisenskrav

Tilgang til SDK krever en aktiv Chloros+-pålogging på et hvilket som helst betalt nivå — **Copper eller høyere**(Copper / Bronze / Silver / Gold); det gratis Iron-nivået har ingen tilgang til SDK/CLI. Håndhevelsen skjer**på serversiden**: hver SDK-forespørsel må inneholde både en aktiv økt og et betalt abonnement, ellers returnerer backend-systemet `403` / `PLAN_UPGRADE_REQUIRED` (utløst som `ChlorosLicenseError` av `ChlorosLocal`, og som `ChlorosConnectError` av `connect_*`-hjelperne). En utlogget anroper får i stedet `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) i stedet – å kjøre `chloros-cli login` på nytt løser det første tilfellet, men ikke det andre.

Offlinebruk fungerer innenfor planens nådeperiode: nivået leses fra servervalideringsbufferen (5 minutter) eller den signerte, maskinbundne lisensbufferen (30 dager for månedlige planer; til abonnementets utløp for årlige planer). Når fristen utløper, går abonnementet over til gratisversjonen, og tilgangen via SDK stanses inntil maskinen har opprettet kontakt med serveren én gang. `chloros-cli status` forblir tilgjengelig på gratisnivået, slik at årsaken alltid er synlig. Se [Chloros+ Innlogging](chloros+-login.md).

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

Alle unntak i behandlingskjeden (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) stammer fra `ChlorosError`. En liten felle: `ChlorosConnectError` — utløses kun av `connect_camera` / `connect_array` / `connect_daq_sensor` — stammer fra vanlig `Exception`, **ikke** fra `ChlorosError`, så `except ChlorosError` vil ikke fange den opp. Det fullstendige hierarkiet finnes i [SDK-referansen](reference/sdk-reference.md#exceptions).

## Se også

* [SDK-referanse](reference/sdk-reference.md) — den komplette API-overflaten, optimalisert for AI-assistenter.
* [CLI-referanse](reference/cli-reference.md) — hver CLI-underkommando speiler et SDK-kall.
* [Last ned](download.md) — installasjonsprogrammer for Windows og Linux.
