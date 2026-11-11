# CLI: Kommandolinje

> **Fullstendig referanse:**[CLI-referanse](reference/cli-reference.md) dokumenterer**alle flagg for alle underkommandoer** og er optimalisert for AI-assistenter — lim inn URL-en i assistenten din og be om en fungerende kommando: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Tips for AI-verktøy:** Enhver side i denne håndboken er tilgjengelig som rå Markdown ved å legge til `.md` i URL-en (f.eks. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), og `https://mapir.gitbook.io/chloros/llms.txt` indekserer hele håndboken for bruk i store språkmodeller (LLM).

> **Vil du at assistenten din skal kjøre Chloros selv?** `chloros-mcp`, en MCP-server installert med Python-SDK, fungerer med Claude Desktop, Claude Code, Cursor og andre MCP-klienter: se [Bruke Chloros med AI-assistenter](ai-assistants.md).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.3.0 build so the version line reads "Chloros CLI 1.3.0" -->


## Hva er CLI?

`chloros-cli` er kommandolinjegrensesnittet til den samme prosesseringsmotoren som Chloros-skrivebordsappen bruker. Det er en «thin» HTTP-klient over Chloros-backenden (en lokal server på `127.0.0.1:5000`) — de fleste kommandoer starter backenden automatisk, så et enkelt `chloros-cli process …`-kall er alt et skript trenger. Backenden fortsetter deretter å kjøre for neste kommando: den stopper ved `chloros-cli logout` (med mindre appen er åpen eller en kjøring eller en kompilering pågår), ved `chloros-cli backend stop`, eller av seg selv etter 30 minutter uten klient, kjøring eller tilkoblet enhet. `chloros-cli backend status` viser dette.

Den kjører på **Windows 10/11 (x64)**og**Linux (x86_64 og NVIDIA Jetson arm64 på JetPack 6)**, i hvilken som helst terminal, uten behov for grafisk brukergrensesnitt. Kontroller installasjonen med:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

Kommandofamiliene, i korthet:

