# Referanse for Chloros Python SDK

**Versjon:**

1.3.0**Generert:**

29.07.2026 kl. 19:19 ·**Revidert:**

06.11.2026**Pakke:** `chloros-sdk` (PyPI)**Målgruppe:** Optimalisert for bruk av store språkmodeller (LLM); lesbar for mennesker.**Omfang:** Alle offentlige klasser, funksjoner og hjelpefunksjoner som er tilgjengelige via `import chloros_sdk`, med eksempler som kan kopieres og limes inn, og som dekker bildebehandling, styring av enkeltkamera, synkroniserte matriser, DAQ-sensorer og prosjektautomatisering, samt `chloros-mcp`-serveren for AI-assistenter.

Hvis du bare trenger høydepunktene, kan du hoppe til:
- [Installasjon og hurtigstart](#installasjon)
- [Smart-Connect for LATTICE-matriser](#smart-connect-for-lattice-kameraer)
- [DAQ-sensorsesjoner](#daq-sensorsesjoner)
- [Kamera-verktøy](#camera-utilities)
- [Transient-matriser, justeringsprofiler og indekser](#transient-arrays-alignment-profiles--indices)
- [Prosjektmappe og oppskrifter](#project-folder--recipes)
- [Prosjektautomatisering](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Live-bilder i din egen arbeidsflyt](#live-frames-in-your-own-pipeline)
- [MCP-server (AI-assistenter)](#mcp-server-ai-assistants)

> **Endret i 1.3.0** (hva et eksisterende 1.2.0-skript registrerer).
>
> **Endringer et 1.2.0-skript kan måtte håndtere:**
>
> - **En utgivelse som backenden ikke bekrefter, utløser nå en feilmelding.** `CameraSession.disconnect()`, `ArraySession.disconnect()` / `force_disconnect()` på en matrise som økten åpnet, `DAQSensorSession.disconnect()` / `force_disconnect()` på en sensor den har åpnet, og `ChlorosProject.disconnect_all()` utløser `ChlorosConnectError` når backenden avviser frigjøringen eller ikke bekrefter den i tide, fordi enheten fortsatt kan være opptatt. I versjon 1.2.0 ble de returnert uten varsel. En avslutning av en `with`-blokk utløser fortsatt aldri noe.
> - **En lånt enhet blir aldri avsluttet.** `with attach_array(...)` og et eksplisitt `disconnect()` på den lar matrisen kjøre videre (i 1.2.0 førte det å forlate blokken til at den ble avsluttet, selv når GUI-en eide den); kall `force_disconnect()` når du ønsker å avslutte den. Tilsvarende vil en `connect_daq_sensor()` som fant sensoren allerede i puljen, inkludert smart-detect med null-flagget, la den forbli åpen på `disconnect()` og når den forlater sin `with`-blokk.
> - **PNG og JPEG tar kun `display` og indeksprodukter.** `capture()` med `format="png"` / `"jpg"` (`ext=".png"` / `".jpg"`) og et `radiance`- eller `reflectance`-nivå, eller `processing="all"`, utløser `ValueError` før enhver forespørsel; en 12-biters `raw` / `debayered` avvises av backenden. Se [PNG / JPEG](#png--jpeg-take-the-display-and-index-products-only).
> - **`connect_daq_sensor(sensor_id=...)` åpner den sensoren eller utløser** `ChlorosConnectError` med en liste over hva den fant; den faller aldri tilbake til en annen sensor. Se [Pinned Transport](#pinned-transport).
> - **`frame_stream()` utløser en feilmelding i stedet for å gå i hvilemodus:** `FrameFetchError` vil ved avslag gjenta (en refleksjonsavlesning uten noen DAQ inkludert), etter 60 sekunder uten ramme og etter 60 sekunder uten ny utløser. Se [Én utløser per avlesning](#one-trigger-per-read--group_id).
> - **En tidsavbrudd ved avlesning eller brutt tilkobling utløser `ChlorosConnectError`** fra hvert smart-connect-anrop og `analyze_array_network()`, der versjon 1.2.0 utløste den underliggende tidsavbruddet eller tilkoblingsfeilen.
> - **`connect_array()` venter som standard i 600 sekunder (`timeout`, 180 sekunder i 1.2.0), og en tilkobling den gir opp, blir tilbakestilt.** Anropet sender sin `timeout` til backenden, som setter hvert kamera tilbake slik det ble funnet og ikke kobler til noe når tilkoblingen ikke er fullført i tide; anropet utløser `ChlorosConnectError` (HTTP-feil 504 `CLIENT_DEADLINE_PASSED`). I versjon 1.2.0 kunne en tilkobling rett etter at kameraene ble slått på, vare lenger enn anropet og koble til matrisen etter at feilen var utløst. `connect_all()` venter på hver lagret matrise på samme måte (120 s i 1.2.0). Se [`connect_array()`-signatur](#connect_array-signature).
> - **`connect_array()` beholder hele sensoren.** Uten `width` / `height` / `binning` er hvert kamera satt til full sensor, og bildefrekvensen faller til det verten og nettverket leverer; 1.2.0 har redusert eller samlet bildene for å opprettholde frekvensen. Se [Synkronisert matrise](#synchronized-array--arraysession-smart-prep).
> - **Et Bayer-pikselformat på et M3M-kamera (mono) er dets dybde.** Et M3M tilbyr kun monoformater, og forhåndsinnstillingene kaller Bayer-formatene: i versjon 1.2.0 avviste kameraet formatet og beholdt sitt eget, mens `settings.pixel_format` ga navn til det Bayer-baserte. `LatticeCamera(preset=...)`, `apply_preset()`, `set_format()`, et `CameraSettings(pixel_format=...)`, `connect_camera(preset=...)` og en matrises felles format angir nå Mono-formatet med samme dybde på en M3M: `"BayerRG8"` og `high_speed` gir `Mono8`; `"BayerRG12"`, `high_quality` og `triggered` gir `Mono12`; `cam.settings.pixel_format` navngir det. Det samme gjelder omvendt: en M3C blir aldri lagt inn i et Mono-format, så `"Mono12"` – fra hvilken som helst av disse, `connect_array(pixel_format=...)`, `set_settings(pixel_format=...)` eller et arrays lagrede format – gir `BayerRG12` på en M3C, uansett hvilket kamera som er arrayets master. Et `CameraSettings` du sender videre, beholder navnet sitt, så det samme objektet ber fortsatt hvert kamera om den dybden i sin egen familie. `connect_camera(preset=...)` beholder forhåndsinnstillingens format (1.2.0 setter kameraet til 12-bit etter at forhåndsinnstillingen er brukt), og et kamera som åpnes på en forhåndsinnstilling, skriver ikke lenger det det tok inn i `PRESETS`. Ethvert annet format et kamera avviser (et format som mangler på `PixelFormat`-listen, for eksempel `RGB8` på en M3M eller `BayerRG16` på fastvare uten det) får heller ikke sitt eget navn: etter at et `LatticeCamera`-tilkobling, angir `apply_settings()` eller `set_format()`, `cam.settings.pixel_format` og `packed_transport` angir formatet kameraet strømmer i, og én utskrevet linje angir formatet det avviste.
> - **`analyze_array_network()` angir hvilken tilkobling som gjelder:** `pixel_format=None` (tidligere `"BayerRG8"`) betyr formatene som tilkoblingen angir (Mono12 / BayerRG12), og `packed="auto"` angir valget av pakket transport, så fps-verdien den oppgir kan avvike fra 1.2.0&#x27;. En `pixel_format` du sender, projiserer en M3M på den dybden. Se [Network-Analysis Helper](#network-analysis-helper).
> - **`CameraSession.capture()` returnerer en `CaptureResult`** (fortsatt en liste med rammedikt) med `.skipped`, `.failed`, `.daq_files` og `.daq_matches`. Et `ArraySession.capture()`-resultatet inneholder også `.alignment` (om justeringsforvrengningen ble brukt, og hvorfor ikke) og `.oriented`.
> - **Array-fangster justeres som standard, akkurat som appenes.** `ArraySession.capture()`, `continuous()`, `capture_repeated()` og `ArrayHandle.capture()` / `capture_smart()` forvrenger hvert medlems `debayered`, `display`, `radiance`, `reflectance` og indeksfilene inn i hovedkameraets bilde og beskjærer dem til medlemmenees overlapping (mindre enn sensoren med kameraenes forskyvninger, `XMP-mapir:AlignmentJSON` `"applied": true`), slik at de er de samme filene som Chloros-appens opptak av den samme scenen; `raw` forblir i opprinnelig format. Tidligere lagret en opptak uten `aligned=True` hver fil i den opprinnelige størrelsen, uten forvrengning. `aligned=False` beholder dette. `chloros-mcp`&#x27;s `capture_array` og `start_capture_run` bruker samme standardinnstilling. Se [`ArraySession`-metoder](#arraysession-methods).
> - **`connect_all()` angir et backend som avviser SDK-et.** Et kjørende backend som er logget ut eller har et abonnement under Copper, rapporteres under `errors` med sin egen årsak (HTTP 401 / 403), ikke som «ingen Chloros-backend kjører», og hvert `connect_all()`-kall sjekker backenden på nytt, så å kalle den etter innlogging fungerer på det samme prosjektobjektet.
> - **Et relativt `output_dir` fungerer mot en lokal backend.** Relative baner gjøres absolutte i forhold til skriptets arbeidsmappe før de sendes. I versjon 1.2.0 løste backenden dem i sin egen installasjonsmappe, noe som mislyktes med feilmeldingen «Tilgang nektet».
> - **`numpy` er nå et krav** i SDK-et (live-frame-kallene returnerer numpy-matriser).
> - **Et eksportnivånavn som ikke er «one», avvises.** `processing=` og `levels=` tar `raw`, `debayered`, `radiance`, `reflectance` eller `display` (eller et alias som for eksempel `preview`; `processing=` samt `"all"`). Ethvert annet navn avvises før kameraet berøres (HTTP 400 `UNKNOWN_LEVEL`: `capture()` utløser `ChlorosConnectError`, `grab()` / `frame_stream()` `FrameFetchError`); 1.2.0 lagret eller returnerte `raw` for det.
> - **Texture Aware avvises for LATTICE-opptak.** I et prosjekt som inneholder LATTICE-opptak, utløser `configure(debayer="Texture Aware (Slow, Highest Quality)")` feilen `ChlorosConfigurationError`, og det samme gjør `process()` etter en `configure()` som ba om dette før importen. En «Texture Aware»-verdi som prosjektet lagret tidligere, settes til «Standard» med en `UserWarning`. LATTICE-opptak får alltid standard demosaik, så versjon 1.2.0 aksepterte innstillingen og ignorerte den; Chloros-appen tilbyr den heller ikke for disse. Se [Støttede verdier](#supported-values).
> - **Kameraer går tilbake til slik de ble funnet — med mindre `keep_settings=True`.** `ArraySession.disconnect()` / `force_disconnect()`, slutten av hver midlertidig funksjon (`capture_cameras_transient()`, `start_multi_camera_stream()`, `test_array_sync()`, `calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index()`) og avslutningen av `start_camera_continuous()` setter hvert kamera tilbake slik det var: utløser- og synkroniseringsledningsforbindelsene, PTP-en og alle kamerainnstillinger som ble endret under økten (eksponering og forsterkning, AE-mål, grenser og demping, hvitbalanse, svartnivå, pikselformat, binning og ROI, speilvendinger, bildefrekvens). `keep_settings=True` lagrer derimot hvert kameras innstillinger i kameraets eget minne, slik at det starter opp med disse; `clear_settings=True` tilbakestiller dette minnet til fabrikkinnstillingene. Uansett blir kameraene stående i frittløp. Et prosjekt konfigurerer kameraene sine ut fra sine egne lagrede innstillinger når det kobler dem til. Se [Kameraminne](#camera-memory--keep_settings--clear_settings).
> - **Alle moduser lagrer et prosjekt på samme måte.** Enhver endring på et kamera, en matrise eller en lyssensor som er tilkoblet under et prosjekt — fra SDK, appen, CLI eller MCP-serveren — lagres i prosjektets `cameras.json` / `sensors.json`, og når prosjektet åpnes i en hvilken som helst modus, konfigureres enhetene ut fra det; en `ChlorosProject` oppretter sin egen mappe i backenden. Tidligere var det kun appen som lagret prosjektet. Se [Én prosjektpost, alle moduser](#one-project-record-every-mode).
> - **Filer laget fra flere kameraer oppgir hvert enkelt kamera.** En multibåndstabel, en live-indeks laget av flere kameras bånd og et `composite/`-bilde fra en array-opptak inneholder hvert kildekameras serienummer, modell og kalibreringspakke som lister (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), og EXIF-listen `SerialNumber` inneholder serienumrene, atskilt med komma. Tidligere inneholdt de ingen kamerainformasjon, eller bare serienummeret til det første kameraet. Se [Filer laget fra flere kameraer](#files-made-from-several-cameras).
> - **Bare en `raw`-fil inneholder kameraets kalibreringstagger.** Det MicaSense-kompatible settet (`BlackLevel`, `RadiometricCalibration`, de-kanals `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`-settene) og nyttelasten `CalibrationJSON` beskriver sensorens råverdier, så de ligger kun i opptakfilen `raw`-opptaksfilen. Alle andre filer — en opptaksfil`debayered`, `display`, `radiance`, `reflectance` og indeksfiler, kontinuerlig opptak og flerkamera-strømrammer, samt alle `process()`-produkter — inneholder kameraets identifikasjon (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) og, ved en opptakssesjon, bildets egne opptakssdata (`XMP-mapir:FrameJSON`: eksponering, forsterkning, temperatur, pikselformat, opptakstid). I versjon 1.2.0 inneholdt de også rådatasettet, som et verktøy (Pix4D, Metashape) brukte på piksler som ikke var råverdier. Processing finner kalibreringen til en slik fil ved hjelp av serienummeret og `CalBundleSha256`.
>
> **Andre endringer:**
>
> - **En matrises automatiske bildefrekvens tilpasses det matrisen leverer.** En frekvens som mister mer enn 1 % av utløserne, reduseres – vurdert over flere lange observasjoner i stedet for én – og en frekvens den ble redusert fra, prøves på nytt av og til. En frekvens du selv angir, forblir uendret.
> - **Autoeksponeringen er mer stabil.** Den holder seg konstant ved jevnt lys, stabiliserer seg i løpet av sekunder etter en endring, ignorerer små høylys og gjenoppretter et kamera som slutter å svare i sterkt sollys. Se [Hvordan autoeksponeringen fungerer](#how-auto-exposure-behaves).
> - **`import_images(recursive=True)` teller kun dine bilder og opptak.** En mappe som inneholder et prosjekts `camera_cal/`, får ikke lenger disse kalibreringsfilene regnet med som bilder og opptak, og et prosjekt som allerede har tatt dem inn, fjerner dem.
> - **`configure(radiometric_output=...)` gjelder.** Den angir nå prosjektets `Export radiance` / `Export reflectance`-produkter; tidligere hadde hvert prosjekts egne produktinnstillinger forrang, slik at `radiance`, `sensor-response` og `all` alle eksporterte stråling og refleksjon.
> - **`test_array_sync()` starter fra kameraenes påslåtte tilstand.** Den slår på kameraenes PTP for kjøringen, venter på at klokkene deres skal låses og stabilisere seg før den første utløsningen (`ptp_lock`) — hvert kamera låses i 120 s, omtrent 2 minutter fra oppstart (`ptp_settle_s=` setter den), siden en klokke som nettopp er låst, avviker med opptil ~200 us i omtrent 2 minutter — og setter hvert kameras PTP tilbake slik den fant den. Et kamera som fortsatt er synkronisert fra en tidligere test, blir ikke ventet på igjen. Tidligere ble PTP holdt av på et nettverk med én bryter der ingen DAQ-E svarte, og hvert kameras frittløpende klokke ble lest som en PTP-klokke som ikke ville synkronisere seg («PTP er ikke låst», ved hver kjøring).
> - **En `.daq`-opptak angir om dekselet ble valgt eller antatt.** DAQ-U, DAQ-M og DAQ-E leveres uten hette, og Chloros kan ikke se om en er montert, så en sensor som er koblet til uten navngitt hette får `sunshine_cosine` som standard, en *antagelse*. Hver Chloros-logg (skjema 1.30) lagrer nå `cap_id_source` ved siden av `cap_id`: `operator` når du har gitt et deksel et navn (inkludert `none`), `auto_default` når den var standard. Behandling av en opptak der kondensatoren ble antatt, med prosjektets DAQ-kondensator satt til «Auto», gir en `[daq-dls] WARNING` som navngir filen: en ren DAQ-U uten solskinnskorreksjon viser en verdi som er omtrent 30 ganger for høy. **Gi navn til den kappen du har montert** når du kobler til (`connect_daq_sensor(cap_id=...)` eller `DAQSensorSession.set_cap()`, `chloros-cli daq pool-connect --cap-id` / `daq pool-set-cap`, eller kondensatorvalget i GUI-en), eller angi prosjektets DAQ-kondensator, så forsvinner advarselen.
> - **Ingen flere pip-konsollvinduer på Windows lenger.** På en PC med Python installert kjørte backenden og hver av dens bildebehandlere `pip` én gang ved oppstart, og hver åpnet et konsollvindu. Det skjer ikke lenger.
> - **Live `.daq`-opptak overlever et strømbrudd.**Et opptak skrives med en logg og en full disksynkronisering ved hver lagring, bak sensoren, slik at en treg disk forsinker filen, men aldri målingene. En opptak som ble stoppet midt i en commit (strømbrudd, avbrudd) kan etterlate `<name>.daq-journal` ved siden av seg:**kopier de to sammen.**Chloros ruller journalen tilbake første gang den leser filen, slik at opptaket åpnes med alt som ble bekreftet før stoppet (omtrent de siste 3 sekundene går tapt); i versjon 1.2.0 kunne et strømbrudd på feil tidspunkt gjøre hele filen uleselig («database disk image is malformed»).
> - **`DAQSensorSession.record_start()` på en opptatt disk fører ikke lenger til at sensoren går i stå**: målingene settes i kø og skrives i en egen tråd, og en som ikke kunne skrives, telles med i `status()`s `record_errors` (sammen med `record_last_error`).
> - **Direkte `LatticeCamera`:**et bilde uten eksponering/forsterkning per blokk rapporterer nå kameraets**faktiske** eksponering og forsterkning (reserveverdien leste en bufret verdi som kunne forbli uendret gjennom en hel automatisk eksponeringssyklus), og `apply_settings(trigger_source='LineN')` / `set_trigger_source('LineN')` bruker den samme 5 µs-inngangsdeglitch-behandlingen som Chloros’ kabelslaver får (ufiltrert: en slave avfyrte bilder uten at det ble sendt noen utløser).
> - **Programvareutløste kameraer dropper ikke lenger annenhver utløser.** Et kamera i programvareutløsermodus setter nå en utløser som ankommer under utlesningen i kø (eller så venter Chloros på at det skal være klart), slik som kabelsynkroniserte kameraer allerede gjorde.
> - **`discover_daq_sensors()` navngir en DAQ-E-S som `DAQ-E-S`.** Dets `model` var `DAQ-E` for hver Ethernet-sensor; en som ikke svarer på identitetssjekken innen ca. 2 sekunder, vises fortsatt på listen som `DAQ-E`.
> - **Nytt: `chloros-mcp`, en MCP-server for AI-assistenter.** Claude Desktop, Claude Code, Cursor og andre MCP-klienter kan behandle bildemateriale (opprette et prosjekt, importere opptak, velge innstillinger, behandle, rapportere hva kjøringen eksporterte) og betjene LATTICE-kameraer, matriser og DAQ-lyssensorer: koble til, forhåndsvise, stille inn eksponering og farge, ta opp, registrere. Verktøyene dekker det samme som dette SDK-et gjør. Det installeres sammen med SDK-et, og .deb-filen for Linux plasserer det i `/usr/bin`. Se [MCP-server (AI-assistenter)](#mcp-server-ai-assistants).
> - **Nytt: kameraverktøy**, `chloros-cli lattice`-kommandoer som SDK-kall: fargeprofil, metning/kontrast, høylys, hvitbalanse (engangsinnstilling, auto, faste forhold), forhåndsinnstillinger, filtermodeller, fastvarestatus og oppdatering, strøm (sjekk, hvilemodus, vekking), målkalibrering, nettverkskort (sjekk, estimering, reparasjon), vignettkorrigering på filer, en DLS-avlesning og vertssjekk. Se [Kameraverktøy](#camera-utilities).
> - **Nytt: resten av CLI som SDK-kall**, slik at CLI, SDK og `chloros-mcp` tilbyr de samme funksjonene (CLI beholder kun pålogging, konfigurering av en DAQ-E og de direkte DAQ-serverne, av sikkerhetshensyn): kameraer som åpnes med én kommando (`start_camera_continuous()`, `start_multi_camera_stream()`, `capture_cameras_transient()`, `test_array_sync()`, `detect_array_master()`, `analyze_array_models()`), justeringsprofilfiler (`calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `alignment_profile_info()`, `reorder_alignment_profile()`), indekser (`index_catalog()`, `compute_index()`, `capture_live_index()`), PTP-grandmasteren (`time_sync_status()` og tilhørende filer), prosjektmappen (`project_folder()`, `set_project_folder()`, `reset_project_folder()`) og oppskrifter (`run_project_recipe()`, CLIs egen kjører). For lyssensorer: `connect_daq_sensor(cap_id=...)`, `DAQSensorSession.set_cap()`, `daq_bindings()` / `set_daq_bindings()`, `daq_band_presets()` og `light_sensor_reflectance()` (to sensorers omgivelses- og objektrefleksjonsgrad samt indekser); for et kamera: `CameraSession.status()` og `capture_repeated()`; for et array: `ArraySession.capability()` og `set_member_settings()` (et elements eksponering, forsterkning, ...). Se [Transiente matriser, justeringsprofiler og indekser](#transient-arrays-alignment-profiles--indices), [Prosjektmappe og oppskrifter](#project-folder--recipes) og [DAQ-sensorsesjoner](#daq-sensor-sessions).
> - **Forbedrede LATTICE-indeksprodukter** (NDVI og andre indekser) fra `process()`, i alle behandlingsmoduser.
> - **LATTICE OCN-kameraer: forbedrede radianse-, reflektans- og indeksprodukter i sollys.** En flåtekorreksjon blir brukt etter båndseparasjonen — i sanntid, i opptak og i `process()`; RGN-kameraer er uendret. Se [Radiometrisk utdata](#radiometric-output-lattice-multispectral-pipeline).
> - **Hvert opptak lagrer DAQ-avlesningen det brukte, akkurat som GUI-en gjør.** `CameraSession.capture()` og `ArraySession.capture()` / `continuous()` skriver `.daq` og dets `.match.json` hver gang en DAQ blir bundet eller lagt i en pool, inkludert opptak som kun er rådata; i versjon 1.2.0 ble en rå eller debayert opptak lagret kun én gang med `force_daq=True`. `daq=False` slår det av, og `daq_tolerance_ms=` angir samsvarstoleransen; begge resultatene angir hva som ble lagret i `.daq_files` og `.daq_matches`. En opptak venter nå på DAQ-enhetens neste avlesning, slik at den kan blande avlesningene på begge sider av eksponeringen. Se [DAQ-avlesningen et opptak lagrer](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
> - **`ArraySession` har live-rammer og en kontinuerlig opptak på serversiden.** `grab()` og `frame_stream()` returnerer numpy-rammer fra én utløser (samme avlesning som prosjekt-API-ene `ArrayHandle.grab()` / `frame_stream()`), og `continuous()` lagrer hver synkronisert gruppe i backenden og returnerer en [`ContinuousHandle`](#continuoushandle). Se [Live-rammer i din egen pipeline](#live-frames-in-your-own-pipeline).
> - **En matrise `grab()` er festet til én utløser** (`ArrayHandle.grab()` og den nye `ArraySession.grab()`). I versjon 1.2.0 leser 12 av 20 firekamera-`ArrayHandle.grab()`-avlesningene spente over to utløsere. Hvert bilde har nå `group_id`, og en matrise `frame_stream()` gir hver utløser én gang. Se [Én utløser per avlesning](#one-trigger-per-read--group_id).
> - **Båndvise og justerte rammer:** `grab()` / `frame_stream()` tar `band=`, `align=`, `bit_depth=` og `refl_max=` på kamera- og matrisehåndtak. Se [Eksportere et enkelt bånd](#exporting-a-single-band--band).
> - **Nye `ArraySession`-metoder og -argumenter:** `set_target_fps()`, `force_disconnect()`, `capture(after_seq=..., orient=...)` (sammen med `CaptureResult.group_seq` og `.failed`), `continuous()`, `connect_array(packed=...)`; `capture_repeated()` kobler sammen påfølgende grupper og stopper når en enhet er full.
> - **`ArraySession.record()` registrerer uten at GUI er åpent.** Når GUI-ets kombinerte visning er lukket, gjengir backenden visningen selv: matrisensNDVI eller indeksen du sender som `expr=`. `RecorderHandle.stop()` utløser `ChlorosConnectError` for en opptak som mislyktes, en som ikke skrev noen ramme, og beholder stopp-resultatet på `.result` (å forlate en `with`-blokk utløser ikke: sjekk `rec.result`). Se [`RecorderHandle`](#recorderhandle).
> - **Nye `DAQSensorSession`-metoder:** `enable_ptp()`, `force_disconnect()`, og for en DAQ-E-enhets egne innstillinger `device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` og `update_firmware()`. `record_stop()` returnerer `warning` og `gaps`. En omstart som avsluttet en opptak, oppgir navnet på denne i sitt svar, eller, når omstarten mislykkes, i `ChlorosConnectError` (`recording_closed`; se [DAQ-E-enhetsinnstillinger](#daq-e-device-settings)). Se [`DAQSensorSession`-metoder](#daqsensorsession-methods).
> - **En samlet DAQ-E som slutter å levere data, kobles til igjen av backenden**, med de innstillingene den hadde, og en opptak på den fortsetter i samme `.daq` med gapet merket i filen; `record_stop()` returnerer hullene som `gaps`. I versjon 1.2.0 forble sensoren stille inntil den ble koblet fra og tilkoblet på nytt, noe som avsluttet opptaket. Se CLI-referansens [En DAQ-E som slutter å levere data](cli-reference.md#a-daq-e-that-stops-delivering).
> - **`open_project()` fungerer fra et installert SDK.** Hjulet (installasjonsprogrammetsog PyPI-ene) inneholder nå prosjektlasterne den trenger; tidligere utløste `open_project()` feilen `ImportError`, der `project_cameras` / `project_sensors` i ethvert pip-installert SDK. `connect_all()` kobler de lagrede kameraene, matrisene og lyssensorene gjennom et kjørende backend (lyssensorer gjennom sin DAQ-pool), og en lagret enhet den ikke har mulighet til å finne, vises under `errors` med en begrunnelse i stedet for å bli hoppet over uten forklaring. Et lagret array som allerede er oppe i backenden (GUI-ene, eller `chloros-cli project connect`s) kobles til og får stå på, der det tidligere ble rapportert som en feil («Kameraet  <sn>er allerede i matrisen<id>»). Se [`ChlorosProject`-metoder](#chlorosproject-methods).
> - **`ChlorosLocal()`, `process_folder()` og `process_lattice_capture()` finner Chloros på Windows.** De ser etter hvor installasjonsprogrammet plasserer backenden (`<install folder>\backend\`, `C:\Program Files\Chloros\backend\` som standard, en tilpasset mappe fra installasjonsprogrammets egen oppføring), og kobler seg til en backend som allerede kjører uten å lete etter en i det hele tatt. I versjon 1.2.0 ga de feilmeldingen «Chloros-backend ikke funnet» på en installert maskin, med mindre du overførte `backend_exe=`.
> - **En backend som krasjer ved oppstart, rapporteres umiddelbart.** Når `ChlorosLocal()` eller et smart-connect-anrop starter en backend som avsluttes før den svarer, utløser anropet innen sekunder feilmeldingene `ChlorosBackendError` fra `ChlorosLocal` og `ChlorosConnectError` fra smart--connect-anropene, med avslutningskoden, kommandoen den kjørte og de siste linjene backenden skrev ut. Versjon 1.2.0 ventet ut oppstartstimeouten på 60 sekunder uten å oppgi noen grunn. Se [Backend Auto-Start](#backend-auto-start).
> - **Backenden fortsetter å kjøre mellom skriptene.** Når man forlater en `with ChlorosLocal(...)`-blokk, `process_folder()` / `process_lattice_capture()` som returnerer og `chloros-mcp` som avsluttes, stopper ikke lenger en backend som SDK-et har startet; det neste skriptet gjenbruker den. Den stopper ved `logout()` (med mindre Chloros-appen er åpen eller en kjøring pågår), ved `shutdown_backend()`, eller av seg selv etter 30 minutter uten at noe bruker den (`CHLOROS_BACKEND_IDLE_MIN` i backendens miljø endrer dette; `0`: aldri).
> - **En backend som SDK-et har startet, blir bedt om å avslutte, ikke avsluttet tvangsmessig.** `shutdown_backend()` lar backenden frigjøre kameraene og lyssensorene sine og fullføre eventuelle opptak; den avsluttes bare hvis den fortsatt kjører 10 sekunder senere. Versjon 1.2.0 avsluttet den umiddelbart (på Windows, en tvangsavslutning), noe som kunne føre til at en DAQ-lyssensor koblet til den backenden ikke lenger svarte. En backend som SDK fant i gang, blir fortsatt ikke berørt.
> - **På Linux kjører automatisk oppstart aldri en `backend_server.py` fra den gjeldende mappen.** Uten installert backend kjørte et skript som ble startet i en mappe som inneholdt en, den; nå startes kun den installerte backenden (eller, for `ChlorosLocal`, den som `backend_exe=` angir).
> - **`configure(indices=...)`**legger til hver forhåndsinnstilling i prosjektet for kameraets bånd, inkludert et nytt prosjekt, og gir en advarsel (`UserWarning`) for et navn det ikke kan legge til.**`import_images()`** rapporterer `.daq`-opptak som `scan_count` og gir ikke lenger advarsel om en LATTICE-opptaksmappe.
> - **`read_image_audit_tags()` fungerer ved installasjon via pip.** Den leser bildets metadata selv; i versjon 1.2.0 returnerte den `{}` uten de interne komponentene fra skrivebordsinstallasjonen på Python-banen.
> - **`capture()` rett etter `connect_array()`** venter i opptil 20 sekunder på den første bildegruppen i serien i stedet for å gi feilmeldingen «HTTP 503: Array still warming up».
> - **`status()` rapporterer en stopp.** Det nye feltet `stalled` er True mens henteløkken kjører, men publiserer ingenting, og `actual_fps` er da 0. I 1.2.0 beholdt `actual_fps` sin siste verdi under en stopp, så en stoppet matrise viste `actual_fps > 0` mens opptak mislyktes med 503 «Ingen ny synkronisert ramme-gruppe».
> - **Færre ufullstendige rammesett og 503-feil ved opptak:** utløserinngangen til hvert slavekamera er nå renset for glitcher.
> - **Pakket 12-bits transport** (`connect_array(packed="auto")`) øker bildefrekvensen for hele sensorarrayet med omtrent en tredjedel. Se [Pakket 12-bits transport](#packed-12-bit-transport-packed).
> - **Array-eksponering under et ROI:** et lite ROI begrenser ikke lenger et medlems eksponering til sensorens friløpsperiode (4,9 ms ved 768 × 304). Se [Eksponering på array-medlemmer](#exposure-on-array-members).
> - **Automatisk eksponering reduserer ikke lenger arrayets automatiske bildefrekvens.** Et kabelsynkronisert array kjører med den hastigheten nettverket og kameraenes utlesning tillater; automatisk eksponering fyller perioden som denne hastigheten gir, og øker deretter forsterkningen. En manuell eksponering som er lengre enn perioden, bremser utløseren for å tilpasse seg den i stedet for å bli avvist. Se [Eksponering på array-medlemmer](#exposure-on-array-members).
> - **`read_nodes()`**legger til `frame_values` og `notes` mens automatisk eksponering kjører (se [`read_nodes()` under automatisk eksponering](#read_nodes-under-auto-exposure--frame_values-and-notes)). Når kameraet avviser en eksponering eller forsterkning, angir**`set_settings()`** området det aksepterer og gjenoppretter den automatiske modusen som ble slått av for den verdien; `set_settings(target_brightness_auto=True)` er ny.
> - **`orient=`** på `ArraySession.capture()` / `capture_fastest()` / `continuous()` / `burst()` og `ArrayHandle.capture()` / `capture_smart()` roterer produktene etter kameraets retning. Se [`ArraySession`-metoder](#arraysession-methods).
> - **`DLS()` krever sensorens egen `mac_address`.** Det finnes ikke lenger noen standard-MAC (det var adressen til én enhet); `DLS()` uten en slik utløser `DLSError`. Se [DAQ-M-båndstørrelser](#daq-m-band-magnitudes--dlsband_magnitudes).
> - **Windows-installasjonsprogrammet** tilbyr «Aktiver lange Windows-stier» og «Ekskluder Chloros-backend fra Microsoft Defender-skanning», begge er merket av som standard. Se [Trinn 1](#step-1--install-the-chloros-platform-package).
> - **Linux-.deb-filen** installerer Python-SDK-pakken for systemet `python3` (tidligere ble bare wheel-filen plassert på disken), øker kjernens mottaksbuffere for sokler som kamerastrømmer trenger, og legger til en backend-tjeneste som kjører under din bruker. Jetson-pakken er `chloros_1.3.0_arm64_jp6.deb`, for JetPack 6, med JetPacks CUDA PyTorch. Se [Trinn 1](#step-1--install-the-chloros-platform-package) og [Trinn 2](#linux-deb).

---

## Arkitektur på 60 sekunder

SDK-et er et tynt Python-lag over Chloros-backenden (den samme Flask-serveren som skrivebords-GUI-en og CLI-en bruker). For automatisering importerer du `chloros_sdk` og kaller metoder på høyt nivå; bak kulissene blir hvert kall en HTTP-forespørsel til den lokale backenden på port 5000 — `http://127.0.0.1:5000/api/...` (bevisst ikke `localhost`, som først løses til `::1` på Windows og tar ca. 2 sekunder per forespørsel mot en backend som kun støtter IPv4). Backenden eier maskinvarepoolen — kameraer, DAQ-sensorer, justeringsprofiler, rammebuffere — slik at SDK-skript kan eksistere side om side med GUI-en uten å måtte konkurrere om serielle porter eller nettverkskortbåndbredde.

Det er tre grensesnitt du vil bruke:

1. **`ChlorosLocal` + frie funksjoner** (`process_folder`, `process_lattice_capture`) – Bildbehandlingsrørledning. Kjør en hel mappe gjennom kalibrering / debayer / indekseksport fra ett Python-kall.
2. **Smart-connect-håndtak** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Åpne en vedvarende backend-økt for live-maskinvare. Samme «smart-prep»-flyt som i GUI: nettverkssondering, automatisk valg av lag, PTP, AE-seeding, GPIO-utløserkonfigurasjon.
3. **`ChlorosProject` / `open_project`** — Last inn et lagret prosjekt (mappe med `cameras.json` + `sensors.json` + `project.json`), koble til kameraene, matrisene og lyssensorene samtidig, og styr opptakene med navngitte håndtak.

Overflater 1 og 2 **starter automatisk en lokal backend** hvis det ikke allerede er en som lytter (den samme medfølgende binærfilen som GUI/CLI-en starter) — slik at et enkelt skript fungerer fra et nytt skall uten at du først må starte en backend: `ChlorosLocal` (og `process_folder()` / `process_lattice_capture()`), `connect_camera()`, `connect_array()`, `connect_daq_sensor()`, `discover_lattice_cameras()` og `discover_daq_sensors()`. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` og `analyze_array_network()` starter aldri av seg selv: de trenger et kjørende backend. Angi `auto_start_backend=False` for å velge bort (f.eks. når du peker mot en ekstern backend som aldri startes). Se [Automatisk oppstart av backend](#backend-auto-start). Surface 3 oppfører seg annerledes: `open_project()` tar ikke imot noen `auto_start_backend`-parameter, og `connect_all()` starter aldri en backend — den søker etter `http://127.0.0.1:5000` én gang, og hvis ingenting svarer, faller den tilbake til direkte (backend-fri) `lattice_sdk`-enhetskontroll. Dette krever Arena SDK-kjøretiden og OpenCV (`cv2`) som kan importeres i Python, noe som ikke installeres av SDK-hjulet; uten disse blir hvert lagret kamera og hver lagret matrise rapportert som en feil, med angivelse av årsaken. Et prosjekts lyssensorer har ingen backend-fri vei: uten en kjørende backend blir hver lagret sensor rapportert som en feil, med angivelse av årsaken (se [`SensorHandle`](#sensorhandle)). Bare `proj.process()` og `stream(..., overlays=True)` konstruerer en `ChlorosLocal()` (som starter automatisk) etter behov.

Alle tre er auth-avhengige: kjør `chloros-cli login` én gang på maskinen, eller logg inn via skrivebordsgrensesnittet. Hva et anrop uten en gyldig økt utløser, avhenger av grensesnittet (se [Unntak](#exceptions)): `ChlorosAuthenticationError` fra `ChlorosLocal` (og `process_folder()` / `process_lattice_capture()`), `ChlorosConnectError` fra smart-connect-kall og deres sesjoner, `status: "error"` med en `error_code` fra `analyze_array_network()`, og en oppføring under `errors` i `connect_all()`-rapporten.

Krav:
- Python 3.7+ (som angitt av pakken; utviklet/testet på 3.10)
- Chloros Desktop installert lokalt (backend-binærfilen følger med i installasjonsprogrammet)
- Aktiv Chloros+-pålogging på **et hvilket som helst betalt Chloros+-nivå (Copper eller høyere)**; det gratis**Iron**-nivået har ingen SDK-/CLI-tilgang. Dette håndheves**på serversiden****:** hver forespørsel merket med SDK/CLI må inneholde både en aktiv økt og et betalt abonnement. Et abonnement under Copper får feilkoden `403` sammen med `error_code: PLAN_UPGRADE_REQUIRED` (`ChlorosLicenseError` fra `ChlorosLocal`, `ChlorosConnectError` fra smart-connect-anropene). En utlogget innringer får i stedet `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError` fra `ChlorosLocal`, `ChlorosConnectError` fra smart-connect-samtalene) — de to er forskjellige fordi en ny kjøring av `chloros-cli login` løser problemet med `401`, men kan ikke løse problemet med `403`, som krever en oppgradering av abonnementet.
- Offlinebruk støttes innenfor abonnementets nådeperiode: nivået hentes fra servervalideringsbufferen (5 minutter) eller den signerte, maskinbundne lisensbufferen (30 dager for månedsabonnementer, til abonnementets utløp for årsabonnementer). Når denne fristen utløper, går abonnementet over til gratisversjonen, og tilgangen til SDK og CLI stanses inntil maskinen kan opprette kontakt med serveren én gang. Feilmeldingen `chloros-cli status` (`GET /api/license-status`) forblir tilgjengelig på gratisnivået, slik at årsaken er synlig.
- Windows 10/11 64-bit, **Ubuntu 22.04 LTS eller nyere**, eller Jetson (JetPack 6). Ubuntu 20.04 støttes**ikke**: avhengighetene til `.deb` er avledet fra det backenden kobler seg mot, inkludert `libc6 (>= 2.35)`, og Focal leveres med glibc 2.31.

---

## Installasjon

Python-SDK-et er et tynt Python-lag over Chloros-backenden. Med mindre du kjører en backend på en annen maskin (se [Fjern-backend-modus](#remote-backend-mode-pip-only-host-via-tunnel)), må du ha **Chloros-skrivebordspakken installert lokalt** (Windows-installasjonsprogram eller Linux `.deb`), og den inkluderer DAQ-lyssensorer. Pakken inneholder backend-binærfilen, Arena SDK-kjøringsmiljøet for LATTICE-kameraer og kalibreringspakkene.

Siste nedlastinger: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Trinn 1 — Installer Chloros-plattformpakken

#### Windows (.exe)

1. Last ned `Chloros Installer 1.3.0.exe` fra nedlastingssiden.
2. Kjør installasjonsprogrammet og følg veiviseren. Standard installasjonssti er `C:\Program Files\Chloros\`.
3. Start Chloros minst én gang og logg inn med din Chloros+-konto.

Installasjonsprogrammet lar backend-delen (`chloros-backend.exe`, ingenting annet) gjennom Windows-brannmuren på alle nettverksprofiler: UDP inn, for kamerastrømmer, PTP, mDNS og DAQ-E-strømmen, og TCP inn kun fra det lokale subnettet, for tilkoblingen en DAQ-E oppretter tilbake til datamaskinen under en fastvareoppdatering ([`update_firmware()`](#daq-e-device-settings)). Avinstalleringsprogrammet fjerner begge reglene. Backendens HTTP-API forblir på `127.0.0.1` uansett.

Veiviserens **Systeminnstillinger**-gruppe har to alternativer, begge merket av som standard:

| Alternativ | Hva det gjør | Avslå stille installasjon |
| --- | --- | --- |
| **Aktiver lange stier i Windows** | Lar prosjektmapper ligge dypere enn Windows’ grense på 260 tegn for stier. Når lange stier er deaktivert (Windows-standard), avvises en prosjektmappe som ligger for dypt for produktene før noe behandles — se [Feilsøking](#feilsøkings-tips). | `/MERGETASKS="!longpaths"` |
| **Ekskluder Chloros-backend fra Microsoft Defender-skanning** | Stopper Defenders sanntids-tidsskanning av hver fil som en kameraopptak skriver. Ved høye opptakshastigheter koster skanningen omtrent 30 % av hastigheten: målt ved ROI 512 på et oppsett med 9 kameraer, kjørte kontinuerlig RAW-opptak 52,7 opptak/s med skanning og 77,7 opptak/s med backend ekskludert. | `/MERGETASKS="!defenderexclusion"` |

En kontinuerlig opptak rapporterer Defenders belastning som `defender_cores` (se [`ContinuousHandle`](#continuoushandle)).

#### Linux amd64 (.deb)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Installer med `apt install ./…`, ikke `dpkg -i`: apt installerer også kravene til Python-SDK, som pakken anbefaler snarere enn krever (se [Trinn 2](#linux-deb)).

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

For Jetson Orin / Orin NX / Orin Nano på JetPack 6 (L4T R36). PyTorch-versjonen er JetPack 6s CUDA-versjon, så GPU-behandlingen kjører på Orins GPU; den bruker CUDA- og cuDNN-bibliotekene som JetPack 6 installerer (`sudo apt install nvidia-jetpack` på et bilde som mangler dem).

#### Linux: mottaksbuffere og backend-tjenesten

Begge .deb-filene øker kjernens mottaksbuffere for sokler, som LATTICE-kamerastrømmer trenger, til 32 MB (`/etc/sysctl.d/60-chloros-gige.conf`). Med kjernens standardinnstilling blir kamerapakker droppet i kjernen, og en array mister bilder og bildefrekvens; `chloros-cli lattice network` rapporterer når buffere er lave. Se CLI-referansens [Linux-verter — socket-mottaksbuffere](cli-reference.md#linux-hosts--socket-receive-buffers).

På en maskin uten skjerm kjører du backenden som en tjeneste **under din bruker**, slik at et skript finner en som kjører (`connect_all()` starter aldri en backend):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
```

Backenden kjører da som deg: opptakene den skriver tilhører deg, og den bruker din `chloros-cli login`. Root-`chloros-backend.service` er fortsatt installert for maskiner som har aktivert den, men hver fil en root-backend skriver tilhører root. Se [Kjøre backend som en tjeneste](cli-reference.md#running-the-backend-as-a-service-headless) i CLI-referansen.

En backend som tjenesten kjører, er delt og stoppes aldri av en klient: SDK-skript og `chloros-cli` bruker den slik den er (`ChlorosLocal.shutdown_backend()` stopper bare en backend som `ChlorosLocal` selv har startet), og `GET /api/status` navngir enheten som `service`. Start den på nytt med `sudo systemctl restart chloros-backend@$USER.service`. Oppgradering av pakken stopper backend-tjenestene mens den installeres, slik at et skript som kjører mister kameraene og sensorene sine, og deretter starter alle de aktiverte på nytt; når pakken fjernes, stoppes de og de aktiverte deaktiveres. Bare én backend kjører om gangen, så aktiver én enhet: en annen nekter å starte og forblir `failed` (avslutningsstatus 98, ikke startet på nytt), og dens `systemctl status` forklarer hvorfor. For å bytte enhet må du deaktivere den du ikke bruker.</id></sn>

### Trinn 2 — Installer Python-SDK

**Chloros-installasjonsprogrammet leveres med et tilhørende SDK-wheel.** Hvert Windows-installasjonsprogram og hver Linux-.deb-pakke plasserer en `chloros_sdk-X.Y.Z-py3-none-any.whl`-fil på disken som stemmer nøyaktig overens med GUI-/CLI-/backend-versjonen. Du trenger ikke å følge med på PyPI for å holde deg oppdatert.

#### Windows

Installasjonsprogrammet kjører automatisk `pip install` mot det medfølgende wheel-filen ved hjelp av systemets Python (`py.exe`-lanseringsprogram foretrekkes, faller tilbake til `python -m pip`). Ingen handling er nødvendig – `import chloros_sdk` fungerer i ditt Python-miljø etter en vellykket installasjon. Hvis det ikke finnes Python på maskinen, hopper installasjonsprogrammet stille over dette trinnet, og GUI + CLI fortsetter å fungere.

#### Linux (.deb)

Begge .deb-filene (x86_64 og Jetson) installerer SDK-et for systemet `python3`, slik at `import chloros_sdk` fungerer etter installasjonen, uten pip og uten nettverk. Pakkene (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` og prosjektlasterne) legges inn i `/usr/lib/python3/dist-packages`, og SDK-kommandoen `chloros-mcp` ([MCP-serveren](#mcp-server-ai-assistants)) i `/usr/bin`, som alle tilhører pakken: en oppgradering erstatter dem, og når pakken fjernes, fjernes også disse.

SDKets egne krav, `python3-numpy` og `python3-requests`, er **anbefalt, ikke påkrevd** (GUI, CLI og backend er selvstendige): `sudo apt install ./chloros_<version>_amd64.deb` (`…_arm64_jp6.deb` på en Jetson) installerer dem, mens `dpkg -i` ikke gjør det. Installasjonsutskriften viser hvilken SDK-versjon `python3` importerer, eventuelle manglende krav, samt en `chloros_sdk` et annet sted i `python3`s søkebane som overskygger pakkens kopi (for eksempel en som ble installert tidligere med `pip install --user`).

I et virtualenv, som ikke har tilgang til systemets pakker, installerer du det medfølgende wheel-filen i det:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

For Jetson-distribusjoner med air-gap foregår dette helt offline — SDK-et og wheel-filen ligger allerede på disken.

#### Offentlig PyPI

For verter som kun bruker pip (ingen Chloros-skrivebordspakke installert; SDK-et driver da en backend på en annen maskin, se [Remote-Backend Mode](#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

PyPI inneholder den nyeste stabile utgaven. Sørg for at SDK-en har samme versjon som Chloros-backenden den styrer (se [Versjonering og kompatibilitet](#versioning--compatibility)). DAQ-M-hjelpeverktøyet `DLS` trenger også `bleak`: `pip install "chloros-sdk[camera]"` (eller `pip install bleak`).

#### Bekreft

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Chloros+-abonnement kreves.** Alle SDK-kall krever en aktiv Chloros+-pålogging. Kjør `chloros-cli login user@example.com 'YourPassword'` én gang per maskin; påloggingsopplysningene lagres i `~/.chloros/`.

### Trenger jeg skrivebordspakken?

Pip-pakken alene er **ikke** tilstrekkelig for de fleste arbeidsflyter. Her er hva hver SDK-flate trenger:

| SDK-flate | Trenger skrivebordspakke? | Hvorfor |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Ja**(for å starte en)**/ Nei** (en backend som allerede er tilgjengelig) | Bruk en kjørende backend (desktop-appens, eller en annen startet skript); ellers start den installerte automatisk: `/usr/lib/chloros/chloros-backend` (Linux) eller `<install folder>\backend\chloros-backend.exe` (Windows; `C:\Program Files\Chloros\backend\` som standard, en tilpasset installasjonsmappe hentes fra installasjonsprogrammets oppføring). Overfør `backend_exe=` for å gi den et navn selv. |
| `chloros-mcp` ([MCP-server](#mcp-server-ai-assistants)) | **Ja**(for å starte en)**/ Nei** (en backend er allerede tilgjengelig) | Bygget på `ChlorosLocal`, altså det samme som raden over. |
| `read_image_audit_tags` | **Nei** | Leser det bearbeidede bildets egne metadata (kun Python-standardbiblioteket). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Ja**(lokalt)**/ Nei**(eksternt) | Rene HTTP-klienter via backenden. Lokal backend → skrivebordspakke kreves. Ekstern backend → `backend_url=`**gjennom en tunnel** (se Ekstern-backend-modus — medfølgende backender binder kun loopback). |
| `ChlorosProject` / `open_project` | **Ja** | Styrer lagrede prosjekter via backend. |
| Direkte LATTICE-klasser (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Ja** | Krever Arena SDK-kjøretid, som følger med i skrivebordspakken, og OpenCV (`cv2`) i Python-miljøet ditt, noe verken skrivebordspakken eller SDK-pakken installerer (`pip install opencv-python`). `CAMERA_AVAILABLE` er ellers `False` ved import. `DLS` trenger også `bleak`. |

### Fjern-backend-modus (pip-only-vert, via tunnel)

> **Den installerte backenden er ikke tilgjengelig over LAN.** Den lytter kun på
> loopback (`127.0.0.1` / `::1`), så `backend_url="http://<lan-ip>:5000"`
> **kan ikke nå en installert Chloros**. For å kjøre en backend på en annen
> maskin, må du selv videresende loopback-porten og peke SDK-et mot
> tunnelen:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"     # the tunnel endpoint
serials = ["213800234", "214000533"]   # master first

# auto_start_backend=False: the tunnel URL looks local, so without it a tunnel
# that is down would start this machine's own backend, if one is installed.
arr = chloros_sdk.connect_array(serials, backend_url=BACKEND,
                                auto_start_backend=False)
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                     backend_url=BACKEND,
                                     auto_start_backend=False)

# The backend opens every path on ITS machine: pass absolute paths valid there.
# (A relative path would be made absolute against this machine's working folder.)
arr.capture("/home/chloros/captures/run1", processing="raw")
```

Headless-/CI-/robotikk-verter kan ha én maskin med full skrivebordsinstallasjon som «Chloros-server» og `pip install chloros-sdk` alle andre steder — men transporten mellom dem er den brukerdefinerte tunnelen ovenfor, ikke en direkte LAN-URL.

> **`ChlorosLocal` på en vertsmaskin som kun kjører pip.** `ChlorosLocal(backend_url=BACKEND)` kobler seg til en backend den kan nå uten en skrivebordsinstallasjon: den søker etter en lokal backend-binærfil bare når den må STARTE en (ingenting svarer og `auto_start_backend=True`). Frem til versjon 1.2.0 søkte den først etter binærfilen og utløste feilmeldingen `ChlorosBackendError` («Chloros-backend ikke funnet…») på en pip-only-vert, selv om en backend var tilgjengelig. Mappene du angir (`process_folder`, `import_images`) sjekkes på denne maskinen og leses av backenden, så med en ekstern backend må de peke på den samme mappen på begge, for eksempel en delt stasjon.

### Arbeidsflyt kun for DAQ (pip-only-vert)

`pip install chloros-sdk` alene kan ikke styre en DAQ-sensor: en DAQ-sensor styres alltid gjennom en Chloros-backend med `connect_daq_sensor()`:

- **Sensor på denne maskinen:** installer Chloros-skrivebordspakken her. `connect_daq_sensor()` starter sin backend hvis ingen kjører.
- **Sensor på en annen maskin:** den maskinen trenger skrivebordspakken; nå backend-en via tunnelen i [Remote-Backend Mode](#remote-backend-mode-pip-only-host-via-tunnel). Backend-en åpner sensoren på sin egen maskin, så en DAQ-U er koblet til den maskinen, en DAQ-M er innenfor Bluetooth-rekkevidde av den, og en DAQ-E er på et nettverk den kan nå.

```python
import chloros_sdk

# http://127.0.0.1:5000 is the local desktop install's backend, or the tunnel.
with chloros_sdk.connect_daq_sensor() as daq:   # smart-detect USB / BLE / ETH
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Uansett håndhever backenden «Chloros+»-planen: logg inn på maskinen som kjører backenden (`chloros-cli login` eller skrivebordsappen) med et hvilket som helst betalt abonnement (Copper eller høyere). En utlogget backend og et abonnement under «Copper» utløser begge `ChlorosConnectError` med backendens begrunnelse (`AUTH_REQUIRED` / `PLAN_UPGRADE_REQUIRED`).

---

## Hurtigstart

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
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
chloros_sdk.CaptureResult                         # list subclass returned by CameraSession.capture / ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job
chloros_sdk.ContinuousHandle                      # handle for an array continuous() capture
chloros_sdk.GrabResult                            # dict returned by grab() / yielded by frame_stream()
chloros_sdk.FrameFetchError                       # raised by grab()/frame_stream() on a refusal

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()
chloros_sdk.daq_bindings() / set_daq_bindings(cameras=, arrays=)   # which DAQ feeds which camera
chloros_sdk.daq_band_presets(filter_code=None)    # band sets for Ambient + Object reflectance
chloros_sdk.light_sensor_reflectance(ambient_id, object_id, ...)   # two DAQs -> reflectance + indices
chloros_sdk.time_sync_status() / time_sync_peers() / time_sync_cameras()   # PTP grandmaster
chloros_sdk.restart_time_sync() / set_time_sync_priority(priority1=, priority2=)

# === Camera utilities (the CLI's lattice commands; by serial) ===
chloros_sdk.camera_color_profile(serial) / set_camera_color_profile(serial, ...)
chloros_sdk.camera_color_adjust(serial) / set_camera_color_adjust(serial, ...)
chloros_sdk.camera_highlights(serial) / set_camera_highlights(serial, value)
chloros_sdk.camera_white_balance_once(serial) / set_camera_white_balance(serial, ...)
chloros_sdk.camera_presets() / apply_camera_preset(serial, preset)
chloros_sdk.camera_filter_models() / camera_filter_model_info(name)
chloros_sdk.camera_firmware_status() / update_camera_firmware(serial)
chloros_sdk.camera_power_probe(serial) / sleep_camera(serial) / wake_camera(serial)
chloros_sdk.calibrate_camera_with_target(serial, ...)
chloros_sdk.network_adapters(...) / fix_network_settings()
chloros_sdk.apply_vignette_correction(input_dir, output_dir, ...)
chloros_sdk.read_dls(mac, ...) / probe_system(...) / system_info()

# === Transient cameras, alignment profiles, indices (cameras opened for one command) ===
chloros_sdk.start_camera_continuous(serial, ..., daq=None, daq_tolerance_ms=None)  # → CliJob; a DAQ bound to the camera records one .daq per frame (daq=False: none)
chloros_sdk.start_multi_camera_stream(serials, ...)   # → CliJob
chloros_sdk.capture_cameras_transient(serials, ...)
chloros_sdk.test_array_sync(serials, count, ptp_settle_s=None) / detect_array_master(line) / analyze_array_models(models)
chloros_sdk.calibrate_alignment_profile(serials, save_path, ...)
chloros_sdk.capture_aligned_multiband(profile_path, ...) / start_aligned_stream(profile_path, ...)  # → dict / CliJob
chloros_sdk.alignment_profile_info(path) / reorder_alignment_profile(path, ...)
chloros_sdk.index_catalog() / compute_index(input_path, ...) / capture_live_index(profile_path, ...)
chloros_sdk.CliJob                                # stats() / stop() / wait()

# === Project lifecycle ===
chloros_sdk.open_project(path, relink=None)       # → ChlorosProject
chloros_sdk.project_folder() / set_project_folder(path) / reset_project_folder()
chloros_sdk.run_project_recipe(recipe, project_path)  # `chloros-cli project run`
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics               # DLS: DAQ-M over Bluetooth; needs bleak

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosProjectFilesMissingError      # a ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True when chloros_sdk.project imported; open_project()
                                 # still raises ImportError without the project loaders
```

---

## Bildbehandling — `ChlorosLocal`

Hovedklassen for pipelinen. Starter backenden når den konstrueres (hvis ingen svarer og `auto_start_backend=True`), oppretter/konfigurerer prosjekter, overvåker fremdriften og returnerer sammendrag etter kjøring.

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
| `open_project(project_path, relink=None)` | Åpner et lagret prosjekt (mappen det ligger i, eller navnet på prosjektet i prosjektmappen) og gjør det til det åpne prosjektet. Stiene til et flyttet prosjekt pekes på nytt mot mappen det nå befinner seg i; `relink` angir navnet på mappen der de andre manglende filene nå befinner seg. Returnerer backendens svar med `source_files`; utløser `ChlorosProjectFilesMissingError` når ingen kan finnes. Se **Et flyttet prosjekt** nedenfor. |
| `import_images(folder_path, recursive=False)` | Importer RAW/TIF/JPG/DNG-bilder **og `.daq` lyssensoropptak**. Returnerer `count` (bilder) og `scan_count` (opptak). Gir kun advarsel hvis mappen ikke inneholder noen av delene. |
| `export_light_sensor(daq=True, csv=True)` | Skriv kalibrerte `.daq` + `.csv` for hver lyssensoropptak i prosjektet, til `<project>/Light Sensor/`. Se [Lyssensoropptak](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Still inn behandlingsknappene. `indices` er forhåndsinnstilte navn (`NDVI`, `GNDVI`, `OSAVI`, …), som hver er lagt til i prosjektet for kameraets bånd; et navn som ikke er en forhåndsinnstilling, eller som båndene ikke kan danne (`NDRE` trenger RE og NIR, noe ingen enkeltkamera tilbyr), får navnet `UserWarning` og ingen indeksbilde. `daq_log_path` betegner en opptak fra en lyssensor (en `.daq`- eller `.csv`-fil, eller en mappe med slike filer) som hvert bilde blir sammenlignet med, både i LATTICE og Survey3, i stedet for opptakene som er importert sammen med bildene; hvis en sti mangler eller ikke inneholder noen opptak, brukes de importerte opptakene. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Kjør pipelinen. Returnerer `{"status": "complete", "async": False}`, pluss en `summary`-nøkkel når backenden leverer en — se [Oppsummering og tips etter kjøring](#post-run-summary--hints) — `export_check_failure` når eksportene fra kjøringen ikke kunne verifiseres, og `image_failure` når et bilde gikk tapt. `wait=False` returnerer `{"status": "started", "async": True}` på en gang. |
| `get_config()` / `get_status()` / `status()` | Kontroller tilstanden til backenden. |
| `logout()` | Logg ut: tømmer de bufret påloggingsopplysningene og frigjør denne maskinens enhetsplass (`device_deregistration` i resultatet). En pågående behandlingskjøring stoppes ikke: den fullføres i den modusen den startet i (`run_in_progress: true` og `run_mode` i resultatet), og neste kjøring bruker standardbehandling. |
| `shutdown_backend()` | Stopp backenden hvis denne instansen startet den; en backend som den fant i gang, blir ikke påvirket. Å etterlate en `with`-blokk eller at instansen blir gjenstandsoppryddet, stopper den ikke: backenden fortsetter å kjøre for det neste skriptet og stopper seg selv etter 30 minutter uten at noe bruker den. Backenden blir bedt om å slå seg av, så den frigjør kameraene og lyssensorene sine og fullfører eventuelle opptak først; den avsluttes bare hvis den fortsatt kjører 10 sekunder senere. |
| `discover_cameras()` | Oppdag LATTICE-kameraer **via denne instansens backend** (`/api/camera/discover`). Returnerer en liste med ordbøker (`serial`, `model`, `ip`, …) — samme struktur som GUI/CLI ser. Tom liste hvis ingen ble funnet eller backend ikke kan nås. |
| `camera_capture(output_dir, format="tiff", **settings)` | Ta opp et enkelt bilde**gjennom backenden**(startes automatisk av dette håndtaket) slik at det får samme forberedelse som GUI/CLI (12-bit standard, gjenbruk av pool, innebygde kalibreringsmetadata). Bestem målet med `serial=` eller `device_index=`; send `exposure`/`gain`/`pixel_format`/`preset` som `**settings`. Returnerer den eldre metadatadiktet (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Genererer forhåndsvisningsbilder sammensatt av overlegg fra et samlet kamera — en lett MJPEG-klient over backendens `/api/camera/<serial>/stream-annotated`-rute (zebra / rutenett / siktemerk / histogram / peaking / punkt tegnet på serversiden). `decode=True` genererer BGR-matriser og krever OpenCV (`cv2`), som SDK-et ikke installerer; `False` genererer rå JPEG-bytes. Også tilgjengelig per prosjekt som `ChlorosProject.stream(overlays=True)`. |

**En kjøring som er i gang beholder sitt prosjekt.** Mens en Processing-kjøring pågår – enten dette skriptets, appens, `chloros-cli process`s eller `chloros-mcp`s – nekter backenden å opprette, åpne, importere til eller endre prosesseringsinnstillingene for et prosjekt (HTTP-feil 409 `PROCESSING_IN_PROGRESS`): `create_project()` og `import_images()` utløser `ChlorosProcessingError`, og `configure()` utløser `ChlorosConfigurationError`, og `process()` med en annen modus enn den som kjører, utløser `ChlorosProcessingError` (en kjøring beholder modusen den startet i), hver med backendens melding. En opptak som er lagret i prosjektet, blir deretter lagt til når kjøringen avsluttes (svaret `project_add` sier dette). Vent til kjøringen er ferdig (`get_status()`), og fortsett deretter.**Et flyttet prosjekt.** Et prosjekts `project.json` navngir hver kildefil (hvert bilde og hver lyssensoropptak) med den fulle banen, så hvis du flytter prosjektmappen – eller hele prosjektmappen til en annen stasjon – vil de fortsatt henvise til den gamle plasseringen. Hver gang du åpner prosjektet, rettes det opp det som er mulig automatisk: en fil som tidligere lå i prosjektmappen, blir søkt etter i den mappen prosjektet nå befinner seg i, og hver sti som finnes der, blir omdirigert og lagret (`project.json` blir først sikkerhetskopiert som `project.json.before-relink-<date>-<time>`). En sti blir kun omdirigert til en fil som faktisk finnes der. Filer som lå utenfor prosjektmappen – en hub-økt, en «Legg til mappe»-import – finnes kun med `relink=<the folder they are in now>` (eller en hvilken som helst mappe over den). `open_project()` returnerer `source_files`: `total`, `found`, `missing`, `missing_roots` (mappene de manglende filene befant seg i), `missing_outside_project` og `relink` (hva som ble omdirigert, fra hvor til hvor). Et prosjekt der noen filer mangler, åpnes sammen med resten; et prosjekt der ingen filer ble funnet, åpnes ikke, og `open_project()` utløser `ChlorosProjectFilesMissingError` (`.missing_roots`, `.audit`; HTTP 409 `PROJECT_SOURCES_MISSING`). CLIs `project open --relink` og `chloros-mcp`’`open_project` (`relink_folder`) gjør det samme.

Bruk som kontekstbehandler for garantert opprydding:

```python
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

### Lyssensoropptak — kalibrert `.daq` + `.csv`

En DAQ-U / DAQ-M / DAQ-E kan registreres **uten** kalibreringspakken. Det er
det de offentlige [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
opptakere (`record_daq.py`) gjør som standard: de skriver ut rå sensortall og merker
filen slik at Chloros henter sensorens fabrikkalibrering **etter serienummer** — først fra lokal cache
først, deretter MAPIR-skyen — og bruker den ved import.

Chloros skriver resultatet tilbake som to produkter per opptak, under
`<project>/Light Sensor/`:

| Produkt | Hva det er |
| --- | --- |
| `<name>_calibrated.daq` | Det arkivet som kan bearbeides på nytt — samme skjema som et live-opptak, men nå med angivelse av pakken som produserte det. Å importere det på nytt fører **ikke** til at det kalibreres en gang til. |
| `<name>_calibrated.csv` | Spektral irradians i W/m²/nm på sensorens eget bølgelengderaster, én rad per måling, pluss fotometriske kolonner (total effekt, fotopisk/skotopisk lux, PPFD og dens oppdeling i blått/grønt/rødt, toppbølgelengde). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Kun sensorer uten pakke (DAQ-A).** Rå spektrale sensortellinger — *ikke* irradians. Se nedenfor. |

`<name>` er opptakets eget filnavn. To opptak i forskjellige mapper kan dele
én — målinger per opptak (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) lagret i
samme millisekund av to kameraer — og hver får likevel sine egne produkter: den første beholder
`<name>`, den neste legger til mappene som skiller den ut **etter** navnet
(`<name>__<capture>_daq_calibrated.daq`), slik at opptakets dato og klokkeslett fremdeles står først.

`process()` utfører denne eksporten som ett av trinnene. Det krever **ikke** bilder:
en lyssensor som flys alene er en førsteklasses arbeidsflyt, og et slikt prosjekt har null
bilder i utgangspunktet.

**DAQ-A-opptak eksporteres som rå tellinger.** DAQ-A-familien er eldre enn det serienummerbaserte
pakkesystemet og har ingen pakke å hente – den kalibreres i stedet i felten mot et
refleksjonsmål, og det er grunnen til at den aldri har trengt en. Disse opptakene eksporteres
med filstammen `_raw` i stedet for `_calibrated`: et annet filnavn i stedet for et flagg
inne i filen, fordi opplysningen må overleve å bli sendt videre via e-post som et rent filnavn.
`.csv`-overskriften angir `raw spectral sensor counts (NOT irradiance)` og advarer om at
verdiene er sammenlignbare **innenfor** filen – akkurat det målbasert kalibrering bruker
dem til — og ikke på tvers av sensorer. De effektavhengige fotometriske kolonnene (total effekt,
fotopisk/skotopisk lux, PPFD) returneres som **NULL** i stedet for å bli integrert fra tellinger.

En DAQ-U / DAQ-M / DAQ-E hvis pakke ganske enkelt ikke kunne hentes, blir fortsatt **hoppet over**,
ikke skrevet i råformat: der finnes pakken, og «koble til på nytt og behandle på nytt» er et reelt råd.

Eldre **v1.01 / v1.02**-opptak (en DAQ-A-SD skriver disse) har ingen epoke per avlesning,
bare filens skrivetid. Bildet↔nedstrømningsmatcher avviser dem fortsatt — å matche en
ramme mot et skrivetidspunkt ville være feil uten at man ser det — men eksportøren leser dem, og
filen CSV-utskriften viser `clock=daq_created_on`, slik at produktet angir hvilken klokke det er på.

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

En opptak der kalibreringspakken ikke kan hentes (offline, eller en sensor uten
kalibrering i filen) rapporteres under `skipped` **med årsaken**. Den blir aldri
skrevet ut som en «kalibrert» fil som inneholder rå tellinger – koble til internett og
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

Når prosessen er fullført, henter `process()` `GET /api/processing-summary` og legger ved brødteksten som `result["summary"]`. Hentingen skjer etter beste evne og blokkerer aldri en vellykket retur — hvis sammendraget ikke er tilgjengelig, faller `process()` tilbake til den enkle `{"status": "complete", "async": False}`-formen. Hver oppføring i `summary["hints"]` — hele setninger med foreslått utbedring, f.eks. hvorfor en kjøring ga null utdata — blir også sendt ut på nytt som en Python-`UserWarning`, slik at kjøringer med null utdata er selvdiagnostiserende selv om du aldri sjekker ordboken:

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
| `targets_found` | Oppdagede refleksjonsmål. |
| `images_calibrated` | Bilder som kjøringen skrev et reflektansprodukt for. |
| `images_failed` | Bilder som mislyktes og ikke ble eksportert. |
| `exported_files` | **Bildeproduktfiler kjøringen skrev**: alle produkter (debayered, forhåndsvisning, radiance, reflektans, indeks, målrenderinger), i alle formater. |
| `reflectance_files` | Reflektansproduktene blant dem (`Reflectance_Calibrated_Images`), uansett om de ble kalibrert av et mål eller lyssensorens (DAQ) irradians. En kjøring uten mål som likevel skrev ut reflektans, angir dette i et hint; kun en kjøring som ikke skrev ut noe, får beskjed om at reflektansutdataene mangler. |
| `clamp_masks_written` | [Unmix-klemmasker](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) som er angitt ved siden av dem — ledsagere, ikke produkter, så de er ikke i `exported_files` (CLI-en teller dem separat på samme måte). |
| `captures_left_out` | Råbilder i prosjektets `captures`-mappe som denne kjøringen ikke tok. En kjøring venter ved oppstart på det som ble tatt opp før den, så dette er 0 med mindre en opptak ble lagret under kjøringen eller ventetiden gikk ut. `summary["captures_left_out_note"]` angir hva som skal gjøres med dem. |
| `daq_recordings_exported` / `daq_recordings_skipped` | Opptak fra lyssensorer, som bevisst telles separat — de kommer fra et annet trinn og finnes for kjøringer uten bilder i det hele tatt, så å inkludere dem ville få en ren DAQ-kjøring til å se ut som om den eksporterte bilder. |

Ved siden av disse: `summary["output_dirs"]` (hver katalog det er skrevet til),
`summary["light_sensor_export"]`, `summary["stopped"]` (sant når brukeren avbrøt
kjøringen, slik at delvise tellinger ikke tolkes som et fullført løp med for lav produksjon), og
`summary["groups"]` (fordelingen per gruppe).

`exported_files` registreres av rørledningen **mens den skriver**, ikke skannes fra
prosjektets bildeobjekter i etterkant. Parallell- og GPU-strategiene bygger sine egne bildeobjekter
(i arbeiderunderprosesser for GPU-banene), så den gamle skanningen rapporterte
`0 file(s) written` for hver slik kjøring og sendte deretter ut «zero-exports»-hintet — på kjøringer
der alt hadde fungert. Hvis du skriver skript basert på dette tallet, rapporterer en vellykket parallellkjøring nå
et antall som ikke er null.

Hopp i lyssensoren rapporterer årsaken som leseren faktisk fastslo for hver fil – et
uleselig skjema, en manglende pakke, en skrivefeil – **deduplisert**, slik at tjue filer
som ble hoppet over av én årsak, vises som én årsak i stedet for tjue gjentakelser av den.

> **`process()` utløses ikke når en kjøring ikke produserer noen bilder.** Dette er ett av de to stedene hvor SDK og
> CLI bevisst skiller seg fra hverandre: `chloros-cli process` behandler «produkter ble forespurt, ingen ble
> skrevet» som en feil og avsluttes med et tall større enn null, mens SDK returnerer normalt og rapporterer
> tilstanden via `summary` / hints. Hvis pipelinen din skal stoppe ved en tom kjøring, må du sjekke det
> selv — undersøk `summary` (eller tell filene i prosjektmappen) i stedet for å stole på
> fraværet av et unntak. De vanligste årsakene er en inndatamappe som ikke ble gjenkjent som en
> opptakskilde, og produkter som ble hoppet over fordi de ikke var relevante for de tilstedeværende kameraene (f.eks. radianse fra kameraer som kun bruker RGB
> ).
>
> `process()` utløses heller ikke når eksportene fra en kjøring ikke kunne verifiseres: det gir en advarsel og setter
> `result["export_check_failure"]`. Produktene blir skrevet ut. For å stoppe ved dette, sjekk den nøkkelen. En kjøring som
> har mistet et bilde, fungerer på samme måte: den gir en advarsel og setter `result["image_failure"]`, som `chloros-cli process`
> avsluttes med en verdi ulik null.

### Hjelpefunksjoner

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "GNDVI", "OSAVI"],
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

# process_folder() / process_lattice_capture() also take input_level=,
# array_alignment=, array_alignment_crop=, array_alignment_interpolation=,
# progress_callback=, poll_interval=, timeout=, processing_stuck_timeout=,
# backend_exe= and api_url= (NOT backend_url=). indices defaults to ["NDVI"].

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

Hver kjøring foregår innenfor sin egen `with ChlorosLocal(...)`; en backend den måtte starte, fortsetter å kjøre når den returnerer, for neste påkalling, og stopper seg selv etter 30 minutter uten at noe bruker den. Ingen av dem tar imot `auto_start_backend`, `radiometric_output` eller `daq_log_path`; bruk `ChlorosLocal` og `configure()` for disse. `process_lattice_capture()` godtar ikke `camera`, `debayer` eller `ppk`: den hopper over paneldeteksjon og bruker standard debayer.

### Støttede verdier

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"TIFF (8-bit)"            # all-8-bit images are written as this; never raised
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer for Survey3 imagery, Chloros+ only.
                                         # Survey3 only: on a project holding LATTICE captures
                                         # (which always use the standard demosaic) configure()
                                         # or process() raises ChlorosConfigurationError; a
                                         # Texture Aware value the project saved earlier is set
                                         # to Standard with a UserWarning, as the app does. Needs a
                                         # CUDA GPU the backend's PyTorch can use (an NVIDIA
                                         # GPU, or a Jetson Orin's). Without one, process()
                                         # uses the standard debayer, and the backend prints
                                         # one "[DEBAYER] ... needs a CUDA GPU" line saying why.
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as Chloros's saved debayered (3-channel) product
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

Eksportnivået for LATTICE multispektral (M3C/M3M) i `process`-pipeline — `reflectance` (standard), `radiance`, `sensor-response` eller `all` (alle relevante moduser per bilde) — angir prosjektets **«Eksporter radianse»**og**«Eksporter reflektans»**: `radiance` eksporterer kun radianse, `reflectance` kun reflektans, `sensor-response` ingen av delene (kun de debayerte og forhåndsvisningsproduktene), `all` begge deler. De debayerte og forhåndsvisningsproduktene kan ikke endres på dette nivået. `configure()` har et eget nøkkelord for dette:

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

Den avanserte nødutgangen — å skrive inn produktene via `custom_settings` — fungerer fortsatt, men husk at den erstatter hele innstillingsblokken (se advarselen nedenfor). Skriv inn nøklene `"Export <product>"`: et prosjekts egne innstillinger for `"Export radiance"` / `"Export reflectance"`-innstillinger har forrang fremfor den eldre `"Radiometric output"`-nøkkelen, slik at den nøkkelen alene ikke endrer noe.

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Export radiance": True, "Export reflectance": False},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (standardinnstillingen) deler kameraets strålingsintensitet med **DAQ-nedstrålingen som er tidsstempelmatcht**, automatisk beregnet fra en innspilt `.daq` (DAQ-U/M/E)**eller en DAQ-M-spesifikk `.csv`**som finnes sammen med bildene; eventuelle kalibreringspakker per kamera eller DAQ som mangler lokalt,**hentes automatisk fra AWS** ved første bruk. CLI-grensesnittet eksponerer dette i henhold tiltype produktbrytere på `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.**Hvem sin avlesning er det?**En `.daq` som en opptak lagrer, er avlesningen fra kameraet/kameraene den beskriver — oppgitt i en `<file>.daq.cameras.json`-sidecar ved siden av den (enkelt-, synkroniserte og array-opptak; en rå serie oppgir hvert medlems avlesning i manifestet) — og samsvarer**kun med bilder fra disse kameraene**. Et frittstående kamera uten egen avlesning låner derfor ikke lenger et annet kameras DAQ; knytt en DAQ til det, eller ta opp en. En DAQs egen opptak (`chloros-cli daq pool-record`, eller en opptak startet i GUI-en), eller ethvert `.daq` uten den sidecaren (inkludert eldre opptak), fungerer fortsatt for alle kameraer.**OCN-kameraer i sollys.** Hver OCN-kalibreringspakke er laget under smalbåndslys, én LED per bånd, mens sollys blander båndene. Chloros 1.3.0 bruker en samlet solbasert korreksjon etter båndseparasjonen, noe som forbedrer OCN-produktene for radianse, reflektans og indeks i sollys. Den kjører overalt der separeringen skjer – `process()`, opptak, `grab()` / `frame_stream()` og live-visningen – for OCN-pakker som støtter det; RGN-kameraer forblir uendret. OCN-bilder tatt med en tidligere versjon får også denne korreksjonen når de behandles på en datamaskin som har kalibreringen de ble tatt med (Chloros beholder kameraets kalibrering når kameraet først er koblet til); ellers behandles de som før. OCN-verdiene for stråling, refleksjon og indeks skiller seg derfor fra versjon 1.2.0.

> `custom_settings` **erstatter** hele blokken med beregnede innstillinger (den omgår `configure()`s andre nøkkelord og validering etter design). Når du bruker den, må du inkludere alle `Project Settings`-nøkler du er interessert i, som i eksemplet ovenfor.

#### Lesing av indekspiksler (NDVI og de andre vegetasjonsindeksene)

**Et heltallsindeksraster er en kontrastutvidelse per bilde, ikke et −1…1-kart.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` og `JPG (8-bit)` indeksprodukter — inkludert 8-bits TIFF-filen som en 8-bits opptak skrives som — lagrer

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

der `IndexMin` / `IndexMax` er **den aktuelle rammens egne laveste og høyeste endelige indeksverdier**. DN 0 er ikke −1, og full skala er ikke +1.

Chloros stempler begge endepunktene slik at avbildningen forblir inverterbar:

| Tag | Betydning |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — konvensjonen nedenfor |
| `Chloros:IndexMin` | indeksverdien lagret som DN 0 |
| `Chloros:IndexMax` | indeksverdien lagret som full skala |

Les dem inn og inverter dem, og normaliser den lagrede DN til 0…1 ved først å bruke den lagrede datatypens maksimumsverdi, slik at ett uttrykk dekker alle heltallsformater:

```python
import json, subprocess
import numpy as np, tifffile

tags = json.loads(subprocess.run(
    ["exiftool", "-j", "-XMP:all", path],
    capture_output=True, text=True).stdout)[0]

# float(): exiftool hands back a negative real as a quoted string, so
# IndexMin arrives as str on most real NDVI data while IndexMax is a number.
lo, hi = float(tags["IndexMin"]), float(tags["IndexMax"])

raw = tifffile.imread(path)                    # uint16 for TIFF (16-bit)
ndvi = lo + raw.astype(np.float64) / np.iinfo(raw.dtype).max * (hi - lo)
```

Lesing krever ikke `-config pix4d.config` — exiftool leser disse XMP-egenskapene uten hjelp. Konfigurasjonen er kun nødvendig for å *skrive* dem.

> **`IndexMin == IndexMax`** betyr at hver piksel i det bildet hadde samme indeksverdi. Rasteret er fylt til midt på skalaen, og det er ingenting å dele med — ta endepunktet.

> **Ingen `Chloros:Index*`-tagger betyr at det ikke er angitt noen tilordning.** En LUT-eksport er en fargelagt gjengivelse der pikslene består av gradientfarger tilpasset LUT-terskelvinduet ditt, og et bilde uten piksler med endelig indeks har ikke noe område å angi. Ikke anta at det foreligger en tilordning i noen av tilfellene — samme regel gjelder som ved manglende `PixelScale`.**Fordi strekkingen er per bilde, er to bilder fra én flyging ikke sammenlignbare som DN**— hvert har sine egne endepunkter, så den samme NDVI-verdien havner på forskjellige DN-verdier i forskjellige bilder. For kvantitativt arbeid på tvers av en flyging (sonestatistikk, faste terskler, tidsserier), eksporter**`TIFF (32-bit, Percent)`**: indeksrastere skrives der som float32**indeksverdiene i seg selv**, uten behov for omskalering eller tagger. Til tross for formatets navn er indeksprodukter ikke ×100 — den skaleringen gjelder kun for reflektans med heltallstypen.

#### Unmix-klampemaske — hvilke piksler et bånd ble klemt på

Den 3×3 spektrale unmixen kan føre til at et bånd blir negativt — der lyset knapt inneholder det båndet, eller der scenen faller utenfor det kameraets unmix-matrise beskriver. Det er ikke en reell verdi, så eksporten setter den til 0 — og livevisningen viser den samme 0. Et bånd kan bli klemt på noen få piksler eller på nesten alle; masken registrerer begge deler. `Chloros:Band{R,G,B}UnmixClampFraction` angir hvor stor del av et bilde dette skjedde på; klemmasken angir **hvilke piksler**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— i en `clamp_masks/`-mappe**ved siden av** produktets mappe, aldri inne i den (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`) — for hvert strålings-, refleksjons- og indeksprodukt der minst én piksel ble klemt. Enkeltbånds `uint8`, på produktets eget pikselrutenett (orientering, ROI, binning og array-justeringsforvrengningen inkludert). Hver maske angir produktets kamera (`Make` / `Model` / serienummer), opptakstid og objektivkoder, og lister opp båndene som bitene står for (`XMP-Camera:BandName`, bit 0 først); dens `ImageDescription` og `Software` markerer den som en klemmaske, og den inneholder ingen kalibrering.
- **Biter**: bit 0 = båndet i produktets R-kanal (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). En satt bit betyr at pikselverdien i det aktuelle båndet ble klemt til 0 — behandle den, og enhver indeks bygget på den, som lavt pålitelig.
- **`Chloros:UnmixClampMask`** er stemplet på hvert produkt som har kjørt unmix: maskens sti i forhold til produktets egen mappe (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, alltid `/`-separert — sett den sammen med bildets mappe for å åpne masken), eller `none` når ingen piksler er begrenset. Ingen merking betyr at produktet aldri har kjørt unmix-prosessen (kameraer med RGB- eller monokrom-modus, rådata/debayered-nivåer).
- Opptakresultater inneholder den som bildets **`clamp_mask`** (maskens sti, eller `None` når ingenting er begrenset) — i `CameraSession.capture()`-resultater, `ArraySession.capture()`&#x27;`CaptureResult`-rammer og HTTP-opptaksresponsene. Offline-versjonen viser maskene som `clamp_masks`.
- En live-indeks’ multibåndstabel (`/api/camera/cli-array-index` med `save_multiband`) får én maske per kildekamera, i utdatamappens egen `clamp_masks/` — se CLI-referansen’`lattice index`. En video har ingen maske: resultatet fra den offline-bygningen rapporterer i stedet klemfraksjonen for hvert bilde.

Siden ingen produktmappe inneholder en maske, uansett dybde, kan en produktmappe overføres til fotogrammetriprogramvare slik den er – inkludert undermapper – og den tar kun imot bilder. Overfør produktmappen, ikke mappen over den. Chloros’ egen import hopper over maskene.

```python
import os, numpy as np, tifffile

mask = tifffile.imread(
    "clamp_masks/radiance/capture_<ts>_SN<serial>-<filter>_radiance_clampmask.tif")
# or, from the image: os.path.join(os.path.dirname(image_path), tag), with
# tag = the image's Chloros:UnmixClampMask
r_clamped = (mask & 1) != 0      # the band Chloros:BandRName names
g_clamped = (mask & 2) != 0
b_clamped = (mask & 4) != 0
```

---

## Smart-Connect for LATTICE-kameraer

Vedvarende backend-økter for live-maskinvare. Samme endepunkter som GUI-en bruker, slik at oppførselen er identisk på tvers av SDK, CLI og GUI.

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

Eksponeringen, forsterkningen og målet som angis her, beholdes som dine: Chloros’ automatiske eksponeringskontroll (inkludert eksponeringen målt av DAQ) lar dem være uendret. `cam.set_settings(exposure_auto="Continuous")` (eller `gain_auto=`) overfører eksponeringen (forsterkningen) tilbake, og `cam.set_settings(target_brightness_auto=True)` gir målet tilbake til Chloros&#x27; automatiske mål.

#### `connect_camera()` Signatur

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

- **Den returnerer når strømmen er startet.** Backenden venter, med en tidsbegrensning (ca. 15 sekunder på at strømmen skal starte, deretter 10 sekunder på den første fullstendige bufferen – ikke på det første forhåndsvisningsbildet, som følger ca. 1,5 sekunder senere), så en `status()` kommer rett etter at den sier `streaming`. Sesjonens `connect_stream` er svaret `stream`-blokken: `state` `streaming`, `no_frames` (strømmen startet og leverte ingenting — dette fremgår av `note`, og for et kamera som bruker jumbo-pakker, skal det kobles til på nytt med en pakkestørrelse på 1500 byte), `starting` eller `start_failed`, med `waited_s`. `None` for et kamera som allerede var tilkoblet. En forhåndsvisning (`get_camera_preview`, `/api/camera/jpeg/<serial>`) som ble forespurt rett etter ventetiden, begrenset til (4 s), på kameraets første forhåndsvisningsbilde i stedet for å svare «Ingen ramme tilgjengelig ennå»; et kamera hvis strøm ikke har levert noe (`no_frames`, eller klargjort for en maskinvareutløser) svarer fortsatt umiddelbart.
- **Du bestemmer selv pakkestørrelsen.** `settings={"gev_packet_size": 1500, "gev_packet_size_auto": False}` åpner kameraet med pakker på 1500 byte, uten sondering (`chloros-cli lattice cam-connect --packet-size standard` sender det samme).
- **Én tilkobling om gangen opptar kamerapoolen**: åpningen, sonderingen av pakkestørrelse, en første kalibreringsnedlasting (ca. 70 sekunder for et kamera som er nytt for denne datamaskinen), og starten på strømmen. En forespørsel som trenger et kamera i mellomtiden (`read_nodes`, `set_settings`, en opptak, en ny tilkobling) venter på det i høyst 5 s, og mislykkes deretter med HTTP-feil 503 `CAMERA_POOL_BUSY`, med navnet på kameraet som kobler seg til (`ChlorosConnectError`); spør på nytt om et øyeblikk. Oppføringene venter aldri: `list_cameras()` (`GET /api/camera/pool`), `list_arrays()` og `CameraSession.status()` svarer umiddelbart. Et kamera som er midt i en tilkobling, vises under pool-svaret `connecting` (`serial`, `phase` — `opening the camera`, `rebalancing the pool bandwidth`, `loading its calibration`, `starting its stream`, `waiting for its stream` — og `elapsed_s`), og dets `status()` sier `connecting: True` med dets `phase`; de andre kameraene leses live når puljen er ledig, ellers svares det fra deres siste avlesning (`live: False`).

#### `CameraSession`-metoder

| Metode | Beskrivelse |
| --- | --- |
| `status(timeout=15.0)` | Kameraets live-tilstand, slik `chloros-cli lattice status` leser den: `streaming`, `stream_state` (`starting`, `no_frames`, `streaming`, `start_failed` eller `stopped`), `device` (modell, serienummer, fastvare, temperatur, `is_mono`) og gjeldende innstillinger. |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Leser GenICam-noder; returnerer `{nodes, errors, enums, device, was_in_pool, in_array_id}`, samt `frame_values` og `notes` mens automatisk eksponering pågår (se nedenfor). |
| `set_settings(**kwargs)` | Skriv noder etter brukervennlig navn (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `target_brightness_auto`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). En `exposure_time` (`gain`) er en manuell eksponering (forsterkning): når den skrives uten `exposure_auto` (`gain_auto`), må automatisk modus først slås av, slik at verdien ikke avvises av en aktiv automatisk modus. Resultatet, `applied`, inneholder det kameraet tok opp; en skriving som kameraet avviser, finnes i `errors` og blir ikke lagret. Feilen angir området kameraet nå aksepterer (et maksimum i `ExposureTime` følger kameraets bildefrekvens), og en automatisk modus som ble slått av kun for den avviste verdien, blir slått på igjen; `restored` viser dette (f.eks. `{"ExposureAuto": "Continuous"}`). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, daq=None, daq_tolerance_ms=None, settings=None, timeout=None)` | Ta et **enkelt** bilde. Returnerer en `CaptureResult`: en liste over metadatadikt for bilder (ett per lagret nivå), med `.skipped`, `.failed`, `.daq_files` og `.daq_matches`. (Serietakning/takning av flere bilder er fjernet — kall `capture()` i en løkke hvis du trenger en serie.) |
| `capture_repeated(output_dir="output", *, count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Back-to-back (`interval_s=0`) eller intervallopptak fra dette kameraet, for `count`-opptak og/eller `duration_s` (minst ett, slik at det avsluttes), slik som `ArraySession.capture_repeated()` gjør for en matrise. Returnerer listen over `CaptureResult`-er; den stopper etter en opptak som melder om fullt minnekort. `on_capture(i, result)` kjører etter hver; andre nøkkelord går til `capture()`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Frigjør fra puljen; kameraet settes tilbake slik det var da det ble funnet ved tilkobling (utløserkabling, PTP, innstillinger); `keep_settings=True` lagrer i stedet innstillingene i sitt eget minne, mens `clear_settings=True` tilbakestiller dette minnet (se [Kameraminne](#camera-memory--keep_settings--clear_settings)); returnerer `camera_memory` når en av disse blir forespurt. Et kamera som er et medlem av en matrise frigjør hele matrisen. Ingen operasjon hvis vi koblet til en allerede åpen økt. Returnerer så snart backenden bekrefter frigjøringen, eller når det ikke er noe å frigjøre (kameraet er allerede borte, eller ingen backend kjører); en frigjøring som har gått ut på tid eller som backenden har avvist, utløser `ChlorosConnectError`, fordi kameraet fortsatt kan være opptatt og den neste tilkoblingen ville komme i konflikt med det. Avslutningen av `with`-blokken utløser aldri noen feil. |

Attributter: `serial`, `device` (kameraets oppdagelsesoppføring) og `backend_url`.

`capture()`-eksportkontroller (samme modell som matrisen + GUI):

- `processing` / `levels` — `processing="all"` lagrer alle relevante eksporttyper; `levels=["raw","radiance"]` lagrer bare disse (overstyrer `processing`). Utelat begge for å bruke standardinnstillingen i backenden.
- `daq` — DAQ-avlesningen av den brukte opptaket lagres som standard som en `.daq`-sidecar, inkludert et rådataopptak, slik at rammen senere kan bearbeides til reflektans/indeks. `daq=False` hopper over den når ingen nivåer bruker den. `force_daq=True` er den eldre skrivemåten for standardinnstillingen. Ingen DAQ, ingen `.daq`.
- `daq_tolerance_ms` — hvor langt fra eksponeringen målingen kan være og fortsatt være innenfor toleransegrensen (standard 500). Bare vurderingen endres.

#### `read_nodes()` under automatisk eksponering — `frame_values` og `notes`

Mens `ExposureAuto` (`GainAuto`) kjører, leser `ExposureTime` (`Gain`) noden leser tilbake den **siste verdien som ble skrevet**, ikke hva kameraet gjør. `nodes` beholder fortsatt den nodeverdien. Når du ber om `ExposureTime` eller `Gain` på et tilkoblet kamera som ikke er i automatisk modus `Off`, legger svaret til:

- `frame_values` — verdien som kameraets nyeste bilde inneholdt, under nodenavnet (`ExposureTime` i µs, `Gain` i dB);
- `notes` — én setning per slik node som angir begge verdiene.

```python
r = cam.read_nodes(["ExposureTime", "Gain"])
print(r["nodes"]["ExposureTime"])                  # last value written, e.g. 2000.448
print(r.get("frame_values", {}).get("ExposureTime"))  # what the newest frame used, e.g. 19600.0
for name, note in r.get("notes", {}).items():
    print(note)
# ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448);
# the camera's newest frame was at 19600 us.
```

Begge nøklene mangler når automatisk modus er `Off` (noden er da kameraetss reelle verdi) og for et kamera som ikke er koblet til backenden. For en matrise gir `ArraySession.status()["member_ae"]` rammeverdiene for hvert element samtidig.

#### DAQ-målt eksponeringstak (HTTP)

Når en DAQ-lyssensor er tilkoblet, begrenser Chloros kameraets eksponering × forsterkning ut fra det målte dagslyset, slik at et hvitt referansepanel ikke blir klippet. Taket gjelder kun så lenge et kalibreringsmål er innenfor kameraets synsfelt (sett i bildet i løpet av de siste 10 sekundene); uten mål i synsfeltet brukes i stedet kameraets vanlige automatiske eksponering, med sin egen høylysbeskyttelse, slik at resten av scenen ikke holdes mørkere enn nødvendig. Det finnes ingen SDK-wrapper; les eller endre innstillingen per kamera med `GET` / `POST /api/camera/<serial>/daq-exposure-cap` (POST-body `{"enabled": false}` deaktiverer funksjonen for det aktuelle kameraet). Svaret inneholder `enabled`, `cap_us` (grensen for live-modus, `None` når funksjonen er inaktiv, inkludert når det ikke er noe kalibreringsmål i synsfeltet), `k_exp_g` (konstanten som taket beregnes med) og:

| Felt | Betydning |
| --- | --- |
| `k_exp_g_source` | Hvilken konstant som er i bruk: `camera` — dette kameraets egen, beregnet fra labpunktet i kalibreringspakken (RGB-kameraer); `fleet` — flåtekonstanten, som brukes der pakken ikke har noe labpunkt; `override` — en feltoverstyring angitt på backend. |
| `k_exp_g_fleet` | Flåtekonstanten, til sammenligning. |

### Synkronisert matrise — `ArraySession` (Smart-Prep)

`connect_array` er **det anbefalte utgangspunktet** for oppsett med flere kameraer. Det kjører hele Smart-Prep-prosessen via grafisk brukergrensesnitt i bakgrunnen:

1. **Nettverksanalyse**(`/api/camera/array/recommend`) — angir**full sensor** (ingen binning, ingen ROI-beskjæring) og lar bildefrekvensen falle til det denne verten og nettverket leverer, beregnet ut fra pikselformatene og pakkemåten som Connect vil bruke. Bildene blir aldri krympet for å opprettholde frekvensen: bruk en mindre `width` / `height` eller `binning=2` / `4` for å bytte oppløsning mot bildefrekvens (endepunktets `alternatives` viser disse alternativene med deres forventede fps).
2. **Automatisk valg av nivå** — `sim-capture-sim-emit` (alle kameraer eksponerer ved samme utløser, sendes i takt for å dele ledningen, bildefrekvensen synker etter hvert som delingen reduseres) på ethvert nettverk der kameraene kan synkroniseres for å dele; `slip-emit-and-capture` (kameraene bruker ledningen etter tur, uten å eksponere samtidig) kun når dette ikke er mulig.
3. **PTP aktivert**som standard — tidsstempler på tvers av kameraene lander på én felles klokke med en nøyaktighet på**~1 ms**, backendens [PTP-grandmaster](#ptp-grandmaster--time_sync_status). Samtidig eksponering skjer via M8-hardwareutløseren (**&lt; 100 µs** mellom modulene), ikke via PTP: PTP synkroniserer *tidsstempler*, ikke eksponeringer.
4. **Automatisk valg av pikselformat per kamera** — svart-hvitt → `Mono12`, farge → `BayerRG12`.
5. **AE-innstilling** — tar øyeblikksbilder av hvert kameras gjeldende AE-tilstand, slik at tilkoblingen ikke tilbakestiller eksponeringen underveis.
6. **GPIO-utløserkonfigurasjon** — `connect_array` aktiverer hvert kamera (`TriggerMode=On`, `TriggerSource=Line2`), slik at masterenspuls styrer slavene via M8-kabelen. Dette er et trinn som kun gjelder for oppstillinger: et enkelt kamera som åpnes med `LatticeCamera`, kjører i stedet fritt. Når oppstillingen frigjøres (`disconnect()` / `force_disconnect()`, `chloros-cli lattice array-disconnect`) setter alle kameraene tilbake til frikjøring (`TriggerMode=Off`), mens alt annet forblir slik det var da tilkoblingen ble opprettet: innstillingene for utløser og synkroniseringslinje (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`; masterens synkroniseringslinje tilbake til en inngang), PTP og kamerainnstillingene som ble endret under økten – med mindre `keep_settings=True` / `clear_settings=True` (se [Kameraminne](#camera-memory--keep_settings--clear_settings)).

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

Attributter: `array_id`, `master_serial`, `slave_serials`, `ptp_enabled`, `sync_mode`, `packed_transport` og `packed_reason` (de fire siste er `None` på et håndtak fra `attach_array`), `connect_stream` og `backend_url`. `connect_stream` er `stream`-blokken i tilkoblingssvaret, formen til den enkelte tilkoblingen pluss `members` (`{serial: {state, why?}}`): `streaming` så snart den første synkroniserte gruppen i matrisen ankom (backenden venter på den, begrenset til ca. 10 s), eller `no_frames` med hvert medlems siste hentingsfeil som sin `why` og, for jumbo-pakker, fallbacken på 1500 byte i `note`. `None` på et `attach_array`-håndtak.

#### `connect_array()`-signatur

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # pin the master trigger rate (auto if None); see set_target_fps
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    packed="auto",                        # packed 12-bit transport: "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=600.0,                        # s; also the backend's deadline (see below)
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`timeout` angir hvor lenge anropet venter på tilkoblingen, og det sendes til
backenden som frist. En tilkobling som ikke er fullført innen da, blir tilbakestilt: hvert kamera som
ble åpnet, settes tilbake slik det ble funnet og lukket, ingen matrise er tilkoblet, og
anropet utløser `ChlorosConnectError` (HTTP 504 `CLIENT_DEADLINE_PASSED`, som angir
trinnet tilkoblingen hadde nådd). Et anrop som utløser feil, etterlater altså aldri et array
tilkoblet etter seg. En matrise med 9 kameraer kobles til på omtrent ett minutt; rett etter at
kameraene slås på, kan det ta flere minutter. En matrise som er oppe, men som ennå ikke har noen
synkronisert bildegruppe, returneres med feilkoden `connect_stream["state"]`
`no_frames`. Backendens logg har én tidslinje per tilkoblingstrinn
(`[array-connect <id>] <step>: <s> s …`) og en siste linje som angir det tregeste.

`force_tier`-verdier:
- `"sim-capture-sim-emit"` — ekte samtidig (alle kameraer utløses på samme klokkekant).
- `"sim-capture-ftd-stagger"` — fleksibel tidsforskyvning (kameraene sender på tidspunkter som er litt forskjøvet, slik at pakkene sendes serielt over ledningen).
- `"slip-emit-and-capture"` — sekvensiell opptak per kamera (ingen tidssynkronisering). Velges automatisk kun når kameraene ikke kan kjøres sakte nok til å dele ledningen samtidig.

`wire_ceiling_mbps` overstyrer **vertens vedvarende ledningskapasitet** i MB/s — det eneste
tallet som hele matrisetildelingen avhenger av. La den stå på `None` for å bruke den automatisk oppdagede
verdien. Senk den når arrayet rapporterer GVSP-korrupte rammer: den automatiske verdien er avledet
fra nettverkskortets oppgitte koblingshastighet, som overvurderer USB-adaptere, smale PCIe-baner og
trafikkbelastede delte nettverk — og denne overvurderingen viser seg som korrupte rammer snarere enn som en
synlig treg kobling. Verdien lagres i prosjektets matriseopptaksblokk, så en
gjenåpning eller en senere `connect_array` gjenoppretter den på samme måte som alle andre matriseinnstillinger.
Se [Matrisetilstand](#array-health--which-subsystem-is-losing-frames).

#### Pakket 12-bits transport (`packed=`)

Ved full sensorinnstilling bestemmes et arrays bildefrekvens av hvor raskt hvert kamera kan sende
sine bilder. 12-bits formater (`BayerRG12`, `Mono12`) overføres normalt som 16-bits
ord; pakket transport sender de samme 12 bitene i 1,5 byte per piksel og
pakker dem ut på verten, slik at hver ramme tar 25 % kortere tid å sende — omtrent
**33 % flere bilder per sekund ved full sensor** (målt: 10,7 → 14,0 fps på et
oppsett med 9 kameraer). **De lagrede dataene er identiske**: samme verdier, samme 16-bits
filer, samme metadata.

- `packed="auto"` (standard) komprimerer bare når det er fordelaktig **og** verten har
  kapasitet til å dekomprimere: ved tilkobling måler backenden sin egen dekomprimeringshastighet,
  antall CPU-kjerner, gjeldende belastning og nettverkskapasitet. En stasjonær PC med ledige kjerner
  pakker ved full sensorbelastning; en opptatt eller strømsparende vert (enkortdatamaskiner,
  innebygde kontrollere) strømmer upakket. Små ROI-er og binned-rammer oppnår liten gevinst.
- `packed="off"` (eller `False`) pakker aldri — for å utelukke det under diagnostisering,
  eller på en vert som må holde CPU-en ledig til annet arbeid.
- `packed="on"` (eller `True`) pakker alltid 12-bits formater.
- Enhver annen verdi utløser `ValueError`. 8-biters og 16-biters formater blir aldri
  pakket.

`analyze_array_network(..., packed="auto")`-prosjekter med samme beslutning,
slik at fps-verdien er hastigheten arrayet vil kjøre med (svaret inneholder
`packed_transport` og `packed_reason`); dens `pixel_format` er som standard satt til det
connect bruker (Mono12 / BayerRG12) i stedet for 8-bit.

Beslutningen og begrunnelsen for den finnes i økten:

```python
arr = chloros_sdk.connect_array(serials)          # packed="auto"
print(arr.packed_transport, arr.packed_reason)
# True  auto: projected 11.0 -> 14.4 fps, unpack 0.16 of 1.50 budgeted cores

arr = chloros_sdk.connect_array(serials, packed="off")
```

`ChlorosProject.connect_all(packed=...)` bruker den samme overstyringen på alle
matriser i et prosjekt (standard `None`, som betyr `"auto"`: et prosjekt lagrer ingen
pakket innstilling per matrise).
For en maskinomfattende standardinnstilling, sett `CHLOROS_ARRAY_PACKED12=off` (eller `on`) i
backendens miljø; en eksplisitt `packed="on"` / `"off"` har fortsatt forrang.
`/api/camera/array/status` rapporterer `packed_transport` og `packed_decision`.

#### Forventede bildefrekvenser

Målte bildefrekvenser for matrisen (hardwaresynkronisert, 12-bit; full sensor = 2048 × 1536). Bruk dem til å sjekke at alt er som det skal med `status()["actual_fps"]`; `analyze_array_network()` beregner bildefrekvensen for din egen vert og ditt nettverk.

| Oppsett | Bildefrekvens for matrise |
| --- | --- |
| Full sensor, PCIe 10 GbE-vertsnettverkskort, pakket transport (`packed="auto"` på en stasjonær PC) | ~14 fps |
| Full sensor, PCIe 10 GbE-vertskort, ukomprimert (`packed="off"`) | ~10,5 fps |
| Full sensor, 8 kameraer på en USB 10 GbE-adapter | ~3,8 fps |
| Full sensor, 1 GbE-vertsforbindelse | 1,2–1,7 bilder per sekund |
| 2 × 2 binning (`binning=2`), 10 GbE-vertskort | ~30–33 bilder per sekund |

- På en PCIe 10 GbE-forbindelse er det **kameraene** som setter grensen ved full sensor, ikke nettverket: 4 og 8 kameraer kjører med samme hastighet.
- **`target_fps` er utløserfrekvensen, og arrayet kjører aldri raskere.** `target_fps=1` kjører matrisen med 1 fps; `None` (standardinnstillingen) kjører den med den høyeste hastigheten som verten og nettverket tåler. En pin som ligger over denne hastigheten, senkes til den (se [`set_target_fps`](#interval-capture-for-drone-surveys)).
- **`link_speed_mbps` av `4294` i `status()` er ikke en koblingshastighet.** Det betyr at Windows ikke klarte å lese adapterens hastighet (driveren rapporterte den største 32-bitsverdien), noe som er typisk for USB-Ethernet-adaptere. En USB-adapters brukbare hastighet på Windows er uansett begrenset til 200 MB/s (se [Array Health](#array-health--which-subsystem-is-losing-frames)); for å justere en, se avsnittet «USB 10GbE-adaptere» i CLI-referansen under [Host NIC Setup &amp; Tuning](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Det gjelder også en USB-begrensning på Linux: 400 MB/s på x86_64, 200 MB/s på NVIDIA Jetson.** Adapteren identifiseres via sysfs (med tilbakefall til `ethtool -i`), så en USB 5 GbE- eller 10 GbE-adapter blir begrenset av denne grensen i stedet for den nominelle hastigheten på 500 / ~1060 MB/s. Tallet for x86_64 ble målt på Linux: det samme kortet som ødelegger rammer over 200 MB/s på Windows, kjørte et oppsett med fire kameraer feilfritt ved alle takverdier opp til 480 MB/s. På NVIDIA Jetson fungerte det samme kortet feilfritt bare ved 200 MB/s og mistet rammer sporadisk over dette. Hvis verten din håndterer mer uten at `gvsp_corrupt`-rammer kan du øke den med `wire_ceiling_mbps`. På Linux er et kort på en USB 2.0-port begrenset til 30 MB/s; koble det til en USB 3-port.

#### Over-Subscription (minimumsgrensen per kamera)

En matrise fungerer på ethvert nettverk; det er bildefrekvensen som avgjør. Sim-emit-pacing tildeler hvert kamera en jevn andel av det kollisjonssikre nettverksbudsjettet, og alle utløser fortsatt på samme trigger. På et tregt nettverk blir derfor andelen – og bildefrekvensen – liten: 8 full-oppløsning 12-bits kameraer på 1 GbE uten jumbo-rammer får 6,5 MB/s hver, omtrent **1 fps**(ca. 1,3 fps pakket). Andelen er begrenset til den laveste hastigheten kameraene kan reguleres til (`per_cam_floor_bps`: leses fra hvert kameras GevSCPD-område ved tilkobling, og aldri under 1 MB/s). Hvis nettverket krever lavere hastighet enn dette, faller tilkoblingen tilbake til**seriell**utsending (`slip-emit-and-capture`): kameraene bytter på å sende med full koblingshastighet, slik at hver ramme kommer frem, men medlemmene**eksponeres ikke samtidig**— responsens nivå og blokkerere (`SERIAL: ...`) bekrefter dette; send `force_tier="sim-capture-sim-emit"` for å avvise tilbakefallet. Bildestørrelsen påvirker ikke delingen, så**binning og ROI øker bildefrekvensen, men endrer ikke om et array passer**. Bare en kabel som ikke kan overføre matrisen ved noen hastighet, verken samtidig eller etter tur,**overbelastes** — feilmodusen er GVSP-pakketap, ikke lavere bildefrekvens. Løsninger: færre kameraer, jumbo-rammer fra ende til ende eller et raskere nettverkskort.

- Svarene `analyze_array_network()` og `/api/camera/array/connect` inneholder `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` og `per_cam_floor_bps`. Når `oversubscribed` er sant, **setter projeksjonen fps-feltene til null** (`achievable_fps_max` / `fps_bright` / `fps_dark`) i stedet for å rapportere en misvisende hastighet som er treg, men som fungerer.
- `POST /api/camera/array/connect` aksepterer en `pin_resolution`-body-parameter (**kun HTTP — ikke en SDK-kwarg**; `connect_array` eksponerer den ikke). Det er streng modus: en overtegnede tilkobling med `pin_resolution` angitt blir**kategorisk avvist** med en feilmelding som angir alle mulige løsninger. Uten denne innstillingen fortsetter tilkoblingen med en advarsel om at matrisen vil miste pakker. Uansett beholder tilkoblingen den forespurte rammestørrelsen — den grupperer eller beskjærer aldri på egen hånd.
- Kun for testing: sett `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` i backend-miljøet for å nedgradere avvisningen til en tydelig advarsel – du kobler til uansett og aksepterer pakketapet.

#### Array-tilstand – hvilket delsystem mister rammer

`ArraySession.capability()` (`GET /api/camera/array/<array_id>/capability`) bærer en aktiv `health`-blokk på et
tilkoblet array, revurdert i et rullende **10-sekunders** vindu. Den deler rammetapet
opp i de to årsakene som krever motsatte løsninger, i stedet for én «ufullstendig» rate som
ikke spesifiserer noen av dem:

| Felt | Hva det betyr | Hvilket undersystem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (per serienummer) | Bildet **kom frem og var strukturelt feil**— GVSP-pakketap. |**Nettverk**: ledningskapasitet, pacing, NIC RX-ring, MTU |
| `never_arrived_rate_pct` (per serienummer) | Rammen **kom aldri i det hele tatt**— kameraet utløste ikke, eller ingenting kom ut av det. |**Utløser / synkronisering**: M8-kabel, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Dårligst frekvens per kamera hver. | — |
| `per_cam_rate_pct` | Kombinert andel ufullstendige bilder per kamera (begge årsakene samlet). | — |
| `stable_for_seconds` | Hvor lenge hvert kamera har ligget under 0,01 %. | — |

I tillegg til `health` viser den samme loggen tallet som hele tildelingen henger på:

| Felt | Hva det betyr |
| --- | --- |
| `wire_ceiling_mbps` | Vertens gjeldende vedvarende båndbredde, MB/s. |
| `wire_ceiling_source` | Hvor tallet kommer fra, i ord — f.eks. `USB-capped 200 MB/s (was theoretical 1062; …)` eller `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` når `wire_ceiling_mbps=` angav det. |
| `nic_is_usb` | `true` for en USB-Ethernet-adapter. |

```python
import chloros_sdk

serials = ["213800234", "214000533"]           # master first
arr = chloros_sdk.attach_array(serials)
h = arr.capability()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    # force_disconnect(), not disconnect(): this handle came from attach_array,
    # and disconnect() never tears down an array the session only attached to.
    # It returns only once the backend confirms the release; a teardown that
    # timed out or was refused raises ChlorosConnectError instead, so the
    # reconnect below never races a half-released array.
    arr.force_disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Tolkning:** `gvsp_corrupt_rate_pct` ulik null med `never_arrived_rate_pct` på 0 betyr
at utløseren og kabelsynkroniseringen er perfekte, og at 100 % av tapet ligger på nettverksbanen — senk
`wire_ceiling_mbps` og koble til på nytt. Det omvendte mønsteret peker i stedet på synkroniseringskabelen eller
utløserlinjen.

> **`target_fps` er ikke avgjørende for korrupte rammer.** GevSCPD-takten skrives inn én gang ved
> tilkobling, så å senke utløserfrekvensen endrer arbeidscyklussen, ikke
> burst-frekvensen for samtidig sending. En målt reduksjon av etterspørselen med 5× ga ingen forbedring, mens
> å senke ledningens tak fra 240 til 200 MB/s reduserte andelen korrupte rammer for samme rigg fra 10,4 % til
> 0,00 %.

> **Et array i drift kan ikke omplanlegge seg selv på LATTICE (M3C / M3M) firmware.** Den kan ikke
> løse dette selv; koble fra og koble til på nytt, slik at tilkoblingstidvelgeren planlegger på nytt i henhold til
> den nye grensen.

En **USB-Ethernet-adapter er begrenset til 200 MB/s** av sonden, uavhengig av dens
typen: effektivitetstabellen som omregner en koblingshastighet til et vedvarende tall, er
avledet fra PCIe, og et USB-nettverkskort oppgir sin Ethernet-koblingshastighet samtidig som det er begrenset av
USB-bussen og driveren. Begrensningen er absolutt, ikke en brøkdel – en USB 1 GbE-adapter
oppnår ~80 MB/s og påvirkes ikke. Dette gjelder for Windows og Linux (inkludert Jetson);
se [Forventede rammehastigheter](#expected-frame-rates) for hvor dette ble målt.

#### `ArraySession`-metoder

| Metode | Beskrivelse |
| --- | --- |
| `status(timeout=10.0)` | Sanntidsstatus for den kjørende matrisen: `target_fps` / `target_fps_auto` (utløserfrekvensen og om frekvensregulatoren eier den), `target_fps_clamped_reason` (hvorfor en fastsatt frekvens holdes lavere, ellers `None`), `actual_fps` (målt; 0 når grab-sløyfen har gitt opp eller gått i stå), `recommended_fps`, `streaming` (False når grab-sløyfen har gitt opp — sjekk dette, ikke bare `last_error`; forblir «True» gjennom en omstart av innstillingene, noe som `restarting` markerer), `stalled` (True mens grab-sløyfen kjører, men ikke har publisert noen ramme-gruppe i omtrent ti rammeperioder, 2–30 s; `last_error` melder dette først etter 60 s), `last_frame_age_s`, `last_error`, `serials`, `ptp_enabled` (`True` kun når matrisen var koblet til PTP **og** hvert medlem sist leste `PtpEnable` på; ikke det samme som låst, som `ptp_epoch` måler), `ptp_configured` (det tilkoblingen ba om), `ptp_off_members` / `ptp_unread_members` (medlemmer som sist leste `PtpEnable` av / ikke svarte), `member_recovery` (`None`, eller et medlem som mistet strømmen eller forbindelsen: `state`, `waiting` mens det ikke svarer, `reopening`, deretter `recovered`, sammen med `members` og `why` — matrisen åpner den igjen når den svarer og gir den tilbake sin PTP-verdi for tilkoblingstid, utløser- og synkroniseringslinjekobling, tempo- og kamerainnstillinger, og strømmer deretter igjen uten nye `connect_array`) og `ptp_epoch` (målt), `sync_mode` / `sync_tier`, `packed_transport` / `packed_decision`, `per_cam_emit_mbps`, `link_speed_mbps` (`4294` betyr at Windows ikke kunne lese adapterens hastighet — se [Forventede bildefrekvenser](#expected-frame-rates)), `trigger_pipeline` / `pipeline_tune`, `radiometric_chain` (om strålingskjeden per bånd er brukbar, og hvilket element som ødela den), og `band_clamp` — mens en livevisning gjengir matrisen: andelen opplyste piksler som unmix-prosessen har begrenset til 0, per element (`members`: `{serial: [band0, band1, band2]}` i kameraets R-, G-, B-båndrekkefølge) og per bånd viser sammensetningen (`composite`: `{band name: share}`), med `warn`, andelen som GUI-grensesnittet markerer som «Bånddata med lav pålitelighet» (målt omtrent en gang i sekundet; `None` når det ikke kjører noen livevisning), og `member_ae` — per element, eksponeringen (`exposure_us`) og forsterkningen (`gain_db`) i det siste publiserte bildet, hentet fra selve bildet (`read_nodes` av `ExposureTime` / `Gain` returnerer den siste verdien som ble skrevet mens automatisk eksponering pågår, ikke hva den gjør). |
| `set_member_settings(serial, **kwargs)` | Skriv inn innstillingene til ett medlemskamera, slik `CameraSession.set_settings()` henter dem (eksponering, forsterkning, mållysstyrke, ...); medlemmet forblir i matrisen. Utløser `ChlorosConnectError` for en serie som ikke er et medlem. Se [Eksponering på matrisemedlemmer](#exposure-on-array-members). |
| `capability(timeout=10.0)` | Det smart-prep fastsatte (opptaksnivå, bildestørrelse, `pixel_format`), den aktive `health`-blokken og vertsmaskinens båndbreddebudsjett. Se [Array-tilstand](#array-health--which-subsystem-is-losing-frames). |
| `set_target_fps(fps, timeout=10.0)` | Endre utløserfrekvensen mens arrayet kjører, uten å koble til på nytt. Et tall låser den fast (frekvenskontrolleren lar den være); `None` setter den tilbake til auto. Returnerer `{requested, target_fps, target_fps_auto, clamped}` — `clamped` er True når frekvensen ble senket til det verten og nettverket kan håndtere. Se [Intervallopptak for dronekartlegging](#interval-capture-for-drone-surveys). |
| `grab(processing="debayered", with_metadata=False, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, bit_depth=None, band=None, align=False, refl_max=None)` | Ett bilde fra hvert kamera, **alle fra én utløser**, som numpy-matriser (ingen filer skrives). Returnerer en `GrabResult` (en `dict`): `{serial: array}`, eller `{serial: (array, metadata)}` med `with_metadata=True`. `.group_id` er utløseren som bildene deler; `.errors` tilordner hvert utelatt kamera til årsaken (et kamera som ikke kan betjenes ved den utløseren, utelates og fylles aldri ut fra et annet). `processing` er `raw` / `debayered` / `radiance` / `reflectance` / `display`. Det samme leses som prosjekt-API-et [`ArrayHandle.grab()`](#one-trigger-per-read--group_id) — én felles implementering — så alle nøkkelordene der (`band`, `align`, `bit_depth`, `refl_max`) fungerer her. Krever `numpy`. |
| `frame_stream(*, processing="debayered", fps=5.0, duration=None, count=None, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, with_metadata=False, bit_depth=None, band=None, align=False, refl_max=None)` | Iterator som gir ett `grab()`-resultat per tick (kun nøkkelordargumenter), med hastighet `fps`, begrenset av `duration` (sekunder) og/eller `count` (`None` = ubegrenset). **Hvert sett som leveres er en ny utløser:** en tick hvis nyeste utløser er settet den allerede har generert, venter på neste utløser i stedet for å gjenta den, slik at strømmen kjører med den laveste av `fps` og matrisens egen bildefrekvens. Utløser `FrameFetchError` i stedet for å gå i dvale: umiddelbart når hvert kamera av en eller annen grunn nekter å gjenta neste tick, etter 60 sekunder med ticks uten ramme, og etter 60 sekunder uten ny utløser («ingen ny utløser på 60 sekunder»). Samme implementering som `ArrayHandle.frame_stream()`. Se [Live-bilder i din egen pipeline](#live-frames-in-your-own-pipeline). |
| `continuous(output_dir="output", *, format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, max_captures=None, duration_s=None, daq=None, daq_tolerance_ms=None, timeout=30.0)` | Start backendens egen kontinuerlige opptak (GUI-ens kontinuerlige modus) → [`ContinuousHandle`](#continuoushandle). Lagrer **hver** synkronisert gruppe som arrayet publiserer, slik som `capture()` ville lagret den (samme eksportkontroller), inntil den stoppes eller inntil `max_captures` / `duration_s` (`None` = inntil den stoppes). Utløser `ChlorosConnectError` hvis en allerede kjører for dette arrayet. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, smart=False, after_seq=None, daq=None, daq_tolerance_ms=None, timeout=300.0)` | Én synkronisert opptaksgruppe. Returnerer en `CaptureResult` (liste over rammediktoner + `.skipped` + `.failed` + `.group_seq` + `.daq_files` + `.daq_matches` + `.alignment` + `.oriented`). Overfør et tidligere resultat’s `group_seq` som `after_seq` for å hente den neste synkroniserte gruppen etter den, i stedet for å vente på en ny. Eksportkontroller nedenfor. |
| `capture(..., smart=True)` | **Smart opptak** — venter på at AE skal stabilisere seg på alle kameraer, og utløser deretter. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, orient=None, timeout=120.0)` | Raskeste opptak: kun rådata + den tildelte DAQ-avlesningen (+ den frie kombinerte indeksen). Speiler «Raskeste opptak»-knappen i brukergrensesnittet. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Enkelt / Kontinuerlig / Intervall i én avgrenset sløyfe. Returnerer `list[CaptureResult]`.**Krever `count` og/eller `duration_s`** for at den skal avsluttes (SDK-et har ikke Ctrl+C). Avsluttes også tidlig, etter gjennomgangen, når et gjennomløp rapporterer en full stasjon (en `.failed`-oppføring med `disk_full`); det gjennomløpet er det siste resultatet. |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, expr=None, timeout=30.0)` | Start opptak av den kombinerte live-indeksvisningen til video/GIF → `RecorderHandle`. Videoen er en Motion-JPEG-fil `.avi` (ingen ffmpeg nødvendig). Hvis ingen GUI-visning er åpen, gjengir backenden den: `expr` (et indeksuttrykk i matrisens båndnavn, f.eks. `"(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)"`), eller matrisens NDVI som standard. Én sammensatt opptaker per matrise. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, orient=None, timeout=30.0)` | Start en rå-Bayer-serie med høy bildefrekvens → `RecorderHandle`. Behandle på nytt offline med `build_video()`. `orient=True`: produktene som `build_video()` lager, roteres etter kameraets orientering (produkter per kamera etter det enkelte kameraets orientering, den kombinerte indeksen etter masterkameraets); råbildene forblir i opprinnelig format, og seriemanifestet registrerer orienteringen til hvert enkelt medlem. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Behandle en lagret rå burst-serie offline til kalibrerte videoer. Blokkerer inntil ferdig (`wait=True`) og returnerer byggeresultatet: `{success, outputs, errors, skipped, substitutions, notes, degraded, combined}` — se [byggeresultatet](#the-build_video-result) nedenfor. |
| `build_video_status(job_id, timeout=15.0)` | Avventer en offline-byggjobb: `{running, result, error, burst_dir}`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Frigjør hele arrayet — **hvis denne økten åpnet det** (`connect_array`): da blir det `force_disconnect()`, og utløser `ChlorosConnectError` når frigivelsen ikke bekreftes (utgangen fra `with`-blokken utløser aldri noe). Ved et håndtak fra `attach_array` lar den matrisen kjøre videre og skriver ut en linje til stderr som sier dette: matrisen kan tilhøre GUI-en eller et annet skript. Hvert kamera blir stående i fri drift og ellers slik tilkoblingen fant det; `keep_settings=True` lagrer innstillingene i kameraets eget minne, mens `clear_settings=True` tilbakestiller dette minnet til fabrikkinnstillingene (se [Kameraminne](#camera-memory--keep_settings--clear_settings)). |
| `force_disconnect(timeout=60.0, keep_settings=False, clear_settings=False)` | Frigjør hele arrayet uavhengig av hvem som åpnet det. Bruk dette på et tilknyttet håndtak når du ønsker å avslutte arrayet (f.eks. for å koble det til på nytt med andre innstillinger). Returnerer når backenden bekrefter frigjøringen, eller når det ikke er noe å frigjøre (matrisen er allerede borte, eller ingen backend kjører); en nedleggelse som har gått ut på tid eller som backenden har avvist, utløser `ChlorosConnectError`. Uansett hvilket flagg som brukes, returnerer den hva hvert kameras minne fikk (`camera_memory`), ellers `None`. |

`capture()` eksportkontroller (samme endepunkt som GUI/CLI bruker):

- `processing` / `levels` — `processing="all"` (eller `levels=["raw","radiance",…]`) lagrer alle relevante eksporttyper per kamera; en enkelt `processing`-verdi lagrer bare det nivået.
- `aligned` — **aktivert som standard (`None`), siden Chloros-appens opptak av det samme arrayet er**(dens**justerte** opptakinnstilling per matrise). Hvert medlems ikke-rå eksport blir tilpasset hovedkameraets ramme ved hjelp av matrisens [justeringsprofil](#array-alignment) og beskåret til medlemmene felles overlappingsområde, slik at filene er pikselregistrerte og kommer ut med samme størrelse, beskjæring og `Alignment*`-tagger som appens (innenfor et ROI blir transformasjonen rebasert til vinduet); `XMP-mapir:AlignmentJSON` angir `"applied": true` med sin `crop_roi`. Raw-filene forblir uforvrengt i opprinnelig størrelse, men bærer transformasjonen i metadataene. `aligned=False` lagrer hver fil i opprinnelig størrelse, uforvrengt; `aligned=True` er standardinnstillingen. `continuous()`, `capture(smart=True)`, `capture_repeated()` og prosjektnivåene `ArrayHandle.capture()` / `capture_smart()` bruker den samme standardinnstillingen. Uansett om forvrengningen brukes eller ikke, registrerer hver medlemsfil kameraets `Alignment*`-transformasjon så snart matrisen har en justeringsprofil — bortsett fra et kamera som profilen ikke justerer (utelatt eller tvunget til identitet på grunn av en mislykket løsning), samt referansekameraet når ingen andre kameraer er justert: disse har ingen `Alignment*`-koder. Uten profil på matrisen lagres opptaket ujustert, og resultatet viser dette: `result.alignment` er `{"requested": True, "applied": False, "warnings": [...]}`, én advarsel per årsak til at forvrengningen ikke kunne brukes (ingen profil, medlemmer med ulik oppløsning, en matrise som ikke kansynkronisert, et bilde som profilen ikke kan tilordnes til). `applied` er `True` bare når hver eksport som ba om forvrengningen, fikk den. Matrisens automatiske justering oppretter profilen så snart eksponeringen har stabilisert seg; `ArrayHandle.calibrate_alignment()` / CLIs `align-calibrate` oppretter en på forespørsel. `result.oriented` angir om `orient=True` har rotert produktene.
- `render_index=False` — hopp over overlegget med vegetasjonsindeks per kamera; standardinnstillingen viser det der det er konfigurert.
- `daq` / `daq_tolerance_ms` — DAQ-avlesningen lagres som en `.daq`-sidecar som standard, selv når ingen valgt nivå trenger den; `daq=False` hopper da over den, og `daq_tolerance_ms` angir samsvarstoleransen (standard 500). `force_daq=True` er den eldre skrivemåten for standardverdien. Se [DAQ-målingen som en opptak lagrer](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
- `orient=True` — slå på hvert medlems **produkter**(`display`, `radiance`, `reflectance`, indeksen per kamera og sammensetningen) med kameraets orientering, etter justeringsforvrengningen: med**masterkameraets**orientering da de ble forvrengt inn i dets ramme (standardinnstillingen `aligned` — medlemmene forblir samregistrerte), ellers med deres egne kameras orientering.**Standard: hver medlemsfil i opprinnelig format**, slik array-opptak alltid har; `raw` og `debayered` forblir i opprinnelig format uansett. Hver fil registrerer orienteringen til rammen som pikslene befinner seg i (`XMP-Camera:ChlorosRotation` / `ChlorosFlip*` / `ChlorosOrientationApplied` — en RAW-fil, sitt eget kamera), og XMP-filen `Alignment*` beholder den opprinnelige rammetransformasjonen, anvendt før orienteringen; en RAW-fil som inneholder denne transformasjonen, registrerer også `ChlorosAligned*` (masterens), retningen til et produkt som er forvrengt inn i masterens ramme. Behandlingen roterer hvert produkt i henhold til den retningen som tilhører dets piksler: den justerte når den forvrenger RAW-filen (Array alignment på), kameraets egen når den ikke gjør det. `capture_repeated()` og `capture_fastest()` sender den videre; `ArrayHandle.capture()` / `capture_smart()` på prosjektnivå tar den også med.

**TIFF-komprimering (knapp kun for HTTP):** `ArraySession.capture()` sender ingen `compression`-nøkkel, så bakendens standardinnstilling gjelder — `POST /api/camera/array/capture` leser en `compression`-parameter i brødteksten, `"none"` som standard: ukomprimert, sensorens piksler nøyaktig slik de leses (~6,3 MB per bilde i full oppløsning). `"deflate"` (tapsfri zlib L1 + horisontal prediktor, ~4,1 MB/ramme) skriver mindre filer saktere — begge er tapsfrie og leses identisk ved import. SDK-et har ingen kwarg for dette; løsningen er `chloros-cli lattice array-capture --compression deflate` eller rå HTTP. DEFLATE holder Python GIL, så komprimerte skrivninger kan ikke parallelliseres på tvers av skrivertrådene per kamera — vedvarende opptak i full oppløsning med sensorhastighet krever den ukomprimerte standardinnstillingen. Detaljer: [CLI-referanse → array-capture](cli-reference.md).**Påfølgende grupper (HTTP):** endepunktet returnerer `group_seq`; en løkke som sender det tilbake som `after_seq` ved neste forespørsel, henter den neste synkroniserte gruppen — inkludert en som ble publisert mens forrige opptak ble lagret — i stedet for å vente på en gruppe publisert etter forespørselen, og `skipped_groups` rapporterer eventuelle grupper sløyfen har gått glipp av. `capture_repeated`, CLIs `--continuous` og GUI-ens kontinuerlige modus gjør alle dette; intervallpasseringer forblir oppdaterte ved hvert planlagte tidspunkt.**Eksportoverstyringer per medlem (kun HTTP):**det samme endepunktet aksepterer også `exclude_serials` (liste – fjern medlemmer fra det lagrede settet; matrisen utløses fortsatt som én synkronisert gruppe, og ekskluderte medlemmer returneres i `excluded`), `serial_levels` (`{serial: [level tokens]}` overstyringer på kameranivå) og `serial_index` (`{serial: bool}` overstyringer av indeksoverlegg per kamera). Dette er GUI-paritet-kroppsparametere og**ennå **ikke SDK-kwargs**; medlemmer som mangler i kartene faller tilbake til de array-omfattende `levels` / `render_index`.

##### Inspisere hoppede kameraer — `CaptureResult.skipped`

`ArraySession.capture()` returnerer en `CaptureResult`, som er en underklasse av `list`: iterer den, indekser den, `len()` — alle eksisterende mønstre fortsetter å fungere. Ny kode kan inspisere `.skipped`-attributtet for å se hvilke kameraer som ble ekskludert og hvorfor. Det vanligste tilfellet er RGB-kameraer i et blandet filterarray når du ber om `processing="radiance"` eller `"reflectance"` — strålingsintensitet per Bayer-piksel er meningsløst for en bredbåndssensor, så backenden hopper over disse kameraene i stedet for å produsere meningsløse resultater.

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

Årsakstokener følger mønsteret `<level>-not-applicable-to-rgb-cam` (én oppføring per hoppet over nivå, hver med `level`). Et kamera hvis kalibreringspakke ikke er lastet inn, hopper over `radiance` / `reflectance` med `<level>-needs-calibration (connect the camera so its cal pack loads)`, og uten brukbar DAQ-måling for reflektans hopper det over med `reflectance-skipped-no-dls`. De andre reflektansspesifikke hoppene er `reflectance-skipped-no-fresh-dls` (ingen ny nedstrålingsmåling tilgjengelig), `reflectance-skipped-bound-daq-unavailable (…)` (den tilknyttede DAQ-enheten kunne ikke nås) og `dls-uncalibrated-band-<nm>` — båndet ligger for det meste utenfor DAQ-lyssensorens radiometrisk kalibrerte område (~374–974 nm), så den absolutte DAQ-baserte reflektansdelingen avvises, og rammen nedgraderes tydelig til sensorrespons. Blant de leverte SKU-ene er det kun F988 som utløser dette; den støttede banen for dette kameraet er arbeidsflyten med reflektanspanelet.

##### Filer som ikke kunne skrives — `CaptureResult.failed`

En opptakssesjon som kun kunne skrive noen av filene sine, returnerer likevel: listen inneholder alle filene som **ble** skrevet, og `.failed` viser resten, én ordbok per fil med `serial` (`None` for den kombinerte sammensatte filen og en matrises `.daq`-sidecar), `level` (`raw`, `radiance`, …, `index`, `composite`, `daq`, eller `match` — opprinnelsen til `.daq`s `.match.json`, da `.daq` ble skrevet, men ikke sidecaren) og `error`. På en full disk inneholder oppføringen også `disk_full: True`; alle senere opptak til den disken ville mislykkes på samme måte, og det er grunnen til at `capture_repeated()` stopper der. Det utløses ingen unntak for en delvis opptak, og en fil i `.failed` ble ikke liggende halvskrevet på disken. `.failed` er tom ved et fullstendig opptak.

```python
result = arr.capture("output/", processing="all")
for f in result.failed:
    print(f"SN:{f['serial']} {f['level']} not written: {f['error']}")
if any(f.get("disk_full") for f in result.failed):
    raise SystemExit("drive full -- free space and capture again")
```

##### DAQ-en som leser av en opptak, lagrer — `.daq_files` og `.daq_matches`

Når en DAQ er knyttet til kameraet eller matrisen, eller er samlet i en pool, lagrer hvert opptak den nedadrettede avlesningen den brukte: en `.daq` i undermappen `daq/` i opptaksmappen, sammen med `.match.json` og `.cameras.json` (hvis måling det er). Dette gjelder også for råopptak eller opptak med debayering, og det er den samme filen uansett om opptaket kom fra GUI, CLI eller SDK. `daq=False` hopper over dette for et opptak der nivåene ikke bruker DAQ-en (et `radiance`-, `reflectance`- eller `display`-nivå skriver det fortsatt); `daq_tolerance_ms` flytter toleranselinjen for den opptaket og aldri den brukte avlesningen. Hvilken DAQ: den som er knyttet til kameraet eller matrisen (GUI-ens valg av lyssensor, et prosjekts tilknytning eller `chloros-cli daq pool-bind`), ellers den nyeste som backenden har. CLI-grensesnittet angir kontrollene `--no-daq` og `--daq-tolerance-ms` ([CLI-referanse](cli-reference.md#the-daq-reading-a-capture-saves)).

```python
r = cam.capture("out", levels=["raw"])      # a CameraSession; ArraySession results match
for m in r.daq_matches:                      # one per .daq in r.daq_files
    print(m["daq_file"], m["gap_ms"], m["in_tolerance"])
```

En opptak lagrer lyssensoravlesningen den brukte som en `.daq` med en `<file>.daq.match.json` ved siden av, som angir hvordan avlesningen var knyttet til eksponeringen: `method`, `gap_ms` (fra eksponeringen til nærmeste DAQ-måling) og `in_tolerance` (innen 500 ms, eller opptakets `daq_tolerance_ms`). Reflektansfilens XMP-data inneholder den samme vurderingen som `Chloros:DlsMatchMethod`, `Chloros:DlsMatchGapS` og `Chloros:DlsInTolerance`.

En opptak venter på DAQ-enhetens første avlesning etter eksponeringen, slik at den kan blande avlesningene på hver side av den (`method` `interpolated`). En DAQ-E sender en avlesning hvert 0,4. s i dagslys og hvert 1–2. s innendørs, så `capture()` returneres opptil omtrent ett og et halvt av disse intervallene senere; selve eksponeringen blir ikke forsinket. Det ventes ikke på en DAQ som har sluttet å sende. En DAQ-E-måling er lyset gjennomsnittlig beregnet over intervallet siden forrige måling, så den plasseres midt i dette intervallet: `gap_ms` måles til dette midtpunktet, og `.match.json`&#x27;`bracket_stamps_ns` gir avlesningene sine egne tidsstempler. `ArraySession.continuous()` venter uten å bremse opptaket. `capture_repeated()`sventer ikke, så de beholder frekvensen sin og blir tilordnet den nærmeste målingen før eksponeringen.

En måling utløper aldri, så ingen ramme går tapt på grunn av en forsinket måling — den merkes i stedet:

- **DAQ-strømmen har stoppet opp** — den nyeste målingen brukes, og `gap_ms` angir hvor gammel den er (`in_tolerance: false` over 500 ms).
- **Ingen DAQ å lese**— frakoblet (`disconnect()`, `chloros-cli daq pool-disconnect`), eller den tilknyttede DAQ-en kan ikke nås — kameraets siste måleverdi gjenbrukes og beskrives i forhold til**denne** eksponeringen: `method`, `retained` og `gap_ms` fra denne eksponeringen, `in_tolerance` beregnes på nytt, og `retained_from_epoch_ns` angir eksponeringen den først ble matchet med. Et kamera som aldri har hatt en måling, får fortsatt ingen reflektans, og resultatet forklarer hvorfor (`reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, eller at opptaket ble avvist).

Filer som en opptak lagrer i mappen til prosjektet som er åpent i Chloros, blir lagt til i prosjektets filliste, akkurat som et opptak i appen (filer som er lagret andre steder, blir stående): `CameraSession.capture()`, `ArraySession.capture()` (også `smart=True`) og `capture_cameras_transient()`. Resultatet fremgår av `.project_add` (`capture_cameras_transient()`: ordbokens `project_add`) -- `held`, `files` og backendens `message`; `held` er `True` mens en behandlingskjøring pågår, og filene legges til når den er ferdig. `None` når ingen filer er lagt til i prosjektet.

`CameraSession.capture()` og `ArraySession.capture()` returnerer dem som resultatene `.daq_files` og `.daq_matches`, siden HTTP-fangstresponsene (`/api/camera/<serial>/cli-capture`, `/api/camera/capture`, `/api/camera/capture-all-synced`, `/api/camera/array/capture`) inneholder `daq_files` og `daq_matches`, én per `.daq`: `daq_file`, `cameras`, `method`, `gap_ms`, `in_tolerance`, `tolerance_ms` og `stream_fault` — DAQ-enhetens egen strømfeil, som kun sjekkes ved måleverdier utenfor toleranseområdet. Før du stoler på reflektansen fra en opptak, må du sjekke `in_tolerance` i dens `.match.json` eller XMP, eller DAQ-enhetens [`status()`](#daqsensorsession-methods).

`processing`-nivåer:

| Nivå | Utgang |
| --- | --- |
| `"raw"` | Enkanals Bayer (monokameraer: det ene båndet) direkte fra sensoren. |
| `"debayered"` *(standard i SDK)* | 3-kanals farge: kalibreringspakkens mørkeforskyvning (DSNU) fjernet fra rådataene, deretter OpenCVs kantbevisste demosaik – uten linearitets-LUT, flatfelt eller unmix (monokameraer: 1-kanals gråtoner). Den lagrede TIFF-filen er i **RGB**-kanalrekkefølge (fotometrisk RGB, slik at alle bildevisere viser ekte farger); matrisene i minnet som `grab()` / `frame_stream()` returnerer, er i OpenCVs**BGR** rekkefølge — bytt om kanalene (`frame[..., ::-1]`) for å sammenligne med en lagret fil. `chloros-cli process` skriver ut det samme resultatet som `Debayered_Images`. |
| `"radiance"` | float32 W/m²/sr/nm via den fullstendige radiometriske kjeden. Kun multispektralt — RGB-kameraer hoppes over. |
| `"reflectance"` | uint16 0..32768 (Pix4D-klar); krever en live DAQ-parring for absolutt referanse. Kun multispektral. |
| `"display"` | Full kjede som samsvarer med forhåndsvisningen i brukergrensesnittet (CCM + WB + gamma i henhold til kameraets profil). |
| `"all"` | **Én fil per gjeldende nivå** for hvert kamera (samsvarer med GUI-innstillingen «Capture All» / CLI-standard). Den returnerte `CaptureResult` inneholder da én rammediktat per `(cam, level)`, med nivået i hvert diktat; nivåer som ikke er relevante vises i `.skipped`. DAQ-en som leste inn opptaket, lagres som en `.daq`-sidecar. |

> **Merk — standardinnstillingen avviker fra CLI.** `ArraySession.capture()` har som standard `processing="debayered"`; kommandoen `chloros-cli lattice array-capture` har som standard `processing="all"`. Overfør `processing="all"` eksplisitt fra SDK for å gjenspeile CLI/GUI-lagring på flere nivåer.

##### Filnavn for matriseopptak

Hvert nivå lagres i sin egen undermappe til `output_dir` (`raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` for `display`, `index/`, `composite/`). Hver fil navngis etter kameraet som `SN<serial>-<filter>`: serienummeret, deretter filterdelen av kameraets modell — `SN214000533-FOCN` for et `M3C-L87-FOCN`, `SN261902142-F450` for en `M3M-L87-F450` — sammen med `-`, så et navn har de samme `_`-separerte feltene som før. Et kamera hvis modell ikke kan leses, er bare `SN<serial>`, og opptak fra tidligere versjoner har ikke noe filter i navnet. Filnavnet avhenger av hvor mange nivåer et medlem ble bedt om:

| Medlem ba om | Filnavn | Eksempel (`reflectance`, standard `format="tiff"`) |
| --- | --- | --- |
| **Ett nivå** (`processing="reflectance"`, eller et `levels` med ett element) | `sync_<timestamp>_SN<serial>-<filter>.<format>` — ingen nivåsuffiks, som i tidligere versjoner, slik at eksisterende skript fortsatt fungerer | `reflectance/sync_<ts>_SN214000533-FOCN.tiff` |
| **Flere nivåer** (`processing="all"`, eller `levels` med mer enn én post) | `sync_<timestamp>_SN<serial>-<filter>_<level>.<format>` | `reflectance/sync_<ts>_SN214000533-FOCN_reflectance.tiff` |

[Klemmasken](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) følger etter produktnavnet, `clamp_masks/<product folder>/<product name>_clampmask.tif`: en enkeltnivå refleksjonsregistrering skriver `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`, mens `processing="all"` skriver `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`. Regelen for suffikset bestemmes for hvert element ut fra nivåene det ble forespurt om, så et utelatt nivå (f.eks. reflektans på et RGB-kamera) endrer det ikke. Les stiene fra de returnerte bildene i stedet for å bygge dem. En opptak med ett kamera (`CameraSession.capture()`) legger alltid til nivået: `capture_<ts>_SN<serial>-<filter>_<level>.tiff`.

##### PNG/JPEG: Tar kun med visnings- og indeksprodukter

En PNG- eller JPEG-fil har 8 bit per kanal. Hvis man ber om en med et nivå den ikke kan håndtere, blir forespørselen avvist før noe blir tatt opp, med en melding som angir nivået og TIFF som formatet som støtter det:

- **`radiance`**(float32) og**`reflectance`**(uint16, `32768` = 1,0) — og dermed også `processing="all"`, som inkluderer dem. SDK-et sjekker dette på argumentene: `capture()` utløser**`ValueError`** uten en forespørsel (`CameraSession`, `ArraySession`, `CameraHandle`, `ArrayHandle`, `capture_smart`).
- **`raw` / `debayered` fra et kamera som strømmer mer enn 8 bit** (12-bit M3C, Mono12 M3M) — backenden avviser disse, da den kjenner hvert kameras bitdybde: HTTP-feil 400 med `error_code` `FORMAT_CANNOT_HOLD_LEVEL`, generert som `ChlorosConnectError` (`CameraSession` / `ArraySession`) eller `RuntimeError` (`CameraHandle` / `ArrayHandle`). Enkeltnivåopptaket `debayered` fra ett kamera henter et 8-bits bilde for PNG/JPEG og blir ikke avvist.

Lagre radiometriske data som `format="tiff"` (`ext=".tiff"`); bruk PNG/JPEG for `display` og indeksrenderinger.

#### Hvordan automatisk eksponering fungerer

- **Eksponering først, forsterkning sist.** Når forsterkningen er på Auto, forblir den på 0 dB inntil eksponeringen har nådd den lengste tillatte verdien.
- **En manuell eksponering lar forsterkningen forbli på «Auto»** inntil du endrer forsterkningen eller stiller den inn på «manuell». Når den settes tilbake på «Auto», er forsterkningens grense den samme som før.
- **Små høylys gjør ikke scenen mørkere.** Et speilende område på opptil 1 % av bildet ignoreres, og i panelene til et kjent kalibreringsmål, opptil 0,5 % av et panel.
- **DAQ-eksponeringstaket gjelder kun mens et kalibreringsmål er i synsfeltet.** Det forhindrer at et hvitt panel blir klippet; når det ikke er noe panel i bildet, eksponerer kameraet for scenen.
- **Konstant lys forblir stabilt, og endret lys stabiliserer seg i løpet av sekunder.** Ved konstant lys forblir eksponeringen og grensen uendret, og etter en stor endring i lyset stabiliserer bildet seg i løpet av få sekunder. En utklipping begrenser ikke lenger eksponeringen permanent, og en endring i ROI finner eksponeringen igjen.
- **Lysstyrkemålet holdes innenfor rekkevidde.** Det stiger ikke mens kameraet er ved eksponeringsgrensen og forsterkningen ikke kan økes, slik at lys som kommer tilbake ikke forårsaker lysreflekser i bildet.
- **Et kamera hvis automatiske eksponering slutter å reagere, holdes på nivået for det lyset det befinner seg i.** I sterk sol kan et kamera falle til sin korteste eksponeringstid og forbli der med et mørkt bilde, mens alle innstillingene viser normale verdier. Chloros overvåker bildet, holder kameraet på det nivået det holdt for det lyset, og slipper det når lyset endres. En mørk eller tildekket lyssensor forstyrrer ikke dette.

#### Eksponering på array-medlemmer

Et LATTICE-kamera begrenser `ExposureTime` til én periode av bildefrekvensen. Mens et array strømmer, kjører Chloros hvert medlem med `AcquisitionFrameRateEnable` aktivert, med en hastighet som ligger litt over utløserhastigheten, slik at et medlems eksponeringstak ligger litt under utløserperioden, **i alle ROI-er**:

- Et elements bildeperiode er utløserperioden minus en slakk på 15 % av perioden (minst 3 ms, høyst halvparten av perioden), og dets eksponeringstak er den bildeperioden minus ca. 0,1 ms. Eksempel: en utløser på 10 fps (100 ms) gir en medlemsfrekvens på ca. 11,8 fps og et eksponeringstak på ca. 85 ms.
- **En manuell `ExposureTime` som er lengre enn dette taket, bremser utløseren for å tilpasse seg den.** Chloros åpner først medlemmets tak for den aktuelle verdien og lar den lavere frekvensen kjøre i noen utløserperioder (skrivingen kan ta noen sekunder lenger), før den skriver den, og utløseren utløses ikke raskere enn eksponeringen tillater. På en matrise med automatisk hastighet gjenopprettes den tidligere hastigheten så snart hvert medlem som ble bremset av manuell eksponering, er tilbake på automatisk eksponering. En verdi som kameraet selv ikke kan ta, blir fortsatt avvist, med det området det aksepterer nå angitt i feilmeldingen, og medlemmets automatiske eksponering blir gjenopprettet hvis den bare var slått av for den aktuelle skrivingen. Før versjon 1.3.0 satte et lite ROI en øvre grense for hvert medlem på sensorens egen friløpsperiode, uansett utløserfrekvens: 4,9 ms ved 768 × 304, slik at ingen medlem kunne gjøres lysere.
- Chloros forkorter aldri en manuell eksponering for å tilpasse seg en raskere utløserfrekvens. En automatisk frekvens holder seg innenfor det manuelle eksponeringene tillater; en fastsatt frekvens som er raskere enn det en eksponering tillater, gir det aktuelle medlemmet et tak som er langt nok for den, og det tar ikke imot utløsere raskere enn det eksponeringen tillater. Senk eksponeringen eller den faste frekvensen når hvert medlem må håndtere hver utløser.
- **Tilbakelesing:** når man leser av et elements noder, vises `AcquisitionFrameRateEnable` True og `AcquisitionFrameRate` med elementets hastighet; aktiveringen slås av igjen når matrisen stopper.
- **Matrisen eier disse to nodene. Ikke skriv `AcquisitionFrameRateEnable` eller `AcquisitionFrameRate` på et matriseelement** — sett i stedet matrisenshastighet med `target_fps` / `set_target_fps()` i stedet.

Skriv et medlems eksponering med `arr.set_member_settings(serial, exposure_time=20000)` (nøkkelordene til `CameraSession.set_settings()`; `chloros-cli lattice exposure --serial` gjør det samme). Medlemmet forblir i matrisen. Ikke åpne et matrisemedlem med `connect_camera()`: connect avviser et kamera som allerede er i en matrise, og `CameraSession`, som den ville returnere, fjerner kameraet fra matrisen når det kobles fra. `ArraySession.status()["member_ae"]` rapporterer hvert medlemseksponering og forsterkning fra det nyeste bildet.

### Én prosjektpost, alle moduser

En prosjektmappe inneholder innstillingene for alle enheter: `cameras.json` — hvert kamera (`cameras[].settings`, for et enkelt kamera og for hvert medlem i matrisen) og hver matrises matriseomfattende innstillinger (`arrays[].capture`: format, ROI, binning, utløserfrekvens, ledningsbudsjett) — samt `sensors.json`, som inneholder innstillingene for hver lyssensor (integrasjonstid, ramme-gjennomsnitt, tak). Appen, CLI, SDK og MCP-serveren leser og skriver alle de samme filene på samme måte:

- **En enhet som er koblet til et prosjekt, lagres i dette prosjektet og registrerer alle endringer der.** «Under et prosjekt» betyr med et prosjekt åpent i backenden (appens), eller — via en `ChlorosProject` (`open_project()`), som angir sin egen mappe til backenden (`project_path`). Et kamera, en matrise eller en lyssensor som prosjektet ennå ikke inneholder, legges til i filene når den kobles til (`connect_camera()`, `connect_array()`, lyssensorkoblingen, CLI, appen, `chloros-mcp`), sammen med innstillingene. En endring fra hvilken som helst modus (`set_settings()`, `ArraySession.set_member_settings()`, `set_target_fps()`, matriseinnstillingene, `set_cap()`, CLI, appens kontroller) lagres i prosjektfilene et øyeblikk senere. En eksponering, forsterkning eller et AE-mål som er angitt på et kamera, lagres som «holdt», slik at det beholdes når prosjektet åpnes på nytt.
- **Når du åpner et prosjekt, blir enhetene konfigurert ut fra dette**, uansett modus: et kamera, en matrise eller en lyssensor som er tilkoblet uten egne innstillinger, får prosjektetslagrede innstillinger (`lattice cam-connect` / `array-connect` når et prosjekt er åpent, `chloros-cli project connect`, `open_project().connect_all()`, når appen åpnes på nytt). Et array som kobles til under en ny ID, oppdaterer det arrayet prosjektet lagret med de samme kameraene; et array med andre kameraer legges til ved siden av det.
- **Appens egen lagring overskriver aldri en enhets live-innstillinger** med en eldre kopi av dem, og beholder overskrivninger per medlem som er lagret av en CLI- eller SDK-kaller.
- **Pikselformatet du angir, er det formatet prosjektet beholder.** Et format angitt i Array Settings, via `--pixel-format` eller via `connect_array(pixel_format=)`, inkludert 8-bit, lagres og gjenkobles nøyaktig. Hvis ingenting er angitt, kobles hvert kamera til med sin 12-bits standardinnstilling (`BayerRG12`, `Mono12` på en monosensor). Et formatnavn angir bitdybden; familien tilhører sensoren, så et fargekamera blir aldri satt til et monofargeformat eller et monokamera til et Bayer-format, og linjen `[PixelFormat <serial>] ... written` angir dette når et format ble tilordnet. Eksportene fra en 8-bits opptak er 8-bits, inkludert reflektans: omskalert og merket med `PixelScale`.
- En enhet som er tilkoblet utenfor et prosjekt, registrerer ingenting.
- **Et opptak tilhører et prosjekt som er opprettet etter det; når det åpnes, forblir det knyttet til prosjektet.**Når et prosjekt opprettes — `create_project()`, `chloros-mcp`s `create_project`, eller appens**Nytt prosjekt**— eller dupliseres (appens**Dupliser**, som åpner kopien) mens en opptak fortsatt skriver til prosjektet som var åpent før, fortsetter opptaket, og det som skrives havner i det nye prosjektet, på samme sted i det (nedenfor). Å åpne et eksisterende prosjekt — `ChlorosProject.process()` (som åpner prosjektet i bakgrunnen), `chloros-cli process`, appens**Åpne prosjekt** — etterlater en opptak som fortsatt pågår i prosjektet den tar opp i: den fullføres der, og filene blir lagt til i det prosjektets filliste (appen stopper alle opptak før den åpner et prosjekt).
  - `ArraySession.record()` og `ArraySession.burst()`: videoen, eller burst-mappen med hvert enkelt bilde, flyttes i sin helhet dit når opptaket stopper. Resultatet av stoppet rapporterer den nye banen (`out_dir`, `video_path`), og en samling av bildeserien lages ut fra den. Hvis stasjonen nekter å flytte filene, forblir opptaket der det ble skrevet, og resultatet angir dette (`relocation_error`). Råbilder fra en stoppet serie og `.daq` legges til i fillisten til prosjektet den havner i; en video legges ikke til i prosjektets filliste.
  - `ArraySession.continuous()`: alle opptak etter endringen lagres i det nye prosjektet; de som ble lagret før, forblir der de er.
  - `DAQSensorSession.record_start()` (og appens egen lyssensoropptak): `.daq` flyttes til det nye prosjektets `light_sensor`-mappe når opptaket stopper; `record_stop()` returnerer den nye banen, og filen blir lagt til i det prosjektets filliste.

  En opptak i en mappe utenfor det åpne prosjektet forblir der den er, og `start_camera_continuous()` / flerstrømsjobber beholder mappen de ble tildelt.
- **I appen havner opptakene etter et nytt prosjekt i det.** Å opprette eller duplisere et prosjekt sørger for at alle pågående opptakskjøringer og opptak fortsetter, og hvert opptak som tas etter det (Capture All, Start Capturing) lagres i det nye prosjektet og blir lagt til i dets filliste. Å åpne et eksisterende prosjekt i appen stopper først alle disse.
- **Et prosjekt flyttes ikke mens det tar opp.**Å flytte det åpne prosjektet til en annen arbeidskatalog (appens**Innstillinger → Arbeidskatalog**; ingen SDK-kall flytter et prosjekt, en HTTP-klient som sender til `/api/move-project` gjør det) avvises med HTTP 409 `PROJECT_RECORDING` mens en matriseopptak eller serieopptak, et kontinuerlig matriseopptak, en kontinuerlig kamera-/multistrømjobb eller et lyssensoropptak skriver til den: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.`

Dette er uavhengig av hvordan kameraene står igjen ved frakobling (se [Kameraminne](#camera-memory--keep_settings--clear_settings)): et prosjekt konfigurerer kameraene selv når det åpnes.

### Kameraminne — `keep_settings` / `clear_settings`

Når et array kobles til, overtar kameraene innstillingene fra arrayet, og du kan deretter endre dem. Når Chloroskobler fra kameraene — `ArraySession.disconnect()` / `force_disconnect()`, slutten på en midlertidig funksjon, avslutningen av `start_camera_continuous()` — setter den hvert enkelt kamera tilbake til slik det var før tilkoblingen: utløser- og synkroniseringsledningskoblingene, PTP- og kamerainnstillingene — `ExposureAuto` / `GainAuto` (samt manuell innstilling av `ExposureTime` / `Gain`), `TargetBrightness` samt AE-grenser og demping, hvitbalanse, svartnivå, `PixelFormat`, binning og ROI, fargepipeline (`GammaEnable` / `Gamma`, `LUTEnable`, `ColorTransformationEnable`), `ReverseX` / `ReverseY`, `UserSetSelector`, PTPs `PtpSlaveOnly` / `PtpDomainNumber`, pakkestørrelsen (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`), stimuleringsfrekvensen (`DeviceLinkThroughputLimitMode` / `DeviceLinkThroughputLimit`, `GevSCPD`, `GevSCFTD`), blokkdataene (`ChunkModeActive`, hver `ChunkEnable` under sin `ChunkSelector`), utløserinngangens deglitch, og en matrise gir en slaves synkroniseringslinje (`LineFilterSelector` / `LineFilterWidth`) og opptakets bildefrekvens (`AcquisitionFrameRate` kun der aktiveringen var på: når den er av, viser frekvensen det høyeste som takten tillater, ikke en innstilling). Kun det som avviker er angitt: når kameraet stoppet først, takten og bildefrekvensen før en eksponering som krever sin egen bildeperiode.

For bruk av kameraene utenfor et prosjekt, skriver to argumenter til kameraets **eget minne** — det det starter opp med, i hvilken som helst programvare (på `disconnect()` / `force_disconnect()`, alle midlertidige funksjoner, `start_camera_continuous()` og `ChlorosProject.disconnect_all()`; MCP: de samme argumentene som for `disconnect_device` og de midlertidige verktøyene; CLI-modus: `--keep-settings` / `--clear-settings`):

- **`keep_settings=True`** lagrer innstillingene for hvert kamera slik de ble etterlatt i minnet ved slutten av økten (UserSet2, som ble oppstartssettet). Kameraene får fortsatt kjøre fritt (`TriggerMode` `Off`), med synkroniseringsledningene slik de ble funnet. Et kamera uten UserSet2 lagrer innstillingene i UserSet1; denne datamaskinen husker dette, så Chloros’ selvhelbredende matrise — som tilbakestiller et UserSet1 som har avviket fra Chloros’ referanseverdi — lar det være i fred.
- **`clear_settings=True`** tilbakestiller hvert kameras minne til fabrikkinnstillingene: Chloros’ referanseverdi, den som matrisens selvreparasjon baserer seg på — fabrikkinnstillinger med `TriggerMode`, `Off`, `TriggerSource`, `Line2`, automatisk eksponering, et RGB-kameras kalibrerte hvitbalanse beholdt — i UserSet1, som utgjør oppstartsettet.
- Ikke begge (`ValueError`; backenden svarer 400). Svaret inneholder `camera_memory`, per kamera `{"saved_to": "UserSet2" | "UserSet1" | None, "errors": [...]}` eller `{"cleared": bool, "errors": [...]}` — `force_disconnect()` returnerer den; for en jobb følger den med `stop()`.

Appen tilbyr ikke disse.

**Hver utgang setter kameraene tilbake.**Tilbakestillingen slik de ble funnet, kjører på hver utgangsvei: `CameraSession.disconnect()` (appens kamera**X**), appens**X** på en matrise eller et hvilket som helst matriseelement (hele matrisen frigjøres), lukker appen (backenden får opptil 20 sekunder på seg til å sette tilbake hvert kamera, hver matrise og alle kontinuerlige opptak som kjører; en midlertidig funksjon eller et engangskall som fortsatt kjører, stopper ved sitt neste sikkerhetspunkt, setter først kameraene tilbake, i opptil 8 sekunder, og svarer med HTTP-kode 503 `SHUTTING_DOWN`; en `connect_array()` som fortsatt kjører, stopper før sitt neste kamera og setter tilbake kameraene den har åpnet, i opptil 10 sekunder, og utløser `ChlorosConnectError` (HTTP 503 `SHUTTING_DOWN`); og en som ankommer under nedstengingen, åpner ingenting: HTTP 503 `SHUTTING_DOWN`), samt en `chloros-cli`-kommando som avsluttes eller hvis terminal lukkes (jobben stoppes og kameraene settes tilbake først). Et backend som krasjer eller avsluttes tvangsmessig, kan ikke sette noe tilbake; en strømavbrudd og oppstart returnerer hvert kamera til minnet.**Et prosjekt setter opp kameraene sine selv.** Når et array kobler seg til via et prosjekt – appens åpne prosjekt eller et `ChlorosProject`, som oppgir navnet sitt til backend-en (`project_path`) – får hvert medlem innstillingene prosjektet har lagret for det: en eksponering, forsterkning eller AE-mål som holdes for det (holdes som ditt, slik at automatisk eksponering lar det være i fred) og dets hvitbalansemodus (korreksjon fra Chloros; kameraets egen hvitbalanse forblir deaktivert), uansett hva kameraets minne inneholder.

### Opptaksmoduser og opptakerenheter

Matrisens overflate speiler opptaksfeltet i brukergrensesnittet: Enkeltbilde / Kontinuerlig / Intervall / Raskeste lukkerhastighet, pluss to opptakerenheter (live-komposittvideo og rå bildeserie → offline etterbehandling).

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

    # Continuous — every synced group for 60 s, saved by the backend itself
    with arr.continuous("run1/", processing="raw", duration_s=60) as run:
        run.wait()
    print(run.result["captures"], run.result["captures_per_s"],
          run.result["stop_reason"])

    # Combined-index video/GIF recorder: the GUI's combined view, or with no
    # GUI open the array's NDVI (pass expr="..." for another index)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    if rec.result and rec.result.get("error"):   # leaving the block does not raise
        print("recording failed:", rec.result["error"])
    else:
        print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**er SDK-ets kontinuerlige/intervall-sløyfe. Siden det ikke finnes noen `Ctrl+C` for å avbryte den fra et skript,**må** du sende `count` og/eller `duration_s` (den stopper når en av dem nås). `interval_s` måles fra starten av hvert gjennomløp (i samsvar med GUI-en). Kontinuerlige gjennomløp (`interval_s=0`) kobler sammen `group_seq` → `after_seq`, slik at de utgjør påfølgende synkroniserte grupper i arrayetshastighet. Gjenværende kwargs sendes direkte videre til `capture()`.
- **`continuous`**er backendens egen kontinuerlige opptak – GUI-ets kontinuerlige modus.**Bruk denne i stedet for `capture_repeated()` når du ønsker at hver utløser skal lagres på disken med høy bildefrekvens:** `capture_repeated()` sender én HTTP-forespørsel per gruppe fra skriptet ditt, mens `continuous()` holder to opptak i gang inne i backenden uten tur-retur mellom gruppene. Målt ved ROI 512 på et oppsett med 9 kameraer: 73–77 opptak/s, mot et oppsett som publiserer ca. 78. Behold `capture_repeated()` for intervallopptak (`interval_s`) og for gjennomganger du inspiserer én etter én. På Windows må du ekskludere backenden fra Microsoft Defender for å oppnå høye hastigheter (se [Trinn 1](#step-1--install-the-chloros-platform-package)).
- **`record`** er *overvåkingsklasse*: den registrerer den gjengitte, kombinerte indeksen i sanntid. Når GUI-ens kombinerte visning er åpen, registrerer den den visningen slik den vises. Når ingen er åpen, renderer backenden visningen selv: `expr` hvis du sender inn en, ellers matrisens NDVI (de røde og NIR-båndene nærmest 660 nm og 850 nm, slik live-visningen plukker dem ut), så snart matrisen er justert. Navngi båndene i `expr` slik matrisen navngir dem: `<band>_<filter>` for et M3C-kamera (`NIR_850_RGN`, `Red_660_RGN`, `Orange_615_OCN`), `Mono_<filter>` for et M3M (`Mono_850`); et kort fargenavn (`NIR`, `Red`) fungerer bare når ett bånd i matrisen har det. En GUI-visning som åpnes under opptaket, blir tatt med så lenge den forblir åpen. Et opptak som ikke skriver noen ramme, mislykkes med feilene `error` og `"no-frames: …"` (matrisen kan fortsatt være i ferd med å justere seg eller stabilisere eksponeringen), og `stop()` utløses (se [`RecorderHandle`](#recorderhandle)). Én sammensatt opptaker per matrise (utløses hvis en allerede kjører).
- **`burst` → `build_video`** er *analyseklasse*: `burst` skriver råbilder + et manifest per bilde + én `.daq` per distinkt DLS-avlesning under `<output>/bursts/<base>/` med grab-sløyfens fulle hastighet (ingen kjede, ingen exiftool, ingen live-visning). `build_video` tidssammenstiller hvert bilde med nærmeste `.daq` og kjører importpipelinens radiance/refleksjonsgrad/indeks-kjeden. `products` er en liste over `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (standard: den kombinerte indeksen). `burst().stop()` starter også automatisk en «best-effort»-kombinert-indeks-versjon, som returneres som `build_job` i stopp-resultatet.

##### `build_video`-resultatet

| Nøkkel | Betydning |
| --- | --- |
| `success` | `True` kun når alle forespurte produkter ble bygget som forespurt: minst én utdata, ingenting i `skipped`, og ikke i `degraded`. |
| `outputs` | Stier til de lagrede videoene (og GIF-ene). |
| `errors` | Problemer, som tekst. |
| `skipped` | Én `{"serial", "level", "reason"}` per forespurt produkt som **ikke** produserte noen video — enten på grunn av et kameraets (ingen kalibreringspakke for det kameraet, ingen DLS-avlesning for reflektans, feil i behandlingskjeden) eller den kombinerte indeksen, som inneholder `"kind": "combined"` og `serial` `None` (ingen rød/ NIR-båndpar blant kameraene, ingen gruppe med strålingsintensitet for begge båndene). Enhver oppføring gir `success` `False`; de andre videoene blir fortsatt skrevet og oppført. |
| `substitutions` | Kameraer gjengitt på et annet nivå enn det som ble forespurt, og hvorfor — et RGB-kamera som har fått forespurt radianse/reflektans, får sin visningsvideo med hvitbalanse når kameraets kalibreringspakke er tilgjengelig. Dette er ikke en feil. |
| `notes` | Informasjonstekst, ikke feil — f.eks. har et kamera uten indeksuttrykk per kamera ingen indeksvideo, eller visningsvideoen til et RGB-kamera er ikke hvitbalansert (ingen kalibreringspakke, eller en uten fabrikkhvitbalanse), med angivelse av kameraets navn. Ikke en feil. |
| `degraded` | `True` når et produkt ble skrevet, men ikke er det som ble forespurt: den kombinerte offline-indeksen er den innebygde normaliserte forskjellen mellom rødt og NIR, så et konfigurert indeksuttrykk blir ikke evaluert. Gir `success` `False`. |
| `combined` | Sammendrag av den kombinerte indeksen: båndene som ble brukt, LUT-området, opprettede rammer. |
| `clamp_masks` | Stier til [unmix-clamp-masker](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) for deramme-kalibrerte TIFF-filene (`save_tiffs=True`), i `clamp_masks/<level>/` ved siden av TIFF-filenes `<level>/`-mappe, én per TIFF-fil der en piksel ble klemt. En video inneholder ingen. |
| `clamp_fractions` | Hva en video angir i stedet for en maske: én `{"kind", "serial", "level", "frames"}` per videoprodukt (`kind` `"combined"` for kameraene som inngår i den samlede indeksen), `frames` en liste over `[frame, R, G, B]` — den andelen av piksler i det aktuelle bildet som unmix-prosessen har begrenset til det båndet som dets R/G/B-kanal inneholder (`Chloros:Band{R,G,B}UnmixClampFraction`). |

#### `RecorderHandle`

Returneres av `ArraySession.record()` og `ArraySession.burst()`. Bruk den som en kontekstbehandler for automatisk stopp ved avslutning av scope, eller styr den manuelt.

| Medlem | Beskrivelse |
| --- | --- |
| `job_id` | Backend-jobb-ID (str). |
| `kind` | `"composite"` (fra `record`) eller `"raw"` (fra `burst`). |
| `start_stats` | Diktet som returneres av `start`-kallet. |
| `result` | `None` under kjøring; den endelige «stop-result»-ordboken når opptaket er stoppet. |
| `stats(timeout=10.0)` | Live-jobbstatistikk (antall innspilte bilder, oppnådd fps, forløpt). |
| `stop(timeout=60.0)` | Stopp opptakeren; returnerer og lagrer det endelige resultatet (`.result` forblir `None` hvis selve stoppforespørselen mislyktes). For en `record()`-opptak som mislyktes – et som ikke skrev noen rammer (`error` `"no-frames: …"`) — lagres resultatet i hurtigbufferen på `.result`, deretter utløses `ChlorosConnectError`. Idempotent (et nytt kall returnerer det hurtigbufrede resultatet, uten å utløse feil). Hvis man lar en `with`-blokk stå, stopper opptakeren uten å utløse feil, så sjekk `rec.result.get("error")` etter den. |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

#### `ContinuousHandle`

Returneres av `ArraySession.continuous()`. Bruk den som en kontekstbehandler — å forlate `with`-blokken stopper opptaket hvis det fortsatt kjører — eller styr den med `status()` / `wait()` / `stop()`. Det kjører én kontinuerlig opptakssesjon per matrise om gangen.

| Medlem | Beskrivelse |
| --- | --- |
| `start_status` | Statusordboken som returneres av startkallet. |
| `result` | `None` under kjøring; den endelige statusdiktet når opptaket er avsluttet (angitt av `stop()`, eller av en `status()` / `wait()` som registrerte at den avsluttet). |
| `status(timeout=10.0)` | Live-statusdiktat (felter nedenfor). |
| `wait(timeout=None, poll_s=0.5)` | Blokker inntil opptaket avsluttes av seg selv — `max_captures` eller `duration_s` nådd, matrisen borte, en full stasjon, en feil — og returner den endelige statusen. Med `timeout` returneres den siste statusen etter det angitte antallet sekunder, selv om opptaket fortsatt pågår: sjekk `running`. |
| `stop(timeout=30.0)` | Stopp opptaket og returner sluttstatusen. Idempotent (et nytt kall returnerer det bufret resultatet). |

Statusfelt:

| Felt | Betydning |
| --- | --- |
| `running` | Sant mens opptaket pågår. |
| `captures` | Synkroniserte grupper lagret. |
| `files` | Filer skrevet. |
| `failed_files` | Filer som ikke kunne skrives (som i [`CaptureResult.failed`](#filer-som-ikke-kunne-skrives--innsamlingsresultat-mislyktes)). |
| `failed_captures` | Innfangingsforsøk som mislyktes i sin helhet, inkludert de som ble forsøkt på nytt (en HTTP-503-feil mens matrisen ikke hadde noen ny gruppe i tide). |
| `skipped_groups` | Grupper som matrisen publiserte, men som ikke ble lagret (innsamlingen kom på etterskudd). |
| `captures_per_s` | Gjennomsnittlig lagringsrate siden starten. |
| `elapsed_s` / `started_at` | Antall sekunder som har gått så langt; starttid (Unix-epoke-sekunder). |
| `last_group_seq` | Den nyeste synkroniserte gruppen som er lagret. |
| `last_error` | Den siste feilen, eller `None`. |
| `stop_reason` | Når avsluttet: `count` (`max_captures` nådd), `duration` (`duration_s` nådd), `stopped` (`stop()` kalt), `array_gone` (matrisen ble koblet fra), `error` (20 feil på rad, eller en forespørsel som backenden avviste), `disk_full` (en fil som stasjonen ikke hadde plass til). `None` under kjøring. |
| `max_captures` / `duration_s` | Grensene den ble startet med (`0` = ingen). |
| `defender_cores` | CPU-kjerner som Microsoft Defender har brukt i løpet av de siste 10 sekundene; `None` inntil en er målt, og alltid `None` på en vert uten Defender. |
| `defender_warned` | Sant når opptaket har logget at Defender bremser det (ved 1 kjerne eller mer). Ekskluder backenden fra Defenders skanning — se [Trinn 1](#step-1--install-the-chloros-platform-package). |

```python
run = arr.continuous("captures/run1", processing="raw", max_captures=5000)
try:
    while True:
        st = run.wait(timeout=5)              # returns every 5 s while running
        print(f"{st['captures']} captures, {st['captures_per_s']:.1f}/s, "
              f"{st['skipped_groups']} skipped")
        if not st["running"]:
            break
finally:
    final = run.stop()                        # no-op once it has ended
if final["failed_files"] or final["stop_reason"] in ("disk_full", "error"):
    raise SystemExit(f"continuous capture: {final['stop_reason']}: "
                     f"{final['last_error']}")
```

### Live-rammer i din egen pipeline

For å mate synkroniserte bilder inn i din egen kode — maskinlæring, en tilpasset visningsprogram, din egen opptaker — bruk `ArraySession.frame_stream()`. Hvert tick gir en `GrabResult`: `{serial: numpy array}` for hvert kamera, **alt fra én utløser**, med utløserens ID på `.group_id`. Det er den samme implementeringen som prosjekt-API-ets `ArrayHandle.frame_stream()`, så alt under [Én utløser per avlesning](#one-trigger-per-read--group_id), [`band`](#exporting-a-single-band--band), [`align`](#aligning-the-streams--align) og [`bit_depth`](#bit_depth--trading-precision-for-bandwidth) gjelder for begge.

```python
import chloros_sdk
from chloros_sdk import FrameFetchError

serials = ["213800234", "214000533", "214701288", "214701292"]   # master first

with chloros_sdk.connect_array(serials) as arr:
    try:
        for frames in arr.frame_stream(processing="raw", fps=10, duration=300,
                                       with_metadata=True):
            if not frames:              # no camera answered this tick
                continue
            if frames.errors:           # cameras missing from this trigger
                print("left out:", {s: str(e) for s, e in frames.errors.items()})
            for serial, (img, hdr) in frames.items():
                # img: numpy array (raw: one Bayer or mono plane)
                # hdr: group_id, frame_id, timestamp_ns, exposure_us, gain_db, ...
                my_pipeline(frames.group_id, serial, img, hdr["timestamp_ns"])
    except FrameFetchError as exc:      # the stream cannot continue
        print("stream ended:", exc.status, exc)
```

- **Hvert utlevert sett er en ny utløser.** `frames.group_id` gjentas aldri innenfor en strøm; når du ber om flere `fps` enn det arrayet kjører med, venter strømmen på neste utløser, slik at den kjører med den laveste hastigheten av de to.
- **Manglende kameraer blir navngitt, ikke erstattet.** Et kamera som ikke kunne betjenes ved den utløseren, er fraværende i ordboken og oppført i `frames.errors`.
- **Det avsluttes med et smell.** `FrameFetchError` vil ved avslag gjenta seg ved neste tick, etter 60 sekunder uten ramme eller etter 60 sekunder uten ny utløser.**Hva det leverer.**`frame_stream()` overfører hele numpy-rammer via den lokale HTTP-tilkoblingen og når en topphastighet på nær**170 MB/s**. Det er nok til live-behandling av rårammer, eller av en binned eller ROI-matrise, men ikke for hver utløser ved full oppløsning — for å beholde**hver** utløser, lagre på disk med [`continuous()`](#continuoushandle). Målt på et oppsett med fire kameraer (FOCN, FRGN, FRGB, F450) med pakket transport; hver tick var et komplett sett med én utløser:

| | Full sensor (matrise 14,2–14,4 fps) | 2 × 2 binning (matrise ~33 fps) |
| --- | --- | --- |
| `frame_stream(processing="raw")` | 6,7 sett/s | 22,2 sett/s |
| `frame_stream(processing="debayered")` | 2,3 sett/s | 9,3 sett/s |
| `frame_stream(processing="display")` | 0,8 sett/s | 3,0 sett/s |
| MJPEG per kamera, 4 strømmer ved 720p | 13,5 bilder per sekund hver; hvert bilde samsvarer med en utløser | ~25 bilder per sekund hver; ~45 % av utløserne er til stede på alle 4 |

Bruk `processing="raw"` for gjennomstrømning og demosaik på din side; `debayered`-nivået fjerner også kalibreringspakkens mørkeforskyvning, noe en demosaik av `raw` ikke gjør.

#### MJPEG-strømmer per kamera (ingen RTSP)

**RTSP støttes ikke.** Strømmingsgrensesnittet er MJPEG over HTTP (`multipart/x-mixed-replace`, én JPEG per del, hver del med en `Content-Length`), én URL per kamera:

| URL | Hva den serverer |
| --- | --- |
| `GET /api/camera/stream/<serial>` | 8-bits forhåndsvisning, opptil ~30 fps. For et array-element angir `max_h` (standard 720, 64–2160) og `jpeg_q` (standard 80) angir størrelse og kvalitet. |
| `GET /api/camera/<serial>/stream-annotated` | Den samme forhåndsvisningen med GUI-overlegg tegnet på (zebra, rutenett, hårkors, histogram, peaking, spotmåler). `ChlorosLocal.camera_stream()` leser den opp for deg. |
| `GET /api/camera/<serial>/stream-reflectance` | 8-biters reflektans per bånd — se [Live reflektansstrøm](#live-reflectance-stream-mjpeg). |

- **Trigger-overskrifter.** For et array-element inneholder hver del av alle tre strømmene `X-Chloros-Group-Id`, `X-Chloros-Frame-Id` og `X-Chloros-Timestamp-Ns`. Deler fra forskjellige kameraer med samme `X-Chloros-Group-Id` utgjør samme utløser: samle delene og grupper dem etter denne overskriften for å sette sammen et sett på nytt. Et frittstående kamera har ingen gruppe: forhåndsvisningsdelene har ingen av disse overskriftene (refleksjonsdelene har en tom `X-Chloros-Group-Id`).
- **Forhåndsvisningsstrømmen dropper bilder under belastning.** Den sender det nyeste bildet og hopper over de bildene den ikke rakk å sende i tide, så ved høye matrisehastigheter når ikke alle utløsere strømmen til hvert kamera (se tabellen ovenfor). Bruk `frame_stream()` eller `continuous()` for hver utløser.
- **`/api/camera/stream/<serial>` settes på pause mens arrayets samlede direktestrøm er åpen** (`/api/camera/array/stream/<array_id>` — GUI-ens arrayvisning åpner den). Strømmen til hvert medlem sender deretter én «Paused (array stream active)»-plassholderramme, uten `X-Chloros-*`-overskrifter, og ingenting mer før den samlede strømmen lukkes; den gjenopptas da av seg selv. `stream-annotated` og `stream-reflectance` pauser ikke. Lukk GUI-ens matrisevisning når et skript leser forhåndsvisningsstrømmene for hvert kamera.
- **For tall, ikke bilder, bruk `frame_stream()`.**JPEG er tapsfylt og 8-bit; `frame_stream()` returnerer de samme produktene uten kodek i banen.

### Intervallopptak for dronekartlegging

En kartleggingsflyging trenger én synkronisert bildegruppe for hver få meter av flyvningen, ikke hvert bilde som matrisen kan produsere. To innstillinger styrer dette:**`target_fps`**angir hvor ofte kameraene tar et bilde, og**`capture_repeated(interval_s=…)`** angir hvor ofte et bilde lagres.**Kameraene fortsetter å streame mellom lagringene.** Hvert intervall lagrer den nyeste synkroniserte gruppen; bildene i mellom blir ikke lagret. Automatisk eksponering justeres for hvert bilde, så eksponeringen har allerede stabilisert seg når lagringen skjer. Det finnes ingen modus der kameraene stopper mellom opptakene og vekkes for hvert enkelt: hvert opptak måtte da vente på at strømmen skulle starte og eksponeringen stabilisere seg.**Fastsett bildefrekvensen til det intervallet krever.**Hvis du lar den stå på «auto», kjører en matrise med den høyeste frekvensen som verten og nettverket tåler. Å fastsette den til en lavere frekvens reduserer varmeutviklingen i kameraet, nettverksbelastningen og belastningen på vertens CPU uten å endre hva som lagres. Velg en frekvens som gir**2–4 bilder per intervall**:

- Hver lagring venter på neste bilde etter at den er forespurt, så en lagret gruppe havner opptil én bildeperiode etter det planlagte tidspunktet. En høyere frekvens holder dette gapet lite.
- Automatisk eksponering justeres én gang per bilde, så en svært lav frekvens reduserer hvor raskt eksponeringen følger en forbigående sky.

**Beregn intervallet ut fra flygeplanen:**

```
spacing    = along-track footprint × (1 − forward overlap)
interval_s = spacing / ground speed
```

For eksempel gir et fotavtrykk på 60 m med 80 % fremoveroverlapping en avstand på 12 m; ved 10 m/s blir det `interval_s=1.2`, og en fastsatt `target_fps=3` gir mellom 3 og 4 bilder per intervall. Opptaket er tidsbasert: SDK-et har ingen avstands- eller GPS-utløst modus, så hold en jevn hastighet over bakken langs kartleggingslinjene.

**Lagre rådata på luftfartøyet og kalibrer senere.** `interval_s` måles fra starten av hver lagring, så en lagring som tar lengre tid enn intervallet gjør at den neste starter for sent og avstanden glir ut. Ved å lagre rådata samt lyssensoravlesningen (`processing="raw"`; avlesningen lagres som standard) holdes hver lagring kort, og hvert bilde kan senere konverteres til reflektans.

```python
import chloros_sdk

INTERVAL_S = 1.2      # from the flight plan
FLIGHT_S = 20 * 60    # bounds the loop; it also stops at count= if given

with chloros_sdk.connect_array(serials, target_fps=3.0) as arr:
    groups = arr.capture_repeated(
        "survey/", interval_s=INTERVAL_S, duration_s=FLIGHT_S,
        processing="raw", render_index=False,
        on_capture=lambda i, r: print(f"group {i}: {len(r)} frames"))
```

- **`connect_array(target_fps=…)`**låser frekvensen fra starten;**`arr.set_target_fps(fps)`**endrer den midt i flyvningen uten å koble til på nytt, og**`arr.set_target_fps(None)`** setter den tilbake til automatisk. En hastighet som overstiger det verten og nettverket kan håndtere, reduseres til dette taket — sjekk `clamped` i resultatet — og den laveste hastigheten er 0,1 fps.
- **Ikke bruk `burst()` til å tynne ut bilder.** Den skriver hvert bilde som matrisen produserer; den følger riktignok en fastsatt `target_fps`, så den er kun det riktige verktøyet når hvert bilde er ønsket.
- **`target_fps` reparerer ikke ødelagte rammer.** Å senke verdien reduserer antall rammer som produseres, ikke hvor raskt hver enkelt sendes; for GVSP-korrupte rammer bør du i stedet senke `wire_ceiling_mbps` (se [Array Health](#array-health--which-subsystem-is-losing-frames)).

```python
r = arr.set_target_fps(5.0)
if r["clamped"]:
    print(f"asked for {r['requested']} fps, running at {r['target_fps']}")
```

### Koble til et allerede tilkoblet array — `attach_array`

Hvis arrayet allerede er oppe (GUI-en har åpnet den, eller en tidligere SDK-økt har kalt `connect_array`), bruk `attach_array` for å hente et håndtak til den i stedet for å koble til på nytt. `connect_array` gir alltid feilmeldingen «Kamera  <sn>er allerede i arrayet<id>» i den situasjonen, fordi en POST-forespørsel med `/array/connect` for et medlem i puljen ikke er idempotent; `attach_array` leser `/api/camera/array/list` og matcher enten etter array_id eller serienummer.

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

Mønster: SDK-skript som deles med skrivebords-GUI-en, bør prøve `attach_array` først og falle tilbake til `connect_array` hvis det ikke er noen matrise i puljen ennå.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Et tilknyttet array er lånt, ikke eid.** Et håndtak fra `attach_array` river aldri ned arrayet: verken avslutningen av `with`-blokken eller en eksplisitt `disconnect()` frigjør det (det eksplisitte anropet skriver ut én linje til stderr som sier at at arrayet fortsatt kjører). Samme regel som for `CameraSession` / `DAQSensorSession`. Det er derfor trygt å dele ressurser med GUI-en, både med og uten `with`:
>
> ```python
> with chloros_sdk.attach_array(serials) as arr:
>     arr.capture("output/", processing="reflectance")
> # … the array stays up for the GUI
> ```
>
> Når du faktisk ønsker å avslutte den – for eksempel for å koble til en annen `wire_ceiling_mbps` – må du kalle `arr.force_disconnect()`. Å koble til på nytt etter en vanlig `disconnect()` på et tilkoblet håndtak mislykkes med «Kameraet er<sn> allerede i matrisen<id>», fordi ingenting ble frigjort.

### Hjelp til nettverksanalyse

Nyttig før du åpner matrisen – viser om de foreslåtte innstillingene vil passe:

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
    # width / height are sensor pixels BEFORE binning (what connect_array
    # takes with binning=); out_width / out_height is what is streamed.
    print(f"Connect with width={r['width']} height={r['height']} "
          f"binning={r['binning']} ({r['out_width']}x{r['out_height']} streamed)")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

```python
analyze_array_network(
    master_serial,
    slave_serials=None,
    width=2048, height=1536,              # sensor pixels before binning
    pixel_format=None,                    # None = what connect applies (Mono12 / BayerRG12)
    binning=1,                            # 1 | 2 | 4
    force_tier=None,
    packed="auto",                        # "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",
    timeout=30.0,
) -> dict
```

Det starter aldri en backend: en som ikke svarer, utløser `ChlorosConnectError`, mens en avvist forespørsel kommer tilbake som resultat. En feilaktig `packed` utløser `ValueError`.

`status` er en av følgende: `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (ellers `error` — inkludert en avvist forespørsel: ikke logget inn, `error_code: AUTH_REQUIRED`, eller en plan under SDK-grensen, `PLAN_UPGRADE_REQUIRED`). `auto_capped_fps` betyr at den forespurte oppløsningen kun passer til RX-ringen ved en begrenset utløserfrekvens — behold oppløsningen og gå fra `target_fps=result["recommended"]["recommended_target_fps"]` til `connect_array` (se [Eksempel 6](#6-capability-probe-before-connecting-a-4-cam-array)). `auto_shrunk` vises kun når den forespurte rammen ikke kan leveres ved noen bildefrekvens; connect reduserer aldri rammer på egen hånd, så bruk `recommended` selv.

**Hvordan lese projeksjonen** (samme modell som GUI-panelet «Array Settings»):

- **Burst (`frame_bytes_total`) summeres per kamera i hvert kameras reelle pikselformat.**Mono**M3M**-kameraer strømmer Mono12 (2 B/px), eller med en `pixel_format` i Mono-formatet til dybden (`"BayerRG8"` er `Mono8`, 1 B/px), så uten dette blir et fulloppløsningsbilde fra fire kameraer**~25 MB** med tre monokameraer, ikke de ~12,6 MB som en antagelse om at alt er 8-bit gir. Backenden bestemmer hvert kameras format ut fra modellen.
- **Admittance (`burst_fits_nic_ring`) tar hensyn til drenering**, ikke «hel burst mot ring»: sim-emit passer når verten tømmer RX-ringen raskere enn kameraene fyller den. En 10G-vert + 1 GbE-kameraer**tillater** full oppløsning selv når bursten overskrider ringen; en 1 GbE-vert blokkerer (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` er et konservativt tak for seriell henting** — `max(readout+emit, N×emit)` med utsending per kamera begrenset til 1 GbE-kamerakoblingen, uavhengig av eksponering. F.eks. ~2,8 fps for et 4-kamera-array med full oppløsning og 12-bit (stemmer overens med runtime-målingene på ~2,7–3,0). Full modell: [CLI-referanse → Array fps &amp; burst-modell](cli-reference.md#array-fps--burst-model).
- **Etter en kontinuerlig opptakssesjon angir projeksjonen hva den holdt.** Et opptak på 30 sekunder eller mer med automatisk hastighet registrerer den hastigheten den stabiliserte seg på ved den aktuelle array-størrelsen på denne datamaskinen; hastighetsregulatorens korte forsøk på en raskere hastighet utelates. Fra da av viser `array-connect`&#x27;linje, `connect_array()`s analyse og panelet «Array Settings» oppgir denne hastigheten når den ligger under modellens, sammen med hva som begrenset den i `cycle_limiter`: `"clock"` (matrisen kjørte med den hastigheten den hadde stabilisert seg på, mens innlesingssløyfen ventet på utløserklokken), `"loop"` (innlesingssløyfen hadde ikke tid igjen), `"capture"` (opptaket kom på etterskudd og hoppet over grupper) eller `"measured"` (ingenting angir hvilken). Modellens tall finnes i `wire_fps_max`. Arrayet stiger fortsatt til det ledningen kan overføre.
- **Over-subscription (`oversubscribed: true`) betyr at ledningen ikke kan overføre arrayet uansett hvilken hastighet kameraene når** — fps-feltene (`achievable_fps_max` / `fps_bright` / `fps_dark`) viser 0, og binning kan ikke løse dette (det reduserer antall byte per bilde, ikke antall byte per sekund ved justert hastighet). En treg kabelforbindelse alene forårsaker ikke dette: bildefrekvensen synker i stedet, og hvis kameraene ikke kan justeres til en lav nok hastighet, blir nivået `slip-emit-and-capture` (seriell). Løsningene er færre kameraer, jumbo-rammer eller et raskere nettverkskort; `max_cams_collision_safe` angir taket for kabelen og kameraene dine. Svaret inneholder også `aggregate_demand_bps`, `collision_safe_ceiling_bps` og `per_cam_floor_bps`. Se [Over-Subscription](#over-subscription-the-per-cam-floor).

### Oppdagelse og oppføring

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

Hver av dem tar kun nøkkelord `backend_url=` og `timeout=30.0`, og returnerer en liste med ordbøker. `discover_lattice_cameras()` (`serial`, `ip`, `mac`, `model`, `lattice_model`, `vendor`, `connected`, `array_id`, …) tar også `auto_start_backend=True`; `list_*`-kallene starter aldri et backend.

---

## Smart-AE / Smart-Capture

LATTICE-matriser kjører kontinuerlig AE i bakgrunnen så snart de er tilkoblet, men det tar et øyeblikk før en nyinnstilt scene konvergerer. **Smart-capture** er den praktiske løsningen: den avleser eksponeringen på hvert kamera, venter til arrayet er stabilt over et vindu, og utløser deretter opptaket. Det tilsvarer GUI: «smart»-opptaksknappen i skrivebordsappen påkaller det samme backend-endepunktet.

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

Når du styrer via `ChlorosProject` (neste avsnitt), får du flere innstillingsmuligheter:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Smart-AE-innstillingen er konservativ som standard. Stram inn `exposure_tolerance_pct` for krevende radiometrisk arbeid; løsne for raskt skiftende scener der du bare ønsker «nær nok».

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

# A specific sensor by id (as list_daq_sensors() shows it; ':' or '-' alike)
daq = chloros_sdk.connect_daq_sensor(sensor_id="CB-7C-A7-5A-73")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

`sensor_id=` angir én sensor, og kun denne sensoren åpnes noensinne. Den sammenlignes med sensorene som allerede er åpne, deretter blir et DAQ-E-enhets vertsnavn eller IP-adresse (et DAQ-E-enhet som nettverksskanningen kun ser via IP-adresse blir spurt, skrivebeskyttet, om navn, MAC-adresse og serienummer, slik at alle disse også finner den), en DAQ-Ms BLE-MAC-adresse og ID-en som en DAQ-U eller DAQ-M rapporterer når den er åpnet (alle andre som åpnes for å sjekke, lukkes igjen). Hvis den ikke blir funnet, utløser `connect_daq_sensor` `ChlorosConnectError` med en liste over hva som ble funnet; inkludert en adresse (`port=` / `mac=` / `eth_host=`) som inneholder en annen sensor, utløses den også. Den faller aldri tilbake til den sensoren som er synlig – det er det null-flagg-kallet er til for.</id></sn></id></sn>

### `connect_daq_sensor()`-signatur

```python
connect_daq_sensor(
    *,                                    # every argument is keyword-only
    sensor_id=None,                       # one named sensor (see above)
    transport=None,                       # "usb" | "ble" | "eth"; None = smart-detect
    port=None,                            # DAQ-U serial port ("COM3", "/dev/ttyUSB0")
    mac=None,                             # DAQ-M BLE MAC (implies transport="ble")
    eth_host=None,                        # DAQ-E hostname or IP (implies transport="eth")
    integration_time=None,                # ms; None = the sensor's own
    frame_avg=None,
    enable_ae=None,
    cap_id=None,                          # cap correction: "sunshine_cosine", "fov_30", ..., "none" (bare)
    start_streaming=True,
    backend_url="http://127.0.0.1:5000",
    timeout=60.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> DAQSensorSession
```

En sensor som allerede er i puljen, returneres som den er (lånt): `integration_time`, `frame_avg` og `enable_ae` brukes ikke på den. Koble den fra først for å åpne den med dine innstillinger. `cap_id` er unntaket: på en lånt sensor settes den med `set_cap()`. Hvis den utelates, gjelder backendens standardinnstilling (se merknaden under metodetabellen).

### `DAQSensorSession`-metoder

| Metode | Beskrivelse |
| --- | --- |
| `status(timeout=10.0)` | Sammendrag av pooloppføring (strømnings-/opptaksstatus, bølgelengdeområde, kalibrerings-sha, integrasjonstid, frame_avg, AE-status). For en DAQ-E der strømmen har stoppet: `reconnect` (der backendens automatiske gjenkobling gjelder: `state`, `attempts`, `next_attempt_in_s`, `last_error`; `None` mens den strømmer), `stream_lost_s`, `reconnects` og `last_reconnect`, samt `record_gaps` for opptaket som pågår; `rx_stream_fault` angir tilkoblingsstatusen inntil det første spektrumet ankommer. |
| `latest(*, n=1, timeout=10.0)` | Returner opptil N av de nyeste spektrumrammene (hver `ts_ns`, `spectrum`, `is_saturated`, `integration_time`, `x`, `y`, `z`). |
| `stream_start()` / `stream_stop()` | Gjenoppta / pause strømming (håndtaket forblir åpent). |
| `record_start(*, output_dir=None, device_name=None, timeout=15.0)` | Start opptak av en .daq-fil. Returnerer filstien. Avvises for DAQ-U/M uten et AWS-kalibreringspakke (DAQ-E er unntatt). |
| `record_stop()` | Stopp opptak. Returnerer `{path, rows, warning, gaps}`; `warning` er `None` for et feilfritt opptak, ellers backendens diagnose (for eksempel en tom fil eller et gap i strømmen), som også skrives ut til stderr. `gaps` viser hver strekning der strømmen ikke leverte noe: `start_utc`, `end_utc` (`None`: fortsatt tapt ved stopp), `duration_s`, `reconnected`, `marked_in_file`. Den er tom ved kontinuerlig opptak. |
| `set_cap(cap_id, timeout=15.0)` | Angi profilen for toppkorreksjon som skal brukes på målingene fra og med den neste: `"sunshine_cosine"`, `"fov_15"` ... `"fov_90"`, eller `"none"` (eller `None`) for en sensor uten deksel. Returnerer sensorenssammendrag, som `cap_id` leser tilbake. En DAQ-E-S holder `as_recorded` uansett hva som blir forespurt (dens diffusor er innebygd). Øker `ChlorosConnectError` for en profil som ikke lastes inn. CLI-kommando: `chloros-cli daq pool-set-cap`. |
| `enable_ptp(timeout=15.0)` | Kun DAQ-E: slå PTP på igjen for en enhet der PTP er slått av (`status()["rx_ptp_enabled"]` er `False`; innstillingen lagres på enheten, så den forblir avslått inntil den slås på igjen). Returnerer den oppdaterte statusen. Utløser `ChlorosConnectError` for en DAQ-U / DAQ-M, eller når enheten ikke bekrefter. Chloros slår aldri en enhet på igjen av seg selv, fordi «av» kan være bevisst. CLI-kommando: `chloros-cli daq pool-enable-ptp`. |
| `device_info(timeout=15.0)` | Kun DAQ-E: enhetens egne innstillinger. `firmware` (`running`, `bundled`, `update_available`), `ip` og `ip_mode` (`dhcp` / `linklocal` / `static`: det som kjører nå), `network` (den lagrede innstillingen: `mode`, pluss `ip` / `mask` / `gateway` når den er i statisk modus), `network_change_pending` (gjelder inntil enheten kjører den lagrede innstillingen, en som er lagret med `set_network(..., reboot=False)` inkludert), `auto_stream`, `ptp`. `firmware["update_available"]` er `None` når enhetensversjon ikke kunne leses. Mens enhetens strøm ikke når denne datamaskinen, leses den ved en ny tilkobling, og ordboken legger til `stream_silent` (`True`), `stream_fault` og `reconnect`; en enhet som ikke svarer, utløser feilen CLI: `pool-info`. |
| `set_network(mode="auto", *, ip=None, mask=None, gateway=None, reboot=True, force=False, comeback_timeout=45.0)` | DAQ-E kun: DHCP (`"auto"`) eller en statisk adresse. Enheten startes på nytt for å ta innstillingen i bruk og kobles til på nytt; denne økten fortsetter å fungere, og `address` følger enheten. `reboot=False` lagrer den bare. Se [DAQ-E-enhetsinnstillinger](#daq-e-device-settings). CLI: `pool-set-network`. |
| `set_auto_stream(enabled, timeout=15.0)` | Kun DAQ-E: strøm ved oppstart uten tilkoblet programvare (lagret på enheten). Returnerer den bekreftede innstillingen. CLI: `pool-set-auto-stream`. |
| `reboot(*, comeback_timeout=45.0)` | Kun DAQ-E: start enheten på nytt og koble den til igjen med de innstillingene den hadde. CLI: `pool-reboot`. |
| `update_firmware(*, comeback_timeout=45.0)` | Kun DAQ-E: oppdater fastvaren som følger med denne Chloros-utgivelsen til en enhet som kjører en eldre versjon, og koble den deretter til igjen (tar omtrent ett minutt; ikke slå den av). Returnerer `updated`, `fw_before`, `fw_after`, `bundled` og en `warning` når enheten var treg med å komme tilbake eller kom tilbake med en annen versjon; `updated` er `False` for en enhet som allerede har den nyeste versjonen. Utløser `ChlorosConnectError` for en enhet hvis versjon ikke kan leses. CLI-kommando: `pool-update-firmware`. |
| `disconnect()` | Frigjør fra puljen. Ingen-op for tilknyttede håndtak som ikke eies. På en sensor som denne økten har åpnet, er det `force_disconnect()`, og utløser `ChlorosConnectError` når frigjøringen ikke er bekreftet (utgangen fra `with`-blokken utløser aldri feil). |
| `force_disconnect(timeout=30.0)` | Frigjør sensoren uavhengig av hvem som åpnet den — for eksempel en sensor som et krasjet skript har etterlatt i puljen, og som et nytt skript kun når som et tilknyttet håndtak der `disconnect()` ikke utfører noen handling. Å avslutte en annen klients sensor avslutter all innspilling den foretar. Returnerer så snart backenden bekrefter frigjøringen, eller når det ikke er noe å frigjøre (sensoren er ikke i puljen, eller ingen backend kjører); en frigjøring som har gått ut på tid eller som backenden har avvist, utløser `ChlorosConnectError`. CLI: `chloros-cli daq pool-disconnect --sensor-id ID`. |

Attributter: `sensor_id`, `model`, `transport`, `address` (følger etter en DAQ-E som `set_network()` har omadressert), `info` og `backend_url`.

> **Gi lokket et navn.** DAQ-U, DAQ-M og DAQ-E leveres uten kondensator, og Chloros kan ikke se om det er montert en: en sensor som er koblet til uten `cap_id` får `sunshine_cosine`, en antagelse som registreringene tyder på at de har gjort (`cap_id_source=auto_default`). En ubeskyttet DAQ-U gir med denne korreksjonen en avlesning som er ca. 30 ganger for høy, så send `cap_id="none"` for en ubeskyttet sensor (eller `set_cap("none")`), og profilen til hetten du ellers har montert.

### DAQ-E-S — fast diffusor og vippesensor

**DAQ-E-S** er en DAQ-E med to forskjeller som har betydning for programvaren. Alt annet — oppdagelse, tilkobling, PTP, opptak og enhetsinnstillingene nedenfor — fungerer nøyaktig som for en DAQ-E.**Diffusoren er fastmontert, så det kreves ingen dekselkorreksjon.** Diffusoren i Sunshine-stil er innebygd og var på enheten da fabrikkalibreringen ble målt, slik at kalibreringen allerede viser den faktiske nedadgående strålingsintensiteten. Chloross setter sin cap til `as_recorded` (ingen profil) uansett hva som blir forespurt: en cap oppkalt etter den (`sunshine_cosine`, `fov_*`, `none`) blir ignorert med en melding, og opptakene lagrer `cap_id_source=model`, slik at advarselen om antatt cap aldri utløses for den. Ikke sett en begrensning på dataene manuelt heller: `sunshine_cosine` på toppen vil vise en verdi som er omtrent 11 ganger for høy.

**Den har et 3-akset akselerometer.** Sollysdiffusorens kosinusrespons er kun gyldig når den er i vater, så en skråstilt sensor rapporterer feil innstråling — og refleksjonen arver feilen — uten andre symptomer. Hver DAQ-E-S-avlesning inneholder derfor sin egen orientering:

| Felt | Betydning |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Vinkler i grader. `None` når fastvaren markerer vinkelen som udefinert (fritt fall, en død akse). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Akselerasjon i milli-g. `mag_mg` langt fra ~1000 betyr at enheten var i bevegelse og at vinklene ikke er meningsfulle for den aktuelle prøven. |
| `cal_applied` | Innstilt: aksene og vinklene er i diffusorens referanseramme (enhetens monteringskalibrering er anvendt). Uinnstilt: rå referanseramme for kortet, som kan vise opptil ~2,3° helling som ikke er reell. |
| `sample_age_ms` | Alderen på akselerometerprøven da spektrumet ble sendt. |
| `temp_raw` | Ukalibrert **trend** for die-temperatur — kun forskjeller, aldri en termometermåling. |

`DAQSensorSession.latest()` returnerer dem under hver rammens `imu`-nøkkel (`None` på en DAQ-U, DAQ-M eller vanlig DAQ-E). En `.daq` registrert fra en DAQ-E-S lagrer de samme verdiene i sine `imu_*`-kolonner (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) ved siden av hvert spektrum, slik at leseren kan skille mellom en sky og et veltet stativ. På alle andre modeller er disse kolonnene tomme.

Holdningsstrømmen rapporterer om sine egne felt i `status()` (`rx_imu_stream_fault`, `rx_imu_stream_faults`, `rx_last_imu_stream_fault`), holdt adskilt fra spektrumfeltene slik at et utfall i akselerometeret aldri tolkes som et hull i opptaket. `discover_daq_sensors()` og øktens `model` angir `DAQ-E-S`.

### DAQ-E-enhetsinnstillinger

```python
with chloros_sdk.connect_daq_sensor(eth_host="daq-e-def330.local") as daq:
    info = daq.device_info()
    print(info["ip"], info["ip_mode"], info["firmware"])

    # A fixed address on the network this computer is on
    daq.set_network("static", ip="192.168.1.50", gateway="192.168.1.1")
    print(daq.address)              # 192.168.1.50: the session follows it

    if info["firmware"]["update_available"]:
        daq.update_firmware()
```

- **Det må finnes en statisk adresse i nettverket til denne datamaskinen.** `set_network` utløser `ChlorosConnectError` for en enhet hvis undernett ikke inkluderer denne datamaskinensadresse på sensorens nettverk: etter omstarten vil ingenting kunne nå sensoren for å endre den tilbake. Godkjenn `force=True` bare når denne datamaskinen skal omadresseres eller rutes til det nye subnettet.
- **`set_network`, `reboot` og `update_firmware` starter sensoren på nytt**, og utløser `ChlorosConnectError` mens den registrerer, og når den ikke kommer tilbake innen `comeback_timeout` sekunder. Sensor-ID-en og vertsnavnet `daq-e-<id>.local` endres aldri.
- **En opptak som ble startet like før omstarten, avsluttes av den og får et navn.** Når sensoren kommer tilbake, inneholder det returnerte ordboken `recording_closed` (filens sti) og en `warning` som angir dette. Når den ikke kommer tilbake, eller hvis firmwareoverføringen mislykkes, angir `ChlorosConnectError` dette i stedet: meldingen avsluttes med en advarsel, og `e.recording_closed` / `e.warning` settes (begge `None` når ingen opptak ble avsluttet). Start opptaket på nytt så snart sensoren er tilbake.
- **Bare selve sensoren kobles til på nytt.** Etter omstarten må enheten svare der den var -- og etter `set_network("auto")` også under navnet `.local` -- med navn, vertsnavn og modell, fra nettverkskortet (MAC) den hadde; `address` blir IP-adressen som svarte. Et svar under navnet teller kun med den MAC-adressen, så fra en datamaskin som når sensoren via en ruter, eller fra WSL2s standard NAT-nettverk, blir en sensor som DHCP har flyttet ikke koblet til på nytt (`ChlorosConnectError`; koble den til igjen på den nye adressen).
- **En firmwareoppdatering krever at sensoren kobler seg til igjen.** `update_firmware()` inviterer enheten, og enheten åpner en TCP-forbindelse tilbake til denne datamaskinen for å hente bildet, på en port som operativsystemet velger. Backenden lytter på nettverkskortet som når enheten (på alle nettverkskort hvis den ikke kan skille mellom dem) og betjener kun den enheten: alle andre tilkoblinger avvises (`[DAQ-FW] … refused a firmware connection from <ip>` i backendens utdata) mens den fortsetter å vente. En brannmur som blokkerer tilkoblingen fører til at oppdateringen mislykkes med feilmeldingen `device never connected back on TCP <port>`. Windows-installasjonsprogrammet legger til regelen for dette (se [Trinn 1](#windows-exe)); på en annen brannmur må du tillate innkommende TCP-trafikk til backenden fra sensorens undernett.
- **Under WSL2 må du bruke speilet nettverk** (`networkingMode=mirrored` i `.wslconfig`): i standard NAT-modus avvises alle statiske adresser (WSLs egen adresse er ikke på sensorens nettverk), strømmen når ikke WSL, og en firmwareoppdatering kan ikke fullføres.
- GUI-en opprettholder sin egen tilkobling til en sensor: en omstart fra et skript avslutter GUI-ens økt med sensoren. GUI-en har de samme innstillingene (fanen «Lyssensorer», sensorens innstillinger, «Nettverk»).

### Oppdagelse — finne en adresse å koble seg til

`discover_daq_sensors()` skanner USB / BLE / ETH etter sensorer du *kunne* åpne. Det er DAQ-motstykket til `discover_lattice_cameras()`, og den eneste måten å finne **DAQ-M-enhetens BLE-MAC** på — en DAQ-E har et vertsnavn og en DAQ-U en COM-port, men en MAC-adresse er verken trykt på enheten eller oppført av operativsystemet.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               DAQ-U  {'vid': '0403', 'pid': '6001', 'manufacturer': 'FTDI'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Felt | Beskrivelse |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM-port / BLE MAC / vertsnavn — sendes videre til `connect_daq_sensor` som `port=` / `mac=` / `eth_host=`. |
| `display` | Lesbar etikett. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` \| `DAQ-E-S`, eller `None`. En Ethernet-sensor er `DAQ-E-S` eller `DAQ-E`, slik den identifiserer seg selv når den blir spurt (skrivebeskyttet) under skanningen (`DAQ-E` når den ikke svarer i tide). En USB-serieport viser `DAQ-U` når brobrikken er av en type som DAQ-U bruker (FTDI, CH340, CP210x, Prolific) — et hint fra portens beskrivelse, ikke en probe — og `None` for alle andre USB-enheter, som fremdeles er oppført, slik at du kan sende den som `port=`. En seriell port uten USB-leverandør-ID og uten en slik brobrikke (en UART på hovedkortet, for eksempel Linuxs `/dev/ttyS*`, en seriell Bluetooth-forbindelse, Intel AMT) kan ikke være en DAQ-U og er ikke oppført. |
| `extra` | Detaljer per transport (BLE-annonsert navn; USB `vid`, `pid`, `manufacturer`, `serial_number`; DAQ-E ip/fw/…). Tomme verdier er utelatt. |

| Parameter | Standard | Beskrivelse |
| --- | --- | --- |
| `transports` | alle tre | Sekvens (eller CSV-streng) som begrenser skanningen. Det lønner seg å angi denne når du vet hva du vil ha — BLE er den tregeste delen. |
| `scan_timeout` | 5 | Skanningsvindu per transport i sekunder; backenden begrenser til 1–20. |
| `timeout` | 60,0 | HTTP-tak for hele anropet (som andre steder i SDK). |
| `auto_start_backend` | `True` | Starter en lokal backend hvis ingen kjører. Startes aldri for en ekstern `backend_url`. |

> **Sensorer som allerede er åpne i puljen, vises ikke.** En tilkoblet BLE-periferienhet slutter å annonsere, og en åpen COM-port kan ikke undersøkes, så oppdagingslisten viser hva som er *tilgjengelig for tilkobling*. Det er forventet et tomt resultat rett etter at du har koblet til noe — bruk `list_daq_sensors()` for det du allerede har. Transportprotokoller som ikke kan skannes (ingen bleak / zeroconf installert) hoppes over i stedet for å gi feilmelding, slik at en maskin uten Bluetooth fortsatt får svarene for USB og ETH.

### Liste

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Hvilken DAQ forsyner hvilket kamera — `daq_bindings()`

Refleksjonsverdien til en opptak og `.daq` den lagrer, bruker DAQ-en som er knyttet til det kameraet eller den matrisen, ellers den nyeste DAQ-en som backenden har. Valg av lyssensor i GUI-en og `chloros-cli daq pool-bind` angir dette; det samme gjør:

```python
chloros_sdk.set_daq_bindings(cameras={"214701281": "COM7"},          # a sensor id or an address
                             arrays={"array-1779862544497": "daq-e-def330"})
chloros_sdk.set_daq_bindings(cameras={"214701281": None})            # None clears it
for serial, b in chloros_sdk.daq_bindings()["cameras"].items():
    print(serial, b["binding"], b["source"], b["resolved"]["in_pool"])
```

`daq_bindings()` returnerer `{"cameras": {serial: b}, "arrays": {array_id: b}}`: hver `b` inneholder `binding` (den angitte verdien), `source` (`project`, fra prosjektets `sensors.json` eller `runtime`), `known` / `known_as` (om kamera- eller matrisenøkkelnavnene er kjent for en Chloros, og hvordan: `connected`, `project` — det åpne prosjektets `cameras.json` — eller, for et kamera, `seen`: åpnet på denne datamaskinen tidligere; `known: False` for en nøkkel som ikke navngir noe) og `resolved` (pool-sensoren den nå er tilordnet: `in_pool`, `sensor_id`, `address`, `model`, `streaming`). En binding under kjøring har forrang fremfor prosjektetsfor sesjonen; prosjektfilen endres ikke. Når et kamera, en matrise eller en DAQ kobles fra, fjernes alle bindinger til den aktuelle maskinvaren, uansett hvem som har satt dem (medlemmene i en matrise bindes sammen med matrisen).

`set_daq_bindings()` binder kun et kamera som er tilkoblet, i det åpne prosjektet eller som tidligere har vært åpnet på denne datamaskinen, og et array som er tilkoblet eller i det åpne prosjektet (eller en tast som allerede er bundet). Enhver annen tast utløser `ChlorosConnectError` med navnet (`400`, `code`, `UNKNOWN_BINDING_TARGET`, sammen med tastene `unknown`), og **ingenting** i det anropet blir bundet. `None` sletter alle nøkler.

### Omgivelses- + objektreflektans — `light_sensor_reflectance()`

To samkjørte DAQ-er, én som ser opp mot himmelen (omgivelser) og én ned mot målet (objekt), gir målets reflektans og vegetasjonsindekser fra de siste målingene: GUI-paret «Omgivelser + Objekt» og `chloros-cli daq reflectance` uten å åpne sensorene en gang til. Begge må være tilkoblet og overføre data.

```python
r = chloros_sdk.light_sensor_reflectance("daq-e-def330", "daq-e-54b774")
print(r["reflectance_type"])                       # "Apparent Reflectance" when both are calibrated
for name, band in r["bands"].items():
    print(name, band["rho"] if band["valid"] else band["reason"])
for idx in r["indices"]:
    print(idx["name"], idx["value"])
```

```python
light_sensor_reflectance(
    ambient_sensor_id, object_sensor_id, *,
    bands=None,          # [{"name", "center_nm", "fwhm_nm", "role"}]; role blue / green / red / nir
    preset=None,         # a preset id from daq_band_presets(); default: the suggested one, else "GENERIC"
    expressions=None,    # custom indices: [{"name": "MyNDVI", "expr": "(NIR_850 - Red_660) / (NIR_850 + Red_660)"}]
    builtins=True,       # include the built-in indices
    backend_url="http://127.0.0.1:5000", timeout=30.0,
) -> dict
```

- **Bånd.** Hvert bånds reflektans er objektets bånd-integrerte irradians over omgivelsene. `daq_band_presets(filter_code=None)` viser båndsettene: ett per LATTICE M3C-filter (`RGN`, `OCN`, `NGB`, kameraets egen båndmodell, slik at en DAQ-NDVI og en kamera-NDVI er den samme størrelsen) og `GENERIC` bredbånd. Dets `suggested` er den forhåndsinnstilte som samsvarer med `filter_code`, ellers filteret til det første tilkoblede kameraet (`suggested_from` angir hvilket).
- **Returnerer** `bands` (per bånd `rho`, `valid`, `reason`, `e_ambient`, `e_object`), `indices` (de innebygde-inns), `custom` (hver med `value`, eller `valid: False` og `error`), `invalid_bands`, `reflectance_type`, `preset` og de to målingene `timestamps` (`ts_ns`). Et bånd som omgivelsessensoren ikke kunne måle (utenfor måleområdet, ved støynivået) er ugyldig, med angivelse av årsaken, og alle indekser som trenger dette utelates, og beregnes aldri mot null.
- **Tilsynelatende eller relativ.** De skalavhengige indeksene (SAVI, OSAVI, EVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) trenger et absolutt forholdstall, så de oppgis kun når begge sensorene er strømkalibrert (hver har en `calibration_bundle_sha`): `"Apparent Reflectance"`. Ellers blir resultatet `"Relative Reflectance"`, med kun de skalainvariable indeksene.
- **To forskjellige bølgelengdegitter** sammenlignes ut fra hvor de overlapper hverandre: omgivelsessensorens punkter innenfor objektets rekkevidde, og objektets spektrum interpolert inn på disse.
- **To sensorer, aldri én.** Å gi én sensor begge navnene (samme ID, uansett hvilken skrivemåte poolen godtar) gir `ChlorosConnectError`: en sensors avlesning av seg selv er 1,0 på hvert bånd, noe som ikke måler noe. Backenden avviser det også (`POST /api/daq/indices` svarer `400`, `code` `SAME_SENSOR`, når `ambient_sensor_id` og `object_sensor_id` som SDK-et sender, er én samlet sensor, f.eks. dens ID og port).

### Felles bruk med CLI-en og andre skript

Backendens DAQ-sensorpool deles av alle CLI- og SDK-klienter på maskinen. Hvis en sensor allerede er i poolen — åpnet av `chloros-cli daq pool-connect` eller av et annet skript — returnerer et kall til `connect_daq_sensor(port="COM3")` fra Python et lånt håndtak: dens `disconnect()` er en no-op, slik at skriptet ditt ikke river sensoren vekk fra klienten som åpnet den når scope avsluttes. Det samme gjelder for `sensor_id=` (skrevet med `:` eller `-`, begge deler samsvarer), for en DAQ-E som ble navngitt etter IP-adressen da den ble åpnet med vertsnavn (eller omvendt), og for smart-detect med null-flagget når backenden svarer med en sensor den allerede hadde — en `with`-blokk frigjør kun en sensor den selv har åpnet.

Chloros-GUI-en er **ikke** en del av denne puljen: DAQ-panelet i GUI-en kjører sin egen tilkobling til sensoren, så `connect_daq_sensor` låner aldri en sensor som GUI-en har åpen. Ikke bruk den samme sensoren fra GUI-et og fra et skript samtidig. En DAQ-U eller DAQ-M som GUI-et har åpen, vil normalt bare mislykkes i å åpnes, men en DAQ-E tillater en ny sesjon: skriptets tilkobling endrer enhetens innsamlingsinnstillinger i GUI-en, og `disconnect()` stopper enhetens strøm, noe som avslutter GUI-ens livevisning og eventuelle opptak den foretar. Lukk sensoren i GUI-en først.

### PTP Grandmaster — `time_sync_status()`

Backenden kjører PTP-grandmasteren som LATTICE-kameraer og DAQ-E-enheter henter tidsstemplene sine fra. Den starter på forespørsel: `connect_camera()`, `connect_array()`, `connect_daq_sensor()` og `connect_all()` starter den hvis den ikke kjører — inkludert en tilkobling som deretter mislykkes, for eksempel når `connect_all()` prøver å koble til en DAQ-E som ikke er til stede — og det samme gjør `test_array_sync()` og hvert anrop som åpner kameraer for én kommando (`capture_cameras_transient()`, `start_multi_camera_stream()`, justeringsanropene, `capture_live_index()`).

- **Hvor det kunngjøres.** På de kablede nettverkskortene som vender mot kameraene (inkludert en `169.254.x.x` link-local-adresse), ellers på kortet til standardruten hvis det er kablet. Aldri på Wi-Fi eller et virtuelt kort (Tailscale, WSL): det er ikke noe kamera eller DAQ-E bak et slikt. Hvis ingen slik adapter er oppe, kjører den med annonsering på ingenting (`health` `NO_SOCKETS`) og binder seg til den neste adapteren som aktiveres når et kamera eller en DAQ kobles til.
- **Etter et tilkoblingsbrudd.** Når kameraadapteren mister tilkoblingen og får den tilbake (en kabel trekkes ut, en bryter byttes ut, en USB-adapter registreres på nytt), gjenoppbygger grandmasteren sine sokler av seg selv, uten omstart, og sjekker på nytt alle tilkoblede DAQ-E-enheter, og aktiverer på nytt den som har blitt stående fast i gapet. Den gjenoppbygger også når alle kameraer og DAQ-E-enheter den betjener, er inaktive i 15 sekunder.
- **Når den oppdager en ledningsfeil?** Les `time_sync_status()` (`GET /api/time-sync/status`). `state` angir bare at grandmaster-tråden kjører; `health` angir om pakkene sendes ut: `OK` (ingen feil oppdaget), `STOPPED`, `NO_SOCKETS`, `SENDS_FAILING`, `DEGRADED` (noen adaptere svikter) eller `PEERS_SILENT` (kameraene og DAQ-ene den betjente sluttet å svare), med en setning i `health_detail`. `announce_sent` / `sync_sent` / `delay_resps_sent` teller kun meldinger som har gått ut; `announce_send_errors`, `sync_send_errors`, `follow_up_send_errors`, `delay_resp_send_errors`, `last_send_error`, `failing_interfaces` og `last_delay_req_age_s` beskriver de som ikke ble sendt, og `rebinds` / `last_rebind` omhandler gjenoppbyggingene. `sync_sends_slow` teller synkroniseringene der sendingen tok mer enn 200 us, og `sync_send_max_us` er den lengste; opprinnelsestiden for hver Follow_Up teller maksimalt 200 us av sendingen. Grandmasteren kjører i en egen prosess (`own_process`, `process_pid`), med høy prioritet på Windows (`process_priority`), slik at en opptatt backend ikke kan forsinke tidsstemplingen og justere klokken til alle kameraene samtidig; hvis denne prosessen ikke kan startes, kjører den inne i backenden som før (`own_process` false).

```python
st = chloros_sdk.time_sync_status()
if st.get("health") not in (None, "OK", "STOPPED"):
    print("PTP:", st["health"], "-", st.get("health_detail"))
    chloros_sdk.restart_time_sync()
```

| Funksjon | Hva den gjør |
| --- | --- |
| `time_sync_status()` | Grandmasterens tilstand, helse, klokkeidentitet, prioriteringer og meldingstellere (ovenfor). `chloros-cli time-sync status`. |
| `time_sync_peers()` | PTP-slavene den hører (kameraer, DAQ-E-er): `peers`, hver `port_identity`, `last_seen_ip`, `last_delay_req_age_ms`, `delay_req_count`. `time-sync peers`. |
| `time_sync_cameras()` | Hvert tilkoblet array-kamera: `PtpStatus`, `PtpOffsetFromMaster` og `PtpMeanPathDelay` (et kamera hvis fastvare ikke har noen PTP-noder, har kun serienummeret). `time-sync cameras`. |
| `restart_time_sync()` | Stopp og start grandmasteren på nytt; PTP-en til hver samlet DAQ-E blir sjekket på nytt etter (`daq_ptp_rechecked`). Utløser `ChlorosConnectError` når den den ikke kommer tilbake. `time-sync restart`. |
| `set_time_sync_priority(priority1=None, priority2=None)` | Dens PTP-prioriteter (0–255, lavere vinner valget av best-master), for et nettverk med en annen PTP-master. Krever at den kjører. `time-sync set-priority`. |

Hver av dem tar `backend_url=` og `timeout=` og starter en lokal backend som ikke kjører.

---

## Kameraverktøy

Hva `chloros-cli lattice` gjør utover å koble til og ta opp, som vanlige funksjoner i `chloros_sdk` (modul `chloros_sdk.camera_utils`). Hver av dem kaller backend-ruten som CLI-kommandoen bruker, med CLI-reglene, slik at de to oppfører seg likt. De tar et kamera **serienummer**, ikke en økt, og de samme `backend_url=` / `timeout=`-nøkkelordene som smart-connect-kallene; en backend som ikke kjører, startes, slik som `connect_camera()` gjør. En avvisning utløser `ChlorosConnectError` med backendens begrunnelse.

```python
import chloros_sdk as c

c.set_camera_color_profile("214701281", profile="natural")
c.set_camera_highlights("214701281", -40)
print(c.camera_firmware_status()["cameras"])
st = c.network_adapters(estimate=True, cameras=4, pixel_format="BayerRG12")
```

| Funksjon | Hva den gjør |
| --- | --- |
| `camera_color_profile(serial)` / `set_camera_color_profile(serial, profile=, custom_cct_k=)` | Skjermens fargeprofil: `raw`, `linear`, `natural`, `enhanced` eller `custom_temp`, og fargetemperaturen (begrenset til 2000–10 000 K; leses når profilen er `custom_temp`). |
| `camera_color_adjust(serial)` / `set_camera_color_adjust(serial, saturation=, contrast=, reset=False)` | Skjermmetning og kontrast, 0–200 % hver (100 = uendret); `reset=True` setter begge tilbake til 100. |
| `camera_highlights(serial)` / `set_camera_highlights(serial, value)` | Reduser de lyseste, hviteste pikslene i skjermbildet: -100..0 (avkortet mot null, deretter begrenset; 0 = av). |
| `camera_white_balance_once(serial, settle_s=None)` | Engangs hvitbalanse: kameraet strømmer kortvarig, konvergerer og returnerer `red_ratio` / `blue_ratio` (`ratios_read` False når det ikke kan rapportere dem). Det beholder hvitbalansen det har beregnet; alle andre innstillinger som «open» skrev, går tilbake til opprinnelig verdi. |
| `set_camera_white_balance(serial, mode=None, red=None, blue=None)` | `mode="auto"` (kontinuerlig) eller `"off"`; eller faste `red` / `blue` balanseforhold, som slår av automatisk først. Et forhold som kameraet avviser, er oppført under `errors`. |
| `camera_presets()` / `apply_camera_preset(serial, preset)` | Kameraets forhåndsinnstillinger, og ved bruk av en av disse: kameraet beholder innstillingene som forhåndsinnstillingen har navn etter (svaret er `settings`, slik kameraet leser dem etterpå, f.eks. bildefrekvensen det kjører med; `None` for et felt som forhåndsinnstillingen ikke navngir), og alle andre innstillinger som åpningen skrev, går tilbake til slik de ble funnet. `"default"` tilbakestiller kameraet til fabrikkstandardene (fabrikkinnstillingene med frittløpende automatisk eksponering, grensen for koblingsgjennomstrømning og feilkorrigering slått av, mens et RGB-kameras kalibrerte hvitbalanse beholdes), uten at dette lagres i minnet: svaret inneholder `factory_defaults: True` og alle feltene i `settings`. I `camera_presets()` er `default` lik `{factory_defaults: True, description}`. |
| `camera_filter_models()` / `camera_filter_model_info(name)` | Filtermodellene og detaljene for én modell (bånd, spektralresponsens kondisjonsnummer). Et kameras filtermodell leses fra modellstrengen; den angis ikke manuelt. |
| `camera_firmware_status(serial=None)` | Hvert kamerafirmware: `current_version`, `available_version`, `update_offered`, `needs_minimum_update` og `minimum_firmware`. |
| `update_camera_firmware(serial, force=False)` | Flashe fastvaren til denne Chloros-versjonen til ett kamera: tilbakestill til fabrikkinnstillinger, flash, omstart, Chloros-grunninnstillingene skrives på nytt; kameraets kalibrering beholdes. Tar flere minutter; kameraet må være strømforsynt. `force=True` flasher et eksisterende kamera på nytt. |
| `camera_power_probe(serial)` | Kameraets strøm- og termiske noder samt temperaturen. |
| `sleep_camera(serial)` / `wake_camera(serial)` | Kameraets strømsparende hvilemodus (det forblir i dvale til det vekkes: grensen for koblingsgjennomstrømning er slått på (`DeviceLinkThroughputLimitMode` `On`) på sitt laveste nivå, strømsparingsmodus er aktiv der kameraet har en slik), med temperaturen før og etter; oppvåkningen setter koblingen tilbake slik den var da den gikk i dvale (grensen, modusen og pakkeforsinkelsen `GevSCPD`, som kameraet holder på 0 mens grensen er aktiv; loggen lagres på denne datamaskinen, slik at en oppvåkning fra en senere backend også finner den), eller, for et kamera som denne datamaskinen ikke har satt i hvilemodus, men som ser ut til å være i hvilemodus (grensen er på ved minimumsnivået), med grensen av og en `GevSCPD` satt til 0 ved 80 (fabrikkinnstillingene); en grense som er satt over minimumsverdien, forblir uendret. Kun strømtilstanden beholdes; alle andre innstillinger som «open» har skrevet, settes tilbake til opprinnelig tilstand. |
| `calibrate_camera_with_target(serial, filter=None, attempts=30, settings=None)` | Ta opp bilder til et MAPIR-kalibreringsmål er i synsfeltet, og tilpass koeffisientene per kanal fra panelene: `target_id`, `panel_dns`, `coefficients`. |
| `network_adapters(estimate=False, cameras=1, pixel_format="BayerRG8")` | Denne datamaskinens Ethernet-kort for GigE Vision (MTU, koblingshastighet, jumbo-rammer) og eventuelle `issues`; `estimate=True` legger til bildefrekvensen og båndbredden som `cameras`-kameraene ville få. |
| `fix_network_settings()` | Bruk GigE Vision-adapterinnstillingene på denne datamaskinenskameraadapter. Dette endrer adapterens innstillinger og fungerer kun når Chloros kjører som administrator; ellers vises en logg med hva programmet fant. |
| `apply_vignette_correction(input_dir, output_dir, lens_model=None)` | Korrigerer vignettering i bildene i en mappe, hvert med sitt eget flatfeltkart for objektivet: `processed`, `lens_models` (objektivet hver fil ble korrigert for), `skipped` (med begrunnelser). Uten `lens_model` hentes objektivet for hver fil fra EXIF-modellen; en `lens_model` gjelder for filer der «Model» ikke angir noe kameranavn, og en fil der «Model» er i strid med dette, hoppes over. Kun filer. |
| `read_dls(mac, filter=None, camera_serial=None)` | Én avlesning fra en DLS (Bluetooth-sensor for nedadrettet lys): spektrum, bestrålingsstyrke per bånd og båndstyrker gjennom kameraets egen respons når `camera_serial` angis. `discover_daq_sensors(transports=["ble"])` finner MAC-adressen. |
| `probe_system(pixel_format="BayerRG12", discover=True)` / `system_info()` | Vertsmaskinens maskinvare for kameraer og den anbefalte konfigurasjonen (`report_text`), samt systemets CPU / GPU / PyTorch-kjøringsmiljø. |

- **Noen trenger kameraet for seg selv.** `camera_white_balance_once()`, `apply_camera_preset()`, `sleep_camera()` / `wake_camera()`, `calibrate_camera_with_target()` og `update_camera_firmware()` åpner kameraet. Mens Chloros holder det (tilkoblet i appen, med `connect_camera()`, som et array-medlem, eller i bruk av en annen kommando akkurat nå, for eksempel en matrisetest eller opptak som åpner kameraene sine), avviser backenden med HTTP-feil 409 `CAMERA_HELD` (både i `code` og `error_code`), som angir hvordan det kan frigjøres. Fargeinnstillingene, høylysene, `camera_power_probe()` og avlesningene fungerer på et tilkoblet kamera.
- **Farge, høylys og hvitbalanse gjelder for RGB-kameraer.** På et enkeltbånds monokromt (M3M) kamera endrer innstillingene ingenting og returnerer `{"skipped": True, "reason": ...}`, slik CLI-en gjør. Dette er skjerminnstillinger: livevisning og opptak på skjermnivå, aldri strålingsintensitet eller refleksjonsgrad.

---

## Transiente matriser, justeringsprofiler og indekser

Hva `chloros-cli lattice` gjør uten en vedvarende tilkobling, i likhet med funksjonene i `chloros_sdk` (modul `chloros_sdk.array_utils`): `continuous`, `multi-capture`, `multi-stream`, `multi-test`, `multi-detect`, `analyze-array`, `align-calibrate`, `align-apply`, `align-stream`, `align-info`, `align-reorder` og `index`. Hver av dem sender forespørselen som CLI-kommandoen sender, med CLIs standardinnstillinger.

```python
import chloros_sdk as c

prof = c.calibrate_alignment_profile(["214000535", "214000551", "214000556"],
                                     "D:/rig/alignment_profile.json", method="orb", model="rigid")
print({sn: t["reprojection_error"] for sn, t in prof["transforms"].items()})

c.capture_aligned_multiband("D:/rig/alignment_profile.json", "D:/rig/out")
idx = c.capture_live_index("D:/rig/alignment_profile.json", preset="NDVI",
                           output_dir="D:/rig/ndvi", capture_level="reflectance")

with c.start_multi_camera_stream(["214000535", "214000551"], "D:/rig/stream", fps=5, count=50) as job:
    print(job.wait()["final_stats"])          # waits for the 50 groups, then closes the cameras
```

- **De åpner kameraene selv.** Hver kommando åpner sine egne kameraer, utfører oppgaven og lukker dem igjen, så den trenger kameraer som Chloros ikke har tilgang til (tilkoblet i appen, med `connect_camera()`, eller som et array-element); backenden avviser med HTTP-kode 409 `CAMERA_HELD` og angir hvordan man frigjør et kamera. En midlertidig åpning tar en stund (ca. 40 sekunder for noen få kameraer). For gjentatte opptak er er `connect_array()` én gang den raskeste løsningen, og den lagrer alle eksporttyper der en midlertidig opptak lagrer et debayert bilde. `calibrate_alignment_profile()` er unntaket: på en tilkoblet matrise av disse kameraene henter den fra matrisens egne bilder (`used_live_array`).
- **`serials`** er kameraene som skal åpnes, hvor det første utløser avtrekkeren; `None` åpner alle kameraene i nettverket.
- **`settings`** er en ordbok med overstyringer av kamerainnstillinger, feltene som CLI-flaggene angir: `exposure_time` (µs), `gain` (dB), `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `frame_rate`, `gev_packet_size` og `preset` (et forhåndsinnstilt kamera med dette navnet). En angitt eksponering eller forsterkning deaktiverer denne automatiske kontrollen, på samme måte som flaggene gjør.
- **`gpu`**: `True` tvinger GPU-warp, `False` tvinger CPU (`--gpu` / `--no-gpu`); `None` lar backenden velge.
- **`keep_settings`**/**`clear_settings`** (alle funksjoner som åpner kameraer, standard `False`): på slutten settes hvert kamera tilbake slik det ble funnet; `keep_settings=True` lagrer i stedet kjøringens innstillinger i kameraets eget minne (frittløpende), `clear_settings=True` tilbakestiller dette minnet til fabrikkinnstillingene — se [Kameraminne](#camera-memory--keep_settings--clear_settings). Svaret (for en jobb, `stop()`s) inneholder `camera_memory`.
- **Stier er backendens**: relative baner blir gjort absolutte i forhold til arbeidsmappen din når backenden er på denne maskinen. Funksjonene for justeringsprofilfiler (`alignment_profile_info()`, `reorder_alignment_profile()`) leser filen på denne datamaskinen, slik CLI-en gjør.

| Funksjon | Hva den gjør |
| --- | --- |
| `start_camera_continuous(serial, output_dir="output", *, format="tiff", jpeg_quality=95, queue_depth=64, settings=None)` | Tar opp fra ett kamera kontinuerlig inntil det stoppes. Returnerer en `CliJob`. Bildene fra en `tiff`-kjøring er den `raw`-filen som en array-opptak skriver av det aktuelle kameraet (ett mosaikkplan ved sensorensegne verdier, ukomprimert); `png` og `jpg` inneholder 8 bit, så disse kjøringene er demosaiserte bilder for visning. En bundet eller samlet DAQ skriver én `.daq` per bilde i kjøringenss `daq/`-mappe (`daq_files` i statistikken). Hvert bilde skrives én gang med fullstendige metadata (modell, opptakstid, objektiv, bånd, kalibrering på et råbilde); `stop()` returneres når hvert enkelt innfanget bilde er skrevet (opptil 25 s), og først da lukkes kameraet. Et kamera som er satt i beredskap for en maskinvareutløser (`TriggerMode` `On`) settes til frikjøring for opptaket (startmeldingens `trigger` angir hva som ble funnet). Ved stopp settes kameraet tilbake slik det ble funnet — utløserinnstillingene og alle innstillinger som kjøringen endret — med mindre `keep_settings=True`, som lagrer kjøringens innstillinger i kameraets eget minne (frittløp), eller `clear_settings=True`, som tilbakestiller dette minnet. En kjøring som ikke tar opp noen bilder, avsluttes automatisk etter 10 sekunder med `stop_reason` `no_frames` og årsaken angitt i `last_error`, og dens `stop()` returnerer denne årsaken som `error`. Bilder som ble droppet fordi skriverne ikke klarte å holde tritt, telles, og `drop_reason` angir årsaken. For et kamera som Chloros-holder, bruk `CameraSession.capture_repeated()`. `lattice continuous`. |
| `start_multi_camera_stream(serials=None, output_dir="output", *, fps=3.0, count=None, format="tiff", jpeg_quality=95, settings=None)` | Strøm synkroniserte bildegrupper ved `fps`, for `count` grupper eller til det stoppes. Returnerer en `CliJob` (dens `start_reply` har `n_cams` og ledningen `pacing`); de endelige statistikkene teller gruppene som er hentet, skrevet og droppet, filene som skriveren ikke klarte å skrive, og gruppene der et kamera ikke leverte noe. `lattice multi-stream`. |
| `capture_cameras_transient(serials=None, output_dir="output", *, format="tiff", jpeg_quality=95, settings=None)` | Én synkronisert opptak, ett debayert bilde per kamera: `results` (`filepath`, `serial`, `frame_id`) og `pacing`. En gruppe der et kamera mangler, oppstår, og angir hvor mange filer som ble lagret først. `lattice multi-capture` uten tilkoblet matrise. |
| `test_array_sync(serials=None, count=10, ptp_settle_s=None, timeout=None)` | `count` synkroniserte opptak med fast eksponering: `sync_groups`, `master`, én `group_timestamps`-rad per utløser (et kamera som gikk glipp av ett bilde, mangler i sin rad) og `sync`: hvert annet kameras avvik fra masteren, utløser for utløser (`offsets_us`: per kamera `mean_us`, `min_us`, `max_us`, `jitter_us`, `groups`), på ÉN klokke (`clock_domain`: `ptp`, eller `device` — PTP av, hvert kameras egen oppetidsklokke, så avviket er når det startet opp, og kun jitteren måler synkroniseringen). En utløser hvis tidsstempler blander enhetsklokken og PTP (PTP av eller fortsatt låser) blir utelatt og telt med (`groups_mixed_domain`), det samme gjelder en der en PTP-klokke fortsatt er mer enn `agree_within_us` fra masteren (`groups_clocks_disagree`); `notes` bekrefter dette. `clock_offset_mean_us` / `clock_offset_jitter_us` er det største gjennomsnittet og den største jitteren per kamera. `success` er falsk, sammen med `error`, når en utløser ble gått glipp av, når færre enn to utløsere har hvert kamera på én klokke, eller når klokkene aldri stemmer overens (`sync.problems`); radene returneres uansett. Den starter PTP-grandmasteren hvis den ikke kjører (`time_sync` sier `grandmaster_running` og `grandmaster_started_by_this_test`), slår på PTP for hvert kamera (slik som `connect_array()` gjør), og før den første utløseren venter den på at PTP-klokken til hvert kamera skal låse seg til grandmasteren og stabilisere seg: `PtpStatus` Slave på én grandmaster, innenfor 100 us av den og stabil i 3 s, og hvert kamera låst på den måten, uten avbrudd, i 120 s — omtrent 2 minutter etter oppstart. En klokke som nettopp har låst seg, avviker med opptil ~200 us i omtrent 2 minutter etter at den leser «Slave», og kameraets egen `PtpOffsetFromMaster` viser ikke dette; 120 s bringer kameraene innenfor ~10 us av en stabil klokke. Et kamera som fortsatt er låst siden en tidligere test der PTP ble stående på, slik denne backenden så det, teller fra den låsingen, så en test rett etter en annen venter ikke på nytt; et kamera som testen slår på, eller et som oppdages med PTP på og som denne backenden ikke har sett låse seg (slått på et annet sted, eller før en omstart av backenden), venter sine 120 s, og i en gruppe med begge kameraene venter de som nettopp har låst seg fortsatt på sin tid. Et kamera som forlater Slave-modus, går over 100 us eller følger en annen grandmaster i 3 avlesninger på rad (1,5 s) under ventetiden, starter tellingen sin på nytt; én eller to slike avlesninger logges (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) og tellingen fortsetter, og en grandmaster-ID som ikke kan leses, blir ansett som ukjent, ikke som en endring. Når tellingen for alle kameraene starter på nytt samtidig (innen ett sekund), er det grandmasterens tid som beveger seg, ikke kameraenes: loggen til backend’logg viser `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` oppfører det i `grandmaster_steps`, og en ventetid som deretter utløper, starter feilmeldingen med `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Grensen utvides ikke for dette. `ptp_settle_s` (0–600; `0` uten avventingstid) angir avventingstiden for kjøringen; andre verdier avvises (HTTP-feil 400). Ventetiden er begrenset til avslutningstiden pluss 90 s (210 s som standard, minst 90 s), og anropet venter i `timeout` s på backenden: som standard 240 pluss avslutningstiden (360 s), slik som `chloros-mcp`s `test_array_sync` gjør. `ptp_lock` har `converged`, `waited_s`, `timeout_s`, `settle_s`, `locked_s` (hvor lenge hvert kamera har vært låst og stått stille sammen), `found_settled` (kameraene som allerede var låst i stabiliseringstiden da de ble funnet) og, per kamera, `status`, `offsets_ns`, `slave_s` (hvor lenge det har vært låst; Ingen: ikke låst) og `remaining_s` (Ingen når ingen av kameraenes PTP slås på). Backend-loggen har en linje hvert 5. sekund med hvert kameras tilstand, hvor lenge det har vært låst og hvor lenge det har igjen. Hvert kamera slås på med PTP slått av; derfra synkroniseres klokkene etter omtrent 10 sekunder. Deretter settes hvert kamera tilbake slik testen fant det – dets PTP, automatisk eksponering og forsterkning holdes av under testens gjennomføring (`ExposureAuto` / `GainAuto`, og en manuell `ExposureTime` / `Gain`), og resten, med mindre `keep_settings=True` — altså et kamera som står igjen med PTP på (`PtpEnable=True`, eller et løp med `keep_settings=True`) forblir låst, og når denne backenden har sett at det har vært låst i stabiliseringstiden, venter ikke en senere test på det. Hvis klokkene ikke stabiliserer seg, er `success` falsk før noen utløser, og `error` angir hva som skal gjøres: et kamera som aldri har låst seg (`not_slave`) betyr at grandmasteren ikke når det (`time_sync_status()`); et kamera som stadig falt ut, navngis etter hvor lenge det hadde vært låst (`remaining_s`); klokker som fortsatt konvergerer (`far`): sjekk `time_sync_status()`, og for å få dem låst før testen, slå på PTP og la det være på (`PtpEnable=True` på hvert kamera, eller `keep_settings=True`; testen gjenoppretter det den fant, så den forblir på), og kjør den deretter. `lattice multi-test`. |
| `detect_array_master(line=None)` | Kameraene på synkroniseringslinjen og en standardmaster: `master_serial`, `slave_serials`, de som `line` har sjekket. `lattice multi-detect`. |
| `analyze_array_models(camera_models=None, *, width=2048, height=1536, n_active=None, force_tier=None, binning=1, packed="auto", pixel_format=None)` | Hvilket utvalg av disse kameramodellene (produktnavn, `"LATT-M3C-L87-FOCN"`) som ville dukke opp på denne datamaskinens nettverk: `tier`, `achievable_fps_max`, kabelsatsen (`max_cams_collision_safe`, `oversubscribed`), `cam_caps` og `fix_suggestions`. Som standard vises modellene til de oppdagede kameraene; er det ikke åpnet noe kamera. For kameraer du har, med innstillingene en tilkobling ville bruke, se [`analyze_array_network()`](#network-analysis-helper). `lattice analyze-array`. |
| `calibrate_alignment_profile(serials=None, save_path="alignment_profile.json", *, frames=3, method="orb", model="rigid", reference_serial=None, checkerboard=None, control_points=None, preview=False, vignette=False, roi=None, mask_path=None, rms_threshold_px=None, per_cam_overrides=None, settings=None, ...)` | Løs en justeringsprofil fra livebilder og lagre den. `method`: `orb`, `akaze`, `phase`, `checkerboard` (sammen med `checkerboard="9x6"`, de indre hjørnene) eller `manual` (sammen med `control_points={serial: [[x, y], ...]}`, referansekameraets også). `model`: `translation`, `rigid`, `affine` eller `homography`. De mer detaljerte alternativene er CLI-flaggene: `max_features`, `ratio_threshold`, `matcher` (`bf` / `flann`), `knn_k`, `prefilter` (`none` / `gradient` / `clahe` / `blur` / `hist_match`), `ransac_threshold_px`, `ransac_max_iters`, `ransac_confidence`, `averaging`, `lock_rotation`, `lock_scale`, `lock_axis`; `per_cam_overrides` er `{serial: {option: value}}`. `rms_threshold_px` nekter å lagre en dårligere løsning. Returnerer `save_path`, `output_width` / `output_height`, `crop_roi`, `reference_serial`, `transforms` (per kamera `reprojection_error`, `num_matches`) og `preview_path`. `lattice align-calibrate`. |
| `capture_aligned_multiband(profile_path, output_dir="output", *, serials=None, format="tiff", bit_depth=None, gpu=None, no_crop=False, bands=None, order=None, vignette=False, per_band=False, per_camera=False, interpolation="linear", border_mode="constant", border_value=0, settings=None)` | Ett justert flerbåndsbilde gjennom profilen: `files` (flerbånds-TIFF-filen med `bands` og størrelse, pluss ett per bånd med `per_band` og ett per kamera med `per_camera`) og `vignette_corrected`. `bands` velger ut og `order` sorterer båndene etter navn (en liste eller en kommaseparert streng). `lattice align-apply`. |
| `start_aligned_stream(profile_path, output_dir="output", *, serials=None, fps=3.0, count=None, ...)` | Strømjusterte flerbåndsbilder (de samme forvrengningsalternativene som `capture_aligned_multiband()`), for `count` eller til det stoppes. Returnerer en `CliJob`; dens endelige statistikk teller `frames_written` og eventuelle `frame_groups_incomplete`. `lattice align-stream`. |
| `alignment_profile_info(path)` | En profilfil oppsummert som `align-info` skriver den ut: `name`, `created`, `method`, `reference_serial`, `sensor_resolution`, `output_width` / `output_height`, `crop_roi`, `camera_serials`, `transforms` (per kamera `model`, `reprojection_error`, `num_matches`, `reference`), `camera_models`, `layer_order` (per kamera `band_names`, `wavelengths_nm`, `enabled`); `profile` er hele filen. |
| `reorder_alignment_profile(path, order=None, enable=None, disable=None)` | Endre profilfilen på stedet: `order` er båndnavn (hvert navn flytter kameraets lag oppover, i den rekkefølgen), `enable` / `disable` er serienumre. Returnerer den nye `layer_order`. `lattice align-reorder`. |
| `index_catalog()` | Indeksforhåndsinnstillingene (`presets`: henholdsvis `name`, `channels`, `formula`) og farge `gradients`. `lattice index --list-presets` / `--list-gradients`. |
| `compute_index(input_path, *, preset=None, formula=None, channel_map=None, output=None, capture_level="unknown", output_format="all", colorize=False, gradient="RdYlGn", vmin=None, vmax=None, percentile=None, bg_mode="clip")` | En indeks over en justert flerbånds-TIFF: en `preset` (`NDVI`, `NDRE`, ...) eller en `formula` over symbolene `channel_map` tilordner til bånd (`{"NIR": "NIR_850", "Red": 0}`, etter navn eller indeks). Et forhåndsinnstilts symbol `channel_map` utelater det ene båndet i filen hvis navn begynner med det (`red` -&gt; `Red_660`, `nir` → `NIR_850_RGN`), navngitt av selve filen (dens båndliste, eller et Chloros-opptak/produkts `Chloros:Band{R,G,B}Name`), så NDVI-data på et Chloros-refleksjonsopptak trenger ikke `channel_map`; to bånd som svarer til ett symbol avvises, ikke gjetes. `capture_level` angir hva båndene inneholder: `reflectance` eller `radiance` gjør verdiene kvantitative; en ukalibrert stabel (`align-apply`) gir kvalitative verdier; `"unknown"` leser det fra filen, inkludert `Chloros:ProcessingLevel` fra en Chloros-fil. Returnerer `stats` (form, 1–99-persentilområdet, LUT-området, den brukte `capture_level`) og den `files` som er skrevet ved siden av inngangen (`<input stem>_<preset>.tif`, `..._LUT.png`) eller ved `output`, en sti: med en bildefiltype navngis filene ut fra stammen (`output="D:/out/ndvi_FRGN.png"` skriver `ndvi_FRGN.tif` og `ndvi_FRGN_LUT.png`), uten en er det et prefiks (`D:/out/ndvi` → `ndvi.tif`, `ndvi_LUT.png`); les stiene fra `files`. De inneholder kameraopplysningene fra opptaket (merke / modell / serienummer, GPS, objektiv, opptakstid i UTC, kalibreringsidentiteten, men aldri dens raw-DN-kalibrering), `Chloros:ProcessingLevel=index`, og indeksen som sitt eneste bånd (`XMP-Camera:BandName`, både TIFF- og LUT-PNG-filene). Kun filer; ingen kamera. `lattice index --input`. |
| `capture_live_index(profile_path, *, preset=None, formula=None, channel_map=None, serials=None, output_dir="output", output_prefix="index", capture_level="reflectance", save_multiband=False, ...)` | Ta opp, kalibrer (`reflectance` eller `radiance`; `raw` hopper over dette), juster og beregne en indeks i ett trinn; `save_multiband=True` beholder den kalibrerte stakken, som `compute_index()` kan hente senere. Returnerer `stats` (sammen med `serials`, hvis bånd den brukte) og `files`. Den åpner `serials` (den første utløser utløseren), ellers profilens kameraer der `layer_order`-oppføringen er aktivert, med referansen først (åpnes selv om båndene er deaktivert); et kamera med deaktiverte bånd blir aldri kalibrert. `channel_map` er kun nødvendig for et symbol som stakken ikke kan svare på, som i `compute_index()`: et forhåndsinnstilt symbols navn tar det ene aktiverte båndet hvis navn begynner med det, sjekket før noe kamera åpnes (ingen eller flere svar blir avvist, ved å navngi dem); et tilpasset `formula` trenger hvert symbol. `index.tif` og LUT-en inneholder stakkens kamera, opptakstid (referansekameraets eksponering, UTC) og objektiv, og begge angir indeksen som sitt eneste bånd (`XMP-Camera:BandName`, f.eks. `NDVI`: farge-PNG-filen angir indeksen, ikke fargekanalene). `lattice index --live`. |

### `CliJob`

Håndtaket som de tre strømmene returnerer. Jobben kjører på backend-siden, som beholder kontrollen over kameraene sine inntil den stoppes.

| Element | Beskrivelse |
| --- | --- |
| `stats(timeout=10.0)` | `running`, `state`, `stats` (bilder lagret / skrevet, droppet, `fps`, `elapsed`, skrivefeil, `stop_reason`; en kontinuerlig jobb også `frames_grabbed` og `frames_queued` – hvert bilde som er hentet inn, blir skrevet, satt i kø, droppet eller mislykkes – og `drop_reason`, `writers` og `write_ms`, gjennomsnittlig tid det tok å skrive én ramme) og `output_dir`. `state` er `running`, `ended` (den har stoppet seg selv og er ikke blitt stoppet ennå), `stopping` (en stopp skriver ut køen sin: live-statistikk, og `stopped_by`) eller `finished` (sluttstatistikken, stoppets `error` og `stopped_by`). En stoppet jobs oppføring beholdes i 10 minutter (høyst de siste 32), for hver klient: et annet skript eller `chloros-mcp` leser den samme tilstanden og de endelige statistikkene. En jobb `chloros-cli` startet (`lattice continuous`, `multi-stream`, `align-stream`) blir stoppet av backenden når CLI-en slutter å sjekke inn i 30 sekunder — avsluttet, maskinen er i hvilemodus — med `stop_reason`, `client_gone` og `stopped_by` i backenden; å lese statistikken herfra holder den ikke i live. En jobb startet fra SDK-et kjører til den stoppes. Hver oppstart sender sin `timeout` som backendens tidsfrist (`client_deadline_s`): et kamera som åpnes etter at den (et som fortsatt starter opp) settes tilbake og lukkes, og ingen jobb startes, og anropet utløser `ChlorosConnectError` (`CLIENT_DEADLINE_PASSED`). Et kamera som en jobb holder, nektes til alle andre åpninger (`connect_camera()`, en ny oppstart, en midlertidig opptak) med HTTP-kode 409 `CAMERA_HELD`, der jobben og den som startet den, oppgis. |
| `stop(timeout=30.0)` | Stopp den og lukk kameraene: `final_stats`, `output_dir` og `error` når stoppet ikke gikk problemfritt. Et nytt anrop returnerer det samme svaret, og det samme gjør en stoppforespørsel fra en annen klient: en som sendes mens den første fortsatt skriver ut køen sin, venter på den (opptil 25 sekunder; etter det utløses en feil, og de endelige statistikkene følger på `stats()`). |
| `wait(timeout=None, poll_s=0.5)` | Vent til den avsluttes av seg selv (når `count` er nådd, en full disk eller en feil), og send deretter `stop()` for å hente de endelige tallene. `timeout` utløser `TimeoutError` mens den fortsatt kjører; en jobb uten `count` avsluttes først når den stoppes. |
| `job_id`, `kind`, `output_dir`, `start_reply`, `stopped` | Dens ID og type (`continuous`, `multi-stream`, `align-stream`), hvor den skriver, svaret fra startruten og om `stop()` har kjørt. |

Bruk den som en kontekstbehandler for å stoppe den ved utgang av omfanget.

---

## Prosjektmappe og oppskrifter

**Prosjektmappen** er mappen der nye prosjekter opprettes, og som deles av Chloros-appen, CLI og SDK (modul `chloros_sdk.system_utils`; `chloros-cli project-folder`):

```python
print(chloros_sdk.project_folder())                # the configured folder, else the default
chloros_sdk.set_project_folder("D:/Flights")       # created if missing
chloros_sdk.reset_project_folder()                 # -> {"path": <default>, "was_set": True}
```

En konfigurert mappe som ikke lenger eksisterer, faller tilbake til standardmappen (`Documents/MAPIR/Chloros Projects`), akkurat som appen gjør. En relativ sti angitt til `set_project_folder()` hentes fra arbeidskatalogen din når backenden kjører på denne maskinen.

**En oppskrift** er YAML-/JSON-handlingslisten som `chloros-cli project run PATH RECIPE` kjører mot et lagret prosjekt. `run_project_recipe()` kjører den med CLI-egen kjører (installert med SDK), så en oppskrift oppfører seg likt uansett om den kjøres fra:

```python
r = chloros_sdk.run_project_recipe("D:/campaign/morning.yaml", "D:/Projects/Field_A")
print(r["ok"], r["actions"])
print(r["log"])                                    # what the CLI would have printed

chloros_sdk.run_project_recipe({"actions": [{"apply": {"name": "FrontLeft", "settings": {"exposure_us": 5000}}},
                                            {"wait": "2s"},
                                            {"capture": {"name": "FrontLeft", "output": "pose_a/", "format": "tiff"}}]},
                               "D:/Projects/Field_A")
```

```python
run_project_recipe(recipe, project_path, *, dry_run=False, echo=False,
                   backend_url="http://127.0.0.1:5000") -> dict
```

- **`recipe`** er en `.yaml` / `.yml` / `.json`-fil eller selve oppskriften: en ordbok med `actions` (`apply`, `capture`, `burst`, `stream`, `sensor`, `wait`), og eventuelt `overrides` og `stop_on_error` (standardverdi true: kjøringen stopper ved den første handlingen som mislykkes). Handlingene er [Recipe DSL](cli-reference.md#recipe-dsl) fra CLI-referansen; en relativ `output` blir løst mot skriptets arbeidsmappe.
- **Returnerer** `ok` (alle handlinger lyktes), `actions` (hvor mange), `log` (utdata fra kjøringsprogrammet; `echo=True` skriver også ut dette under kjøring) og `validation`. En oppskriftsfil som mangler eller ikke kan analyseres, utløser `ChlorosConnectError`.
- **Kontrolleres før den kjører.**`validation` er `{"ok", "errors", "warnings", "steps"}`: hver handling må ha nøyaktig ett kjent verb som nøkkel (`{"capture": {"name": ...}}`, ikke `{"action": "capture"}` eller `{"type": "capture"}`) som navngir et kamera, en matrise eller en sensor prosjektet har, av riktig type. En oppskrift med en feil utfører**ingen** handling (`ok` false, feilene i `log`), uansett hva `stop_on_error` sier. Ubrukte nøkler, og alle matriser eller lyssensorer som oppskriften styrer, men som ikke er tilkoblet nå, er `warnings`. En oppskrift kobler ikke til enheter: koble til prosjektets første (`ChlorosProject.connect_all()`, `chloros-cli project connect`, eller MCP-verktøyet `connect_project_devices`). `dry_run=True` kjører ingenting og returnerer `{"recipe": ..., "validation": ...}`.
- Det kjører kun én oppskrift om gangen i en prosess. En YAML-oppskrift krever PyYAML.

---

## Prosjektautomatisering — `ChlorosProject`

Et lagret Chloros-prosjekt er en mappe som inneholder `cameras.json` + `sensors.json` + `project.json`. `open_project(path, relink=None)` laster inn manifestet (`relink`: mappen der de manglende kildefilene til et flyttet prosjekt nå befinner seg, brukes når backenden laster det inn — `process()` — se **Et flyttet prosjekt** under `ChlorosLocal`), og `connect_all` setter alle lagrede kameraer, matriser og lyssensorer i drift med de lagrede innstillingene — samme maskinvaretilstand som GUI-en ville ha produsert. Lyssensorer kobles til via backendens [DAQ-sensorpool](#daq-sensor-sessions), så de trenger en kjørende backend (se [`SensorHandle`](#sensorhandle)).

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

# Light sensors are addressable by name OR slot_id
reading = proj.sensors["Sky"].read()   # (spectrum, is_saturated, ...) or None

# One frame from every camera and array, one spectrum from every sensor
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
| `connect_all(*, cameras=True, arrays=True, sensors=True, verbose=False, align=None, packed=None)` | Oppdager og kobler til alle lagrede kameraer, matriser og lyssensorer. Returnerer en tilkoblingsrapport per klasse (`{"cameras": {...}, "arrays": {...}, "sensors": {...}}`, hver med `connected`, `missing` og `errors`; `arrays` inneholder også `alignment`). Med en backend som lytter på `127.0.0.1:5000`, blir de lagrede kameraene og matrisene funnet og koblet til via denne. Hvis det ikke finnes noen, finner og styrer denne prosessen dem selv via `lattice_sdk`, som trenger Arena SDK-kjøretiden og OpenCV (`cv2`) — SDK-hjulet installerer ingen av delene; når det ikke kan gjøre det, vises hvert lagret kamera og hver lagret matrise under `errors` med årsaken oppgitt, og `missing` forblir tomt. Et backend som kjører, men avviser SDK-et (logget ut: HTTP 401; en plan under Copper: 403), regnes ikke som «ingen backend»: hvert lagret kamera, array og lyssensor vises under `errors` med backendens årsak, og ingenting åpnes i denne prosessen. Hvert anrop sjekker backenden på nytt, så `connect_all()` etter innlogging, eller etter at Chloros er startet, kobler seg til det samme prosjektet. Et lagret array som allerede er oppe i backenden med de samme kameraene (GUI-enes eller `chloros-cli project connect`s) kobles til som det er, uten at de lagrede innstillingene brukes på nytt, og `disconnect_all()` lar den kjøre videre. Lyssensorer går alltid gjennom backenden (se [`SensorHandle`](#sensorhandle)). Den oppretter aldri en backend. |
| `disconnect_all()` | Avslutt alt. Hver enhet frigjøres først; deretter, hvis backenden ikke bekreftet frigjøringen av et kamera, en matrise eller en lyssensor (avslutningen gikk ut på tid eller ble avvist), opprettes det én `ChlorosConnectError` for hver. En enhet som allerede er borte, eller hvis det ikke kjører noen backend, er stille. Et array eller en lyssensor som en annen klient hadde åpent, forblir i drift. Prosjektets `with`-avslutning utløses aldri. |
| `capture_all(output_dir=".")` | Én ramme fra hvert tilkoblet kamera + array, og ett spektrum (`SensorHandle.read()`) fra hver tilkoblet lyssensor, tatt etter tur (kameraer, deretter matriser, deretter sensorer), ikke samtidig. |
| `stream(camera, overlays=False, fps=10.0)` | Generator som genererer BGR `numpy`-rammer fra et navngitt kamera (eller array). `overlays=False` leser rammer med håndtaket`grab()` på `debayered`-nivå: gjennom backenden når prosjektet er koblet til via en slik, ellers en direkte `lattice_sdk`-hentesløyfe (matriser gir `{serial: frame}`-ordbøker). `overlays=True` ruter gjennom `ChlorosLocal.camera_stream()` → backendens `/api/camera/<serial>/stream-annotated` MJPEG-feed, med kameraets lagrede `ui.overlay`-blokk sendt videre som spørringsparametere. Krever backend-modus, OpenCV (`cv2`) og et **frittstående kamera**: et kamera i direkte modus utløser `RuntimeError` (backenden kan ikke hente et kamera som denne prosessen eier), og en matrise utløser `NotImplementedError` (overlegger sammensatt per kamera — strømmer et element etter navn). Tilsvarende engangsoperasjon: `CameraHandle.capture(annotated=True)`. |
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

**Behandlingsnivåer.** `capture()`, `grab()` og `frame_stream()` bruker alle det samme `processing`
token, og kjeden er kumulativ — hvert nivå kjører alt som ligger over det:

| Nivå | Utgang | Merknader |
| --- | --- | --- |
| `raw` | 1-kanals Bayer, sensor-innfødt | Ingen demosaik. Overlegg er ikke tilgjengelige på dette nivået. |
| `debayered` | 3-kanals farge (**standard** for `CameraHandle` og alle `grab()` / `frame_stream()`; `ArrayHandle.capture()`, `capture_smart()` og `stream()` har `raw` som standard): BGR i minnet (`grab()` / `frame_stream()`), lagret som RGB av `capture()` | I backend-modus, som en opptak: kalibreringspakkens mørkeforskyvning (DSNU) fra rådataene, deretter OpenCVs kantbevisste demosaik — ingen linearitets-LUT, flatfelt eller unmix. Det eneste nivået som fungerer uten backend-modus, der det er kamerabibliotekets vanlige bilineære demosaik (uten mørkforskyvning). Den lagrede TIFF-filen er fotometrisk RGB, så reverser kanalene i et bilde i minnet (`frame[..., ::-1]`) for å sammenligne den med en fil. |
| `radiance` | float32, W/m²/sr/nm, **ett plan per bånd** | Full radiometrisk kjede: demosaik + 3×3 unmix (multispektral) + DSNU + flat-field + NIST-skala, med eksponering × forsterkning dividert ut slik at verdiene er absolutte. |
| `reflectance` | uint16, 32768 = 1,0 (eller uint8, 255 = 1,0 — se `bit_depth`), **ett plan per bånd** | Strålingsintensitet delt på nedadgående irradians (ρ = π·L/E). Krever en DLS/DAQ-avlesning — se merknaden nedenfor. |
| `display` | 8-bit sRGB-lignende | GUI-ekvivalent gjengivelse: CCM + hvitbalanse + gamma via kameraets aktive fargeprofil. |

Kun i direkte (backend-fri) modus er `debayered` tilgjengelig: `grab()` på et annet nivå hever
`NotImplementedError`, `capture()` ignorerer `processing`, og `frame_stream()` øker
`RuntimeError` på hvert nivå. `reflectance` trenger en brukbar nedadrettet avlesning — rammeendepunktet trekker
den samlede DAQ-en inn i kameraets DLS-spor automatisk, men uten en tilknyttet DAQ avviser kjeden
refleksjonsutgangen og merker nedgraderingen tydelig i de returnerte metadataene i stedet for å
å levere tilbake et dårligere produkt.

> **Refleksjons-DN-skala — ikke hardkod den.** LATTICE-reflektans bruker `32768` = ρ 1,0 og merker
> XMP `Chloros:PixelScale=32768`; Survey3-reflektans bruker `65535` = ρ 1,0 og har ingen
> `Chloros:*`-tagger. Les taggen og del med den. Den er definert i uint16-domenet, så den forblir
> `32768` for alle formater som skaleres om (16-bits TIFF, 8-bits TIFF, 8-bits PNG/JPG, 32-bits prosent)
> — normaliser først den lagrede datatypen tilbake til uint16 (×257 fra 8-bit, ×65535 fra float). Det eneste
> unntaket er en fil fra en versjon før 1.3.0: en 8-bit TIFF-fil ble da *beskåret*, ikke omskalert, så
> ingen skala beskriver den, og den inneholder ingen `PixelScale`- eller MicaSense-tuple. Behandle en manglende tag på en
> LATTICE-reflektansfil som «ingen gyldig skala», ikke som en standard.

**Kalibreringstagger: kun på `raw`.** `raw`-filen til en opptak inneholder kameraets kalibreringspakke: `CalibrationJSON`-nyttelasten og de MicaSense-kompatible taggene (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), som beskriver sensorens råverdier. Filene på hvert annet nivå (`debayered`, `display`, `radiance`, `reflectance`, samt `index/`-renderingen) og hvert bilde fra kontinuerlig opptak/strøm fra flere kameraer inneholder kameraets identifikasjon (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) og bildets egen opptaksprofil, `XMP-mapir:FrameJSON` (eksponering, forsterkning, temperatur, pikselformat, opptakstid, DAQ-samsvar), ettersom hvert `process()`-produkt bærer denne identiteten. Behandlingen finner kalibreringen til en slik fil ved hjelp av serienummeret og `CalBundleSha256` i Chloros’ kalibreringsbuffer, og laster ned pakken én gang dersom denne datamaskinen ikke har den.

#### Filer laget fra flere kameraer

En fil laget fra båndene til mer enn ett kamera – en multibåndsstabel (`capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index(save_multiband=True)`, `Registration.save_multiband()`), en live-indeks laget av flere kameraer (`index.tif`, `index_LUT.png`), et «array capture»-bilde med navnet `composite/` — oppgir navnene på hvert kamera som inngikk i det:

- **Per-kamera-lister, i båndrekkefølge** (sammensetningen: bakgrunnslag, deretter forgrunn; et indekslag, kameraene som båndene stammer fra): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (kalibreringspakken som hvert kameras bånd har gått gjennom, ellers — en `capture_aligned_multiband()` / `start_aligned_stream()`-stabel, hvis bånd ikke går gjennom noen — kameraets pakke som Chloros har lagret i hurtigbufferen, den som opptakfilene er oppkalt etter), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — parallelle lister, hvor element *n* i hver beskriver det samme kameraet; `-` der et kamera ikke har noen verdi (ingen kalibreringspakke, et ukjent objektiv). Hvert kamera er **ett** element: der pakken har flere verdier for et felt — en pakkes halogenlampesertifikater — de kobles sammen med `;` (`F-1479-1;F-1480-1;F-1481-1`), aldri med komma, slik at en liste viser én post per kamera uansett hvordan den skrives ut.
- **EXIF `SerialNumber`** inneholder de samme serienumrene, skilt med komma (`214000535,214000551`).
- Den inneholder **ingen** identitet for enkeltkameraer (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): disse angir ett kamera og dets pakke, og behandlingen vil beregne filens kalibrering ut fra dem. En indeks beregnet fra flere kameras bånd (`index.tif`, `index_LUT.png`) har EXIF-koden `Model` `LATTICE` (rigg), ikke én enkelt kameras `LATT-…`-modell.

En fil med båndene fra **ett** kamera – en TIFF-fil per bånd, et `capture_aligned_multiband(per_camera=True)`-bilde, en stabel eller en live-indeks fra et enkelt kamera – inneholder det aktuelle kameraets egne identitetsmerker, akkurat som alle andre opptaksfiler gjør. `capture_aligned_multiband()`s stabel, båndvise og kameravise filer, samt hver `start_aligned_stream()`-stabel og dens båndvise filer, inneholder gruppens opptakstid (referansekameraets eksponering, UTC), og et monokameras `per_camera`-ramme er ett plan, på `bit_depth` akkurat som stakken.**Kameraets orientering (speilvendt / rotert).** Rammer fra `grab()` / `frame_stream()` er alltid i
kameraets opprinnelige orientering, på alle nivåer. Lagrede opptak følger kameraets orienteringsinnstilling
(GUI-innstillingene «Mirror H» / «Mirror V» / rotasjon):

- **`raw`- og `debayered`-filer er nøyaktig det sensoren har avlest**, uansett innstilling – en Bayer-
  mosaikk som er rotert eller speilvendt, vil være et annet fargefiltermønster, og kalibreringskartene er
  på sensorens eget rutenett.
- **`display`, `radiance`, `reflectance` og indeksfiler er orientert** – rotert én gang, etter all
  kalibreringsberegningene. Klemmaskerne deres ligger på samme rutenett.
- **Hver opptaksfil registrerer innstillingen** i XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, med klokken, etter
  speilvendingen) og `ChlorosOrientationApplied` — `false` på rådata og debayerte bilder, `true` på de orienterte
  produktene. `ChlorosProcessing` viser operasjonene som er brukt på filens piksler (`none` når
  den er i opprinnelig format). Dette inkluderer hvert enkelt bilde i en kontinuerlig opptaksserie (kameraets egen innstilling,
  ikke anvendt) og i en strøm fra flere kameraer (identiteten: bildene i denne er i opprinnelig format, som ved et
  array-opptak).
- **Behandling av et prosjekt anvender den registrerte orienteringen** på produktene det lager fra et rå
  (først kalibrering i sensor-koordinater, deretter matrisejustering, til slutt orientering),
  slik at de samsvarer med produktene fra liveopptaket — inkludert de ukalibrerte sensorrespons- og vignettkorrigerte
  reserveproduktene; det debayerte produktet forblir i opprinnelig format.
- **Chloros-bildeviseren viser en rå eller debayert fil som er rotert i henhold til opptaket** – kun for visning,
  filen endres ikke; en fil uten opptak vises slik den er lagret.
- **Opptak uten opptak behandles som native.** Før versjon 1.3.0 roterte et opptak med ett kamera
  selv sine RAW- og debayered-filer; Chloros kan ikke skille en slik RAW-fil fra en originalfil og
  gjør ingen antakelser.
- **Array-opptak beholder alle medlemsfilene i originalformat som standard** og registrerer identiteten;
  `ArraySession.capture(orient=True)` behandler produktene sine (se kontrollene for array-opptak).

#### Utdata per bånd og blandede matriser

På nivåene `radiance` og `reflectance` returnerer et kamera **ett plan per bånd for det
kameraet** — tre for et M3C (dets tre ublandede bånd, BGR-kanalrekkefølge), ett for et M3M (dets
enkelt smalbåndsfilter). Det monokameraets replikerte planer er sammenfoldet, så et **blandet
M3C/M3M-matrise gir forskjellige matriseformer per serie**. Les `bands`-nøkkelen fra rammens
metadata for å finne ut hva hvert plan er; ikke trekk slutninger ut fra antall kanaler:

```python
for serial, (arr, hdr) in arr_handle.grab(processing="reflectance",
                                          with_metadata=True).items():
    print(serial, arr.shape, hdr["bands"])   # ('…', (1536, 2048, 3), ['nir','green','red'])
                                             # ('…', (1536, 2048, 1), ['F685'])
    rho = arr.astype("float32") / hdr["pixel_scale"]
```

Sjekk alltid `hdr["processing_level"] == "reflectance"` før du bruker verdiene. En nedgradering
(ingen DAQ-avlesning, ingen kalibreringspakke, ukjent båndoppsett) rapporteres i `processing_level` og
`radiometric_refused` i stedet for oppgradering — matrisen du får tilbake er fortsatt en gyldig ramme, bare
ikke det produktet du ba om. En lagret opptak som ble nedgradert, har den samme koden i XMP-filen
som `Chloros:RadiometricRefused` (f.eks. `no-band-layout`, `no-dls-snapshot`).

#### Én utløser per avlesning — `group_id`

**Array-medlemmer hentes fra arrayets synkroniserte bildegruppe, ikke fra hvert enkelt kameras egen
opptaksbuffer.** Kameraene i et maskinvaresynkronisert array eksponerer samtidig, men N avlesninger per kamera
*sampler* ikke samtidig — avlesning av hvert kameras siste bufrede bilde strekker seg over utløsergrensene
grenser, og et enkelt utelatt bilde desynkroniserer settet uten at noen legger merke til det.

Levering fra gruppen er nødvendig, men ikke tilstrekkelig i seg selv: hver forespørsel tar fortsatt
et øyeblikksbilde av *den nyeste* gruppen på sitt eget tidspunkt, og henteløkken kan publisere i mellomtiden.
Målt på et live-array med 4 kameraer, **strakte 12 av 20 parallelle avlesninger seg over to grupper** — alltid
nøyaktig ved siden av hverandre. Så avlesningen er **festet**:

1. Hvert bilde bærer `group_id`, oppstillingenes monotone publiserings teller.
2. `grab()` leser fra masteren (eller, hvis masteren ikke kan betjenes, det neste kameraet som kan),
   og fester deretter de gjenværende kameraene til gruppe-ID-en den landet på (`?group_id=N` på HTTP-
   endepunktet).
3. Backenden beholder nylige grupper i **minst et halvt sekund**, slik at en søsken som allerede
   er forbigått av senere publiseringer, fortsatt betjenes av den utløseren — lenge nok til at en
   refleksjonseksport per kamera, selv på et fullsensor-array med 9 kameraer. En tilknytning som backend ikke
   lenger opprettholder, avvises med `409` og `group_aged_out: true` i stedet for å bli besvart uten melding
   med en annen utløser – `grab()` leser settet på nytt.

Om målt med pinning: **20 av 20 avlesninger returnerte én utløser.** `grab()` gjør dette for deg;
du trenger bare `group_id=` når du driver HTTP-endepunktet direkte.

- Et kamera som ikke kan betjenes i den festede gruppen — enten fordi det mangler der, eller fordi det avviser forespørselen
  (et `band` det ikke har, et nivå det ikke kan produsere) — blir **utelatt i stedet for å fylles inn i etterkant**
  fra en annen utløser, og de andre kameraene returneres fortsatt.
- Diktet som `grab()` returnerer, er en `GrabResult` (en ren `dict`-underklasse;
  `from chloros_sdk import GrabResult`): `.errors` tilordner hver
  utelatt serienummer til unntaket som lesingen utløste, og `.group_id` er den delte gruppen. En tom
  `.errors` betyr at alle kameraer inngår i denne gruppen.
- Et frittstående kamera har ingen gruppe, rapporterer `group_id: None` og **avviser** en `group_id=`
  pin i stedet for å ignorere den.

```python
frames = arr_handle.grab(processing="reflectance")
for serial, err in frames.errors.items():
    print(f"SN:{serial} omitted: {err}")       # e.g. status=400 ... band 'F685' not in [...]

frames = arr_handle.grab(processing="reflectance", with_metadata=True)
ids = {hdr["group_id"] for _a, hdr in frames.values()}
assert len(ids) == 1, f"straddled triggers: {ids}"     # belt and braces
```

Den samme telleren publiseres på **hver MJPEG-strøm per kamera** for et medlem av en matrise —
forhåndsvisningen (`/api/camera/stream/<serial>`), den kommenterte forhåndsvisningen (`/stream-annotated`) og
refleksjonsstrømmen (`/stream-reflectance`) — som `X-Chloros-Group-Id` på hver del, i tillegg til
`X-Chloros-Frame-Id` og `X-Chloros-Timestamp-Ns`. Strømmene er kontinuerlige snarere enn
punktvise, så de samsvarer ved å samle deler og gruppere etter den overskriften – det er ingen
pin å angi. Et frittstående kamera har ingen gruppe. Se
[MJPEG-strømmer per kamera](#per-camera-mjpeg-streams-no-rtsp).

**`frame_stream()` blir aldri stille.** En avkryssing som mislykkes av en årsak som kan løse seg av seg selv,
hoppes over – en kamerastrøm gir ingenting for den, en array-strøm gir kameraene som faktisk
svarte (muligens ingen). Årsakene er: ingen bilde ramme ennå; kameraets hentetråd er satt på pause i et
sekund eller to (`404 not streaming`, som backenden svarer på når et annet kamera kobler seg til
eller fra, eller når en innstillingsendring balanserer puljen på nytt); en omstart av backend eller `5xx`; justering
er fortsatt i ferd med å stabilisere seg (`409 … retry shortly`); og en avvisning av reflektans fordi DAQ-en ikke har noen måling
ennå (`no-dls-snapshot`, en foreldet eller tom måling). En avvisning ved neste tikk ville bare gjenta seg — en
ukjent `band`, `bit_depth=8` på strålingsintensitet, en kalibreringspakke uten strålingsintensitetsskala, en avvisning av reflektans
med **ingen DAQ i det hele tatt** (ingen knyttet til kameraet eller dets matrise, ingen koblet til
backenden: avvisningens `dls_source` er `"none"`), enhver annen `4xx` —
utløser `chloros_sdk.FrameFetchError` (en `RuntimeError`, som bærer `.status` og `.body`) på
en gang. Det samme gjør **60 sekunder** med hoppede takter uten ramme imellom, uansett hva `fps` er — lang
nok til å vente ut justeringen og en omstart av backenden, kort nok til at et kamera som er
virkelig er borte, avslutter strømmen. Denne oppgivelsesfeilen er også en `FrameFetchError`, både for et kamera og for
et array, med unntaket fra den siste avlesningen på `.errors` (og, for et kamera, som `__cause__`). For
et array betyr «refused» at hvert kamera har nektet; unntaket `.errors` inneholder hvert kameras
årsak.

**En matrise `frame_stream()` gir hvert utløsersignal én gang** (både `ArrayHandle` og `ArraySession`). En
tick hvis nyeste sett er utløseren den allerede har levert, venter på neste utløser i stedet for å
levere det samme settet igjen, slik at strømmen kjører med den laveste av `fps` og matrisens egen rammefrekvens
, og påfølgende sett deler aldri en `group_id`. En matrise som slutter å publisere, utløser
`FrameFetchError` («ingen ny utløser på 60 sekunder») i stedet for å gjenta sitt siste sett. Uten dette ville en
strøm som ba om mer enn det arrayet publiserte – eller startet rett etter tilkobling – ha gitt
gjentakelser: i én målt kjøring inneholdt 20 sett 11 forskjellige gruppe-ID-er.

```python
from chloros_sdk import FrameFetchError

try:
    for arr in cam.frame_stream(processing="radiance", bit_depth=8):
        my_analysis(arr)
except FrameFetchError as exc:
    print(exc.status, exc.body)        # the HTTP status and the backend's reason
```

#### Eksportere et enkelt bånd — `band`

`grab()` / `frame_stream()` bruk `band="<name>"` for å eksportere bare ett av kameraets bånd som en
enkeltkanals `(H, W, 1)`-matrise — samme form som et monokamera produserer, så utdata fra ett bånd
håndteres på samme måte uansett om det kommer fra en M3M eller fra oppdeling av en M3C. Et FRGNs røde bånd:

```python
red = cam.grab(processing="reflectance", band="red")        # (1536, 2048, 1)
```

Godkjente skrivemåter er det fullstendige layoutnavnet (`Red_660`, `NIR_850`, `Green_550`) eller dets korte
hode (`red`, `nir`, `green`), uten skille mellom store og små bokstaver. Et ukjent navn avvises med en liste over tilgjengelige
navn.

> **Valget skjer etter navn, ikke etter kanalnummer — dette er bevisst.** «Kanal 1» er `Green_550` i
> matrisens BGR-kanalrekkefølge, men *rød* i filterkodensbokstavrekkefølge (`RGN`), og de to
> stemmer ikke overens. Velger man feil, får man en fullstendig plausibel ramme fra feil bånd, som
> ingenting lenger nedstrøms kan oppdage. Det finnes ingen numerisk indeks av den grunn.

#### Justering av strømmene — `align`

`align=True` forvrenger rammen til matrisens felles referanseramme. Hvert kamera forvrenges
gjennom **samme** profiloppløsning og beskjærings-ROI, slik at strømmer per kamera som åpnes uavhengig av hverandre,
blir **gjensidig registrert** — du trenger ikke å be om dem samtidig:

```python
frames = arr_handle.grab(processing="reflectance", align=True)   # {serial: array}
# or one MJPEG stream per camera, each co-registered on its own:
#   GET /api/camera/<serial>/stream-reflectance?align=1
```

Krav og oppførsel:

- Kameraet må være i et **hardwaresynkronisert** array (M8-synkroniseringskabel) som har en justeringsprofil
  — samregistrering forutsetter at medlemmene eksponeres samtidig, noe et programvare- eller
  frittløpende oppstilling ikke gir. Avvist med `409` ellers.
- **Avvist, blir aldri stille ujustert.** Opptakspassen kan lagre ujustert med en advarsel
  fordi en fil beholder sine opprinnelsestagger og kan registreres på nytt senere; en direktestrøm har ingen
  slik ny sjanse, så en manglende profil er en feil snarere enn en reserve.
- **Å be om justering starter prosessen.** Å be om `align` på et array uten profil setter i gang
  kalibreringen og returnerer `409` med det den venter på (f.eks. *«venter på at eksponeringen skal
  stabilisere seg på 214000551 (1/3)»*) — prøv på nytt, så lykkes det, vanligvis i løpet av noen titalls sekunder.
  Dette er viktig ved headless-bruk: den automatiske justeringen ved tilkobling gir opp etter 30 sekunder, og hvert forsøk
  innenfor dette tidsvinduet hoppes over mens smart-AE fortsatt stabiliserer seg, så et array der eksponeringen
  konvergerer sakte pleide å ende opp uten profil for hele økten og uten mulighet til å skaffe
  en uten å åpne den kombinerte visningen på skrivebordet. `alignment_status` rapporterer også
  `ae_align_holdouts`, og nevner kameraene som fortsatt stabiliserer seg og deres fremgang.
- Gi hver strøm i en matrise **samme `max_h`**, ellers havner utskjæringene deres på forskjellige piksel
  rutenett.
- Justeringen utføres *før* bitdybdekodingen, slik at forvrengningen interpolerer verdier med full presisjon
  .

#### `bit_depth` — bytter presisjon mot båndbredde

`grab()` og `frame_stream()` aksepterer `bit_depth=16` (standard) eller `bit_depth=8`:

| `bit_depth` | Refleksjonskoding | Kostnad |
| --- | --- | --- |
| `16` | uint16, 32768 = ρ 1,0, spillerom til ρ 2,0 | Full presisjon. |
| `8` | uint8 som spenner over `0..refl_max` (standard ρ 1,0) | Halvparten av byteene; ~0,004 ρ-kvantisering ved standardtaket. Verdier over taket blir avskåret — se nedenfor. |

##### 8-bits taket — `refl_max`

`refl_max` angir ρ-verdien som 8-bitsområdet spenner over, så `pixel_scale = 255 / refl_max` og ρ er fortsatt
`pixel / pixel_scale`. `grab()` og `frame_stream()` bruker det som et nøkkelord på både kamera- og
matrisehåndtak (`?refl_max=` på HTTP-endepunktet). Øk verdien for å skaffe mer spillerom for speilende
høylys, med proporsjonalt grovere trinn:

```python
frame = cam.grab(processing="reflectance", bit_depth=8, refl_max=1.5)   # 0..1.5 in 0..255
```

| `refl_max` | `pixel_scale` | trinn |
| --- | --- | --- |
| `1.0` (standard) | 255 | 0,0039 ρ |
| `1.5` | 170 | 0,0059 ρ |
| `2.0` | 127,5 | 0,0078 ρ |

> **Automatisk eksponering kan ikke rette opp en 8-bits klipping, og å prøve å bruke den gjør dataene dårligere.**
> ρ = π·L/E med eksponering og forsterkning dividert ut, så ρ er en egenskap ved motivet, ikke ved
> kamerainnstillingen — målt på et live-array førte en halvering av eksponeringen til en forskyvning av gjennomsnittlig ρ på 1,3 %. Å øke
> eksponeringen for å «tilpasse» seg området fører i stedet til at piksler går inn i **sensor**-metning, noe som
> undervurderer L og skaper en **lav** skjevhet i ρ uten at det oppstår feilmelding (målt: maks. ρ 1,203 → 0,985 ved
> 2× eksponering, gjennomsnitt −23 %). Velg området med `refl_max`; å holde sensoren utenfor
> metning er det AE allerede er til for, og det beskytter ρs *gyldighet* snarere enn omfanget.

**Les `reflectance_clipped_fraction`, ikke flagget.** `reflectance_clipped_at_1_0` utløses av en
enkelt piksel — i en virkelig scene rapporterte den `True` mens brøkdelen var `0.000001`, én piksel
av en million. Brøkdelen er det som forteller deg om du skal heve taket.

**`bit_depth=8` gjelder kun reflektans.** Å be om det for `radiance` blir avvist i stedet for
å bli betjent: refleksjonsgrad har et fast område på 0..1 som tåler en 8-bits koding, mens strålingsgrad ikke har noe
fast område, så for å få plass til den i 8 bits kreves det en prosentilstrekking per bilde, der kartleggingen
endres for hvert bilde — et bilde, ikke en måling.

#### Live-strøm av reflektans (MJPEG)

`GET /api/camera/<serial>/stream-reflectance` leverer 8-bits reflektans per bånd som
`multipart/x-mixed-replace` — en M3C som 3-kanals JPEG, en M3M som gråtoner. Spørringsparametere:
`fps` (standard 10), `jpeg_q` (standard 95), `max_h` (0 = opprinnelig), `band` (ett navngitt bånd, som
ovenfor), `align` (`1` for samregistrering — samme regler som SDKs `align`), `refl_max` (8-bits
kodingstak, som ovenfor). Hver del inneholder
`X-Chloros-Group-Id`, `X-Chloros-Frame-Id`, `X-Chloros-Timestamp-Ns`, `X-Chloros-Bands`,
`X-Chloros-Pixel-Scale`, `X-Chloros-Aligned` og `X-Chloros-Reflectance-Clipped`.
**Deler fra forskjellige kameraer som har samme `X-Chloros-Group-Id`, utgjør den samme utløseren** —
slik settes flere strømmer per kamera fra én matrise sammen igjen, og dette garanteres
på serversiden fordi matrisemedlemmer hentes fra den synkroniserte ramme-gruppen (se
[Én utløser per avlesning](#one-trigger-per-read--group_id)). Den returnerer `409` når reflektansen er
utilgjengelig ved tilkoblingstidspunktet og avslutter strømmen hvis den blir utilgjengelig under kjøring, så en stoppet
strøm betyr «ikke lenger reflektans» snarere enn «fortsatt i orden, bare statisk».

> **JPEG er tapsfylt — disse pikslene er en måling.** Komprimeringsfeil påvirker direkte
> reflektansverdiene og er verst ved kantene, der materialgrensene ligger. Kvaliteten
> er derfor satt til 95 som standard. Hvis du trenger tallene intakte, bruk
> `grab(processing="reflectance")` / `frame_stream(...)`, som gir samme resultat uten
> kodek i banen (og `bit_depth=8` hvis ledningskostnaden, ikke kodeken, er begrensningen).

> **EXIF overføres til eksporten.** `process()` kopierer GPS-blokken fra kildeopptaket
> **og dens ExifIFD** til hvert produkt, slik at eksportene inneholder `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` og `CameraSerialNumber` samt
> georefereringen. `FocalLength` er det Pix4D bruker til å beregne avstanden mellom bakkepunkter – uten dette
> ender rekonstruksjonen opp med en helt feil skala (i et målt tilfelle ble et område på 411 m
> omgjort til et på 47,8 km). Kopien er bevisst ikke `-all:all`: IFD0s strukturelle koder ødelegger
> LATTICE-utdataene, og `ExifImageWidth`/`Height` er utelatt fordi de beskriver kildeopptaket
> snarere enn det eksporterte rasteret.

Underflagg for opptakstrinnet (gjelder de radiometriske nivåene — `radiance`, `reflectance`, `display`):

| Flagg | Standard | Betydning |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + flat-field + 3x3 unmix + NIST-radiometrisk skala. |
| `apply_white_balance` | `True` | WB LUT. DLS-kompatibel når en DAQ er koblet til kameraet. |
| `apply_index` | `False` | Evaluering av vegetasjonsindeks. |
| `index_expression` | `None` | Overstyr formel. Ikke-tom → aktiverer indeksen automatisk. |
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

> **Returverdien er en `chloros_sdk.CapturePaths`, av typen `CapturePathMap`, ikke `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` er `Dict[str, Union[str, List[str]]]`: en enkeltnivå
> `processing` gir hver serie én bane, mens `processing="all"` gir den en
> **ordnet liste** over alle produkter som er lagret for det kameraet. `ArrayHandle.capture()` tar
> ingen `levels=`, og den ignorerer nøkkelordargumenter den ikke kjenner (`levels`, `daq`,
> `render_index`, `force_daq`), så send kun de som er vist her (`aligned=False`
> lagrer native filer, som på `ArraySession.capture()`); bruk
> `ArraySession.capture()` for disse kontrollene. En live kombinert kompositt, hvis en ble strømmet, kommer inn under den ekstra
> `"combined"`-nøkkelen i stedet for under en serienummer. Kode som forutsetter `str` bryter på
> listeformen uten at noen typekontroll protesterer — merknaden sa `Dict[str, str]`
> en stund etter at listeformen ble lansert, og det er grunnen til at aliaset eksisterer. Normaliser
> når du ønsker den flate formen:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

> **Det som ikke kunne skrives: `.failed`.** Kartet som `capture()` returnerer, er en
> `chloros_sdk.CapturePaths` — en ren `dict`-underklasse, så alt ovenfor fungerer
> uendret — som også inneholder `.failed`: filene som backenden ikke klarte å skrive,
> de samme oppføringene som [`CaptureResult.failed`](#filer-som-ikke-kunne-skrives--captureresultfailed)
> (`serial`, `level`, `error` og `disk_full: True` på en full disk). En delvis
> opptak returnerer fortsatt alle banene som ble skrevet; på en full harddisk er oversikten
> tom, og `.failed` forklarer hvorfor. `.skipped` viser overspringene per kamera og
> `.group_seq` den synkroniserte gruppen, og `.alignment` / `.oriented` er
> backend-ene, som på `CaptureResult`. `capture_smart()` returnerer dem som
> `result["frames"].failed` og `result["failed"]`.
>
> ```python
> paths = arr.capture("./out", processing="all")
> if any(f.get("disk_full") for f in paths.failed):
>     raise SystemExit("drive full -- free space and capture again")
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

# Manual control points: each camera's pixel coordinates of the same
# features, the reference camera's included (>= 2 per camera). Every one
# of num_frames is solved from these points.
arr.calibrate_alignment(method="manual", model="rigid", control_points={
    "214701173": [[412, 300], [1630, 310], [1020, 1180]],   # reference (master)
    "214701292": [[418, 297], [1637, 306], [1027, 1177]],
})

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

En lagret lyssensor (DAQ-U, DAQ-M eller DAQ-E), åpnet av `connect_all()` i backendens [DAQ-sensorpool](#daq-sensor-sessions) — den samme poolen som `connect_daq_sensor()` og `chloros-cli daq pool-connect` bruker.

```python
reading = proj.sensors["Sky"].read()
if reading is not None:
    spectrum, is_saturated, integration_time, x, y, z = reading
```

- `read()` venter på sensorens **neste** spektrum — et som allerede er bufret når du kaller, teller ikke — og returnerer `(spectrum, is_saturated, integration_time, x, y, z)`, eller `None` hvis ingen kommer innen 5 s. En pauset strøm gjenopptas. `spectrum` er i W/m²/nm når sensoren er kalibrert.
- `.sensor` er sensorens [`DAQSensorSession`](#daq-sensor-sessions), for alt annet: `latest(n=...)`, `status()`, `record_start()` / `record_stop()`.
- `.slot_id`, `.model`, `.serial`, `.name` og `.settings` er den lagrede oppføringen fra `sensors.json`.

**Hva `connect_all` gjenoppretter.** Hver sensor åpnes på sin lagrede adresse — dens COM-port, Bluetooth-adresse eller DAQ-E-vert — og når prosjektet har registrert sensorens ID, må det være den sensoren: en annen enhet på den adressen avvises, den er ikke tilkoblet. Den lagrede integrasjonstiden, rammeutjevningen og automatisk eksponering brukes, samt kap-korreksjonen:

| Lagret cap | Koblet til med |
| --- | --- |
| En cap-profil (f.eks. `fov_30`) | Den profilen. |
| Ingen valgt | `sunshine_cosine`, standardinnstillingen for tilkobling, registrert som antatt (`cap_id_source=auto_default`), så behandlingen gir en advarsel. Sensorene leveres uten deksel: velg det dekselet som er montert. |
| `none` — valgt som **Ingen (sensor uten hette)** i brukergrensesnittet | Ingen hettekorreksjon. |

En DAQ-E-S bruker ingen dekselprofil, uansett hva som ble lagret: diffusoren er en del av kalibreringen (se [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor)). Et prosjekt som ble lagret av en tidligere Chloros, registrerte en bar sensor som det eneste alternativet, så den kobler seg til igjen med `sunshine_cosine`; velg **Ingen (bar sensor)** i GUI-en igjen og lagre prosjektet.

En sensor som en annen klient allerede har åpen i puljen (CLI-en, et annet skript) kobles til som den er, uten at innstillingene må angis på nytt, og `disconnect_all()` lar den kjøre; en sensor som `connect_all()` åpnet, frigjøres. Som med `connect_daq_sensor()` er Chloros-brukergrensesnittet **ikke** en del av puljen: Lukk en sensor i GUI-en før du kobler til prosjektet fra et skript.**Krav og rapporten.** Lyssensorer trenger en kjørende Chloros-backend som er logget inn på et betalt Chloros+-nivå (Copper eller høyere), og `connect_all()` starter aldri en slik. Når ingen kjører, vises alle lagrede sensorer under `report["sensors"]["errors"]`, med en merknad om dette. En sensor som backenden ikke kan åpne, vises også der, med backendens begrunnelse; `report["sensors"]["missing"]` forblir tom for lyssensorer.

---

## Direkte maskinvare (uten backend)

Når du ønsker null avhengighet av backend for LATTICE-kameraer (CI, headless-roboter, innebygd), importerer du `lattice_sdk` direkte — den reeksporteres av `chloros_sdk`. Vær oppmerksom på `CAMERA_AVAILABLE`: `lattice_sdk` finnes i SDK-hjulet, men krever at Arena SDK-runtime og OpenCV (`cv2`) er til stede, og hjulet installerer ingen av delene. Lyssensorer går gjennom backend — [`connect_daq_sensor()`](#daq-sensor-sessions) eller et prosjekts `proj.sensors` — med ett unntak, DAQ-M-båndmagnitud-hjelpefunksjonen [`DLS`](#daq-m-band-magnitudes--dlsband_magnitudes), som også trenger `bleak`.

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

Tre av de fire forhåndsinnstillingene er **free-run**: kameraet eksponerer kontinuerlig, og en
`capture()` returnerer neste bilde. `triggered` er unntaket — den aktiverer
kameraet for en maskinvareflanke på linje 2, så den tar ikke noe bilde før en slik flanke oppstår.

| Forhåndsinnstilling | Utløser | Bruk den når |
| --- | --- | --- |
| `default` | fri drift | generell bruk |
| `high_speed` | fri drift | 8-bit, 60 fps-begrensning, kort eksponering |
| `high_quality` | fri drift | 12-bit, ingen fps-begrensning — det vanlige valget for stillbilder |
| `triggered` | **klar, linje 2** | kameraet er koblet til en M8-synkroniseringskabel, og noe annet utløser det |

Hvis du velger `triggered` (eller selv angir `trigger_mode="On"`) uten at noe
driver linje 2, vil alle `capture()` gå ut på tid — helt riktig, siden du ba
kameraet om å vente. SDK-et forklarer dette når det skjer; se
[SC_ERR_TIMEOUT under opptak](#direct-hardware-backend-free).

> **Merk — «GVSP-probe» / `SC_ERR_TIMEOUT -1011`-meldinger ved tilkobling er ikke feil.**
> Ved tilkobling prøver SDK-et å forhandle om**jumbo-rammer** (9000-byte GVSP-pakker) for høyere gjennomstrømning. På en direkte punkt-til-punkt-NIC-forbindelse (f.eks. en link-local `169.254.x.x`-adresse) kan nettverket vanligvis ikke håndtere jumbo-rammer, så denne sonderingen går ut på tid og logger linjer som:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Dette er den **innbygde fallback-løsningen**: SDK-en går automatisk tilbake til standardpakker på 1500 byte, og kameraet fortsetter å koble seg til som vanlig (`[chunk-enable …]`-linjene som følger, er en del av den normale tilkoblingssekvensen). Opptak fungerer fortsatt.
>
> Du kan hoppe over denne testen, men **den er ikke bare en loggdemper – den deaktiverer jumbo-rammer.** Kameraet svarer kun på «Don&#x27;t-Fragment»-ping opp til 1500 byte, uansett hvor godt nettverket ditt er, så ping-testen alene kan aldri oppdage jumbo-rammer; denne sonden er det eneste som kan det. Deaktiverer du den, vil kameraet for alltid bruke standardpakker på 1500 byte, på hvilket som helst nettverk:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Det er bare verdt det på et nettverk du *vet* ikke kan håndtere jumbo-rammer, der det sparer omtrent ett sekund i tilkoblingstid per kamera. Siden dette er en reell avveining og ikke bare en kosmetisk endring, opplyser SDK-et nå om dette når du bruker det:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **La den være som den er, med mindre du har en grunn til å endre den.**Hvis den er aktivert, måles hvert kamera én gang per backend-økt, og igjen når tilkoblingen endres (annen adapter, annen hastighet eller en tilkobling som har falt ut, slik som ved bytte av switch); ved gjenoppkobling gjenbrukes resultatet.**Etter en endring bak en switch som har vært oppe, må du starte backenden på nytt for å måle igjen.** En første tilkobling kan av og til ta noen minutter ved denne sjekken, og loggen viser dette når det skjer.
>
> Hvis du *ønsker* jumbo-gjennomstrømning, aktiver jumbo fra ende til ende (NIC MTU 9000 + en switch som slipper dem gjennom), eller lås den med `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` når du vet at koblingen støtter det — men foretrekker en kommandobasert `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` fremfor å sette den permanent, siden en fastsatt størrelse hopper over sonden og slutter å tilpasse seg nettverket foran den. **Hver** enhet i banen må kunne sende jumbo-pakker — inkludert eventuelle PoE-splittere eller -injektorer, som er den vanligste årsaken til at et ellers jumbo-kompatibelt oppsett ikke klarer å overføre dem.

> **`SC_ERR_TIMEOUT -1011` under `capture()` / `grab*()` er et annet problem — det er en reell feil.**
> Merknaden ovenfor gjelder kun `-1011` logget av**connect-time probe**. Den samme feilen som oppstår ved en**capture**, betyr at kameraet ble koblet til uten problemer, men ikke sender noen bilder:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Det som avslører dette, er et kamera der *kontroll*kanalen fungerer som den skal — oppdagelsen fungerer, innstillingene og `[chunk-enable …]`-skrivingene lykkes alle — mens *hver* ramme går ut på tid.
>
> **Den vanligste årsaken er at kameraet er klargjort for en maskinvareutløser.** Med `trigger_mode="On"` og `trigger_source="Line2"` sender kameraet ikke noe som helst før det kommer en elektrisk flanke på M8-synkroniseringskabelen. Hvis du ikke har noen kabel som driver den linjen, venter hver eneste bildeopptak i all evighet. Kameraet er ikke ødelagt, og nettverket fungerer som det skal — det gjør akkurat det det ble bedt om.
>
> `CameraSettings()` og `default` / `high_speed` / `high_quality` forhåndsinnstiller frikjøring, og en opptakssøknad som går ut mens den er aktivert, forklarer seg selv i stedet for å skrive ut bare `-1011`. `PRESETS["triggered"]` aktiverer Line2, som planlagt.
>
> For å tvinge et hvilket som helst kamera til friløp:
>
> ```python
> import copy
> settings = copy.deepcopy(PRESETS["high_quality"])   # PRESETS entries are shared
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Hvis det fortsatt går ut med `trigger_mode="Off"`, leverer kameraet virkelig ikke data – send oss loggen og `ip link show`.

#### Fargeprofiler (RGB-forhåndsvisning) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` velger skjermens fargeprofil for **forhåndsvisningen** på RGB-kameraer (multispek-kameraer ignorerer denne innstillingen):

| Profil | Betydning |
| --- | --- |
| `raw` | Omgå den radiometriske kjeden fullstendig. |
| `linear` | DSNU + flat + WB, ingen CCM, ingen gamma. |
| `natural` | Lineær + målt CCM + sRGB-gamma, kun med den enkle etterbehandlingen (kromatisk utjevning + desaturering av høylys) — den realistiske standardinnstillingen. |
| `enhanced` | `natural` pluss full «hub-parity»-etterbehandling (defringe, vibrance, CLAHE lokal kontrast). Rikere utseende til omtrent **dobbel kostnad per bilde**, og dermed lavere LIVE-bildefrekvens. |
| `custom_temp` | `natural`, men hvitbalanse låst til `custom_cct_k` Kelvin (DLS ignoreres; begrenset til 2000–10 000 K på backend-siden). |

Profilen er en **kun for live-forhåndsvisning**-knapp for hastighet/utseende: lagrede opptak får alltid den fulle, rike finishen uavhengig av valgt profil, så å velge `natural` for å spare inn rammetid reduserer ikke kvaliteten på det som lagres på disken. En ukjent profil utløser `ValueError`; når en chloros-backend er tilgjengelig, blir endringen også sendt til den via POST, slik at neste forhåndsvisningsramme gjenspeiler den (brukere av direkte SDK-tilkobling uten backend får fortsatt endringen i innstillingene).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Høydepunkter (RGB-skjerm) — `set_highlights`

`LatticeCamera.set_highlights(value)` trekker ut de lyseste, hviteste pikslene fra **skjermen** på RGB-kameraer — løsningen på at en lys hvit overflate gjengis som en flat hvit klatt. `value` er `-100`..`0` (begrenset); `0` er slått av, og gjengivelsen blir da nøyaktig slik den er uten innstillingen. Sterkere innstillinger senker skjermens hvitnivå (til ca. 60 % ved `-100`) og utvider tonekurven, slik at en overflate som forhåndsvisningens automatiske lysstyrke har gjort helt hvit, får detaljene tilbake.

Det er en skjermkontroll, men i motsetning til metning og kontrast (som bare endrer live-forhåndsvisningen) endrer den også det du lagrer: live-forhåndsvisningen og **skjermnivå**-opptakene følger den, så disse filene blir også mørkere (de har `Chloros:DisplayHighlights`); eksport av stråling og refleksjon gjør det aldri, og den kan ikke gjenopprette sensordata som er klippet – senk eksponeringen for det. Multispek- og monokameraer ignorerer den. Verdien lagres på `settings.highlights` og sendes via POST til en tilgjengelig chloros-backend, akkurat som `set_color_profile`; CLI-ekvivalenten er `lattice highlights --set N`.

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_highlights(-60)                      # 0 = off, -100 = strongest
```

#### Monokameraer (M3M) og `Calibration`

Et monokamera **M3M** (`M3M-<lens>-F<wavelength>`) er enkeltbånds: ett gråtonenivå, ingen Bayer-mosaikk, ingen 3×3 spektral-crosstalk-matrise. `Calibration` gjenkjenner det og viser et `is_mono`-flagg. Reflektans gjelder fortsatt som et radiometrisk kart per bånd (unmix er identitetsmatrisen), men flerbåndsberegninger på et enkelt kamera gir meningsfulle resultater i stedet for meningsløse:

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

For å beregne et vegetasjonsindeks fra monokromt utstyr, kombiner flere M3M-kameraer med forskjellige bølgelengder til en justert flerbåndsstabel (se [Arrayjustering](#array-alignment)) og beregne indeksen over hele stabelen i stedet for på ett enkelt kamera.

#### DAQ-M-båndstyrker — `DLS.band_magnitudes`

`DLS(mac_address, filter_model="RGN-IMX265")` kobler seg til en DAQ-M via Bluetooth ved hjelp av sin egen BLE-MAC. Dette er den eneste lyssensorklassen som kommuniserer direkte med sensoren i stedet for via backend, så den trenger det samme som de direkte kameraklassene trenger (`CAMERA_AVAILABLE`: Arena SDK-kjøretiden og OpenCV) pluss `bleak` for Bluetooth (`pip install "chloros-sdk[camera]"` eller `pip install bleak`; uten den utløser `DLS` feilen `DLSError`). En DAQ-M som er koblet til et annet sted (backendens pool, et prosjekt, GUI-en) slutter å annonsere, så frigjør den der først. **`mac_address` er påkrevd**: det finnes ingen standard, siden hver enhet har sin egen, og `DLS()` uten en utløser `DLSError` før noen tilkobling forsøkes. Finn MAC-adressen med `discover_daq_sensors(transports=["ble"])` (se [Oppdagelse](#discovery--finding-an-address-to-connect-with)) eller `chloros-cli daq pool-discover --only ble`.

`DLS.band_magnitudes(spectrum=None, *, spectral_response=None)` integrerer et DAQ-M-spektrum (den siste avlesningen når `spectrum` er `None`; `DLSError` hvis det ikke finnes noen ennå) gjennom kameraets tre kanalresponser og returnerer `{"red": …, "green": …, "nir": …}` — nøklene navngir kanalene i den rekkefølgen; med kurvene fra et RGB-kamera er den tredje (`nir`) dets blå kanal.

- Uten `spectral_response` bruker den filtermodellens standardkurve (den `filter_model` som `DLS` ble bygget med), som ble målt på én enhet.
- `spectral_response` bruker i stedet kameraets **egne** målte kurver: en dict `{"wavelengths": [...], "R": [...], "G": [...], "B": [...]}`, slik den er lagret i kameraets kalibreringspakke (`color_correction.spectral_response_smoothed` — RGB-kameraer; multispektrale pakker inneholder ingen slik kurve). Hvis en dict ikke kan brukes, faller den tilbake til flåtekurven.

```python
from chloros_sdk import DLS

# camera_curves: {"wavelengths": [...], "R": [...], "G": [...], "B": [...]}
with DLS(mac_address="AA:BB:CC:DD:EE:FF") as dls:
    reading = dls.acquire()
    fleet = dls.band_magnitudes(reading)                         # fleet curve
    own = dls.band_magnitudes(reading,
                              spectral_response=camera_curves)   # the camera's own
```

CLI-kommandoen `chloros-cli lattice dls --camera SERIAL` gjør det samme via backenden, ved å finne kameraets kurve i den innlastede eller bufret kalibreringspakken, og oppgir hvilken kurve som ble brukt. Over HTTP er det `POST /api/dls/cli-read` med `mac` (påkrevd: uten dette blir svaret HTTP-feil 400, `code` `MAC_REQUIRED`) og `camera_serial` i brødteksten; svarets `band_magnitudes_source` er `camera <serial>` når kameraets egen kurve ble brukt, eller `model <filter>` for flåtekurven (et kamera uten kurve: ikke RGB, eller ingen pakke lastet inn eller lagret i hurtigbufferen).

---

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

> `ChlorosAuthenticationError` og `ChlorosConfigurationError` er attributter til `chloros_sdk` (og av `chloros_sdk.exceptions`), men ikke i `__all__`, så `from chloros_sdk import *` henter dem ikke inn: importer dem etter navn. `ChlorosError`-familien er avledet av `ChlorosLocal` og funksjonene som er bygget på den (`process_folder()`, `process_lattice_capture()`); Smart-Connect-flaten utløser `ChlorosConnectError`.

Hierarki:

```

ChlorosError
├── ChlorosBackendError           (backend binary not found / failed to start)
├── ChlorosConnectionError        (HTTP transport failure; no backend running
│                                  with auto_start_backend=False)
├── ChlorosLicenseError           (403 PLAN_UPGRADE_REQUIRED: plan below Copper)
├── ChlorosAuthenticationError    (401 AUTH_REQUIRED: not logged in)
├── ChlorosConfigurationError     (bad configure() inputs)
└── ChlorosProcessingError        (pipeline failed; create/import refused)
    └── ChlorosProjectFilesMissingError  (open_project(): none of the
                                  project's source files found, even after
                                  the relink; .missing_roots, .audit)

ChlorosConnectError                (the smart-connect surface: connect_*, attach_array,
                                    list_*, discover_*, every CameraSession /
                                    ArraySession / DAQSensorSession method,
                                    RecorderHandle, ContinuousHandle,
                                    disconnect_all(), and analyze_array_network()
                                    when no backend answers; 401 and 403 included.
                                    Derives from plain Exception, NOT from
                                    ChlorosError, so `except ChlorosError` will
                                    not catch it. .warning: a warning the
                                    backend sent beside its error (also at the
                                    end of the message); .recording_closed: the
                                    recording a failed DAQ-E restart closed;
                                    both None otherwise)

FrameFetchError (RuntimeError)     (grab() / frame_stream() refusals; .status,
                                    .body, .errors)
ValueError                         (capture() arguments: PNG/JPEG with a level they
                                    cannot hold; a bad packed=)
FileNotFoundError / ImportError    (open_project(): no such folder / no project
                                    loaders)
RuntimeError                       (CameraHandle / ArrayHandle: a backend refusal)

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
        indices=["NDVI", "GNDVI", "OSAVI", "SAVI"],
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

### 3. Prosjektdrevet innspillingskampanje

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

### 6. Funksjonstest før tilkobling av et 4-kamera-array

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
        width=r["width"], height=r["height"],     # before binning
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Tilsvarende opptakskonfigurasjon (ren Python)

`run_project_recipe()` kjører en CLI-konfigurasjon slik den er (se [Prosjektmappe og konfigurasjoner](#project-folder--recipes)). De samme trinnene, skrevet i Python, er:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply: connect_all() applied each camera's saved settings. CameraHandle
    # has no settings method; a CameraSession on the same pooled camera does
    # (it is borrowed, so it never releases the camera)
    chloros_sdk.connect_camera(cam.serial).set_settings(
        exposure_time=5000, gain=6.0)

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

## MCP-server (AI-assistenter)

`chloros-mcp` lar en AI-assistent som bruker [Model Context Protocol](https://modelcontextprotocol.io) (MCP), for eksempel Claude Desktop, Claude Code eller Cursor, styre Chloros: behandle bilder (opprett et prosjekt, importer en mappe med opptak, velge innstillinger, behandle, rapportere hva kjøringen eksporterte) og betjene LATTICE-kameraer, matriser og DAQ-lyssensorer (koble til, forhåndsvise, stille inn eksponering og farge, ta opp, registrere, oppdatere fastvare). Verktøyene dekker det samme som dette SDK-et gjør. Du spør på vanlig språk, og assistenten påkaller serverens verktøy:

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

The assistant calls:
  create_project          {"project_name": "Flight_2026-10-17"}
  import_images           {"folder_path": "D:\\Flights\\2026-10-17"}
  configure_processing    {"indices": ["NDVI"]}
  start_processing        {}
  get_processing_status   {}      (every 30-60 s until its state is "finished")

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.

The assistant calls:
  list_devices            {}
  get_camera_preview      {"array_id": "array-1779862544497"}
  start_capture_run       {"array_id": "array-1779862544497", "mode": "interval",
                           "interval_s": 10, "duration_s": 300,
                           "processing": "reflectance"}
  get_job_status          {"job_id": "..."}   (until its state is "finished")
```

Serveren er bygget på denne SDK-en: behandlingsverktøyene påkaller [`ChlorosLocal`](#image-processing--chloroslocal), og maskinvareverktøyene [smart-connect-kallene](#smart-connect-for-lattice-cameras), [DAQ-økter](#daq-sensor-sessions), `ChlorosProject` og [kamera-verktøyene](#camera-utilities), som alle kommuniserer med samme backend som Chloros-appen bruker. Forespørslene inneholder `X-Chloros-Client: sdk`, slik at backend-systemet gjennomfører SDK-ets påloggings- og abonnementskontroller, og SDK-ets regler følger med (for eksempel nektes «Texture Aware» for LATTICE-opptak).

### Installasjon

Serveren leveres i `chloros-sdk`-pakken; det er ikke noe annet å installere. SDK-et installerer en `chloros-mcp`-kommando, som er den samme som `python -m chloros_sdk.mcp_server`:

```bash
pip install chloros-sdk        # not needed where the Chloros installer set up the SDK (Step 2)
chloros-mcp --help
```

- **Windows:** Chloros-installasjonsprogrammet pip-installerer SDK-et i din Python, noe som plasserer `chloros-mcp.exe` i den aktuelle Python-installasjonens `Scripts`-mappe.
- **Linux (.deb):** pakken installerer SDK-en for systemet `python3` og plasserer kommandoen `chloros-mcp` i `/usr/bin`, slik at den finnes i hver brukers `PATH`. I et virtualenv legger `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` til `chloros-mcp` i virtualenvens `bin`.

Serveren bruker kun standardbiblioteket i tillegg til SDK-et (Python 3.7+). En MCP-klient starter den og kommuniserer med den via stdin/stdout; du kjører den ikke selv, bortsett fra med `--help`.

### Klientoppsett

**Claude Desktop:**legg til serveren i `claude_desktop_config.json` (på Windows: `%APPDATA%\Claude\claude_desktop_config.json`;**Innstillinger &gt; Utvikler &gt; Rediger konfigurasjon** åpner den), og start deretter Claude Desktop på nytt:

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

**Claude Code:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Cursor:** den samme `mcpServers`-oppføringen som i Claude Desktop, i `~/.cursor/mcp.json` (alle prosjekter; `%USERPROFILE%\.cursor\mcp.json` på Windows) eller `.cursor/mcp.json` i ett prosjekt.

Alternativene angis i `args`, eller etter kommandoen for Claude Code (`claude mcp add chloros -- chloros-mcp --no-auto-start`). For å bruke en bestemt Python-SDK i stedet for den som er angitt i `chloros-mcp` på `PATH`, kjør modulen med den Python-versjonen (den fullstendige banen som `command` hvis den ikke er den første `python3` på `PATH`):

```json
{
  "mcpServers": {
    "chloros": {"command": "python3", "args": ["-m", "chloros_sdk.mcp_server"]}
  }
}
```

En klient starter serveren uten ditt skall, så hvis den melder at serveren ikke kunne startes, er `chloros-mcp` sannsynligvis ikke på den `PATH` som klienten ser. Angi i stedet den fullstendige banen (`where chloros-mcp` på Windows, `which chloros-mcp` på Linux, der .deb-filene er `/usr/bin/chloros-mcp`), eller kjør modulen med den Python-versjonen som har SDK-et.

### Alternativer

| Alternativ | Standard | Hva det gjør |
| --- | --- | --- |
| `--backend-url URL` | `http://127.0.0.1:5000` | Chloros-backenden som skal brukes. |
| `--no-auto-start` | av | Start aldri en backend: bruk en som allerede kjører, for eksempel Chloros-appens, Linux-backend-tjenesten eller en tunnel til en annen maskins (se [Fjern-backend-modus](#remote-backend-mode-pip-only-host-via-tunnel); mappestier må da peke på den samme mappen på begge maskinene). |
| `--backend-exe PATH` | der Chloros installerer den | Backenden som skal startes når ingen kjører, for en Chloros-installasjon der SDK-et ikke leter. |
| `--log-level LEVEL` | `WARNING` | `DEBUG`, `INFO`, `WARNING` eller `ERROR`. Logger sendes til stderr, som de fleste klienter viser i sin MCP-logg; stdout inneholder kun protokollmeldinger. |

### Verktøy

Argumentene til hvert verktøy er de samme som for SDK-kallet det omslutter; klienten viser hvert enkelt med fullstendig skjema. Banene er absolutte.

**Behandling**

| Verktøy | SDK-kall | Hva det gjør |
| --- | --- | --- |
| `create_project` | `ChlorosLocal.create_project` | Oppretter et prosjekt og gjør det til det åpne prosjektet, som behandlingsverktøyene virker på. Et vanlig navn oppretter mappen i standard prosjektmappen; en absolutt sti oppretter den der. |
| `open_project` | `ChlorosLocal.open_project` | Åpner et lagret prosjekt og gjør det til det åpne prosjektet. Stiene til et flyttet prosjekt pekes på nytt mot mappen det nå befinner seg i; `relink_folder` (absolutt) angir navnet på mappen der de andre manglende filene nå befinner seg. Returnerer `source_files` og `image_count`. Ingen funnet: en feilmelding som oppgir hvor de var og navngir `relink_folder`; ingenting er åpent. |
| `import_images` | `import_images` | Importerer Survey3-parene `.RAW`+`.JPG`, «LATTICE» `.tif`-opptak og DAQ `.daq`-opptak; returnerer `count` (bilder) og `scan_count` (opptak). |
| `configure_processing` | `configure` | Endrer de overførte innstillingene og ingen andre (verdier: [Støttede verdier](#supported-values)); `custom_settings` sender en hel konfigurasjon slik den er angitt. |
| `get_project_settings` | `get_config` | Leser de lagrede innstillingene for behandling, eksport og indeksering for det åpne prosjektet. |
| `start_processing` | `process(wait=False)` | Starter behandlingen av det åpne prosjektet og returnerer umiddelbart. |
| `get_processing_status` | `get_status` | Rapporterer `state` (`idle`, `starting`, `processing` eller `finished`) og fremdrift; når den er ferdig, blir kjøringens `summary`, `export_check_failure` når eksportene ikke kunne verifiseres, og `image_failure` når bilder gikk tapt: de andre produktene skrives ut; rapporter kjøringen som mislykket. |
| `process_folder` | `process_folder` / `process_lattice_capture` | Oppretter et prosjekt for en mappe, importerer den, konfigurerer og starter kjøringen i ett kall (`lattice: true` for LATTICE-opptak). Mens en kjøring pågår, avviser den før den oppretter noe, og backenden avviser da også prosjektendringer fra ethvert verktøy. |
| `export_light_sensor` | `export_light_sensor` | Skriver kalibrerte `.daq`- og `.csv`-filer for prosjektets lyssensoropptak opptakene uten å behandle bildene, og henter fabrikkalibreringen fra MAPIR-skyen når den ikke er lagret i hurtigbufferen (se [Lyssensoropptak](#light-sensor-recordings--calibrated-daq--csv)). |
| `read_image_audit_tags` | `read_image_audit_tags` | Leser Chloros-revisjonsmetadataene til et bearbeidet bilde (kilde, kalibrering, anvendte trinn). |

**Kameraer og matriser**

| Verktøy | SDK-kall | Hva det gjør |
| --- | --- | --- |
| `list_devices` | `list_cameras`, `list_arrays`, `list_daq_sensors` | Hva som er tilkoblet nå, i hvilken som helst klient, hver med `connected_here`; pluss serverens pågående oppgaver. |
| `discover_devices` | `discover_lattice_cameras`, `discover_daq_sensors` | Kameraer og lyssensorer som kan kobles til. |
| `get_device_status` | `CameraSession.status`, `ArraySession.status` / `capability`, `DAQSensorSession.status` | Live-status for ett kamera, en matrise (med tilhørende kapasitet og tilstand) eller en lyssensor. |
| `analyze_array_network` | `analyze_array_network` | Hva denne verten og dette nettverket kan håndtere for et foreslått array. |
| `connect_camera` / `connect_array` | `connect_camera` / `connect_array` | Koble til et kamera eller et synkronisert array (master først). |
| `connect_project_devices` | `open_project(...).connect_all` | Koble til alle enheter som et lagret prosjekt husker, med tilhørende innstillinger. |
| `disconnect_device` | `disconnect` / `force_disconnect` | Frigjør et kamera, et array eller en lyssensor (se Oppførsel). `keep_settings: true` på et array lagrer innstillingene for hvert kamera i sitt eget minne, mens `clear_settings: true` tilbakestiller dette minnet (resultatet er `camera_memory`). |
| `disconnect_project_devices`, `align_project_arrays`, `capture_project_devices` | `disconnect_all`, `align_arrays`, `capture_all` | Prosjektets enheter, samlet. |
| `read_camera_settings` / `set_camera_settings` | `CameraSession.read_nodes` / `set_settings` | Les eller skriv kamerainnstillinger, etter brukervennlig navn eller GenICam-nodenavn. |
| `set_array_frame_rate` | `ArraySession.set_target_fps` | Fastsett hastigheten til en matrise, eller sett den tilbake til automatisk hastighet. |
| `capture_camera` / `capture_array` | `CameraSession.capture` / `ArraySession.capture` | Én opptak (`fastest` og `wait_for_exposure` er `capture_fastest` og opptaket med fastsatt eksponering). Uten `output_dir` lagres det i det åpne prosjektets`captures`-mappe, og filene blir lagt til prosjektet (holdes tilbake til en behandlingsrunde er avsluttet); svaret `project_add` bekrefter dette. |
| `start_capture_run` | `ArraySession.continuous`, `capture_repeated` | Kontinuerlig eller intervallopptak, som en jobb. |
| `start_array_recording` | `ArraySession.record` / `burst` | En video med kombinert indeks eller en rå serieopptak, som en jobb. |
| `build_burst_video` | `ArraySession.build_video` | Behandle en serie på nytt til kalibrert video, som en jobb. |
| `get_job_status` / `stop_job` | | Følg eller stopp en jobb. |
| `get_camera_preview` | | Hvert kameras live-forhåndsvisning, returnert som et bilde. |
| `grab_frames` | `ArraySession.grab`, `CameraHandle.grab` | Ett bilde (eller ett synkronisert sett) på et behandlingsnivå, uten lagring: statistikk per plan og en PNG-fil av hvert. |
| `calibrate_array_alignment`, `get_array_alignment`, `clear_array_alignment`, `nudge_array_alignment`, `export_array_alignment`, `import_array_alignment` | `ArrayHandle.calibrate_alignment`, `alignment_status`, `clear_alignment`, `tweak_alignment`, `export_alignment`, `import_alignment` | En matrises justering. |

**Lyssensorer**

| Verktøy | SDK-kall | Hva det gjør |
| --- | --- | --- |
| `connect_light_sensor` | `connect_daq_sensor` | Koble til en DAQ-U, DAQ-M eller DAQ-E (`cap_id` angir kappe-korreksjonen). |
| `get_light_sensor_reading` | `DAQSensorSession.latest` | De siste målingene: tidsstempel, metning, spektrumområde og topp, samt spektrumet gjennomsnittsberegnet i bølgelengdeintervaller (`full_spectrum` for hvert punkt). |
| `set_light_sensor_streaming` | `stream_start` / `stream_stop` | Start eller stopp strømmen. |
| `start_light_sensor_recording` / `stop_light_sensor_recording` | `record_start` / `record_stop` | Ta opp en `.daq`-fil som en jobb; `duration_s` stopper den automatisk. |
| `set_light_sensor_cap` | `DAQSensorSession.set_cap` | Korreksjonen for kapsel (`none` for en bar sensor). |
| `get_light_sensor_bindings` / `set_light_sensor_bindings` | `daq_bindings` / `set_daq_bindings` | Hvilken sensor leverer reflektansen til hvert kamera og hver matrise. |
| `get_light_sensor_band_presets` / `get_light_sensor_reflectance` | `daq_band_presets` / `light_sensor_reflectance` | Båndsettene, samt to strømmende sensorers reflektans og indekser for omgivelser og objekt. |
| `get_light_sensor_device_info`, `set_light_sensor_network`, `set_light_sensor_auto_stream`, `enable_light_sensor_ptp`, `reboot_light_sensor`, `update_light_sensor_firmware` | `device_info`, `set_network`, `set_auto_stream`, `enable_ptp`, `reboot`, `update_firmware` | En DAQ-E-enhets egne innstillinger, lagret i enheten. |

**Kameraverktøy** ([Kameraverktøy](#camera-utilities))

| Verktøy | SDK-kall | Hva det gjør |
| --- | --- | --- |
| `get_camera_color_settings` / `set_camera_color_settings` | `camera_color_profile`, `camera_color_adjust`, `camera_highlights` og deres setters | Et RGB-kameras skjermfarge: profil og fargetemperatur, metning / kontrast, høylys. |
| `run_camera_white_balance` / `set_camera_white_balance` | `camera_white_balance_once` / `set_camera_white_balance` | Engangs hvitbalanse, eller automatisk / av / faste forhold. |
| `list_camera_presets` / `apply_camera_preset` | `camera_presets` / `apply_camera_preset` | Forhåndsinnstillinger for kameraet. |
| `list_filter_models` | `camera_filter_models`, `camera_filter_model_info` | Filtermodellene, eller én etter navn. |
| `get_camera_firmware_status` / `update_camera_firmware` | `camera_firmware_status` / `update_camera_firmware` | Firmwareversjoner og en blits (en oppgave som standard). |
| `get_camera_power_info` / `set_camera_power` | `camera_power_probe`, `sleep_camera`, `wake_camera` | Strøm- og termisk tilstand; hvilemodus eller våken. |
| `calibrate_camera_with_target` | `calibrate_camera_with_target` | Koeffisienter fra et kalibreringsmål i visningen. |
| `get_network_adapters` / `fix_network_settings` | `network_adapters` / `fix_network_settings` | Kameraets nettverkskort, et estimat og løsningen (Kun administrator). |
| `apply_vignette_correction` | `apply_vignette_correction` | Vignettkorrigering av en mappe med bilder. |
| `read_dls` | `read_dls` | Én DLS-avlesning. |
| `probe_system` / `get_system_info` | `probe_system` / `system_info` | Vertsmaskinens maskinvare og kjøretid. |

**Transiente kameraer, justeringsprofiler og indekser** ([Transiente matriser, justeringsprofiler og indekser](#transient-arrays-alignment-profiles--indices)). Hvert verktøy her som åpner kameraer, bruker `keep_settings` / `clear_settings` (standardinnstilling false): se [Kameraminne](#camera-memory--keep_settings--clear_settings).

| Verktøy | SDK-kall | Hva det gjør |
| --- | --- | --- |
| `start_camera_continuous` / `start_multi_camera_stream` | `start_camera_continuous` / `start_multi_camera_stream` | Ta opp fra ett kamera, eller strøm flere synkronisert, som en jobb (`duration_s` stopper den automatisk). |
| `capture_cameras_transient` | `capture_cameras_transient` | Én synkronisert opptak uten å koble til et array. |
| `test_array_sync` / `detect_array_master` | `test_array_sync` / `detect_array_master` | Sjekk av utløsersynkronisering, og hvilke kameraer som er på synkroniseringslinjen. |
| `analyze_array_models` | `analyze_array_models` | Hva et array av disse modellene ville oppnå på dette nettverket. |
| `calibrate_alignment_profile` | `calibrate_alignment_profile` | Løs og lagre en justeringsprofilfil (`options` inneholder de mer avanserte løsningsalternativene). |
| `capture_aligned_multiband` / `start_aligned_stream` | `capture_aligned_multiband` / `start_aligned_stream` | Et justert flerbåndsbilde, eller en strøm av slike bilder som en jobb. |
| `read_alignment_profile` / `reorder_alignment_profile` | `alignment_profile_info` / `reorder_alignment_profile` | Innholdet i en profilfil; båndrekkefølgen og aktiverte kameraer. |
| `list_index_presets`, `compute_index`, `capture_live_index` | `index_catalog`, `compute_index`, `capture_live_index` | Indeksforhåndsinnstillinger og gradienter; en indeks over en flerbånds-TIFF; opptak til indeks i ett trinn. |

**Tidssynkronisering, prosjektmappe og oppskrifter** ([PTP Grandmaster](#ptp-grandmaster--time_sync_status), [Prosjektmappe og oppskrifter](#project-folder--recipes))

| Verktøy | SDK-kall | Hva det gjør |
| --- | --- | --- |
| `get_time_sync_status` | `time_sync_status`, `time_sync_peers`, `time_sync_cameras` | PTP-grandmasterens tilstand, dens likestilte enheter og PTP-tilstanden til hvert array-kamera. |
| `restart_time_sync` / `set_time_sync_priority` | `restart_time_sync` / `set_time_sync_priority` | Start den på nytt; angi prioriteringene. |
| `get_project_folder` / `set_project_folder` / `reset_project_folder` | `project_folder` / `set_project_folder` / `reset_project_folder` | Mappen der nye prosjekter opprettes. |
| `run_project_recipe` | `run_project_recipe` | Kjør oppskriften til et lagret prosjekt som en jobb med tilhørende logg. Kontrolleres først: en oppskrift med en feil avvises og kjører ingenting; det gis advarsel om ikke-tilkoblede matriser/sensorer (den kobler dem ikke til: `connect_project_devices` først). `dry_run` avslutter kontrollen og kjører ingenting. |

Det finnes ikke noe verktøy for å åpne et lagret prosjekt for behandling: hver behandlingsjobb starter med `create_project` (eller `process_folder`).

### Krav

- **Chloros installert på samme datamaskin.** Chloros-backenden utfører behandlingen. Serveren kan i stedet bruke en backend som nås via en tunnel (`--backend-url` med `--no-auto-start`, se [Alternativer](#options)).
- **Logg inn på den datamaskinen**, i Chloros-appen eller med `chloros-cli login`. Serveren kan ikke logge inn for deg: et verktøy som startes mens du er logget ut, svarer «Ikke logget inn på Chloros» med backendens begrunnelse, og assistenten informerer deg om dette.
- **Et hvilket som helst betalt Chloros+-nivå (Copper eller høyere).** Serveren har SDK-tilgang, og det gratis Iron-nivået har ingen: et verktøy som kalles på det gratis nivået svarer «Ikke tilgjengelig på denne Chloros-planen» med begrunnelsen fra backenden.
- Python 3.7 eller nyere med SDK, og en MCP-klient som starter lokale (stdio) servere.

### Oppførsel

- **Backenden.** Å starte serveren eller vise en liste over verktøyene starter ingenting; det første verktøykallet oppretter forbindelse. Hvis Chloros-appen (eller Linux-backend-tjenesten) kjører, bruker serveren denne backenden og lar den kjøre videre. Hvis ingen backend kjører, starter serveren den installerte, uten vindu. Når serveren avsluttes (du avslutter klienten, eller den starter serveren på nytt), stopper den opptaksjobbene og lyssensoropptakene den startet, og frigjør enhetene den koblet til; backenden fortsetter å kjøre, slik at en ufullført behandlingsrunde fortsetter til den er ferdig, og den avslutter seg selv etter 30 minutter uten aktiv bruk. Med `--no-auto-start` starter serveren aldri en backend: et verktøy som kalles opp uten at noen kjører, ber deg om å åpne Chloros-appen, og neste oppkall etter at du har gjort det oppretter forbindelse.
- **Ett åpent prosjekt.** Backenden har ett åpent prosjekt, som deles med Chloros-appen når appen er åpen, så ikke arbeid med et annet prosjekt i appen mens assistenten er i gang.
- **Bare de overførte innstillingene endres.** `configure_processing` lar alle innstillinger som ikke er angitt, forbli på den lagrede verdien (SDK-versjonen `configure()` tilbakestiller debayer, vignettkorreksjon, reflektanskalibrering, PPK og eksportformat til standardverdiene når de ikke er angitt; serveren fyller dem inn fra prosjektet). Hvis den ikke kan lese prosjektets lagrede innstillinger, endrer den ingenting og melder dette, med mindre alle fem er angitt. `indices` erstatter prosjektets indeksliste. Resultatet viser det som ble sendt som `settings_sent`, og navngir under `warnings` alle indekser som kameraets bånd ikke kan danne.
- **Behandlingen foregår etter «start-then-poll»-prinsippet.** `start_processing` returnerer så snart kjøringen starter, noe som deretter tar fra minutter til timer, og assistenten følger opp med `get_processing_status`. `state` er enten `idle` (ingen behandling), `starting` (backenden har ikke rapportert kjøringen ennå, i opptil 20 sekunder), `processing` eller `finished`, med kjøringens [sammendrag](#post-run-summary--hints): hva den eksporterte, samt tips når den eksporterte mindre enn forventet (ingen bilder, ingen kalibreringsmål funnet). Produkter skrives til prosjektmappen, akkurat som fra appen. `start_processing` avvises mens en kjøring pågår, og et backend som slutter å svare, rapporteres som en feil, ikke som en fullført kjøring.
- **Hva assistenten ser.** SDK-advarslene vises i verktøyresultatet under `warnings`, og feil (logget ut, plan, feil sti) vises som feiltekst som den kan lese og videreformidle til deg. En liste med mer enn 20 elementer blir avkortet til de første 20, med full lengde angitt som `<key>_total`. Baner må være absolutte.
- **Enheter deles.** Kameraer, matriser og lyssensorer ligger i backend, slik at assistenten kan styre en som er koblet til Chloros-appen, og endringene vises i appen. `disconnect_device` viser hva serveren er koblet til; en enhet som en annen klient er koblet til (appen, `chloros-cli`, et skript, en annen `chloros-mcp`) frigjøres kun med `force: true`, som er SDK-ens `force_disconnect`, og avvisningen angir hvem som koblet den til når backenden vet det; en enhet som backenden ikke har i det hele tatt, rapporteres som ikke tilkoblet (`released: false`, `connected: false`, ingenting å frigjøre). En opptak eller strøm som backenden kjører, fører til at dens `get_job_status`-visning har `counts` — `frames_written`, `frames_dropped`, `frames_failed`, `drop_reason` (bildegrupper for en strøm med flere kameraer) — og en enkeltlinje `summary` («220 bilder skrevet, 55 droppet, 0 mislykket: ...»), mens den kjører, og, når den er ferdig, de endelige tallene; en annen klients oppføring av den (`other_clients_jobs`) inneholder det samme (`list_cameras()`, `list_arrays()` og `list_daq_sensors()` inneholder hver sin enhetss `connected_by`: `client`, `label`, `name`, `at`). Hver `chloros-mcp` identifiserer seg selv overfor backenden som `chloros-mcp (pid N)` (`chloros_sdk.connect.CLIENT_LABEL`, sendt som `X-Chloros-Client-Label`; ethvert SDK-skript kan angi dette). Å stoppe en annen klients lyssensoropptak krever også `force`.
- **Langvarig arbeid er en jobb.** `start_capture_run`, `start_array_recording`, `build_burst_video`, `start_light_sensor_recording`, `start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`, `run_project_recipe`, og ved enhver forespørsel om innfanging, løsning eller indeksering av `background`, returneres umiddelbart en `job_id`; `get_job_status` følger etter, og `stop_job` avslutter den. En kjøring fortsetter til den stoppes, med mindre du gir den `max_captures`, `duration_s` (eller, for en burst, `max_frames`); `min_free_gb` nekter å starte når stasjonen har for lite ledig plass. Det gjelder ingen begrensning med mindre du angir en. En opptak eller strøm som backenden kjører (`start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`) beholder backendens jobb-ID, slik at en annen `chloros-mcp` — en som startes senere — finner den ved hjelp av denne ID-en: `get_job_status` følger den (og viser opp slike jobber som `other_clients_jobs`, fra `GET /api/camera/cli-jobs`, hver med `started_by`), og `stop_job` stopper den først med `force: true`. Enhver annen jobb (en opptakskjøring, en matriseopptak eller burst, et bakgrunnsopptak) kjører i `chloros-mcp` som startet den, og bare den serveren vet om den: en annen som spurte etter den, sier det. En stoppet oppgave leser `stopping` mens den skriver ut køen sin, deretter `finished` med de endelige tallene, i 10 minutter og fra hvilken som helst `chloros-mcp`: den som startet det, ser en annens tvungne stopp på samme måte. `get_job_status` venter aldri på et stopp: under et (sin egen `stop_job`, en annen klients, eller stoppet som frigjør kameraene fra en jobb som avsluttet seg selv) svarer den umiddelbart med `stopping` og de sanntids tellingsverdiene, og `stop_job` fra en jobb som allerede er i ferd med å stoppe, opplyser om dette. En jobb som er avsluttet, oppgir årsaken i `stop_reason`: sin egen årsak (`no_frames`, `disk_full`, `member_missing`, eller for en kontinuerlig kjøring av en matrise `count`, `duration`, `array_gone`), `limit reached` bare når tellerne viser `count` (eller en bursts `max_frames`, en opptaketss `duration_s`) nås, `error`, `stopped` (med `stopped_by`) når en klient stoppet den, ellers `finished`.
- **Hvor opptakene havner.** Til det åpne prosjektets `captures`-mappe, der appen lagrer dem, med mindre `output_dir` er angitt.
- **Bilder.** `get_camera_preview` og `grab_frames` returnerer bilder som assistenten kan se på, i tillegg til deres JSON-data. `grab_frames`-bilder er kontrastjustert for visning; statistikken er de reelle verdiene.
- **Når serveren avsluttes**, stoppes jobber den har startet, og en lyssensoropptak den har startet, lukkes. Hvis serveren startet backenden, frigjør den også enhetene den koblet til før den stopper backenden; i appens backend forblir de tilkoblet.
- **Protokoll.** Kun stdio (ingen HTTP-endepunkt), kun verktøy (ingen ressurser eller spørsmål), tekst- og bilderesultater, MCP-protokollversjoner 2024-11-05 til 2025-11-25.**Hva som er utelatt.** Å logge inn eller ut (kontoen er din; serveren kan ikke logge inn), og å stoppe en backend som serveren ikke startet. Alt annet denne SDK-en gjør med prosessering og maskinvare har et verktøy. (Provisjonering av en DAQ-E – navn, autentiseringstoken, OTA-passord, signaturkrav, tilbakestilling til fabrikkinnstillinger – og de direkte DAQ-serverne tilhører utelukkende `chloros-cli`, også i SDK-et: et feil navn eller token sperrer alle Chloros-installasjoner fra enheten.)

---

## Automatisk oppstart av backend

Smart-connect-inngangspunktene – `connect_camera`, `connect_array`, `connect_daq_sensor`, `discover_lattice_cameras` og `discover_daq_sensors` — er enkle HTTP-klienter som forutsetter at en backend lytter på `127.0.0.1:5000` (standard-URL for smart-connect-grensesnittet). Når GUI-en eller CLI-en allerede kjører, er det en som gjør det. Fra et rent skript er det kanskje ikke tilfelle — så disse funksjonene **starter den medfølgende backend-binæren automatisk** (uten vindu, på samme måte som `ChlorosLocal` gjør) før deres første påkalling, og venter deretter opptil 60 s på at den skal komme opp. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` og `analyze_array_network()` starter aldri en.

Regler:

- **Bare en lokal URL blir noensinne opprettet.** En `backend_url` som peker mot `localhost` / `127.0.0.1` / `[::1]` er kvalifisert; enhver annen vert antas å være en annens maskin og blir aldri opprettet.
- **En backend som avsluttes før den svarer, avslutter ventetiden.** Anropet returnerer umiddelbart med avslutningskoden, kommandoen den kjørte og de siste linjene backenden skrev ut, i stedet for å vente i 60 sekunder. Avslutningskode 98 er unntaket: en annen backend har allerede porten eller låsefilen (Chloros-appens, backend-tjenesten eller et annet skript som fortsatt starter), så anropet fortsetter å vente på at den skal svare. `ChlorosLocal` gjør det samme.
- **Backenden blir stående i gang for gjenbruk** (på samme måte som i CLI-en) — det skjer ingen implisitt avslutning når skriptet avsluttes. Når du kjører skriptet på nytt, gjenbrukes den aktive backenden.
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

Hvis den medfølgende binærfilen ikke kan finnes eller startes, utløser det påfølgende HTTP-kallet en handlingsbar, **plattformavhengig** `ChlorosConnectError` i stedet for en ren «forbindelse avvist»-logg — på Windows viser den deg til skrivebordsappen eller en `chloros-cli`-kommando; på Linux (uten GUI) viser den deg til en `chloros-cli`-kommando eller `.deb`.

---

## Miljø og overskrifter

SDK-et merker hvert HTTP-anrop til backenden med `X-Chloros-Client: sdk`. Backenden bruker SDK/CLI-lisensieringsregler (pålogging **og** et betalt Chloros+-abonnement kreves) i stedet for GUI-gratisversjonen. Dette settes automatisk ved import – du trenger ikke å gjøre noe.

Hodet legges til i hvert `requests` / `urllib`-anrop der URL-en inneholder `localhost` eller `127.0.0.1`, uansett port – inkludert dine egne lokale tjenester — og erstatter en verdi du selv har angitt. Anrop til andre verter forblir uendret.

Overstyr backend-URL-en ved å sende med `backend_url=` (`ChlorosLocal` tar imot `backend_url=` eller `api_url=`; `process_folder()` og `process_lattice_capture()` godtar kun `api_url=`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(En installert backend lytter kun på loopback, så en LAN-`backend_url` kan ikke nå den; se [Remote-Backend Mode](#remote-backend-mode-pip-only-host-via-tunnel) for tunnelmønsteret.)

---

## Versjonering og kompatibilitet

- SDK-versjonen eksponeres som `chloros_sdk.__version__`.
- SDK-et er avhengig av den medfølgende backend-versjonen. Å kombinere et eldre SDK med en nyere backend fungerer vanligvis (fremoverkompatible endepunkter), men å kombinere et nyere SDK med en eldre backend kan føre til `404`-feil på nye endepunkter — oppgrader skrivebordsappen for å sikre kompatibilitet.
- Smart-connect-grensesnittet (`connect_camera` / `connect_array` / `connect_daq_sensor`) og endepunktet for nettverksanalyse returnerer stabile JSON-skjemaer; nye felt legges til.

---

## Tips til feilsøking

- **`ChlorosAuthenticationError: Login required`** (fra `ChlorosLocal`; Smart Connect-anropene utløser `ChlorosConnectError: Login required. …` av samme årsak) → Kjør `chloros-cli login EMAIL PASSWORD` én gang på denne maskinen, eller logg inn via Chloros-skrivebordsappen.
- **`ChlorosProcessingError: Failed to start processing: … The project folder is too deep for this computer …`** (eller `Failed to create project: …` fra `create_project()`) → Windows-funksjonen for lange baner er deaktivert på denne maskinen (Windows-standard), så ingen sti kan være lengre enn 260 tegn, og produktene ligger opptil ca. 130 tegn under prosjektmappen. Ingenting ble behandlet. Meldingen angir den lengste prosjektmappen som passer: bruk en kortere, eller slå på lange stier i Windows og start Chloros på nytt. Windows-installasjonsprogrammet tilbyr dette som «Aktiver lange Windows-stier», som er merket av som standard; ellers bruk gruppepolicy: *Datamaskinkonfigurasjon &gt; Administrative maler &gt; System &gt; Filsystem &gt; Aktiver lange Win32-stier*.
- **`ChlorosConnectError: No Chloros backend is running …`** → Smart-connect-funksjonen starter automatisk en lokal backend, så denne feilen vises bare når den medfølgende binærfilen ikke kan finnes eller startes (f.eks. en pip-enhet uten skrivebordspakke). Meldingen er plattformavhengig: på Windows åpner du skrivebordsappen eller kjører en hvilken som helst `chloros-cli`-kommando; på Linux: kjør en `chloros-cli`-kommando (det finnes ingen grafisk brukergrensesnitt) eller installer `.deb`. For en ekstern backend, send `backend_url=` (og `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** ved import → `lattice_sdk` kunne ikke lastes inn: Arena SDK-kjøretiden, eller OpenCV (`cv2`), kan ikke importeres i denne Python-versjonen. Den backend-drevne overflaten (`connect_*`, projiserer gjennom en backend, `ChlorosLocal`) fungerer fortsatt.
- **Array kobler til med full sensor, men lav bildefrekvens**→ Slik er det designet: auto beholder full sensor og lar frekvensen falle til det kabelen tåler. Bruk `binning=2` / `4` eller en mindre `width` / `height` for flere bilder per sekund, eller aktiver jumbo-rammer / oppgrader forbindelsen. Binning og ROI dekker**ikke** samlet overabonnement (`oversubscribed: true`, fps-felt 0): for mange kameraer for kabelen kan ikke løses ved hjelp av binning/ROI — reduser antall kameraer, aktiver jumbo-rammer eller bytt til et raskere nettverkskort (se [Overtegnelse](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` rapporterer at nettverkskortets mottaksring er svært liten (~0,26 MB) / koble sammen portene med «FRAMES WILL DROP»** → Vertsnettverkskortets mottaksring er på standardverdien (blir ofte tilbakestilt til 32 etter en oppdatering av nettverkskortdriveren). På et Realtek USB 10GbE-kort må du sette `ReceiveBufferLen=256` og `PendingReceives=64` (forhøyet), og deretter starte backenden på nytt slik at den leser ringen på nytt. Fullstendig fremgangsmåte: [CLI-referanse → Konfigurering og justering av verts-NIC](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Verten henger seg opp ved omstart/avslutning, senere WMI-feil `Invalid class` / NIC-kortet aktiveres ikke** → Utdatert USB 10GbE-driver forårsaker `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Oppdater adapterdriveren til en aktuell versjon (≥ 2026) og bruk innstillingene for mottaksringen på nytt. Se [CLI-referanse → Konfigurering og justering av verts-NIC](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Array-innsamling mislykkes med HTTP-feil 503 «No fresh synced frame group»** → Arrayet publiserte ingen ny ramme-gruppe innenfor ventetiden. Sjekk `arr.status()`: `stalled: True` (eller `streaming: False`) med `actual_fps`. 0 betyr at innhentingssløyfen har sluttet å publisere — koble fra og koble til arrayet på nytt. Hvis `stalled` er False og `actual_fps` er over 0, prøv på nytt. Rett etter `connect_array()` venter en opptak allerede i opptil 20 sekunder på den første gruppen.
- **`status()` viser `link_speed_mbps: 4294`** → Windows kunne ikke lese adapterens hastighet; dette er typisk for USB-Ethernet-adaptere og er ikke en feil. Se [Forventede rammehastigheter](#expected-frame-rates).
- **`update_firmware()` mislykkes med `device never connected back on TCP <port>`** → En brannmur blokkerte DAQ-E-enhetens tilkobling tilbake til denne datamaskinen. Windows-installasjonsprogrammet legger til regelen (se [DAQ-E-enhetsinnstillinger](#daq-e-device-settings)); med en annen brannmur må du tillate innkommende TCP-trafikk til backenden fra sensorens undernett.
- **Refleksjonsmåling avvist** → En aktiv DAQ må være koblet til kameraet (eller arrayet) for å oppnå reflektans i absolutt skala. Koble enten til via brukergrensesnittet, eller bruk `processing="radiance"` (W/m²/sr/nm), som ikke krever en tilkoblet sensor.
- **`smart=True`-opptak tar lengre tid enn forventet** → AE-konvergens avhenger av scenens dynamikk. Med `ArrayHandle.capture_smart()`, utvid `exposure_tolerance_pct`, forkort `stability_window_s` eller senk `settle_timeout_s` for en raskere, mindre stabil utløser.

---

## Se også

- [CLI-referanse](cli-reference.md) — hver CLI-underkommando tilsvarer et SDK-kall.
- [DAQ-lyssensorer](https://mapir.gitbook.io/chloros/daq-light-sensors) i Chloros-håndboken — sensorene, deres deksler og kalibrerte måleområde, registrering, DAQ-E-nettverk.
- [LATTICE-kameraer](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) i Chloros-håndboken — tilkobling, kamerainnstillinger, opptaksmoduser, matriser.
- Online dokumentasjon: `https://mapir.gitbook.io/chloros/api-python-sdk`
