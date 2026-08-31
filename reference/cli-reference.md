# Chloros CLI Referanse

**Versjon:**

1.2.0**Generert:**

29.07.2026 kl. 19:19 ·**Revidert:**

30.08.2026**Målgruppe:** Optimalisert for bruk av store språkmodeller (LLM); lesbar for mennesker.**Omfang:** Alle brukerrettede underkommandoer til `chloros-cli`, med alternativer og eksempler som kan kopieres og limes inn.

Dette dokumentet er den fullstendige referansen for kommandolinjeverktøyet `chloros-cli`-linjeverktøyet som følger med MAPIR Chloros. Det er bevisst uttømmende, slik at en LLM (eller et menneske) kan sette sammen enhver støttet arbeidsflyt fra listene nedenfor uten å måtte undersøke kildekoden.

Hvis du bare trenger det viktigste, kan du hoppe til:
- [Fem minutters hurtigstart](#five-minute-quickstart)
- [LATTICE-kamera – arbeidsflyt for første tilkobling](#lattice-camera-first-connect-workflow)
- [DAQ-sensor – arbeidsflyt for første tilkobling](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Opptakmoduser, opptakerenheter og offline-bearbeiding](#capture-modes-recorders--offline-reprocess)

---

## Konvensjoner

- Alle kommandoer har prefikset `chloros-cli`. På Windows er binærfilen `chloros-cli.exe`; på Linux/Jetson er den `chloros-cli`.
- Valgfrie argumenter vises som `--flag`. Obligatoriske posisjonsargumenter vises uten parenteser.
- Der det er angitt en standardverdi, vil utelatelse av flagget føre til at denne verdien brukes.
- CLI er en «thin» HTTP-klient over Chloros-backenden (Flask-server på `127.0.0.1:5000`). Backenden startes automatisk av de fleste kommandoer. `CHLOROS_BACKEND_URL=<url>` peker mot **`lattice`**,**`project`**og**`daq pool-*`**-kommandofamiliene til en ekstern backend – kjerne-kommandoene (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) låser bevisst `http://127.0.0.1:<port>` og ignorerer den (IPv4-litteralen unngår Windows&#x27; `localhost`→`::1` ~2 s per forespørsel). Se [Miljøvariabler](#environment-variables).
- Det kreves innlogging med en Chloros+-konto for alle SDK/CLI-kall (kjør `chloros-cli login` én gang per maskin; lagres i `~/.chloros/`).
- Eksemplene bruker Linux-baner; på Windows erstattes `/home/user/...` med `C:/Users/.../...`.

---

## Oversikt på øverste nivå

```
chloros-cli [global options] COMMAND [command options]
```

### Globale alternativer

| Flagg | Beskrivelse |
| --- | --- |
| `--backend-exe PATH` | Overstyr den automatisk oppdagede kjørbare filen for backend. |
| `--port N` | Backend HTTP-port (standard: `5000`). |
| `-v, --verbose` | Aktiver detaljert utdata. |
| `--restart` | Tving omstart av backend (avslutter alle kjørende `backend_server.py`-prosesser). |
| `--version` | Vis versjon (`Chloros CLI 1.2.0`). |
| `--help` | Vis hjelp på øverste nivå. |

### Kommandoindeks

| Kommando | Formål |
| --- | --- |
| [`process`](#chloros-cli-process) | Behandle en mappe med Survey3- eller LATTICE-opptak fra start til slutt. |
| [`login`](#chloros-cli-login) | Autentiser denne maskinen med en Chloros+-konto. |
| [`logout`](#chloros-cli-logout) | Tøm bufret påloggingsinformasjon. |
| [`status`](#chloros-cli-status) | Vis gjeldende lisens-/autentiseringsstatus. |
| [`export-status`](#chloros-cli-export-status) | Eksportfremdrift for Live Thread-4 under en `process`-kjøring. |
| [`language`](#chloros-cli-language) | Angi eller vis CLI-visningsspråk (38 støttes). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Standard prosjektmappe (delt med GUI). |
| [`update`](#chloros-cli-update) | Sjekk etter og installer CLI-oppdateringer (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | Systemdiagnostikk + røykprøver. |
| [`time-sync`](#chloros-cli-time-sync) | PTP-grandmaster-status / -kontroll. |
| [`lattice`](#chloros-cli-lattice) | LATTICE-kamerakontroll og -opptak (45+ underkommandoer). |
| [`daq`](#chloros-cli-daq) | DAQ-spektralsensorstyring (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Åpne og kjør et lagret Chloros-prosjekt (kameraer + DAQ-er). |

---

## Installasjon

`chloros-cli` leveres som en del av Chloros-installasjonsprogrammet for skrivebordet på alle støttede plattformer — det finnes ingen separat nedlasting av CLI. Når du installerer plattformpakken, legges `chloros-cli` til `PATH` sammen med skrivebordsappen og backend-binærfilen den kjører.

Siste nedlastinger: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> installasjonsprogrammet leveres også med praktiske startskript (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`) som åpner et bruksklartbruk CLI-skall; disse er beskrevet i [CLI brukerhåndboken](../CLI.md) og gjengis ikke her.

### Windows (.exe)

1. Last ned installasjonsprogrammet Windows fra nedlastingssiden.
2. Kjør `Chloros-Setup-x.y.z.exe` og følg veiviseren. Standard installasjonssti er `C:\Program Files\Chloros\` (CLI havner i `C:\Program Files\Chloros\cli\`, som installasjonsprogrammet legger til i PATH).
3. Åpne en ny terminal (`cmd.exe`, PowerShell eller Windows Terminal) slik at den oppdaterte `PATH` blir hentet inn.

```powershell
chloros-cli --version
```

Installasjonsprogrammet legger automatisk til `chloros-cli.exe` i systemveien din `PATH` og inkluderer Arena SDK-kjøretiden som trengs for LATTICE-kameraer.

### Linux amd64 (.deb)

For Ubuntu 22.04 LTS eller nyere / Debian-baserte x86_64-arbeidsstasjoner.

> **Ubuntu 20.04 støttes ikke.** Pakkens avhengighetsliste er avledet fra
> det som backend faktisk kobler seg mot, og det inkluderer `libc6 (>= 2.34)`;
> focal leverer glibc 2.31. `apt` avviser installasjonen i stedet for å la den mislykkes ved
> kjøring.

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

.deb-filen installerer:
- `chloros-cli` til `/usr/bin/chloros-cli`
- Den kompilerte backenden til `/usr/lib/chloros/chloros-backend`
- Arena SDK-kjøringstid (for LATTICE-kameraer)
- Støyreduksjonsmodeller, kalibreringspakker og konfigurasjon for oppdateringskanal

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Samme oppsett som amd64 .deb, med en CUDA-versjon tilpasset Jetson Orin / Orin NX / Orin Nano.

### Autentiser én gang per maskin

Hver plattform krever en engangsinnlogging med Chloros+ før SDK/CLI-kall fungerer:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Påloggingsopplysningene lagres i `~/.chloros/user_session.json`.

### Bekreft installasjonen

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **Chloros+-abonnement kreves.**CLI krever et aktivt Chloros+-abonnement.**Copper**er inngangsnivået Chloros+-nivået — alle betalte Chloros+-nivåer har tilgang til CLI/SDK; kun det gratis**Iron**-nivået har ikke dette. (Abonnements-ID-oversikt: `0`=Iron/gratis, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Oppgrader på [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Denne grensen håndheves av backend-systemet, ikke bare av CLI: en forespørsel merket med SDK/CLI-merket forespørsel uten et betalt abonnement blir avvist med `403 PLAN_UPGRADE_REQUIRED`, uansett om den kommer fra `chloros-cli`, Python SDK eller en manuelt utviklet HTTP-klient. En utlogget bruker får i stedet feilmeldingen `401 AUTH_REQUIRED`. Tilgang fungerer offline i abonnementets nådeperiode (30 dager for månedsabonnement, til utløpet for årsabonnement) og opphører når denne utløper; `chloros-cli status` fortsetter å fungere slik at årsaken er synlig (det er den eneste SDK/CLI-ruten som er unntatt fra nivåbegrensningen — `GET /api/license-status`).

---

## Fem minutters hurtigstart

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

Behandle en mappe med bilder gjennom hele Chloros-rørledningen (målgjenkjenning → kalibrering → vignett → refleksjonsgrad → indekseksport).

### Oversikt

```
chloros-cli process INPUT [OPTIONS]
```

### Posisjonsargumenter

| Argument | Beskrivelse |
| --- | --- |
| `INPUT` | Sti til inndatamappe som inneholder `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) eller `.dng`-filer. |

### Vanlige alternativer

| Flagg | Standard | Beskrivelse |
| --- | --- | --- |
| `-o, --output PATH` | en ny mappe med tidsstempel under standard prosjektstien (`~/Chloros Projects` med mindre annet er konfigurert) | Prosjektmappe som skal opprettes eller gjenbrukes. Hvis mappen allerede inneholder en `project.json`, opprettes en `_1`/`_2`-søskenmappe i stedet for å overskrive. |
| `-n, --project-name NAME` | auto (tidsstempel) | Prosjektnavn. |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` bruker en Chloros+ nevral debayer; tregere, men høyere kvalitet. |
| `--vignette / --no-vignette` | `--vignette` | Vignettkorreksjon. |
| `--reflectance / --no-reflectance` | `--reflectance` | Refleksjonskalibrering (bruker panelmål hvis det finnes, NIST-kalibrering per serienummer for LATTICE). For LATTICE multispektral fungerer dette også som bryteren for **produkt**-refleksjonsgrad — se [Eksportbrytere per produkt](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | av | Bruk PPK GNSS-korreksjoner fra sidecar-filer. |
| `--exposure-pin-1 MODEL` | av | Fest en Survey3-modell med to kameraer «pin-1»-modellen. |
| `--exposure-pin-2 MODEL` | av | Fest «pin-2»-modellen. |
| `--recal-interval SECONDS` | 0 | Tving omkjøring av kalibreringsberegninger hvert N sekund av opptakstiden. |
| `--timezone-offset HOURS` | lokal | Overstyr tidssoneforskjellen som er innebygd i utdatametadataene. |
| `--format FORMAT` | `TIFF (16-bit)` | En av følgende: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | ingen | Vegetasjonsindekser (`NDVI`, `NDRE`, `GNDVI`, `EVI`, `SAVI`, `OSAVI`, `CIG`, …). |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Tving inngangspunktet til behandlingskjeden for LATTICE TIFF-filer (Survey3 .raw påvirkes ikke). Også unntaksmuligheten som gjør at en opptak med **ingen raw** kan behandles i det hele tatt — se [Hvordan en opptaksmappe ser ut](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | på | Send ut det lineære debayerte produktet (`Debayered_Images`). Se [Eksportbrytere per produkt](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | på | Send ut forhåndsvisningen (`Preview_Images`): RGB = hvitbalanse (DAQ-lyskilde når tilgjengelig, ellers gråverden) + gamma; multispec = falskfargestrekning. |
| `--radiance / --no-radiance` | på | Send ut radianse i float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referanse for LATTICE-refleksjonsproduktet: `auto` = QA-godkjent mål i bildet er den absolutte referansen, DAQ-nedadgående (ρ = π·L/E) som reserve; `target` = strengt (ingen DAQ-erstatning); `daq` = DAQ-autoritativ. Se [Eksportbrytere per produkt](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | ingen | Katalog over **målte** målrefleksjonsskanninger per enhet (`<serial>.csv`); faller tilbake til de nominelle T3/T4P-spektrene ved manglende data. |
| `--array-alignment / --no-array-alignment` | på | LATTICE-matriser: bruk modul-til-modul-justeringen som er stemplet i hver opptakets `Chloros:Alignment*` XMP-fil på hvert bearbeidet produkt (debayered / forhåndsvisning / strålingsintensitet / reflektans / indeks). Ingen operasjon for bilder uten disse taggene. |
| `--array-alignment-crop / --no-array-alignment-crop` | beskjæring | Beskjær justerte eksporter til matrisens felles overlappingsområde slik at alle moduler deler ett fotavtrykk; `--no-…` beholder hele sensorområdet (svart fylling utenfor kilden). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Resampling for justeringsforvrengningen. `nearest` bevarer eksakte kilde-DN-er (ingen blanding av radiometriske verdier mellom piksler). |

### Alternativer for måldeteksjon

| Flagg | Beskrivelse |
| --- | --- |
| `--min-target-size PIXELS` | Minste panel-målstørrelse (px) for detektor. |
| `--target-clustering 0-100` | Klyngesensitivitet. |
| `--target / --targets` | Behandle inndatamappen som kun målpanel (hopp over kartleggingsdeteksjon). |

### Eksempler

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### Eksportbrytere per produkt (LATTICE multispektral)

LATTICE-behandlingen fordelers seg på **alle relevante produkter i én gjennomgang**. De fire bryterne per type — `--debayered`, `--preview`, `--radiance`, `--reflectance` — er alle**slått på som standard**; bruk skjemaet `--no-<type>` for å deaktivere en. RGB-hovedkameraer sender kun ut debayert + forhåndsvisning (ingenbånds radianse/reflektans), så `--radiance`/`--reflectance` har ingen effekt for dem. Vekslerne ignoreres for Survey3 og `.raw` (som følger standardbanen for reflektans/mål). *(Det gamle `--radiometric-output {reflectance,radiance,sensor-response}`-flagget ble **fjernet** og erstattet av disse bryterne; det finnes ikke lenger noe `sensor-response`-nivå.)*

| Produkt | Utgang | Kreves DAQ-nedstråling? |
| --- | --- | --- |
| `--debayered` | Lineær demosaik (`Debayered_Images`). | Nei. |
| `--preview` | Vis forhåndsvisning (`Preview_Images`): RGB = WB + gamma; multispek = falskfargeutvidelse. | Nei. |
| `--radiance` | float32 W/m²/sr/nm fra den fullstendige radiometriske kjeden (`Radiance_Images`). | Nr. |
| `--reflectance` | uint16 reflektans ρ (`32768` = 1,0), klar for Pix4D. | **Ja**, med mindre et QA-godkjent mål innenfor rammen forankrer den (se nedenfor). |

`--reflectance-source` velger reflektansreferansen:**`auto`**(standard) gjør et QA-godkjent mål innenfor rammen til den**absolutte referansen**— de målforankrede empiriske linjekjedene kryssscores på holdte-ut-paneler, og den målte vinneren blir brukt — med tilbakefall til DAQs nedadgående delingsgrense (ρ = π·L/E) når det ikke finnes noe mål eller QA mislykkes;**`target`**er strengt (ingen DAQ-erstatning);**`daq`**velger bort til den DAQ-autoritative oppførselen. Målgeometri (ArUco / fast ROI / stripe) hentes fra prosjektets målkonfigurasjon; `--target-reflectance-dir DIR` lagrer**målte** skanninger per enhet (`<serial>.csv`) slått opp etter målenhetens serienummer/QR, med de nominelle T3/T4P-spektrene som reserve.

DAQ-refleksjonsbanen løser **tidsstempel-matchet nedadgående**automatisk fra en registrert**`.daq`**(DAQ-U/M/E)**eller en DAQ-M-egendefinert `.csv`**som finnes sammen med bildene. Hvis en kalibreringspakke per kamera eller DAQ kalibreringspakke ikke er lagret lokalt i hurtigbufferen,**henter rørledningen den automatisk fra AWS** ved første bruk (krever internettilgang én gang; lagres i hurtigbufferen under `~/.chloros/`).

#### Lesing av refleksjonspiksler (Pix4D / Metashape / dine egne skript)

Refleksjonen lagres som et heltall (DN), og **den DN-verdien som tilsvarer ρ = 1,0 avhenger av kildekameraet**:

| Kilde | ρ = 1,0 er | Hvordan se det |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (margin opp til ρ 2,0) | XMP-taggen `Chloros:PixelScale=32768` er stemplet på filen. |
| Survey3 | `65535` (avkortet ved ρ 1,0) | Ingen `Chloros:*` XMP-koder — denne fraværet *er* selve signalet. |

**Les `Chloros:PixelScale` og del med det** i stedet for å anta en konstant. Taggen er definert i uint16-domenet, så den forblir `32768` på tvers av utdataformater som skalerer om — `TIFF (16-bit)`, `PNG (8-bit)`, `JPG (8-bit)` og `TIFF (32-bit, Percent)` er alle selvbeskrivende (normaliser først den lagrede datatypen tilbake til uint16: ×257 fra 8-bit, ×65535 fra float).

> **Ett tilfelle har ingen skalering, slik det er utformet.** Når en 8-biters kildeopptak (BayerRG8) skrives som 8-biters TIFF, *klipper* til 0..255 i stedet for å skalere om, slik at alle verdier over ρ≈0,008 blir flatet til 255, og ingen skala beskriver filen. Chloros utelater bevisst både `Chloros:PixelScale` og tuplen `MicaSense:RadiometricCalibration` der, og logger årsaken til dette. **Hvis taggen mangler i en LATTICE-refleksjonsfil, må du ikke anta at det finnes en skala — eksporter på nytt i 16-bit eller 32-bit** i stedet for å dele opp piksler som aldri var delbare.

#### EXIF overføres til eksporten

`process` kopierer kildebildets **GPS-blokk og dens ExifIFD** til hvert produkt, slik at en
eksport inneholder `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` og
`CameraSerialNumber` sammen med georefereringen.

**`FocalLength` er ikke valgfritt for fotogrammetri.** Pix4D beregner avstanden mellom bakkepunkter ut fra
brennvidde pluss høyde; uten denne taggen faller beregningen tilbake til en helt feil skala. Under en
49flytur over en appelsinlund forvandlet den manglende taggen et område på 411 m × 160 m til et rekonstruert
område på 47,8 km × 13 km — en ortofoto på 455 MP bestående hovedsakelig av «nodata», som deretter ble tolket som et flisleggingsproblem og
et BigTIFF-problem før noen sjekket GSD. Hvis ortofotoet ditt ender opp med en usannsynlig
skala, kjør først `exiftool -FocalLength` på det eksporterte produktet.

Kopien er bevisst **ikke** `-all:all`: IFD0s strukturtagger ødelegger LATTICE-utdata når de
kopieres, og `ExifImageWidth` / `ExifImageHeight` er utelatt fordi de beskriver
*kilden* opptaket — en eksport som noen gang har blitt endret i størrelse, ville ellers inneholde dimensjoner
som strider mot dets eget raster. XMP skrives direkte i stedet for å kopieres, fordi ExifTool
kasserer XMP-tagger fra samme påkalling når XMP-blokken kopieres (noe som ville føre til at MAPIR
kalibreringstaggene ble fjernet).

### Hvor utdataene havner

Produktene lagres **i prosjektmappen, gruppert etter kamera og deretter etter filformat**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Kameramappen er `LATT-<sensor>-<lens>-F<filter>` for LATTICE (samsvarer med opptakets EXIF
`Model`) og `<model>_<filter>` for Survey3 — to kameraer som deler sensor og filter, men som skiller seg
fra hverandre når det gjelder objektiv, har separate trær, fordi vignettering, synsfelt og forvrengning varierer. Formatmappen
følger `--format`: `tiff16`, `tiff8`, `png8`, `jpg8`, eller `tiff32` for
`TIFF (32-bit, Percent)`.

> **Hvert eksportert produkt beholder navnet på KILDE-filen.** En Radiance-eksport av
> `capture_…_raw.tif` heter fortsatt `capture_…_raw.tif` — den ligger bare i
> `tiff32/Radiance_Images/`. **Mappen identifiserer produktet, ikke filnavnet**, så globbing
> etter `*radiance*.tif` gir ingen treff; bruk i stedet mappen som søkekriterium.

### Lys-sensoropptak — kalibrert `.daq` + `.csv`

`process` håndterer også `.daq`-opptakene i inndatamappen din, og det **trenger ikke**
trenger noen bilder for å gjøre det: en DAQ-U / DAQ-M / DAQ-E som flys alene, utgjør en fullstendig
opptak, og en mappe som kun inneholder `.daq`-filer, er en gyldig inndata.

En DAQ kan registreres **uten** kalibrering – det er det de offentlige
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) opptakerne
(`record_daq.py`) gjør som standard: de skriver ut rå sensortall og merker filen slik at
Chloros henter sensorens fabrikkalibrering **via serienummer** (lokalt cache først,
deretter MAPIR Cloud) og bruker den. `process` skriver resultatet tilbake:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv` inneholder én rad per måling: UTC-tidsstempel, integrasjonstid, total effekt,
fotopisk/skotopisk lux, PPFD (og dens oppdeling i blått/grønt/rødt), toppbølgelengde, deretter det
fullstendige spektrumet på sensorens eget bølgelengdegitter. `.daq` importeres på nytt uten å bli
kalibrert en gang til.

Ved vellykket gjennomføring rapporterer kjøringen `Light-sensor products written: N (calibrated .daq + .csv)`.
Teksten i parentes beskriver hva som faktisk ble skrevet, så den lyder
`(RAW COUNTS — this sensor has no calibration bundle)` for en sensor uten pakke og
`(N calibrated, M raw counts)` for en mappe som inneholder begge. Backendens egne
overskrifter `[DAQ-EXPORT]` og `[RUN-SUMMARY]` har samme ordlyd — ingen av
de tre kan kalle en rå eksport for kalibrert.

En DAQ-U / DAQ-M / DAQ-E-opptak der kalibreringspakken ikke kan hentes – du er
offline, eller sensoren har ingen kalibrering i filen – **hoppes over med en begrunnelse** på en
`[DAQ-EXPORT]`-linje, og skrives aldri ut som en «kalibrert» fil som inneholder rå tellinger.
Koble deg til internett og kjør på nytt. Årsaken er den som leseren faktisk
fastslo for den filen (uleselig skjema, ingen pakke, en skrivefeil), og kjøringssammendraget
viser **distinkte** årsaker — tjue filer hoppet over på grunn av én årsak vises som én
årsak, ikke tjue gjentakelser av den.

#### DAQ-A-opptak eksporteres som rå tellinger

**DAQ-A**-familien er eldre enn systemet med serienummerbaserte bundler og har ingen kalibreringsbundle
å hente – den kalibreres i stedet i felten mot et refleksjonsmål, og det er
grunnen til at den aldri har trengt en. Å avvise disse opptakene gjorde at de ikke hadde noen måte å få ut
tallene sine i det hele tatt, så de eksporteres under et **annet navn**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Et annet filnavn i stedet for en markør inne i filen, fordi opplysningen må overleve
å bli sendt videre via e-post som et rent navn. `.csv`-overskriften angir
`raw spectral sensor counts (NOT irradiance)` og advarer om at verdiene er sammenlignbare
**innenfor** filen — noe som er nøyaktig hva målbasert kalibrering bruker dem til — og
ikke på tvers av sensorer. De effektavhengige fotometriske kolonnene (total effekt, fotopisk og
skotopisk lux, PPFD) skrives som **NULL** i stedet for å integreres fra tellinger, og
oppsummeringen sier `RAW COUNTS`, så «eksportert» i en logg kan ikke leses som bestrålingsintensitet.

Eldre **v1.01 / v1.02**-registreringer (en DAQ-A-SD skriver disse) har ingen epoke per avlesning,
bare filens skrivetid. Bildet↔nedstrålingsmatcher avviser dem fortsatt — å sammenligne et
bilde med en skrivetid ville være feil uten at man ser det — men eksportøren leser dem, og
CSV skriver ut `clock=daq_created_on`, slik at produktet angir hvilken klokke det er på.

### Merknader

- `process` oppdager automatisk om mappen din er Survey3, LATTICE eller en blanding.
- Fremdriftsstrømmer via Server-Sent Events; CLI viser sanntids fremdrift per tråd (detektering, analyse, behandling, eksport).
- For Linux/Jetson sjekker CLI bytteplass og kan gi en advarsel før behandling av store mapper. Tekstur-bevisst debayer setter også automatisk en GPU-frekvensbegrensning på Jetson-enheter med lav effekt (Nano, Orin Nano).
- Ved suksess rapporterer kjøringen hvor mange bildeprodukter den skrev (`Image products written: N`).

#### En kjøring som ikke skriver ut noen bilder, mislykkes

Hvis du ba om produkter og kjøringen ikke skrev ut **noen** — bare `project.json` og
`calibration_data.json` — behandler `process` dette som en feil: den skriver ut
`Processing finished but wrote no image products.` og **avsluttes med et tall større enn null**, slik at et skript kan
oppdage det. Meldingen oppgir prosjektmappen og de vanlige årsakene:

- inndatamappen ble ikke gjenkjent som en opptak (sjekk oppsettet og `--input-level`), eller
- alle forespurte produkter ble hoppet over som uaktuelle for disse kameraene (f.eks. ved å be om
  strålingsintensitet/refleksjonsgrad fra kameraer som
  kun støtter RGB-kameraer).

Kjør på nytt med `--verbose` og sjekk backend-loggen for linjer med `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
som forklarer utelatelser per kamera som ellers ikke kommer med i utdataene fra CLI.

En bevisst kjøring kun med metadata – alle produkter slått av og ingen `--indices` – er fortsatt en
**suksess**, fordi et tomt bildeutdata er det riktige resultatet i dette tilfellet.

Det samme gjelder en **kjøring kun med lyssensor**: en mappe med `.daq`-opptak har per definisjon ingen bilder å eksportere,
og kjøringen vurderes ut fra den kalibrerte `.daq` / `.csv` den skrev i stedet.

---

## `chloros-cli login`

Autentiser denne maskinen med en Chloros+-skykonto. Påloggingsopplysningene lagres sikkert i `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Eksempler

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (ved å fjerne eller duplisere deler av passordet). Ved en 401-feil prøver CLI automatisk på nytt med `$$` lagt til på nytt, deretter med enhalvdel av passordet som ikke er duplisert; hvis et nytt forsøk lykkes, logger den deg inn og viser den riktige syntaksen med enkelt anførselstegn som skal brukes neste gang.

> **Bruk uten grafisk grensesnitt/med skript: ingen bufret økt betyr en interaktiv kommandoprompt, ikke en rask feilmelding.** Enhver kommando som starter en backend (`process`, `status`, `export-status`, `time-sync`, …) som kjøres uten en bufret lisens/sesjon, havner i en interaktiv `Email:` / `Password:`-prompt på stdin før den fortsetter. En uovervåket jobb uten bufret sesjon vil derfor henge seg opp i påvente av inndata — kjør `chloros-cli login EMAIL PASSWORD` én gang per maskin før du planlegger headless-arbeid.

---

## `chloros-cli logout`

Tømmer den bufrede økten og tvinger frem en ny pålogging ved neste oppkall.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Viser gjeldende lisensnivå (Iron/Copper/Bronze/Silver/Gold), autentisert bruker og antall enhetsbindinger.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Henter sanntidsstatus for Thread-4-eksporten. Kan trygt påkalles **under** en `process`-kjøring fra et annet skall.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Angi visningsspråket for CLI (38 støttes, inkludert CJK, RTL, og Indic). Faller elegant tilbake til engelsk på eldre konsoller som ikke kan gjengi skriptet.

```
chloros-cli language [LANG_CODE] [--list]
```

### Eksempler

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Kommandoer for prosjektmappe

Disse styrer standardplasseringen for prosjektmappen (delt med GUI).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

---

## `chloros-cli update`

Linux / Kun Jetson. Sjekker `version_url` fra `/etc/chloros/update.conf` og tilbyr å laste ned og installere den tilsvarende `.deb`.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

På Linux/Jetson utfører CLI også en **automatisk oppdateringssjekk ved hver oppstart** (blokkerer ikke, forsinker aldri kommandoen): den leser `/etc/chloros/update.conf`, lagrer resultatet i hurtigbufferen i 1 time i `~/.chloros/update_cache.json` og skriver ut `Update available: vX.Y.Z / Run: chloros-cli update` når en nyere versjon finnes. Hoppes over uten varsel ved eventuelle feil og på Windows.

---

## `chloros-cli selftest`

Kjører en 7-trinns røykprøve: versjon, porttilgjengelighet, oppstart av backend, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), tilstedeværelse av støydempingsmodell, CUDA+støydempingsklarhet.

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

PTP-grandmaster-status og -kontroll. Chloros-verten kjører PTP-grandmasteren; LATTICE-kameraer og DAQ-E-enheter fungerer som slaver for denne for tidsstempler på tvers av enheter.

| Underkommando | Beskrivelse |
| --- | --- |
| `status` | Vis grandmaster-tilstand, BMCA-prioriteter, klokkeidentitet. |
| `peers` | Vis liste over slaver som er registrert via Delay_Req (kameraer + DAQ-E-sensorer). |
| `cameras` | PTP-tilstand per kamera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Start grandmaster-prosessen på nytt. |
| `set-priority --priority1 N --priority2 N` | Overstyr BMCA-prioriteringer. |

### Eksempler

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

LATTICE-kamerakontroll. Hver underkommando går gjennom Chloros-backenden; backenden eier kamerapoolen poolen, slik at påfølgende CLI-kall gjenbruker det samme åpne håndtaket.

### Vanlige alternativer (felles for de fleste underkommandoer)

| Flagg | Beskrivelse |
| --- | --- |
| `-d, --device N` | Kameraindeks (standard: 0). |
| `-s, --serial SN` | Spesifikk serienummer; overstyrer `--device`. |
| `--serials SN1,SN2,…` | Kommaseparerte serienumre for drift med flere kameraer. |
| `--all` | Utfør på alle oppdagede kameraer. |
| `--exposure US` | Eksponeringstidtid i mikrosekunder. |
| `--gain DB` | Forsterkning i dB. |
| `--pixel-format FMT` | f.g. `BayerRG8`, `BayerRG12`. |
| `--width N` / `--height N` | Bildedimensjoner. |
| `--preset {default,high_quality,high_speed,triggered}` | Bruk en forhåndsinnstilling. Alle kjører fritt, bortsett fra `triggered`, som aktiverer kameraet ved en maskinvarekant på linje 2 — uten noe som driver den linjen, vil det vente i all evighet i stedet for å ta bilder. |
| `-o, --output DIR` | Utdatakatalog (standard: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | GVSP-pakkestørrelse. `auto` kjører ICMP+GVSP-sonder; `jumbo` = 9000; `standard` = 1500. |

### LATTICE-kamera – arbeidsflyt for første tilkobling

```bash
# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Auto-shrink frame size to fit the wire
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Referanse for underkommandoer

#### Oppdagelse og informasjon

| Underkommando | Formål |
| --- | --- |
| `lattice info` | Vis liste over tilkoblede kameraer (leverandør, modell, serienummer, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analyser vertsystemet for optimal kamerakonfigurasjon. `--no-discover` hopper over kamerasøk (raskere, analyse kun av nettverkskort). |
| `lattice network [--fix] [--estimate] [--cameras N]` | Kontroller/rett opp nettverkskortinnstillinger; estimer båndbredde/FPS. |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | Nettverkskapasitet i backend med stabilt skjema + anbefaling av matrise (returnerer `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `auto_capped_fps` beholder den forespurte oppløsningen, men setter en øvre grense for mål-fps — les `recommended.recommended_target_fps` og send den som tilkoblingsmål; behandle dette som en suksess, ikke en feil. |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | «Hva-hvis»-analyse uten å åpne kameraer. **`--n-active` er det totale antallet kameraer på nettverket, ikke bare denne matrisen&#x27;**— øk den når frittstående kameraer strømmer samtidig, eller når nettverksbudsjettet beregnes ut fra et behov som undervurderer antallet (standard: `len(--models)`). Skriver alltid ut det samlede `Wire budget:` (krevd MB/s mot kollisjonssikkert tak) og `Max cameras:`-linjene, og markerer `** OVER-SUBSCRIBED**` når matrisen overbelaster nettverket — se [Matrise-fps og burst-modell](#array-fps--burst-model). |
| `lattice gpu` | Vis GPU-status. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Sjekk eller oppdater kameraets fastvare. Lokalt valg av `.fwa` er fastlåst: filen i `firmware/<MODEL_PREFIX>/` som samsvarer med buildens `MIN_FIRMWARE_VERSION`, blir flashet når den er til stede (kun høyeste versjon som reserve), så et nyere leverandørbilde som er lagret på disken, er inaktivt inntil den festede versjonen oppdateres — bevisst nyere utgivelser når enhetene via det signerte AWS-manifestet, noe som foretrekkes når det er nyere. |
| `lattice presets [--apply NAME]` | Vis eller bruk kamerainnstillinger. |
| `lattice status` | Live kamerastatus. |

#### Opptak

| Underkommando | Formål |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | Enkeltbilde. **Lagrer alle eksporttyper som standard** (`--processing all`); se [Eksportnivåer for opptak](#capture-export-levels-the-all-default). `--levels` lagrer en eksplisitt delmengde (overstyrer `--processing`); `--force-daq` skriver den tildelte DAQ-avlesningen som en `.daq`-sidecar selv ved en ren-opptak. `--jpeg-quality` = JPEG kvalitet 1–100 (standard 95). |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | Strøm til disk til Ctrl+C. |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Nettleser-basert live MJPEG-forhåndsvisning. `--ae-damping` angir demping av automatisk eksponering (0,4–100). |

#### Sensorinnstilling

| Underkommando | Formål |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | Les/skriv til hvilken som helst GenICam-node. |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | Eksponering og AE. |
| `lattice gain [--auto] [--off] [--set DB]` | Forsterkning og automatisk forsterkning. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | Sensor-ROI og binning. |
| `lattice format [--set FMT] [--list]` | Pikselformat. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Maskinvare-/programvaretrigger. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (ingen flagg = engangs-hvitbalanse) | Hvitbalanseoperasjoner. Kun RGB/Bayer-kameraer; ingen-operasjon (hoppes over) på mono M3M. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | RGB-fargesignalbehandling. `natural` (standard) er den enkle live-bearbeidingen; `enhanced` legger til defringe + vibrance + CLAHE lokal kontrast for det fullstendige «hub-parity»-utseendet til ca. 2× bearbeidingskostnaden per bilde, så en lavere **live** bildefrekvens — lagrede opptak får uansett alltid full etterbehandling. RGB/Kun Bayer-kameraer; hoppes over på mono M3M. |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | Vis metning/kontrast (RGB-filterkameraer). Hoppes over på mono M3M. |
| `lattice filter [--set NAME] [--list]` | Angi kameraets filtermodell (`RGN-IMX265`, `OCN`, `NGB`, …). |
| `lattice power [--sleep]` | Måle strøm-/termiske noder; slå på lavstrømshvilemodus. |

#### Kalibrering og sensorer

| Underkommando | Formål |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibrer fra et refleksjonsmål. |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | Kommandoer for innebygd sensor for nedadrettet lys. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Bruk vignettkorreksjon på eksisterende bilder. |

#### Flere kameraer (midlertidige økter)

| Underkommando | Formål |
| --- | --- |
| `lattice multi-info` | Vis alle kameraer med synkroniseringsroller. |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | Ett synkronisert bilde fra hvert kamera. Lagrer **alle eksporttyper som standard**når en permanent matrise er tilkoblet; den midlertidige fallback-løsningen uten matrise er**kun debayert** (kjør `array-connect` først for resten). |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | Strøm synkroniserte bilder (midlertidig). |
| `lattice multi-test [--count N]` | Test av GPIO-synkroniseringstid. |
| `lattice multi-detect [--line LINE] [--json]` | Automatisk gjenkjenning av GPIO-master/slave-kabling. |

#### Justering

| Underkommando | Formål |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — pluss detektor-/matcher-knapper `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, RANSAC-knapper `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, kombinasjon av flere bilder `[--averaging mean\|median\|inlier_weighted]`, geometriske begrensninger `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, romlig begrensning `[--roi X0,Y0,X1,Y1] [--mask PATH]` og overstyringer per slave `[--per-cam-override SN:KEY=VALUE]` (gjentakelig) | Beregn justeringsprofil fra live-kameraer. `--prefilter` bruker som standard `gradient` (kantkart; samsvarer med GUI/matrisejustereren — kantene forblir intakte på tvers av spektralbåndene). `--matcher flann` gir best resultat ved over ~5000 treff; `--averaging median` er robust mot én feilopptak, `inlier_weighted` vektlegger etter antall treff; `--lock-scale` projiserer til nærmeste rotasjon (ingen skalering), `--lock-axis` setter én translasjonskomponent til null; `--mask` gjelder for alle kameraer (bruk `--per-cam-override` for innstillinger per kamera, f.eks. `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` nekter å lagre en kalibrering der RMS-verdien for reprojeksjonen overskrider grensen. |
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | Ta opp ett justert flerbåndsbild. `--bit-depth` bruker som standard innstillingen som samsvarer med kameraet; `--no-crop` beholder hele bildet (fyller ut med svart); `--interpolation` (standard `linear`) og `--border-mode`/`--border-value` (standard `constant`/0) styrer CPU-warp — GPU-banen er uansett bilineær. |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | Strømjusterte flerbåndsrammer (samme warp-regulatorer som `align-apply`). |
| `lattice align-info --profile PATH [--json]` | Vis profildetaljer. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Endre lagrekkefølge. |

#### Indeks / Vegetasjonsmatematikk

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

Fullt flaggsett: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND` (kan gjentas), `--capture-level raw|debayered|radiance|reflectance|unknown` (overstyrer opptaknivået som er registrert i kilden TIFF; standard: leses fra TIFF-metadata), `--output PATH`, `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Med `--live` gjelder også justeringsknappene: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`.

> **`--channel`-symbolene er store og små bokstaver.** Symbolsiden må stemme nøyaktig overens med forhåndsinnstillingens kanalnavn (forhåndsinnstillingene bruker små bokstaver, f.eks. NDVI = `red`, `nir` — sjekk `--list-presets`), og båndsiden må samsvare med et båndnavn i den tilpassede stakken (eller være en 0-basert båndindeks i offline-modus). `--channel red=Red_660 --channel nir=NIR_850` fungerer; `--channel RED=660` mislykkes med en `channel_map missing entries`-feil.

#### Vedvarende tilkoblinger (Smart-Prep, GUI-ekvivalent flyt)

Disse kommandoene holder kameraene åpne i backend-poolen på tvers av CLI-kall.

| Underkommando | Formål |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Legg til ett kamera i puljen (enkeltkamera, ikke array). |
| `lattice cam-disconnect [--serial SN] [--all]` | Frigjør. |
| `lattice cam-list` | Viser kameraene i puljen. |
| **`lattice array-connect`**|**Kobler til en vedvarende synkronisert matrise (DET anbefalte inngangspunktet).** Kjører hele Smart-Prep-flyten i GUI. |
| `lattice array-disconnect [--array-id ID] [--all]` | Frigjør et array. |
| `lattice array-list` | Vis tilkoblede matriser. |
| `lattice array-status [--array-id ID]` | Live fps, PTP, siste feil. |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Én synkronisert opptak fra live-matrisen — Enkelt / Kontinuerlig / Intervall / Raskest. **Standardinnstillingen er `all`** (én fil per relevant eksporttype per kamera). Kameraer som er hoppet over (f.eks. RGB ekskludert fra radianse/reflektans) rapporteres med `Skipped: SN:<serial> (<reason>)`; DAQ-målingen som brukes for refleksjonsgrad, lagres ved siden av og rapporteres med `DAQ: <path>`. Se [Opptakmoduser, opptakerenheter og offline-etterbehandling](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | Ta opp den kombinerte indeksvisningen i sanntid til video/GIF (overvåkingskvalitet; krever at den kombinerte strømmen er åpen). |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | Rå-Bayer-serie med høy bildefrekvens (analyse-kvalitet; etterbehandle offline). |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Etterbehandle en lagret rå-serie til kalibrert(e) video(er). |

##### `array-connect`-alternativer

| Flagg | Standard | Beskrivelse |
| --- | --- | --- |
| `--serials SN1,SN2,…` | automatisk oppdagelse av alle LATTICE-kameraer (krever ≥2) | Det første serienummeret er MASTER. Når dette utelates, filtreres oppdagelsen til LATTICE (`TRI032*`)-modeller, og alle kobles til. |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO-synkroniseringslinje. |
| `--target-fps F` | auto | Master-utløserens avfyringsfrekvens. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Overstyr nivåvelgeren. |
| `--wire-ceiling-mbps MB_PER_S` | auto-detected | **Vertens vedvarende båndbredde, i MB/s — tallet som hele array-tildelingen avhenger av.** Senk det når arrayet rapporterer GVSP-korrupte rammer: den automatiske verdien er avledet fra nettverkskortets oppgitte koblingshastighet, som overvurderer USB-adaptere, smale PCIe-baner og travle delte nettverk. Lagres i prosjektets array-opptaksblokk, så en gjenåpning / CLI / SDK gjenoppretter den. Se [Array-tilstand](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | auto | Maskinvarebinning. |
| `--no-recommend` | av | Hopp over trinnet med nettverksanalyse. |
| `--no-ptp` | av | Deaktiver PTP (tidsstempler på tvers av kameraer er da **ikke** sammenlignbare). |

### Smart-AE / Smart-Capture

LATTICE-arrayer kjører kontinuerlig AE i bakgrunnen så snart de er tilkoblet, men det tar et øyeblikk før en nyinnstilt scene konvergerer. `array-capture --smart` er den **praktiske løsningen**: den venter til AE har stabilisert seg på alle kameraene i arrayet, og utløser deretter opptaket. Bruk den når du bytter scene midt i en økt.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Standardinnstillingen for stabiliseringspolitikken er konservativ: 5 sekunders tidsavbrudd, 1,5 sekunders stabilitetsvindu, ±5 % toleranse for eksponeringsspredning. Juster via SDK (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) hvis du trenger en annen oppførsel fra automatiseringen.

### Eksportnivåer for opptak (standardinnstillingen for `all`)

Fra og med denne utgivelsen er `lattice capture`, `lattice multi-capture` og `lattice array-capture` **er som standard satt til `--processing all`** — én lagret fil per eksporttype som gjelder for hvert kamera, i tråd med «Capture All»-funksjonen i brukergrensesnittet. Nivåene er:

| Nivå | Utdata | Gjelder for |
| --- | --- | --- |
| `raw` | Enkanals Bayer (monokameraer: det ene båndet) direkte fra sensoren. | Alle kameraer. |
| `debayered` | 3-kanals BGR-demosaik (monokameraer: 1-kanals gråtoner). | Alle kameraer. |
| `radiance` | float32 W/m²/sr/nm via hele den radiometriske kjeden. | Multispektrale (M3C/M3M) kun — **hoppes over for RGB-filterkameraer**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), Pix4D-klar. | Kun multispektral, og **kun når en DAQ er tilknyttet + kameraet er kalibrert**; ellers hoppes over. |
| `preview` / `display` | Full GUI-forhåndsvisningskjede (CCM + WB + gamma i henhold til kameraets profil). `lattice capture` navngir dette `preview`; `array-capture`/`multi-capture` bruker `display`. | Alle kameraer. |

Angi et enkelt nivå for å lagre bare det ene (`--processing debayered`). Når du ber om `all`, hoppes nivåer som ikke gjelder for et gitt kam over (og rapporteres), uten at det oppstår feil — et ikke-tilkoblet eller ikke-kalibrert kamera får fortsatt `raw` / `debayered` / `preview`.

For alle reflektansrammer skrives den faktisk brukte DAQ-nedstrålingsmålingen til en **`.daq`**-sidecar ved siden av bildene (slik at opptaket kan behandles på nytt senere) og rapporteres på en `DAQ:`-linje.

### Slik ser en opptakmappe ut

Hver eksporttype havner i sin **egen undermappe** under `-o`, slik at et opptak på flere nivåer aldri blander typer:

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

`<ts>` er tidsstempelet for opptaket og `<serial>` er kameraets serienummer, slik at en synkronisert gruppe deler et
felles tidsstempel på tvers av kameraene. **Legg merke til den ene asymmetriske:** `display`-nivået lagres i en mappe
med navnet `preview/`, mens filene selv beholder `_display` i navnet — mappe og suffiks skiller seg
kun for dette nivået. Ukjente nivåer faller tilbake til en mappe med sitt eget navn, og hvis undermappen
ikke kan opprettes, skrives filen til utdatarotkatalogen i stedet for å gå tapt.

**Nybehandling av en «captures»-mappe:**pek `chloros-cli process` mot**«captures»-rotmappen**
(`output/`). `process` importerer normalt bare den mappen du angir, men når den mappen ikke inneholder noen
bilder og har undermapper, går den automatisk nedover i hierarkiet — slik at undermappene på rotnivå og selve
roten `.daq` hentes inn på én gang. Hvert nivå i en opptak importeres som ett enkelt bilde, mens
de andre nivåene er tilgjengelige som moduser, i stedet for ett bilde per nivå.

Å navngi en **undermappe på et nivå** direkte (f.eks. `output/raw/`) fungerer også. Da blir rotmappen
`.daq` stående igjen, så kopier eller pek DAQ-avlesningen inn ved siden av når du beregner et radiometrisk
produkt på nytt fra `raw/` – ellers har tidsstempelavstemningen ingenting å sammenligne seg med.

**Behandlingen starter alltid fra `raw`.** Innenfor hver opptak er råbildet kilden til behandlingsrørledningen;
`debayered`, `radiance`, `reflectance` og `preview` følger med som visningsmoduser, men føres aldri
tilbake gjennom prosesskjeden. Å behandle et avledet produkt på nytt ville innebære å påføre vignett, CCM og
strålingsberegninger som allerede er bakt inn i pikslene, så Chloros avviser i stedet for å
behandle det to ganger. To konsekvenser det er verdt å vite:

- Renderingene `index/` og `composite/` blir **aldri** behandlet. De er utdata, ikke opptak —
  en NDVI LUT-rendering har ingen meningsfull radiance-tolkning.
- En opptak-mappe eksportert **uten** `raw` (f.eks. `array-capture --processing reflectance`) har
  ingen gyldig pipeline-kilde. Disse opptakene importeres og vises normalt, men `process` hopper
  over dem og melder følgende:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Hvis du virkelig trenger å sende et avledet produkt videre – en hub-økt som er tatt opp med
  `demosaic` aktivert, eller en eldre mappe — tvinger `--input-level {raw,debayered,processed}` inngangspunktet
  og overstyrer hoppingen. Dette flagget er den bevisste nødutgangen; `auto` (standardinnstillingen)
  behandler aldri en opptak som mangler rådata.

### Hoppede opptak i blandede filteroppsett

Når du blander RGB og multispektrale kameraer i ett oppsett, lagrer `array-capture --processing radiance` (eller `reflectance`) de multispektrale bildene og **hopper over** RGB-kameraene — strålingsintensitet per Bayer-piksel er ikke relevant for en bredbåndssensor. CLI skriver ut hver lagrede fil (med eksportnivå), hver `.daq` som er skrevet, og hvert hopp eksplisitt, så antall filer er ikke overraskende:

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Tokenene for hoppårsaker følger mønsteret `<level>-not-applicable-to-rgb-cam`. Refleksjonsgraden kan også hoppes over med `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, og med `dls-uncalibrated-band-<nm>` når båndet hovedsakelig ligger utenfor det radiometrisk kalibrerte området til DAQ-lyssensoren (~374–974 nm) — blant de tilgjengelige SKU-ene er det kun F988, som støtter refleksjonspanel-arbeidsflyten.

Bruk `--processing debayered` (eller `display`) for å inkludere alle kameraer uavhengig av filtertype, eller standardinnstillingen `all` for å hente alle relevante nivåer per kamera i ett opptak.

---

## Opptaksmoduser, opptakere og offline-etterbehandling

Disse fungerer alle på et **permanent matrise** (kjør `array-connect` først). De speiler opptakspanelet i brukergrensesnittet.

### `array-capture`-moduser

`array-capture` er en enkelt kommando med fire lukkermoduser pluss et sett eksportbrytere:

| Modus | Flagg | Oppførsel |
| --- | --- | --- |
| **Enkelt** *(standard)* | (ingen) | Én synkronisert opptaksgruppe, deretter avsluttes. |
| **Kontinuerlig** | `--continuous` | Påfølgende gjennomganger inntil `Ctrl+C`, `--count N` eller `--duration S`. |
| **Intervall** | `--interval S` | Én gjennomgang hvert `S` sekund (målt fra starten av hver gjennomgang), samme grenser. |
| **Raskest** | `--fastest` | Kun rådata + den tildelte DAQ-avlesningen + den kombinerte-indekssammensetningen; hopper over beregningene for stråling/refleksjon/visning slik at rammen vises raskt. Impliserer `--processing raw --force-daq`. Behandle de lagrede `.daq`-dataene på nytt til kalibrerte produkter senere. |

Eksportalternativer (kan kombineres med hvilken som helst modus; alle deler GUI/SDK-endepunkt):

| Flagg | Effekt |
| --- | --- |
| `--processing LEVEL` | Enkelt eksportnivå, eller `all` (standard). |
| `--levels L1,L2,…` | Eksplisitt delmengde av eksporttyper (f.eks. `raw,radiance,reflectance`); **overstyrer `--processing`**. |
| `--aligned` / `--no-aligned` | Warp alle medlemmers ikke-rå eksport til matrisens [justeringsprofil](#justering) (samregistrert). Rådata forblir uforvrengt, men bærer transformasjonen i metadataene. Faller tilbake til ujustert (med en advarsel) hvis matrisen ikke har noen profil. |
| `--index` / `--no-index` | Lagre / hopp over vegetasjonsindeks-overlegget per kamera der et slikt er konfigurert. Standard: gjengi det. |
| `--force-daq` | Lagre den tildelte DAQ/DLS-avlesningen som en `.daq`-sidecar selv når ingen valgt nivå trenger det (f.eks. et-opptak), slik at bildene kan bearbeides på nytt til reflektans/indeks offline. |
| `--smart` | Vent til AE har stabilisert seg på alle kameraer før utløsning (se [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | TIFF pikselkomprimering. `deflate` (standard) = tapsfri zlib L1 + horisontal prediktor, ~4,1 MB per bilde i full oppløsning; `none` = ukomprimert, ~5 ganger raskere skriving med ~6,3 MB per bilde — brukes for maksimal vedvarende hastighet når disken tillater det. Begge er tapsfrie og leses identisk ved import. |

> **Enkelt-skriving TIFF + modellen for vedvarende hastighet.**Opptakene skrives i**én**TIFF-filomgang som inneholder piksler + XMP + IFD0 Merke/Modell (målt på full oppløsning Mono12: 36 ms komprimert / 6,5 ms ukomprimert, mot ~148 ms for den gamle metoden med først skriving og deretter omskriving med ExifTool); det eneste gjenværende ExifTool-arbeidet (finpussing av EXIF-under-IFD) kjører på en asynkron bakgrunnsarbeider, og et bilde er ferdig og klart for import selv om dette aldri kjører. Merk at DEFLATE-komprimering holder Python GIL, så komprimerte skrivninger**parallelliseres ikke**på tvers av skrivertrådene per kamera — vedvarende opptak med 8 kameraer i full oppløsning ved sensorhastighet (~10,4 fps) krever `--compression none`**og** en disk i NVMe-klassen (~500 MB/s vedvarende skriving). Den samme innstillingen er tilgjengelig som `compression` på `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — kombinert indeks for video/GIF (overvåkingskvalitet)

Tar opp det som vises i **live-visningen med kombinert indeks** til en `.avi` (og eventuelt en `.gif`). Siden den henter fra live-komposittbildet, må den kombinerte strømmen være åpen (f.eks. matrisen forhåndsvises i brukergrensesnittet) for at bildene skal komme inn. Den sjekker fremdriften hvert 2. sekund og stopper på `--duration`, `Ctrl+C` eller når opptakeren avsluttes automatisk.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| Flagg | Standard | Beskrivelse |
| --- | --- | --- |
| `--array-id ID` | kun matrise | Målmatrise (utelat hvis bare én er tilkoblet). |
| `-o, --output DIR` | `output` | Utdatakatalog (backend-lokal). |
| `--fps F` | `10` | Opptak bildefrekvens. |
| `--duration S` | til Ctrl+C | Stopp automatisk etter `S` sekunder. |
| `--gif` | av | Skriv også en animert GIF. |
| `--gif-only` | av | Skriv bare en GIF (ingen `.avi`). |

### `array-burst` — rå-Bayer-serieopptak med høy bildefrekvens (analyse-kvalitet)

Leser direkte fra synkroniseringsgruppens buffer i opptakssløyfen — **ingen kalibreringskjede, ingen exiftool, ingen live-visning nødvendig** — slik at den kjører med kameraets fulle opptakshastighet. Skriver råbilder + etramme-manifest + én `.daq` per distinkt DLS-avlesning under `<output>/bursts/<base>/`. Behandle på nytt offline (neste kommando), eller send `--build` for å gjøre det umiddelbart ved stopp.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Flag | Standard | Beskrivelse |
| --- | --- | --- |
| `--array-id ID` | kun matrise | Målmatrise. |
| `-o, --output DIR` | `output` | Utdatakatalog (burst havner i `<DIR>/bursts/<base>/`). |
| `--duration S` | til Ctrl+C | Automatisk stopp etter `S` sekunder. |
| `--max-frames N` | ubegrenset | Automatisk stopp etter `N` råbilder. |
| `--build` | av | Etter stopp, behandle serien umiddelbart på nytt (samme som `array-build-video`). |
| `--products …` | `combined:index` | Med `--build`: hvilke videoer som skal bygges (se nedenfor). |
| `--fps F` | `10` | Med `--build`: bildefrekvens (fps) for utgangsvideoen. |
| `--save-tiffs` | av | Med `--build`: lagre også kalibrerte TIFF-filer for hvert bilde. |
| `--gif` | av | Med `--build`: lagre også animerte GIF-filer. |

### `array-build-video` — offline-bearbeiding av en lagret bildeserie

Tidsjusterer hvert råbilde til nærmeste lagrede `.daq`-avlesning og sender det gjennom **den samme strålings-/refleksjons-/ indekskjede som importpipeline**, og renderer ett eller flere videoer.

`--products` er en kommaseparert liste over `kind:level`-elementer, der `kind` ∈ `per_cam` | `combined` og `level` ∈ `radiance` | `reflectance` | `index`. En ren `level` (uten `kind:`) bruker `per_cam` som standard. Standard er `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Flagg | Standard | Beskrivelse |
| --- | --- | --- |
| `--burst-dir DIR` | (påkrevd) | Sti til burst-mappen (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | `kind:level`-liste, som ovenfor. |
| `--fps F` | `10` | Bildfrekvens for utgangsvideo. |
| `--save-tiffs` | av | Lagre også kalibrerte TIFF-filer for hvert enkelt bilde sammen med videoene. |
| `--gif` | av | Skriv også animerte GIF-filer. |

> **Velg riktig opptaker.** `array-record` er *overvåkingsklasse* — den tar opp det sammensatte livebildet slik det vises og krever at strømmen er åpen. `array-burst` → `array-build-video` er *analyseklasse* — den lagrer rå sensordata med full hastighet og rekonstruerer kalibrerte strålings-/refleksjons-/indeksvideoer i etterkant, uten at det kreves livevisning.

### Mono (M3M) enkeltbåndskameraer

**M3M**-serien er den monokrome søskenmodellen til Bayer**M3C**: ett smalbåndsinterferensfilter per kamera (`M3M-<lens>-F<wavelength>`, f.eks. `M3M-L87-F685`), slik at sensoren leverer et**enkelt gråtonebånd** uten Bayer-mosaikk. Det er ingenting å demosaikere, ingen kryssforstyrrelser mellom kanalene å skille ut, og ingen hvitbalanse å stille inn — hele fargebehandlingsprosessen for RGB-skjermen gjelder ganske enkelt ikke.

Hva dette betyr for CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**oppdager et monokamera og**hopper over med en enlinjemelding** i stedet for å påføre meningsløse innstillinger. De fungerer fortsatt normalt med et RGB/Bayer M3C-kamera i samme økt.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** fungerer fortsatt — stråling og refleksjon er *per-bånd* radiometriske kart og er perfekt definert for ett bånd. Monobilder har en **identitets**-sensorresponsmatrise (ingen 3×3-separering), så bildet går uendret gjennom kalibreringsberegningene.
- **Et enkelt monokamera kan ikke produsere et vegetasjonsindeks.**NDVI/NDRE/osv. trenger minst to bånd (f.eks. Red + NIR). For å få et indeks fra monokameraer, rett**flere** M3M-kameraer med forskjellige bølgelengder, samle dem i én multibåndsstabel og beregne indeksen *for den*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel`-symbolene må samsvare **nøyaktig** (skiller mellom store og små bokstaver; NDVI er små bokstaver `red`, `nir` — se `--list-presets`), og båndsidenavnene må peke på et bånd i den justerte stakken (offline-modus godtar også 0-baserte båndindekser, f.eks. `--channel red=0 --channel nir=1`).

Det som skiller dem fra hverandre i hele stakken er tokenet `M3M` i modellstrengen (det forekommer aldri i en `M3C`-streng), som vises i GUI/SDK som `is_mono`.

---

## Konfigurering og justering av verts-NIC (LATTICE-oppsett)

LATTICE-kameraer strømmer GVSP via vertens Ethernet-adapter, så for oppsett med flere kameraer er adapterens **driver**og**mottaksringstørrelse** like viktig som koblingshastigheten. Feil innstillinger vises som en `FRAMES WILL DROP` / `Reduce ROI to enable`-port i panelet «Array Settings» (og i `lattice network-analysis` / SDKs `analyze_array_network()`), selv når kameraene i seg selv fungerer som de skal.

### USB 10GbE-adaptere — Realtek RTL8157 («Realtek USB 10GbE Family Controller»)

| Element | Nødvendig verdi | Hvorfor det er viktig |
| --- | --- | --- |
| **Driverversjon**|**≥ v10.67 (jan. 2026)**, INF `rtump64x64sta.inf` | Den eldre**2016**-driveren (v10.65, `rtump64x64.inf`) håndterer ikke strømavbrudd og feilmeldinger med**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**ved avslutning/omstart/hvilemodus. Overgangen henger seg opp (~5 minutters tidsavbrudd), brukeren må slå av maskinen manuelt, og gjentatte uregelmessige avslutninger**ødelegger WMI-repositoriet**(PowerShell/verktøy begynner å svikte med `Invalid class`) og**blokkerer USB-stakken** ved neste oppstart (nettverkskortet aktiveres ikke; USB-stasjoner slutter å bli gjenkjent). Oppdater fra realtek.com (eller dongle-leverandøren) før du stoler på rene omstarter. |
| **Mottaksbuffere**— nøkkelord `ReceiveBufferLen` |**256**(maks. i driveren) | Nettverkskortets RX-ring. Driverens standardverdi på**32**etterlater bare ca. 0,26 MB brukbar ring – altfor lite for en multikamera-burst – så array-panelet rapporterer `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` og blokkerer tilkoblinger. Ved**256**er ringen stor (**~13,5 MB målt på laboratoriets 10GbE-vert**), noe som gir RX-rørledningen reelt spillerom for GVSP-burst med flere kameraer. (Hvorvidt en gitt konfigurasjon faktisk *kobles til*, avgjøres av to kontroller – den **drain-aware**-admittanskontrollen og kontrollen for**aggregat over-subscription** – ikke en ren sammenligning mellom burst og ring; se [Array fps &amp; burst-modell](#array-fps--burst-model).) |
| **Mottak-URB-er**— nøkkelord `PendingReceives` |**64** (maks.) | USB-forespørselsblokker under overføring; økes sammen med mottaksbuffere for å absorbere burst-trafikk. |
| **Jumbo-ramme** — nøkkelord `*JumboPacket` | **9014** | Nødvendig for 9000-byte GVSP-pakker (6× færre pakker/ramme enn 1500). |

> ⚠️ **En oppdatering av nettverkskortdriveren TILBAKESTILLER disse avanserte egenskapene til standardinnstillingene.**Etter å ha oppdatert eller byttet ut nettverkskortdriveren, må du**angir på nytt** `ReceiveBufferLen=256` og `PendingReceives=64`, ellers vil array-panelet gå i gate-modus igjen, selv om «ingenting er endret i maskinvaren». Dette er den vanligste årsaken til at et utstyr som tidligere fungerte, plutselig nekter å koble seg til.

Bruk fra en **administratorrettighetsbasert** PowerShell (erstatt med navnet på nettverkskortet ditt, f.eks. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` gjelder USB 10GbE-kort.** Den oppdager nå adaptertypen og stiller inn riktig nøkkelord for mottaksring: `*ReceiveBuffers`→2048 for PCIe-nettverkskort (Intel I219, osv.), eller `ReceiveBufferLen`→256 + `PendingReceives`→64 for Realtek **USB** 10GbE-kontrolleren (som ikke eksponerer `*ReceiveBuffers`). Målverdiene er begrenset til den maksimale verdien som hver driver rapporterer (`NumericParameterMaxValue`), slik at det aldri skrives en verdi utenfor området. Kjør det fra en **privilegert** terminal; som med alle registerbaserte justeringer, trer endringen i kraft etter en omstart av nettverkskortet eller en omstart av systemet. De manuelle `Set-NetAdapterAdvancedProperty`-kommandoene ovenfor er fortsatt et godt alternativ – de trer i kraft umiddelbart (binder adapteren på nytt) uten omstart.

### Nettverksgrunnleggende (alle LATTICE-koblinger)

- **Adressering:** link-local `169.254.0.0/16` (GigE Vision LLA). Verten tar en statisk `169.254.x.x/16`; kameraer og DAQ-E tildeler seg selv adresser i samme område. Ingen DHCP/gateway nødvendig.
- **Pakke størrelse:**foretrekker jumbo (9000), men la den automatiske sonden finne den – den måler på nytt ved hver tilkobling og ser allerede forbi kameraets ICMP-grense på 1500 byte via en GVSP-sonde, slik at den ender opp med jumbo uansett hvor kabelen faktisk klarer å overføre den. Bruk pin med `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` kun når du vet bedre enn sonden, og foretrekker per kommando fremfor permanent: en pin-innstilling hopper over sonden, så hvis banen faktisk ikke kan bære 9000, vil**hver** innfanging gå ut på tid med `SC_ERR_TIMEOUT -1011` (se [Miljøvariabler](#environment-variables)).
- **RX-ringen skaleres med `ReceiveBufferLen`:**ved standardverdien `32` er den brukbare ringen ~0,26 MB (for liten for enhver multikamera-burst); ved maksimal `256` er den stor (~13,5 MB målt på laboratorieverten med 10 GbE), noe som gir reell kapasitetsreserve. Hvorvidt en konfigurasjon kobles til, avgjøres deretter av den dreneringsbevisste adgangskontrollen**og** den samlede overskrivningskontrollen nedenfor — ikke en rå sammenligning mellom burst og ring.

### Array-fps og burst-modell

Slik leser du panelet «Array Settings» (og `lattice analyze-array` / SDKs `analyze_array_network`):

- **Burst summeres per-kamera i hvert kameras faktiske pikselformat.**Mono**M3M**-kameraer streamer**Mono12 (2 B/px)**;**M3C**Bayer-kameraer streamer 8- eller 12-bit (TRI032S sender automatisk ut BayerRG12 selv når BayerRG8 er forespurt). Et bilde i full oppløsning fra 4 kameraer er altså**~12,6 MB hvis alle er 8-bits, men ~25 MB med tre 12-bits mono-kameraer**. Projeksjonen avleser hvert kameras format ut fra modellen (identitetsbuffer), slik at dataoverføringen samsvarer med hva kabelen faktisk overfører – ikke en standardantagelse om BayerRG8.
- **En USB-Ethernet-adapter har en øvre grense på 200 MB/s, uavhengig av hva som står på typeskiltet.** Effektivitetstabellen som omregner en koblingshastighet til et vedvarende tall, er avledet fra PCIe; et USB-nettverkskort oppgir sin *Ethernet*-koblingshastighet, men er begrenset av USB-bussen og driveren. En USB 10GbE-dongle oppnådde tidligere ~1063 MB/s «vedvarende» – et tall som aldri ble verifisert— og den resulterende hastighetsreguleringen ødela 6–18 % av rammene, samtidig som den fortsatt rapporterte en god mål-fps. USB-tilkoblede nettverkskort er nå begrenset til **200 MB/s** som et absolutt tall (grensen er bussen, så den skalerer ikke i henhold til spesifikasjonene; en USB 1 GbE-adapter oppnår ~80 MB/s og påvirkes ikke). `wire_ceiling_source` i kapasitetsoppføringen sier dette med ord, og `nic_is_usb` markerer det. Overstyr uansett med `--wire-ceiling-mbps`.
- **Admittans er avhengig av drenering, ikke av forholdet mellom hel burst og ring.** En samtidig burst trenger bare å passe inn i den *midlertidige backloggen* = `max(0, Σ per-cam arrival − host drain) × emit_window`, ikke hele bursten. I et nettverk med rask vert og treg kamera (en **PCIe**10G-vert + 4× 1 GbE-kameraer: innkommende ≈ 320 MB/s, utgående ≈ 1063 MB/s) tømmer verten raskere enn kameraene fyller, backlog ≈ 0, så**tillater**full-res sim-emit selv om 25 MB-bursten overstiger 13,5 MB-ringen. Plasser de samme fire kameraene bak en**USB**10 GbE-adapter, og uttaket blir 200 MB/s, ikke 1063 — innkommende trafikk overgår det, og tapet viser seg som ødelagte rammer i stedet for en lavere bildefrekvens. På en 1 GbE-vert gjør kameraenes DLThr-grense på 31,25 MB/s at til at innkommende data overgår uttaket → den**blokkerer** korrekt (for *denne* blokkklassen, reduser ROI eller bruk binning ≥ 2). Adgang er en av **to** tilkoblingsporter — den andre er den samlede kontrollen for overtegnelse nedenfor.
- **Projisert fps er et konservativt tak for seriell henting.**Vertsmaskinens hentingsløyfe henter for øyeblikket hver kameras buffer**serielt**(~ett utsendingsvindu per kamera hver), så syklusen er begrenset av `max(readout+emit, N × emit)`, der utsendingen per kamera er begrenset til kameraets**tilgangslink**(1 GbE ≈ 80 MB/s), ikke vertsens uplink. For et oppsett med 4 kameraer i full oppløsning og 12-bit er dette**~2,8 fps**, noe som samsvarer med de målte ~2,7–3,0 fps. Dette er bevisst**eksponeringsuavhengig**, så i svakt opplyste scener kan den faktiske verdien synke litt under takverdien når eksponeringstiden øker. Den serielle hentingen er den virkelige fps-begrenseren; å parallellisere den ville heve takverdien mot hastigheten for enkeltutsendelse.
- **Samlet overtegnelse er en hard tilkoblingsblokkering.**Båndbreddetildelingen per kamera har en nedre grense på**8 MB/s**(`ARRAY_PER_CAM_FLOOR_BPS`), så når denne nedre grensen nås, kan den samlede etterspørselen (`per_cam × N`) overstige den**kollisjonssikre kabelgrensen**(`sustained × sim_emit_factor`). Praktiske tak for full oppløsning på 1 GbE:**6 kameraer ved 1500 MTU, 9 med jumbo**. Dette taket er en egenskap ved nettverket og minimumsgrensen alene — det er**uavhengig av rammestørrelse**, så**binning og mindre ROI hjelper IKKE** (de reduserer antall byte per *ramme*, ikke antall byte per *sekund* som styres av GevSCPD); de eneste løsningene er færre kameraer, jumbo-rammer fra ende til ende eller et raskere nettverkskort. Symptomet vil være pakketap i GVSP, ikke en jevn reduksjon i bildfrekvensen, så `analyze-array` setter tallene for oppnåelig fps til null og skriver ut `**OVER-SUBSCRIBED**`, og `array-connect` med en fastlåst oppløsning **nekter å koble til** (walk-down-funksjonen sorterer ellers bort rammer, noe som heller ikke fjerner denne typen blokkering). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` nedgraderer avvisningen til en tydelig advarsel for benkarbeid — se [Miljøvariabler](#environment-variables).

### Array-tilstand — hvilket delsystem mister rammer

`GET /api/camera/array/<array_id>/capability` i et tilkoblet array inneholder en aktiv
`health`-blokk, som revurderes i et rullende **10-sekunders** vindu. Den deler rammetap
opp i de to årsakene som krever motsatte løsninger, i stedet for å rapportere én «ufullstendig»
rate som ikke spesifiserer noen av dem:

| Felt | Hva det betyr | Hvilket undersystem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (per seriell port) | Bildet **kom frem, men var strukturelt feil**— GVSP-pakketap. |**Nettverk**: ledningskapasitet, pacing, NIC RX-ring, MTU |
| `never_arrived_rate_pct` (per serienummer) | Rammen **kom aldri i det hele tatt**— kameraet utløste ikke, eller ingenting kom ut av det. |**Utløser / synkronisering**: M8-kabel, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Det dårligste kameraets frekvens for hver av dem. | — |
| `per_cam_rate_pct` | Kombinert andel ufullstendige rammer per kamera (begge årsakene samlet). | — |
| `stable_for_seconds` | Hvor lenge hvert kamera har ligget under 0,01 %. | — |

Over 5 % logger backenden en `[array-health <id>] WARN`-linje som angir fordelingen — ved
første brudd, ved endring av alvorlighetsgrad, én gang i minuttet så lenge det vedvarer, og én gang når
det forsvinner. Den korrupte halvdelen skriver ut `[gvsp-corrupt <SN>]` ved første treff per kamera og
årsak, deretter en oppsummering hvert 60. sekund. Hver evaluering havner fortsatt i backend-loggfilen;
tellerne går videre for hver buffer uavhengig av hva som skrives ut.

Den samme posten rapporterer tallet som hele tildelingen henger på:

| Felt | Hva det betyr |
| --- | --- |
| `wire_ceiling_mbps` | Vertsmaskinens gjeldende, vedvarende båndbreddebudsjett, MB/s. |
| `wire_ceiling_source` | Hvor tallet kommer fra, i ord – f.eks. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` eller `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` når `--wire-ceiling-mbps` (eller GUI-ens **Wire Budget**-feltet) angir det. |
| `nic_is_usb` | `true` for en USB-Ethernet-adapter — se grensen på 200 MB/s ovenfor. |

**Tolkning:** `gvsp_corrupt_rate_pct` ulik null med `never_arrived_rate_pct` på 0
betyr at utløsningen og kabelsynkroniseringen er perfekte, og at 100 % av tapet skyldes nettverksbanen
— senk `--wire-ceiling-mbps` og koble til på nytt. Det omvendte mønsteret peker i stedet mot
synkroniseringskabelen eller utløserlinjen.

> **`--target-fps` er ikke avgjørende for korrupte rammer.** GevSCPD-takten skrives
> én gang ved tilkobling, så å senke utløserfrekvensen endrer arbeidscyklussen og ikke
> burstfrekvensen for samtidig utsending. En målt reduksjon av etterspørselen med 5× ga ingen forbedring;
> å senke ledningstaket fra 240 til 200 MB/s reduserte andelen korrupte rammer for samme rigg fra 10,4 %
> til 0,00 %.

> **Automatisk reduksjon midt i strømmen er ikke tilgjengelig på TRI032S-firmware.** Et array i drift
> kan ikke løse dette selv; koble fra og koble til på nytt, slik at tilkoblingstidvelgeren kan
> planlegge på nytt med den nye grensen.

### Symptom → løsning

| Symptom (Array-innstillinger / tilkobling / `analyze_array_network`) | Årsak | Løsning |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` tilbakestilt til 32 (vanligvis etter en driveroppdatering) | Sett `ReceiveBufferLen`→256, `PendingReceives`→64; åpne panelet på nytt (start backend på nytt hvis den har lagret den gamle ringstørrelsen i hurtigbufferen) |
| Omstart/avslutning henger seg opp; senere oppstår `Invalid class` WMI-feil, nettverkskortet aktiveres ikke, USB-stasjoner mangler | Gammel Realtek USB 10GbE-driver fra 2016 → BSOD `0x9F` → tvungne avstengninger | Oppdater adapterdriveren til ≥ v10.67 (2026), og bruk deretter mottaksringinnstillingene ovenfor på nytt |
| Tilkoblingen lykkes, men gir en oppløsning lavere enn den opprinnelige | Smart-prep krympet rammen automatisk for å passe til kabelen | Oppgrader lenken / godta krympingen / `--force-tier slip-emit-and-capture` |
| Array rapporterer en normal mål-fps, men leverer bare en brøkdel av den; `health.gvsp_corrupt_rate_pct` er ikke null, `never_arrived_rate_pct` 0 | Vertsmaskinensantatte ledningskapasitet er høyere enn den faktisk tåler (typisk på en USB-Ethernet-adapter, en smal PCIe-bane eller et delt nettverk) | Koble til på nytt med en lavere `--wire-ceiling-mbps` og sjekk helseblokken på nytt. **Ikke** `--target-fps` — GevSCPD-tempoet er fastsatt ved tilkobling |
| Kameraer mangler fra publiserte grupper; `health.never_arrived_rate_pct` er ikke null, `gvsp_corrupt_rate_pct` 0 | Utløser-/synkroniseringsbane — kameraene utløses ikke, dette er ikke et nettverksproblem | Kontroller M8-synkroniseringskabelen og `--line`; bekreft at alle enheter er aktivert (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` overskredet i `analyze-array`, eller tilkoblingsfeil med fastløsning (`array over-subscribes the wire`) | Samlet behov per kamera (8 MB/s minimum × N kameraer) overskrider det kollisjonssikre taket for båndbredde — 6 kameraer i full oppløsning på 1 GbE @1500 MTU, 9 med jumbo | Færre kameraer, jumbo-rammer fra ende til ende, eller et raskere nettverkskort. **ROI/binning vil IKKE hjelpe** (taket er uavhengig av rammestørrelse). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` overstyrer på testbenken (aksepterer pakketapet) |

---

## `chloros-cli daq`

Kommandoer for spektralsensor. To klasser:
- **`pool-*`**— enkle HTTP-klienter som styrer sensoren gjennom backendens permanente pool.**Dette er den støttede banen, og den eneste som finnes i den medfølgende CLI.** Backenden eier transporten, så GUI-en, CLI- og SDK-skriptene deler alle ett aktivt håndtak i stedet for å kjempe om den serielle porten.
- **Alt annet**(`test`, `record`, `live`, `stream`, `connect`, `info`, `net`, `ota`, `sample-rate`, `calibrate`, `serve`, `ws`, `udp`, `mqtt`, `reflectance`, `login`, `logout`, `status`) — direkte maskinvaretilgang, dokumentert nedenfor for fullstendighetens skyld. Disse krever pakken `daq` Python, som**ikke er inkludert i noen av de leverte artefaktene**: den kompilerte CLI utelukker den (`scripts/Build-CLI.ps1` setter `--nofollow-import-to=daq`, og transportene `pyserial` / `bleak` / `zeroconf` med den), og PyPI-pakken SDK inneholder den heller ikke. De kjører kun fra en kildekode-checkout, så behandle dem som en MAPIR-intern utviklingsbane snarere enn noe man bør ty til.
- **`discover` / `list`** ligger mellom de to: de er direkte maskinvarekommandoer fra en kildekode-checkout, men på en ferdig distribuert versjon faller de tilbake til `pool-discover`, og backenden utfører skanningen. Så skanning fungerer overalt — noe som er viktig fordi det er den eneste måten å finne ut BLE-MAC-en til en DAQ-M på.

> **`chloros-cli daq --help`** (og `-h` / `help`) viser en liste over `pool-*`-underkommandoene — hjelpen blir bevisst sendt til pool-klienten, slik at den gjenspeiler kommandoene som faktisk kjøres. Hvis du påkaller en underkommando for direkte maskinvare på en utgitt versjon, avsluttes den med en eksplisitt feilmelding som angir det manglende pakken og henviser deg tilbake til `pool-*`; ingenting mislykkes i det stille. (`discover` / `list` er unntaket — de omdirigerer til `pool-discover` og fungerer uten problemer.)
>
> **Alt en kunde trenger, er tilgjengelig via `pool-*`** – koble til, strømme, ta opp kalibrerte `.daq`-filer og bytte kap-profiler. DAQ-enheten kan også styres fra Python med `chloros_sdk.connect_daq_sensor()`, som bruker den samme samlede banen.

### Arbeidsflyt for første tilkobling av DAQ-sensor

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### `pool-*`-referanse

| Underkommando | Formål |
| --- | --- |
| `daq pool-connect` (smart-detect) | Åpne en sensor i backend-poolen. |
| `daq pool-connect --port PORT` | DAQ-U på en spesifikk seriell port. |
| `daq pool-connect --ble` | DAQ-M over BLE, MAC skannes automatisk. |
| `daq pool-connect --mac MAC` | DAQ-M via BLE på en kjent MAC-adresse (forutsetter `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E via Ethernet på en kjent vert. |
| `daq pool-connect --eth` | DAQ-E over Ethernet, vert automatiskoppdaget (mDNS + ARP-reserve; fungerer fra en tom ARP-cache på Windows og Linux). |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | Juster integrasjonsvindu / AE-tilstand. |
| `daq pool-connect --no-stream` | Koble til, men ikke start strømming ennå (fortsett med `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Cap-korreksjonsprofil. Standard på backend er `sunshine_cosine`. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Skann alle transportprotokoller etter sensorer du kan koble til, uten å koble til. **Slik finner du BLE-MAC-adressen til en DAQ-M.** `daq discover` / `daq list` rutes automatisk hit i de leverte versjonene. Sensorer som allerede er åpne i puljen, vises ikke — en tilkoblet DAQ-M slutter å annonsere — så bruk `pool-list` for disse. |
| `daq pool-list` | Vis alle sensorer i backend-poolen. |
| `daq pool-disconnect --sensor-id ID [--all]` | Frigjør. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | De nyeste N spektrumrammer. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Gjenoppta / pause strømming. |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | Start / stopp en .daq-opptak. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Bytt cap-korreksjonsprofil under kjøring. |

### Direkte maskinvare-underkommandoer (kun i kildekoden – ikke i de ferdige versjonene)

> Oppført for fullstendighetens skyld. Disse krever pakken `daq` Python samt `pyserial` / `bleak` / `zeroconf`, og ingen av disse følger med i den kompilerte CLI eller PyPI-versjonen SDK — de kjører kun fra en MAPIR-kildekodeutgave. **Hvis du bruker en utgitt Chloros-versjon, bruk i stedet `pool-*`-kommandoene ovenfor**; de dekker tilkobling, strømming, opptak og valg av cap.

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

Åpne, koble til og kjør et lagret Chloros-prosjekt (en mappe med `cameras.json` + `sensors.json` + `project.json`). Alt går gjennom backend, slik at GUI og CLI gir identisk maskinvarestatus.

### Referanse for underkommandoer

| Underkommando | Formål |
| --- | --- |
| `project open PATH` | Skriv ut prosjektets enhetsmanifest (kameraer, matriser, sensorer). |
| `project devices PATH [--reconnect]` | Liste opp eller kjør oppdagingen på nytt. |
| `project connect PATH [--cameras-only] [--sensors-only]` | Koble til alle lagrede kameraer / matriser / sensorer. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Enkeltopptak fra et navngitt kamera eller matrise. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | N-rammers serieopptak fra et navngitt kamera eller en matrise (`-n/--count` standard 5; `-i/--interval` sekunder mellom rammer, standard 0). Matriseserieopptak fjerner-dupliserer gjentatte synkroniserte grupper (foreldelsesvakt), slik at et delvis syklende array ikke kan levere tilbake N kopier av ett bilde; skriver ut resultater per iterasjon. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | Strøm til-disk via en backend-jobb. `--poll-interval` = sekunder mellom `/stats`-avlesninger (standard 2,0). |
| `project sensor read PATH NAME [--json]` | Siste spektrumramme. |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | Ta opp .daq. |
| `project run PATH RECIPE.yaml` | Kjør en YAML/JSON-opptaksoppskrift. `--dry-run` validerer uten å kjøre. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Beregn justering for en matrise — se [flaggtabellen nedenfor](#project-align-calibrate-alternativer). |
| `project align status PATH NAME [--json]` | Skriv ut gjeldende justeringsprofil. |
| `project align clear PATH NAME` | Slett den bufret profilen. |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | Juster transformasjonen til én slave. |
| `project align export PATH NAME --to FILE` | Lagre profilen til JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Last inn en lagret profil. |

#### `project align calibrate`-alternativer

| Flagg | Standard | Beskrivelse |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Justeringsmetode. **Disse stavemåtene skiller seg fra `lattice align-calibrate`**, som bruker kortformene `orb` / `akaze` / `phase`; de to kommandoene er ikke utskiftbare på dette flagget. |
| `--model {translation, rigid, affine, homography}` | `affine` | Transformere modellen for å passe. |
| `--frames N` | `1` | Synkroniserte rammeøyeblikksbilder til gjennomsnittet. |
| `--reference SN` | masteren | Referansekameraets serienummer; hvert annet medlem blir forvrengt til å passe til det. |
| `--max-features N` | `5000` | Øvre grense for antall ORB-trekk. |
| `--ratio-threshold F` | `0.75` | Lowes forholdstest. |
| `--ransac-threshold-px F` | `3.0` | RANSAC-terskel for inliers. |
| `--min-matches N` | `15` | **Kvalitetsgrense** — avvis løsningen dersom antall inlier-treff er lavere enn dette. |
| `--max-reproj-err-px F` | `4.0` | **Kvalitetsgrense** — avvis løsningen hvis RMS-feilen ved reprojeksjon overstiger denne verdien. |
| `--checkerboard RxC` | — | Brettgeometri for `--method checkerboard`, f.eks. `9x6`. |
| `--name PROFILE` | tom | Profilnavn innebygd i den lagrede JSON. **Ikke matrisenavnet** — det er den posisjonelle `NAME`. |

De to kvalitetskontrollene er årsaken til at en kalibrering kan lykkes med å løse oppgaven, men likevel
avvises ved lagring: en profil som ikke oppfyller ett av kravene, vil uten varsel føre til feilregistrering av alle
senere opptak, så den avvises i stedet for å lagres.

### Eksempler

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### Oppskrifts-DSL

`project run RECIPE.yaml` godtar en YAML- eller JSON-fil som beskriver en sekvens av handlinger:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Støttede handlinger: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. Handlingen `burst` tar `name` (påkrevd), `count` (standard 5), `interval` (sekunder, standard 0), `output`, `format` og `settings` (samme innstillingsstruktur per kamera som `apply`); array-serier bruker samme nysynkroniserte gruppevakt som `project burst`.

Kjør det:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## Miljøvariabler

| Variabel | Effekt |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Overstyrer backend-innstillingen URL (standard `http://127.0.0.1:5000`) — **respekteres kun av kommandofamiliene `lattice`, `project`, og `daq pool-*`-kommandofamiliene.** Kjerne-kommandoene (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) kobler til `http://127.0.0.1:<port>` og ignorerer denne variabelen (IPv4-litteral omgår Windows `localhost`→`::1` ~2 s-per-forespørsel-straff), så de retter seg alltid mot den lokale maskinen. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1` nedgraderer avslaget på tilkobling ved overabonnement i arrayet (samlet etterspørsel per kamera &gt; kollisjonssikker ledningsgrense med `pin_resolution`) til en høylydt «advarsel og fortsett», og aksepterer tap av GVSP-pakker. Kun for benkebruk — se [Array fps &amp; burst-modell](#array-fps--burst-model). |
| `CHLOROS_CLI_MODE` | Angis av selve CLI; ber backenden om å aktivere parallellbehandling. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` hopper over GVSP-fallback-sonden (kun ICMP-resultater). **Dette slår jumbo av, det demper ikke bare loggen** — kameraet svarer kun på DF-ping opp til 1500 på hver bane, så denne sonden er det eneste som kan oppdage jumbo. Sparer ~1 s per kamera per tilkobling; koster ~1,45× kabeltak hvis nettverket *kunne* ha håndtert jumbo. SDK gir en advarsel når du konfigurerer den. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Fastsett GVSP-pakkestørrelse til N byte; hopper over sondering helt. Anbefalt per kommando (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) fremfor å sette den permanent: en fastsatt størrelse slutter å tilpasse seg nettverket foran den, og å fastsette 9000 på en bane som ikke kan håndtere jumbo-pakker, fører til at **hver** opptakstid går ut med `SC_ERR_TIMEOUT -1011`. |
| `TMPDIR` (Linux) | Overstyr Nuitkas utpakkingsmappe for én fil. CLI bruker automatisk `/mnt/ssd/tmp` hvis den finnes. |

---

## Avslutningskoder

| Kode | Betydning |
| --- | --- |
| `0` | Vellykket. |
| `1` | Generell feil (de fleste underkommandofeil). |
| `2` | Argumentfeil. |
| `130` | Avbrutt med Ctrl+C. |

---

## Tips til feilsøking

- **«Pålogging kreves»** → Kjør `chloros-cli login EMAIL PASSWORD` én gang på denne maskinen.
- **«Backend utilgjengelig»** → Start skrivebordsappen Chloros, eller kjør backend-binærfilen direkte (`chloros-backend`), eller sjekk `CHLOROS_BACKEND_URL` hvis det er eksternt.
- **`lattice`-kommandoer mislykkes med «LATTICE-kameradrivere ikke funnet»** → Arena SDK-kjøretiden er ikke installert; CLI leveres med `win32api` inkludert i Windows, men C kjøringsmiljøet er en del av GUI-installasjonsprogrammet.
- **Array connect / Array Settings viser «FRAMES WILL DROP» eller «Reduce ROI to enable»** → Mottaksringen på verts-NIC-en er for liten (blir ofte tilbakestilt til 32 etter en oppdatering av NIC-driveren). Se [Konfigurering og justering av verts-NIC](#host-nic-setup--tuning-lattice-arrays) — sett `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Maskinen henger seg opp ved omstart/avslutning, deretter WMI `Invalid class` / nettverkskortet aktiveres ikke / USB-stasjoner mangler** → Utdatert driver for USB 10GbE-adapter forårsaker `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Oppdater adapterdriveren — se [Konfigurasjon og justering av verts-NIC](#host-nic-setup--tuning-lattice-arrays).
- **Jetson-swap-advarsel** → Legg til filbasert swap; CLI viser de nøyaktige `fallocate` / `swapon`-kommandoene.
- **DAQ-direktekommandoer mangler** → Forventet: den leverte `chloros-cli` utelukker bevisst `daq`-pakken, slik at kun `pool-*` er til stede (PyPI-versjonen SDK inneholder den heller ikke). Bruk `pool-*`, som styrer den samme sensoren via backenden, eller `chloros_sdk.connect_daq_sensor()` fra Python.

---

## Se også

- [Python SDK-referanse](sdk-reference.md) — programmatisk ekvivalent til alle CLI-kommandoer.
- [Veiledning for DAQ-sensorer](../daq/README.md) — sensorspesifikk kabling og kalibrering.
- Online dokumentasjon: `https://mapir.gitbook.io/chloros/cli`