* **Behandling og konto** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 språk — se [Støttede språk](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (kun Linux/Jetson)
* **Live-maskinvare** — `lattice` (LATTICE-kamerakontroll, over 45 underkommandoer), `daq pool-*` (DAQ-lyssensorer), `time-sync` (PTP)
* **Automatisering** — `project` (kjør et lagret Chloros-prosjekt uten grafisk grensesnitt, inkludert YAML-opptaksoppskrifter)

Globale alternativer det er verdt å kjenne til: `--port N` (backend-port, standard `5000`), `-v/--verbose`, `--restart` (tving omstart av backend), `--backend-exe PATH`. Se [CLI-referansen](reference/cli-reference.md) for fullstendig oversikt.

***

## Installasjon

CLI-en **følger med i Chloros-installasjonsprogrammet** på alle plattformer — det finnes ingen separat nedlasting av CLI-en. Last ned installasjonsprogrammet fra [Nedlastingssiden](download.md).

### Windows


Installasjonsprogrammet plasserer CLI-en i:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

og legger til denne mappen i systemet ditt `PATH` én gang (en oppgradering fjerner også dupliserte oppføringer etter tidligere installasjonsprogrammer) — **åpne en ny terminal**etter installasjonen, slik at den oppdaterte `PATH` blir hentet inn. Installasjonsprogrammet plasserer også startskript (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) i installasjonsroten, samt en**Chloros CLI** snarvei i Start-menyen, som hver åpner en terminal med `chloros-cli` klar til bruk.

### Linux


Installer `.deb` for din arkitektur med `apt` (ikke `dpkg -i`, som hopper over de anbefalte kravene til Python SDK):

```bash
# Linux x86_64
sudo apt install ./chloros_1.3.0_amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
```

Dette installerer `chloros-cli` til `/usr/bin/chloros-cli` (allerede på `PATH`), backend- og CLI-programmene som er pakket ut i `/usr/lib/chloros/backend/` og `/usr/lib/chloros/cli/` (`/usr/lib/chloros/chloros-backend` er et startprogram som starter backend-modulen), Arena SDK-kjøretiden som backend-modulen trenger for LATTICE-kameraer, og Python-SDK for systemet `python3`. Se [Linux-installasjon](linux/linux-installation.md) for detaljer.

### Bekreft

```bash
chloros-cli --version    # "Chloros CLI 1.3.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Pålogging og lisensiering

Tilgang til CLI (og Python SDK) krever et **betalt Chloros+-abonnement**— alle betalte nivåer har dette; det gratis nivået har det ikke. Begrensningen håndheves**på serversiden** av backend-systemet, ikke av CLI-binærfilen: et anrop fra en utlogget bruker avvises med `401 AUTH_REQUIRED`, og et innlogget anrop på gratisabonnementet med feilkoden `403 PLAN_UPGRADE_REQUIRED`, uansett om det kommer fra `chloros-cli`, SDK-et eller en håndskrevet HTTP-klient. Oppgrader på [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Logg inn **én gang per maskin**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.3.0 build showing the license tier line -->


{% hint style="warning" %}
**Passord med spesialtegn**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` blir forvrengt av skallet (CLI-en oppdager dette ved en 401-feil og prøver på nytt automatisk, men enkelt anførselstegn unngår problemet helt).
{% endhint %}

Økten lagres i `~/.chloros/user_session.json` og fungerer fortsatt offline i abonnementets nådeperiode (30 dager for månedsabonnementer, frem til utløpet for årsabonnementer). `chloros-cli status` fungerer selv uten et betalt abonnement, så årsaken til avslaget er alltid synlig.

{% hint style="danger" %}
**Skal du planlegge headless-arbeid? Logg inn først.**En kommando som starter en backend (`process`, `status`, `export-status`, …) som kjøres uten**bufret økt**, mislykkes ikke umiddelbart — den går over til en interaktiv `Email:` / `Password:`-prompt på stdin. En uovervåket cron-jobb eller et CI-trinn vil derfor**henstå i ventemodus på inndata**. Kjør `chloros-cli login EMAIL 'PASSWORD'` én gang på maskinen før du planlegger noe.
{% endhint %}

***

## Din første behandlingskjøring

Pek `process` mot en mappe med opptak — den oppdager automatisk Survey3 (`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` eller en blanding:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Fremdriftsstrømmer sendes live per rørledningstråd (detektering, analyse, behandling, eksport), og en vellykket kjøring avsluttes med en rapport om hvor mange bildeprodukter den skrev: `Image products written: N`, etterfulgt av `(plus M clamp masks)` når kjøringen også skrev [clamp-masker](output-image-formats.md#clamp-masks) (disse regnes ikke som produkter).

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N (plus M clamp masks)" summary line -->


Hvis en Chloros-backend allerede kjører (for eksempel desktop-appens), bruker `process` den og åpner prosjektet sitt i den. Den avviser mens appen er åpen på et prosjekt appen har åpnet: kjør fra appen, eller lukk det prosjektet. Hvis en tidligere kjøring fortsatt er i ferd med å avsluttes, venter den i opptil 15 sekunder på den før den avviser. `--restart` tvinger frem en ny backend. På Linux blir en backend som kjører som en systemd-tjeneste aldri startet på nytt: `process` bruker den som den er, og der den ellers ville ha startet den på nytt (`--restart`, eller en import som ikke fant noe), skriver den ut `sudo systemctl restart <unit>` og avslutter `1`.

### Hvor utdataene havner

`process` skriver til en **prosjektmappe**, ikke til inndatamappen din:

* Uten `-o`: prosjektet opprettes under standard prosjektmappen din (delt med GUI; administrer den med `get-project-folder` / `set-project-folder`; hvis ingen er angitt, `Documents/MAPIR/Chloros Projects` i brukermappen din), navngitt med `-n/--project-name` eller et tidsstempel (`YYYYMMDD_HHMMSS`) hvis dette utelates.
* Med `-o PATH`: den mappen **er** prosjektmappen. Hvis den allerede inneholder en `project.json`, opprettes en `_1`/`_2`… med suffiks opprettes i stedet for å overskrive. En relativ `PATH` angis relativt til mappen du kjører `chloros-cli` fra.

{% hint style="info" %}
**Relative baner (endret i 1.3.0).** Hver sti som backenden åpner — `-o`/`--output` ved enhver kommando (inkludert standard `output` for opptakskommandoer), `--input`, `--burst-dir`, `--profile`, `--save` og `--mask` — blir løst mot mappen du kjører `chloros-cli` fra, slik at `-o captures/` skriver til `<current folder>/captures/`. I versjon 1.2.0 løste backenden dem i sin egen installasjonsmappe, noe som mislyktes med feilmeldingen «Tilgang nektet». Mot en ekstern backend (`CHLOROS_BACKEND_URL`) sendes stiene slik de er angitt og løses på den maskinen.
{% endhint %}

{% hint style="warning" %}
**Windows: hold prosjektmappen kort med mindre lange baner er aktivert.** Når lange baner er deaktivert i Windows (standardinnstillingen), kan ingen filbane være lengre enn 260 tegn, og produktene ligger opptil ca. 130 tegn under prosjektmappen. `process` sjekker dette før det starter: en prosjektmappe som er for dyp, avvises med **«Prosjektmappen er for dyp for denne datamaskinen»**, den lengste prosjektmappen som passer, og hvordan man aktiverer lange baner — ingenting blir behandlet. Bruk en kortere `-o` (eller `set-project-folder`), eller aktiver lange stier i Windows (installasjonsprogrammets alternativ **Aktiver lange stier i Windows**, som er merket av som standard) og start Chloros på nytt.
{% endhint %}

Inne i prosjektet er produktene gruppert **etter kamera, deretter etter filformat**:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Kameramappen er `LATT-<sensor>-<lens>-F<filter>` for LATTICE (som samsvarer med opptakets EXIF-data `Model`) og `<model>_<filter>` (f.eks. `Survey3N_RGN`) for Survey3. Formatmappen følger `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` eller `tiff32` for `TIFF (32-bit, Percent)`.

For et multispektralt M3C-kamera kan det også forekomme en `clamp_masks/`-mappe i en formatmappe, ved siden av produktmappene (for eksempel `tiff16/clamp_masks/Reflectance_Calibrated_Images/` og `tiff32/clamp_masks/Radiance_Images/`). Den inneholder maskene per piksel for strålings- og refleksjonsprodukter der båndseparasjonen har klemt en negativ verdi til null — se [Klemmasker](output-image-formats.md#clamp-masks). Ingen produktmappe inneholder noen gang en maske.

{% hint style="info" %}
**Hvert eksportert produkt beholder navnet på KILDE-filen.**En strålings-eksport av `capture_..._raw.tif` heter fortsatt `capture_..._raw.tif` — den ligger bare i `tiff32/Radiance_Images/`.**Mappen identifiserer produktet, ikke filnavnet**, så bruk glob for katalogen, ikke for suffikset `*radiance*`.
{% endhint %}

### Alternativene du faktisk vil bruke

| Flagg | Standard | Hva det gjør |
| --- | --- | --- |
| `-o, --output PATH` | standard prosjektmappe | Plassering av prosjektmappen (se ovenfor). |
| `-n, --project-name NAME` | tidsstempel | Prosjektnavn. |
| `--format FMT` | `TIFF (16-bit)` | En av `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros oppskalerer ikke: på bilder som utelukkende består av 8-bit-data skrives et dypere format som `TIFF (8-bit)`, og dette oppgis én gang. |
| `--indices NAME [NAME ...]` | ingen | Vegetasjonsindekser som skal eksporteres (se [Vegetasjonsindekser](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = nevral debayer, tregere, høyeste kvalitet (Chloros+, krever en CUDA-GPU: uten en slik bruker kjøringen `standard`; avvist for LATTICE-opptak, som alltid får `standard`). |
| `--vignette / --no-vignette` | på | Vignettkorreksjon. |
| `--reflectance / --no-reflectance` | på | Refleksjonskalibrering; for LATTICE er dette også bryteren for refleksjonsproduktet. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Tving inn inngangspunktet til behandlingskjeden for LATTICE-TIFF-filer. |

For alt annet — justering av måldeteksjon, PPK, eksponeringspinner, flagg for matrisejustering — se [`process`-delen av CLI-referansen](reference/cli-reference.md).

***

## Velge hva som skal eksporteres (LATTICE-produkter)

LATTICE-behandlingen fordeles på **alle aktuelle produkter i én gjennomgang**. Fire brytere per produkt er alle**slått på som standard**; bruk skjemaet `--no-` for å slå av én:

| Bryter | Produkt |
| --- | --- |
| `--debayered` | Lineær demosaik → `Debayered_Images/`: kalibreringspakkens mørkeforskyvning fjernes, deretter en kantbevisst demosaik, uten linearitetskorreksjon, flatfelt eller unmix — den samme filen som en opptak lagrer på sitt `debayered`-nivå. Bevares i sensorens opprinnelige orientering (registrert i XMP). |
| `--preview` | Vis forhåndsvisning (hvitbalanse + gamma; falskfargestrekning for multispektral) → `Preview_Images/` |
| `--radiance` | float32 strålingsintensitet, W/m²/sr/nm → `Radiance_Images/` (alltid `tiff32/`) |
| `--reflectance` | uint16 reflektans, Pix4D-klar → `Reflectance_Calibrated_Images/` |

RGB-hovedkameraer sender kun ut debayert + forhåndsvisning — radianse/reflektans per bånd er ikke meningsfullt for en bredbåndssensor, så disse bryterne har ingen effekt for dem. Survey3 `.raw` ignorerer bryterne og følger standardbanen for reflektans/mål.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (standard `auto`) velger reflektansreferansen: `auto` gjør et QA-godkjent [kalibreringsmål](calibration-targets.md) innenfor rammen til den absolutte referansen og faller tilbake til DAQ-lyssensorens nedadgående lysdelingsforhold (ρ = π·L/E) når det ikke er noe mål til stede (et mål som består QA med et flagg konkurrerer med DAQ om målt poengsum); `target` er strengt (ingen DAQ-erstatning); `daq` er DAQ-autoritativ. Målskanninger målt per enhet kan leveres med `--target-reflectance-dir`.

{% hint style="info" %}
**Avlesning av reflektanspiksler:**DN-verdien som betyr ρ = 1,0 er**per kilde** — LATTICE-filer merker `Chloros:PixelScale=32768` i XMP; Survey3-filer bruker 65535 og har ingen `Chloros:PixelScale`-tag (de har imidlertid andre `Chloros:*`-behandlings-tagger). Les taggen og del med den i stedet for å anta en konstant. Detaljer og det ene bevisste spesialtilfellet uten skala finnes i [CLI-referansen](reference/cli-reference.md).
{% endhint %}

**Behandlingen starter alltid fra `raw`.** Avledede produkter (eksport av debayering/stråling/refleksjon) føres aldri tilbake gjennom behandlingskjeden — å importere dem på nytt og behandle dem ville føre til at kalibreringsberegningene ble brukt to ganger, så Chloros hopper over dem og opplyser om dette. `--input-level` er den bevisste nødutgangen når du virkelig trenger å tvinge frem et startpunkt.

***

## Når en kjøring mislykkes

Fra og med versjon 1.2.0 mislykkes `process` med et tydelig feilmelding i stedet for å «lykkes» uten å vise noe:

* En kjøring som **ba om produkter, men ikke skrev ut noen**— kun `project.json` og `calibration_data.json` — skriver ut `Processing finished but wrote no image products.` og**avsluttes med en verdi ulik null**, slik at skript kan oppdage det. De vanligste årsakene: inndatamappen ble ikke gjenkjent som en opptak (sjekk oppsettet og `--input-level`), eller hvert forespurt produkt var uanvendelig for de aktuelle kameraene (f.eks. å be om radianse/reflektans fra kameraer som kun bruker RGB).
* En **bevisst kjøring kun med metadata** (alle produkter slått av, ingen `--indices`) regnes fortsatt som vellykket — et tomt bilde som utdata er det riktige resultatet i dette tilfellet.
* Kjør på nytt med `--verbose` og sjekk backend-loggen for linjer med `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, som forklarer utelatelser per kamera.
* På Windows, der lange baner er deaktivert, avvises en prosjektmappe som er for dyp for banegrensen på 260 tegn **før** behandlingen starter, og den lengste prosjektmappen som passer, blir brukt (se [Hvor utdataene havner](#where-the-outputs-land)).
* Opptakskommandoer (`lattice capture`, `lattice array-capture` og de andre opptaksfamiliene) oppgir navnene på alle filene de ikke klarte å skrive, og avslutter med `1`; gjentatte opptak stopper når stasjonen er full.

Avslutningskoder: `0` suksess · `1` generisk feil · `2` argumentfeil · `130` avbrutt av Ctrl+C.

***

## Vegetasjonsindekser

Kjør `--indices` med ett eller flere forhåndsinnstilte navn; hver indeks havner i sin egen `<INDEX>_Index_Images/`-mappe:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI GNDVI OSAVI
```

De 22 forhåndsinnstilte navnene som `process --indices` godtar:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

Et navn som prosjektet ennå ikke inneholder, legges til. Et navn som ikke er en forhåndsinnstilling, eller som kameraenes bånd ikke kan danne, hoppes over med en advarsel som angir navnet. Hvis CLI-en ikke kan lese prosjektets gjeldende indeksliste, lar den listen være uendret og gir en advarsel i stedet for å erstatte den.

{% hint style="warning" %}
**Det finnes tre indekslister – ikke bland dem sammen.**Rullemenyen «Prosjektinnstillinger» i GUI-en har 27 formler (legger til `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` – disse fem finnes kun i brukergrensesnittet og gjelder**ikke** for `--indices`). Live-/offline-kommandoen `lattice index --preset` bruker sin egen separate liste med 22 forhåndsinnstillinger. Formler og båndberegninger er dokumentert i [Formler for multispektrale indekser](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## DAQ-lyssensorer: En rask innføring

`daq pool-*`-familien styrer MAPIRs DAQ-spektralsensorer (DAQ-U via USB, DAQ-M via BLE, DAQ-E via Ethernet) gjennom backendens permanente pool — GUI, CLI og SDK deler alle ett live-håndtak. **`pool-*` er den støttede DAQ-banen i den medfølgende CLI-en**; andre `daq`-underkommandoer du kanskje ser referert til, er en MAPIR-intern overflate kun for kilde og avsluttes med en eksplisitt feilmelding som henviser deg til `pool-*`.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record` uten `--duration` kjører til `pool-record --stop`; standardutdatakatalogen er `~/Documents/DAQ Live View/` **på backend-maskinen**. Cap-korreksjonsprofilen velges ved tilkobling (`--cap-id`, standard for backend `sunshine_cosine`) og kan byttes under drift med `pool-set-cap` — cap-profiler og sensorens kalibrerte område er omtalt i DAQ-kapitlene i denne håndboken.

{% hint style="warning" %}
**DAQ-E på en vert med flere nettverkskort:** Den første automatiske oppdagelsen av `pool-connect --eth` etter oppstart kan mislykkes selv med en sensor i god stand. `--eth-host <ip-or-hostname>` er den pålitelige varianten — bruk den når oppdagelsen ikke gir resultater.
{% endhint %}

***

## LATTICE-kameraer, PTP og prosjektautomatisering

`lattice`-familien (over 45 underkommandoer) dekker LATTICE-kameradrift fra start til slutt: oppdagelse, enkeltopptak, vedvarende synkroniserte matriser med GUI-ens «smart-prep»-tilkoblingsflyt, live forhåndsvisning i nettleser, justering, indeksberegninger og verts-NIC-diagnostikk. Et utdrag:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

I tillegg: `chloros-cli time-sync` rapporterer om PTP-grandmasteren som Chloros-verten kjører (LATTICE-kameraer og DAQ-E-sensorer fungerer som slaver for denne for tidsstempler på tvers av enheter), og `chloros-cli project` åpner et lagret Chloros-prosjekt og styrer kameraene, matrisene og sensorene uten skjerm – inkludert skriptbaserte YAML-opptaksoppskrifter.

Disse tre familiene (`lattice`, `project`, `daq pool-*`) er også de eneste som støtter `CHLOROS_BACKEND_URL` for styring av en **ekstern** backend; kjerne-kommandoene retter seg alltid mot den lokale maskinen.

Fullstendige gjennomganger finnes i LATTICE-kapitlene i denne håndboken; alle flagg er oppført i [CLI-referansen](reference/cli-reference.md).

***

## Feilsøking: Topp 5

| Symptom | Løsning |
| --- | --- |
| `Login required`, eller en planlagt jobb henger seg opp ved en `Email:`-prompt | Kjør `chloros-cli login EMAIL 'PASSWORD'` én gang på denne maskinen — kommandoer uten en bufret sesjon kjører interaktivt i stedet for å mislykkes umiddelbart. |
| `backend unreachable` | Start Chloros-skrivebordsappen, eller start backend-tjenesten selv: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` på Windows (mappen `backend` i installasjonsmappen; den finnes ikke i `PATH`), `chloros-backend` på Linux. `chloros-cli backend status` viser om en kjører; `lattice`/`project`/`daq pool-*` starter aldri en. Hvis du peker dem mot en ekstern backend, sjekk `CHLOROS_BACKEND_URL`. |
| Array-tilkobling blokkert: `FRAMES WILL DROP` / `Reduce ROI to enable` | Verts-NICs mottaksring er tilbakestilt til standardinnstillinger — den vanligste årsaken til at et system som tidligere fungerte, nå nekter å koble seg til, vanligvis etter en oppdatering av NIC-driveren. Kjør `chloros-cli lattice network --fix` fra en **terminal med utvidede rettigheter** (eller sett `ReceiveBufferLen=256`, `PendingReceives=64`); se referansens *Host NIC Setup & Tuning*. |
| `daq`-underkommando avsluttes: «krever hele DAQ-pakken…» | Bruk kommandoene `daq pool-*`, som styrer den samme sensoren via backenden: tilkobling, strømming, opptak, valg av cap, og for en DAQ-E dens nettverksinnstillinger, omstart og firmwareoppdateringer (eller `chloros_sdk.connect_daq_sensor()` fra Python). |
| Jetson viser en advarsel om swap før store mapper | Legg til filbasert swap — CLI-en viser de nøyaktige `fallocate`/`swapon`-kommandoene som skal kjøres. |

***

## Hjelp

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Alle flagg, alle underkommandoer:** [CLI-referanse](reference/cli-reference.md)
* **Python-ekvivalent:** [Python SDK](api-python-sdk.md) og [SDK-referansen](reference/sdk-reference.md)
* **Støtte:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
