# Chloros-CLI-referanse

**Versjon:**

1.3.0**Generert:**

29.07.2026 kl. 19:19 ·**Revidert:**

06.11.2026**Målgruppe:** Optimalisert for bruk av store språkmodeller (LLM); lesbar for mennesker.**Omfang:** Alle brukerrettede underkommandoer i `chloros-cli`, med alternativer og eksempler som kan kopieres og limes inn.

Dette dokumentet er den fullstendige referansen for kommandolinjeverktøyet `chloros-cli` som følger med MAPIR Chloros. Det er bevisst uttømmende, slik at en LLM (eller et menneske) kan sette sammen enhver støttet arbeidsflyt fra listene nedenfor uten å måtte undersøke kildekoden.

Hvis du bare trenger det viktigste, kan du hoppe til:
- [Fem minutters hurtigstart](#five-minute-quickstart)
- [Arbeidsflyt for første tilkobling av LATTICE-kamera](#lattice-camera-first-connect-workflow)
- [Arbeidsflyt for første tilkobling av DAQ-sensor](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Opptakmoduser, opptakere og offline-bearbeiding](#capture-modes-recorders--offline-reprocess)

> **Styrer du Chloros fra en AI-assistent?** Claude Desktop, Claude Code, Cursor og andre MCP-klienter kan kjøre prosessering, LATTICE-kameraer og -matriser samt DAQ-lyssensorer via `chloros-mcp`, MCP-serveren som installeres sammen med Python-SDK-et (og med Linux-.deb-filen, i `/usr/bin`): `claude mcp add chloros -- chloros-mcp`. Oppsett, alternativer og verktøy finnes i SDK-referansens [MCP-server (AI-assistenter)](sdk-reference.md#mcp-server-ai-assistants). En assistent med et skall kan også kjøre alle kommandoene på denne siden direkte.

> **Endret i 1.3.0** (noe et eksisterende 1.2.0-skript eller en arbeidsflyt vil merke).
>
> **Endringer et 1.2.0-skript kan måtte håndtere:**
>
> - **`--preset` gjelder.** `lattice capture`, `continuous`, `calibrate`, `multi-capture`, `multi-stream`, `align-calibrate`, `align-apply` og `align-stream` ble fjernet i versjon 1.2.0 og kjørte på kameraets egne innstillinger. Den angir nå forhåndsinnstillingens format, eksponering, forsterkning og bildefrekvens: `--preset high_speed` tar opp 8-bit ved 5 ms, 6 dB, 60 fps. `lattice cam-connect --preset` beholder også forhåndsinnstillingens format (versjon 1.2.0 satte kameraet til 12-bit etter at forhåndsinnstillingen ble brukt). Se [Vanlige alternativer](#common-options-shared-by-most-subcommands).
> - **Et Bayer-pikselformat på et M3M-kamera (mono) er dets dybde.** Forhåndsinnstillingene angir Bayer-formater, som en monosensor ikke kan ta: i versjon 1.2.0 avviste kameraet formatet og beholdt det det hadde, så `--preset high_speed` var ikke 8-bit der. Det er nå Mono-formatet med samme dybde: `high_speed` og `--pixel-format BayerRG8` gir `Mono8`, `high_quality`, `triggered` og `BayerRG12` gir `Mono12`. I en blandet M3C + M3M-matrise bruker hvert kamera det felles formatet i sin egen familie, og `network-analysis` / `analyze-array --pixel-format` projiserer hvert kamera på den dybden. Det samme gjelder omvendt: et M3C som blir bedt om et Mono-format, bruker Bayer-formatet for den dybden, så et array med et M3M som master åpner ikke lenger sine M3C-kameraer i Mono. Se [Mono (M3M) enkeltbåndskameraer](#mono-m3m-single-band-cameras).
> - **`lattice array-capture`-eksportfiler er som standard justert, akkurat som appens filer.** Hvert medlems `debayered`, `display`, `radiance`, `reflectance` og indeksfilene blir forvrengt inn i hovedkameraets ramme og beskåret til medlemmenees overlapping (mindre enn sensoren på grunn av kameraenes forskyvninger, `XMP-mapir:AlignmentJSON` `"applied": true`), slik at en CLI-opptak består av de samme filene som Chloros-appens opptak av den samme scenen; `raw` forblir i opprinnelig format. Tidligere lagret en opptak uten `--aligned` alle filene i opprinnelig størrelse, uten forvrengning. `--no-aligned` beholder dette. Den samme standardinnstillingen gjelder for `--smart`, `--continuous` og `--interval`, samt for SDK og `chloros-mcp`. Se [`array-capture`-moduser](#array-capture-modes).
> - **`lattice array-capture --compression` bruker `none` som standard.** Filene skrives ukomprimert (ca. 6,3 MB per bilde i full oppløsning) for å oppnå den raskeste vedvarende hastigheten; versjon 1.2.0 skrev `deflate` (ca. 4,1 MB). Bruk `--compression deflate` for mindre filer. Begge er tapsfrie.
> - **PNG og JPEG støtter kun `display` / `preview` og indeksprodukter.** En `--format png`- eller `jpg`-opptak med et nivå som en 8-bits fil ikke kan håndtere, avvises før noe blir tatt opp (HTTP-feil 400, avslutt `1`), og det samme gjelder standardinnstillingen `--processing all` med `--format png`/`jpg`. Se [Eksportnivåer for opptak](#capture-export-levels-the-all-default).
> - **`lattice dls` trenger `--mac`.**Det finnes ikke lenger noen standard-MAC (det var adressen til én enhet); `daq pool-discover --only ble` finner adressen til en sensor.**`lattice dls --camera SERIAL`** er ny.
> - **Flere feil fører til avslutning av `1`.** `process` når eksportene fra en kjøring ikke kunne verifiseres eller den mistet et bilde, selv om de andre produktene er skrevet; en innfangingskommando med en hvilken som helst fil den ikke klarte å skrive (de andre filene lagres fortsatt); `lattice continuous` og `multi-stream` ved full harddisk; `array-burst` / `array-record` når opptakeren melder en feil; `array-build-video` når et forespurt produkt ble hoppet over eller ikke bygget som forespurt; `project burst` / `stream` når en opptak mislykkes underveis; `project sensor log` når opptaket ikke inneholder noen avlesning; `lattice exposure --set`, `gain --set` og `configure --set` med en verdi som kameraet avviser. `lattice network-analysis --json` avsluttes på samme måte som sammendraget (`1` for `needs_force_slip`). Omvendt: `daq pool-disconnect` fra en sensor som ikke lenger er i puljen, avsluttes med `0`. Se [Avslutningskoder](#exit-codes).
> - **En treg `lattice array-connect` mislykkes aldri når matrisen er tilkoblet.** Den venter i opptil 600 s (180 s i 1.2.0; `project connect` 90 s), og skriver ut backendens trinn underveis, og en tilkobling som ikke er fullført innen da, blir opphevet av backenden — hvert kamera settes tilbake slik det ble funnet — så en `Error:` (avslutning `1`) betyr at ingen matrise er tilkoblet. I versjon 1.2.0 kunne en tilkobling rett etter at kameraene ble slått på, vare lenger enn ventetiden og koble til matrisen etter at kommandoen hadde mislyktes. Ctrl+C avbryter nå tilkoblingen. Se [Vedvarende tilkoblinger](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-connect` beholder hele sensoren.** Uten `--width` / `--height` / `--binning` er hvert kamera satt til full sensor, og bildefrekvensen faller til det denne datamaskinen og nettverket kan levere; tilkoblingsutdataene viser raskere alternativer (binning eller et mindre ROI) med deres forventede hastigheter. Versjon 1.2.0 komprimerte eller binnet bildene for å opprettholde hastigheten. Se [`array-connect`-alternativer](#array-connect-options).
> - **Et tregt nettverk avviser ikke lenger et array.** Hvert kamera tilpasses sin andel av båndbredden, og bildefrekvensen synker tilsvarende. Der kameraene ikke kan tilpasses så langsomt, kjører arrayet i seriell utsending (`slip-emit-and-capture`): hvert bilde kommer frem, men kameraene eksponeres ikke samtidig. `--force-tier sim-capture-sim-emit` krever samtidig eksponering. Se [Array fps & burst-modell](#array-fps--burst-model).
> - **`lattice network-analysis` / `analyze-array` angir hvilken tilkobling som gjelder**: uten `--pixel-format`, formatene som «connect» angir (Mono12 / BayerRG12, tidligere BayerRG8), og valget av [pakket 12-bit](#packed-12-bit-transport---packed). Den samme kommandoen kan derfor angi en annen bildefrekvens enn i versjon 1.2.0.
> - **Relative baner angis der du kjører kommandoen.** `-o` og de andre banene som backenden åpner (`--input`, `--burst-dir`, `--profile`, `--save`, `--mask`), et oppskriftstrinns `output`, og standardverdien `output/` for `lattice index --live` blir løst i forhold til mappen du kjører `chloros-cli` fra. I versjon 1.2.0 løste backenden dem i sin egen installasjonsmappe, noe som på en installert versjon resulterte i feilmeldingen «Tilgang nektet». Se [Konvensjoner](#conventions).
> - **`lattice index` kjører på backenden i alle former**, akkurat som de andre `lattice`-kommandoene, så start backenden først. Den offline indeksen (`--input`) beregnes av backenden, som leser TIFF-filen og skriver resultatene ved siden av den eller i `-o`, og `--list-presets` / `--list-gradients` skriver ut backendens tabeller. På Windows fungerer disse nå: Tidligere svarte Windows-CLI på alle former for `lattice index`, inkludert `--live`, med `Error: lattice_sdk not available`. På Linux kjørte den offline indeksen tidligere inne i CLI-en uten backend og uten pålogging; nå trenger den begge deler, akkurat som alle andre CLI-kommandoer. Se [Indeks / Vegetasjonsmatematikk](#index--vegetation-maths).
> - **Nye filer ved siden av produktene.** Hvert radiance-, reflektans- og indeksprodukt der båndseparasjonen har klemt en piksel, får en maske i en `clamp_masks/`-mappe ved siden av produktmappen, og utskriften viser en `clamp mask:`-linje. Se [Unmix-klampemaske](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on).
> - **Kameraorientering.** Et opptak med ett kamera lagrer `raw` og `debayered` slik sensoren leste dem, og registrerer kameraets speilvendning/rotasjon i XMP; `process` bruker dette på produktene det lager. Filer i arrayet forblir i opprinnelig format med mindre du sender den nye `--orient` (`array-capture`, `array-burst`, `project capture` / `burst` / `stream`). Se [Kameraorientering](#hvordan-en-opptaksmappe-ser-ut).
> - **Et eksportnivånavn som ikke er «one», blir avvist.** `lattice capture --processing` og alle `--levels`-lister tar `raw`, `debayered`, `radiance`, `reflectance` eller `display` (eller et alias som `preview`; `--processing` samt `all`). Ethvert annet navn avvises før kameraet berøres (HTTP 400 `UNKNOWN_LEVEL`, avslutt `1`); 1.2.0 lagret `raw` for det og avsluttet `0`.
> - **`daq pool-set-cap` trenger `--cap-id`.** Uten det avsluttes kommandoen `2`; 1.2.0 fjernet dekselkorreksjonen, slik at sensoren ble blottet. `--cap-id none` ber om den rene sensoren.
> - **`lattice resolution --binning-mode` er borte.** Binning er alltid «Average»; flagget aksepteres fortsatt, men det vises en advarsel og ingenting skjer. Se [Sensorkalibrering](#sensor-tuning).
> - **`process --debayer texture-aware` avvises for LATTICE-opptak.** For en mappe som inneholder LATTICE-opptak, avslutter `process` `1` etter importen og oppgir hvor mange det er, før noe behandles. LATTICE-opptak får alltid standard demosaik, så versjon 1.2.0 aksepterte alternativet og ignorerte det; Chloros-appen tilbyr det heller ikke for dem. Se [`chloros-cli process`](#chloros-cli-process).
> - **Kameraene går tilbake til den tilstanden de ble funnet i — med mindre `--keep-settings`.** `lattice array-disconnect`, alle midlertidige kommandoer (`multi-capture` uten matrise, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`) og `lattice continuous` setter hvert kamera tilbake slik de fant det: utløser- og synkroniseringsledningsforbindelsene, PTP-innstillingene og alle kamerainnstillinger som ble endret under økten (eksponering og forsterkning, AE-mål, grenser og demping, hvitbalanse, svartnivå, pikselformat, binning og ROI, speilvendinger, bildefrekvens). `--keep-settings` lagrer derimot hvert kamerasinnstillinger i kameraets eget minne, slik at det starter opp med dem; `--clear-settings` tilbakestiller dette minnet til fabrikkinnstillingene. Uansett blir kameraene satt i frittløp. Et prosjekt konfigurerer kameraene sine ut fra sine egne lagrede innstillinger når det kobler dem til. Se [Kameraminne](#camera-memory----keep-settings----clear-settings).
> - **Alle moduser lagrer et prosjekt på samme måte.** Enhver endring på et kamera, en matrise eller en lyssensor som er tilkoblet under et prosjekt – fra appen, CLI, SDK eller MCP-serveren – lagres i prosjektets `cameras.json` / `sensors.json`, og når prosjektet åpnes i en hvilken som helst modus, konfigureres enhetene ut fra dette. Tidligere var det kun appen som lagret prosjektet, og neste lagring overskrev alt annet. Se [Én prosjektpost, alle moduser](#one-project-record-every-mode).
> - **Filer laget fra flere kameraer navngir hvert enkelt kamera.** En multibåndsstabel (`align-apply`, `align-stream`, `index --live --save-multiband`), en live-indeks laget av bånd fra flere kameraer, og et bilde fra en array-opptak med navnet `composite/` inneholder hvert kildekameras serienummer, modell og kalibreringspakke som lister (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), og EXIF-listene `SerialNumber` inneholder serienumrene, adskilt med komma. Tidligere inneholdt de enten ingen kameraopplysninger i det hele tatt, eller kun serienummeret til det første kameraet. Se [Filer laget fra flere kameraer](#files-made-from-several-cameras).
> - **Bare en `raw`-fil inneholder kameraets kalibreringstagger.** Det MicaSense-kompatible settet (`BlackLevel`, `RadiometricCalibration`, den kanalvise `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`-settene) og nyttelasten `CalibrationJSON` beskriver sensorens råverdier, så de ligger kun i opptaksfilen `raw`. Alle andre filer – en opptakets `debayered`, `display`, `radiance`, `reflectance` og indeksfiler, `lattice continuous` og `multi-stream`-rammer, og hvert `process`-produkt — inneholder kameraets identifikasjon (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) og, ved en opptak, rammens egen registrering (`XMP-mapir:FrameJSON`: eksponering, forsterkning, temperatur, pikselformat, opptakstid). I versjon 1.2.0 inneholdt de også rådatasettet, som et verktøy som bruker det (Pix4D, Metashape) brukte på piksler som ikke er råverdier. `process` finner kalibreringen til en slik fil ved hjelp av serienummeret og `CalBundleSha256`.
>
> **Andre endringer:**
>
> - **En matrises automatiske bildefrekvens tilpasses det matrisen leverer.** En frekvens som mister mer enn 1 % av utløserne, reduseres – vurdert over flere lange observasjoner i stedet for én – og en frekvens den ble redusert fra, prøves på nytt av og til. En frekvens du selv angir, forblir uendret.
> - **Automatisk eksponering er mer stabil.** Den holder seg konstant ved jevnt lys, stabiliserer seg i løpet av sekunder etter en endring, ignorerer små høylys og gjenoppretter et kamera som slutter å svare i sterkt sollys. Se [Hvordan automatisk eksponering fungerer](#how-auto-exposure-behaves).
> - **`process` teller kun dine bilder og opptak.** Hvis du kjører den på nytt på en mappe som inneholder et prosjekts `camera_cal/`, teller den ikke lenger disse kalibreringsfilene («Fant 96 bilder» på et sett med 90 bilder), og et prosjekt som allerede har tatt dem inn, sletter dem.
> - **`lattice multi-test` starter fra kameraenes oppstartstilstand.** Det slår på kameraenes PTP for kjøringen, venter på at klokkene deres skal låses og stabilisere seg før den første utløsningen — hvert kamera låses i 120 s, omtrent 2 minutter fra oppstart (`--ptp-settle SECONDS` setter den), siden en klokke som nettopp har låst seg, har en avvik på opptil ~200 us i omtrent 2 minutter – og setter hvert kameras PTP tilbake slik den fant den. Et kamera som fortsatt er låst fra en tidligere test, blir ikke ventet på igjen. Tidligere ble PTP slått av i et nettverk med én bryter der ingen DAQ-E svarte, og hvert kameras frittløpende klokke ble tolket som en PTP-klokke som ikke ville synkronisere seg (`PTP is not locked`, ved hver kjøring). Se [Flerkamera (forbigående økter)](#multi-camera-transient-sessions).
> - **En `.daq`-logg angir om lokket ble valgt eller antatt.** DAQ-U, DAQ-M og DAQ-E leveres uten hette, og Chloros kan ikke se om en er montert, så en sensor som er koblet til uten at det er angitt en hette, får `sunshine_cosine` som standard, en *antagelse*. Hver Chloros-registrering (skjema 1.30) lagrer nå `cap_id_source` ved siden av `cap_id`: `operator` når du har gitt lokket et navn (inkludert `none`), og `auto_default` når det var standard. Ved behandling av en opptak der lokket ble antatt, og prosjektets DAQ-lokk er satt til «Auto», skriver ut en `[daq-dls] WARNING` som navngir filen: en ren DAQ-U under solskinnskorreksjon viser omtrent 30 ganger for høye verdier. **Gi navn til den capen du har valgt** når du kobler til, eller sett prosjektets DAQ-cap, så forsvinner advarselen.
> - **Ingen flere pip-konsollvinduer på Windows lenger.** På en PC med Python installert kjørte backenden og hver av dens bildebehandlere `pip` én gang ved oppstart, og hver av dem åpnet et konsollvindu. Det skjer ikke lenger.
> - **Live-opptak av `.daq` overlever et strømbrudd.**Et opptak skrives med en logg og en full disksynkronisering ved hver lagring, bak sensoren, slik at en treg disk forsinker filen, men aldri målingene. En opptak som stoppes midt i en commit (strømbrudd, avbrudd) kan etterlate `<name>.daq-journal` ved siden av seg:**kopier de to sammen.** Chloros ruller journalen tilbake første gang den leser filen, slik at opptaket åpnes med alt som ble bekreftet før stoppet (omtrent de siste 3 sekundene går tapt); i versjon 1.2.0 kunne et strømbrudd på feil tidspunkt gjøre hele filen uleselig («database disk image is malformed»).
> - **En opptak på en opptatt disk setter ikke lenger sensoren i stå.** `daq pool-record` setter avlesningene i kø og skriver dem på en egen tråd; en avlesning som ikke kunne skrives, telles (`daq pool-list` viser `N write error(s)` med den siste feilen) i stedet for å fryse sensoren (én skriving tok 8,4 sekunder ved siden av fire kameraer som lagret TIFF-filer).
> - **`lattice multi-capture` uten en matrise mister ikke lenger bildene den har lagret når en utløser går ut på tid.** En kamerautløser som går ut på tid (`SC_ERR_TIMEOUT -1011`, observert med fire kameraer på én 1 GbE-switch) avslutter kjøringen ved den bildegruppen: de allerede lagrede gruppene vises med `Saved:`, deretter feilen, exit `1`. I versjon 1.2.0 var det en HTTP-500-feil uten liste, selv om filene lå på disken.
> - **Programvareutløste kameraer dropper ikke lenger annenhver utløsning.** Et kamera i programvareutløsningsmodus setter nå en utløsning som kommer inn under avlesningen i kø (eller Chloros venter på at den skal bli klar), slik som kabelsynkroniserte kameraer allerede gjorde.
> - **`daq pool-discover` navngir en DAQ-E-S `DAQ-E-S`** i sin `MODEL`-kolonne (og `--json`s `model`). Den angav `DAQ-E` for hver Ethernet-sensor; en som ikke svarer på identitetskontrollen innen ca. 2 sekunder, er fortsatt oppført som `DAQ-E`.
> - **Forbedrede LATTICE-indeksprodukter** (NDVI og andre indekser) fra `process`, i alle behandlingsmoduser.
> - **OCN-kameraer: forbedrede radianse-, reflektans- og indeksprodukter i sollys.** En flåtekorreksjon blir anvendt etter båndseparasjonen; RGN-kameraer er uendret, og OCN-verdiene avviker fra versjon 1.2.0. Dette gjelder også for et OCN-opptak laget med en tidligere versjon, når det behandles på en datamaskin som har kalibreringen som opptaket ble tatt med (Chloros beholder kameraets kalibrering så snart kameraet er koblet til).
> - **`lattice continuous` registrerer råbildet.** En `tiff`-kjøring skrev demosaiserte 3-plans bilder, som `process` ikke kan ta utgangspunkt i. Dens bilder er nå den `raw`-filen som en array-opptak skriver: ett mosaikkplan med sensorens egne verdier, ukomprimert, med kalibreringspakken. Et skript som leser tre plan fra disse filene, leser bare ett.
>
> - **Hver opptak lagrer DAQ-avlesningen den brukte, akkurat som GUI-en gjør.** `lattice capture`, `lattice continuous` (én `.daq` per ramme), `array-capture` og `multi-capture` med et tilkoblet array (og SDK) skriver `.daq` og dens `.match.json` hver gang en DAQ blir bundet eller lagt i en pool, inkludert en opptak som kun er i råformat; i versjon 1.2.0 ble en rå eller debayert opptak lagret kun med `--force-daq`. `--no-daq` slår det av, `--daq-tolerance-ms MS` angir samsvarstoleransen for det opptaket, og den nye `daq pool-bind` binder et kamera eller en matrise til en DAQ fra CLI-en. Med `.daq` blir et opptak som lagres, i `process` kun matchet mot de kameraene det navngir. En opptak venter nå på DAQ-enhetens neste avlesning, slik at den kan blande avlesningene på begge sider av eksponeringen. Se [DAQ-avlesningen som en opptak lagrer](#the-daq-reading-a-capture-saves) og [En DAQ-avlesning langt fra eksponeringen](#a-daq-reading-far-from-the-exposure).
> - **`process --indices NAME`** legger til indeksen i prosjektet for hvert kameras bånd, inkludert et nytt prosjekt. I versjon 1.2.0 ble et navn som prosjektet ennå ikke hadde oppført, ignorert med `Unknown index name(s) ignored`.
> - **`process` ved siden av et live-array:**det starter ikke lenger på nytt en backend som inneholder kameraer eller DAQ-er, og venter i opptil 15 sekunder på en kjøring som er i ferd med å avsluttes.**Ctrl+C** stopper kjøringen i backenden, ikke bare i CLI-en. Se [`process`-merknader](#notes).
> - **En prosjektmappe som ligger for dypt i strukturen til å overholde Windows’ grense på 260 tegn i stien** avvises umiddelbart med den lengste mappen som passer, i stedet for at kjøringen ikke skriver noe. Windows-installasjonsprogrammet tilbyr nå «Aktiver lange Windows-stier» og et unntak i Microsoft Defender for backend-delen, begge merket av som standard. Se [Windows (.exe)](#windows-exe).
> - **`lattice array-capture --continuous`** kjører backendens egen kontinuerlige opptak: en fremdriftslinje hvert 2. sekund, en oppsummering i stedet for en filvis liste, og avsluttes med en ikke-null-kode ved eventuelle ubeskrevne filer eller en full harddisk. Se [`array-capture`-moduser](#array-capture-modes).
> - **Den første `array-capture` etter `array-connect`** venter i opptil 20 sekunder på matrisens første bildegruppe i stedet for å gi feilmeldingen HTTP-kode 503 «Array still warming up».
> - **Array-eksponering under et ROI:** et lite ROI begrenser ikke lenger eksponeringen til et array-medlem til sensorens friløpsperiode (4,9 ms ved 768 × 304). Se [Eksponering på et array-medlem](#exposure-on-an-array-member).
> - **Automatisk eksponering reduserer ikke lenger et arrays automatiske bildefrekvens.** Et kabelsynkronisert array kjører med den frekvensen nettverket og kameraenes utlesning tillater; automatisk eksponering fyller perioden som denne frekvensen gir, og øker deretter forsterkningen. En manuell eksponering som varer lenger enn perioden (`lattice exposure --set`, `configure --set`) reduserer utløsningshastigheten for å tilpasse seg den i stedet for å bli avvist, og hastigheten gjenopprettes når elementet går tilbake til automatisk modus. Se [Eksponering på et array-medlem](#exposure-on-an-array-member).
> - **Arrayer leverer færre ufullstendige bildegrupper**, og færre 503-feil («Ingen ny synkronisert bildegruppe»): utløserinngangen til hvert slavekamera er nå renset for glitcher.
> - **Pakket 12-bits transport** (`--packed`, standard `auto`) øker bildefrekvensen for fullsensor-matriser med omtrent en tredjedel. `array-connect --packet-size` er nå tatt i bruk (versjon 1.2.0 aksepterte den, men ignorerte den). Se [Pakket 12-bits transport](#packed-12-bit-transport---packed).
> - **Et array der kameraene gjenoppretter seg selv etter strømavbrudd.** Etter et strømavbrudd på bryteren gjenoppretter `array-connect`-arrayet-åpner kameraene når de svarer og strømmer igjen med PTP på; i 1.2.0 stoppet den for godt og trengte en ny `array-connect`. `array-status` rapporterer `ptp_enabled: False` (og `ptp_off:`) mens et kamera leser PTP som deaktivert, der det tidligere rapporterte innstillingen som arrayet var koblet til med. Se [Persistent Connections](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-record` tar opp uten at GUI-en er åpen.** Når GUI-ens kombinerte visning er lukket, gjengir backenden visningen selv: arrayetsNDVI, eller indeksen du sender med den nye `--expr`. En opptak som ikke skriver noen ramme, mislykkes nå (`no-frames: …`, avslutt `1`). Tidligere ble det kun registrert mens GUI-ets kombinerte visning var åpen, og en opptak som ikke skrev noe, avsluttet likevel `0`. Se [`array-record`](#array-record--combined-index-videogif-monitoring-grade).
> - **Et kamera som Chloros allerede holder, åpnes ikke en gang til.** En kommando som ville åpnet det selv, svarer i stedet med HTTP-kode 409 `CAMERA_HELD`, og oppgir hva som holder det og kommandoen som frigjør det. Dette gjelder også et kamera som en annen kommando bruker akkurat nå (en `multi-test` eller `multi-capture` som åpner sine kameraer, en `lattice capture`): den andre kommandoen avvises inntil den første er fullført. Å åpne det på nytt kan stoppe strømmen til den som holder det, eller føre til at backenden krasjer. Se [Persistent Connections](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **Midlertidige kommandoer for flere kameraer leverer alle kameraene eller mislykkes.** `lattice multi-capture` uten tilkoblet array, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream` og `index --live` avsluttes med at `1` navngir et kamera som ikke leverte noen ramme. Strømmene hopper over en bildegruppe der et kamera mangler og stopper etter tre på rad; `multi-test` skriver ut en rad per utløser, `(no frame)` der et kamera ikke leverte; `align-stream` avslutter også `1` ved en jobbfeil. Hver kjøring setter deretter hvert kamera tilbake slik det fant det: dets utløser- og synkroniseringslinjekabling, dets PTP (`PtpEnable`) og alle kamerainnstillinger som kjøringen endret — eksponering og forsterkning, AE-mål, grenser og demping, hvitbalanse, svartnivå, pikselformat, binning og ROI, speilvendinger, bildefrekvens — med mindre `--keep-settings` / `--clear-settings` (se [Kameraminne](#camera-memory----keep-settings----clear-settings)). Tidligere beholdt disse kommandoene alle bilder som kom inn og avsluttet `0` (`align-stream` ga en jobbfeil som `Note:`), og lot hvert kamera forbli i `TriggerMode` `On`. De satte også hvert kameras eksponering fra scenen før de lagret et bilde (høyst noen få sekunder), på målet starter en tilkoblet matrise hvert kamera på: 40 multispektralt, 60 RGB (`--brightness N` angir dette), og beholder disse innstillingene gjennom hele opptaket; det samme gjelder et enkelt `lattice capture` for et kamera som Chloros ikke styrer. Tidligere ble hvert kameras første bilde tatt ved kameraets egen eksponeringsgrense, ca. 100 ms, uansett lysforhold, og ble ofte klippet. Se [Flerkamera (midlertidige økter)](#multi-camera-transient-sessions).
> - **Midlertidige flerkamerakommandoer deler nettverket på samme måte som `array-connect` gjør, og åpner kun `--serials`.** Hvert kamera tilpasses sin andel av vertsforbindelsen, slik at fire kameraer kjører på en 1 Gbps-forbindelse; tidligere ba de fire om 125 MB/s til sammen, alt på 1 GbE, og hvert bilde mistet pakker. En konfigurasjon som nettverket ikke kan håndtere, avvises før noen kameraer begynner å streame. `--serials` velger nå kameraene, der det første utløser kommandoen; tidligere åpnet hver kommando alle kameraene på nettverket, uansett hva `--serials` sa. En feil som skyldes GVSP-pakketap, angir nå dette i stedet for å peke på M8-synkroniseringskabelen. Se [Flerkamera (midlertidige økter)](#multi-camera-transient-sessions).
> - **`lattice array-disconnect` lar kameraene kjøre fritt, slik de ble funnet.** `TriggerMode` `Off`, og alt annet slik Connect fant det – innstillingene for utløser og synkroniseringslinje (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`), PTP og kamerainnstillingene som ble endret under økten – slik at en enkeltkamera-kommando på et tidligere medlem henter inn bilder på en gang, og masteren blir ikke lenger stående på `TriggerSource` `Software`. `--keep-settings` lagrer sesjonens innstillinger i hvert kameras minne, mens `--clear-settings` tilbakestiller dette minnet (se [Kameraminne](#camera-memory----keep-settings----clear-settings)). `lattice continuous` setter også et kamera det finner i aktivert tilstand til frikjøring for løpet (og setter det tilbake etterpå), og et løp uten ramme avsluttes med `1`.
> - **`lattice configure --get` / `lattice exposure`** skriver ut en `note:` med kameraetsfaktiske eksponering eller forsterkning mens automatisk modus er aktiv. Et avvist `lattice exposure --set` / `configure --set` angir området kameraet aksepterer og setter automatisk modus tilbake i stedet for å la automatisk eksponering være deaktivert. Se [Sensorjustering](#sensor-tuning).
> - **Nye kamerakontroller:**`lattice highlights` (trekker lyse hvite overflater ned i RGB-visningen) og `lattice exposure --brightness-auto`.**`lattice multi-detect`** finner de kablede kameraene i én sondering og setter masteren til RGB-kameraet som standard.
> - **Nye DAQ-kommandoer:** `daq pool-bind`; `pool-enable-ptp`; og for en DAQ-E-enhets egne innstillinger: `pool-info`, `pool-set-network`, `pool-set-auto-stream`, `pool-reboot` og `pool-update-firmware`. Se [DAQ-E-enhetsinnstillinger](#daq-e-device-settings).
> - **En DAQ-E som slutter å levere data, kobles til igjen av backenden.** Når strømmen fra en DAQ-E i en pool blir stille (for eksempel ved en strømavbrudd i bryteren som forsyner den med strøm, en brutt kamerakobling), kobler backenden den til igjen med de innstillingene den hadde, og prøver på nytt med stadig lengre intervaller mens den er borte. En opptak på den fortsetter i samme `.daq` med gapet merket i filen, og `pool-record --stop` viser hvert avbrudd. I versjon 1.2.0 forble sensoren stille helt til `pool-disconnect` + `pool-connect`, noe som også avsluttet opptaket. Se [En DAQ-E som slutter å levere](#a-daq-e-that-stops-delivering).
> - **`time-sync status`** rapporterer om PTP-grandmasterens pakker når nettverket (`health`), med en `WARNING:` på stderr når de ikke gjør det. Se [`time-sync`](#chloros-cli-time-sync).
> - **Linux .deb:** den øker størrelsen på kjernens mottaksbuffere for sokler som kamerastrømmer trenger (med kjernens standardinnstilling går det tapt bilder og bildefrekvens), legger til en backend-tjeneste som kjører som din bruker, og `lattice network` sjekker bufferne. CLI-en stopper aldri en backend som en tjeneste kjører, og en oppgradering starter de aktiverte backend-tjenestene på nytt. Se [Kjøre backenden som en tjeneste](#running-the-backend-as-a-service-headless) og [Linux-verter — mottaksbuffere for sokler](#linux-hosts--socket-receive-buffers).
> - **`lattice array-capture` angir når det ikke var mulig å justere:**hver årsak til at en opptak ble lagret uten justering (ingen justeringsprofil, elementer med ulik oppløsning, en matrise som ikke er kabelsynkronisert, en ramme som profilen ikke kan tilordnes til) vises som en `Alignment:`-linje.**`chloros-cli daq help`** er `daq --help` (den avsluttet `2`), og på Linux viser hjelpen en liste over de direkte maskinvarekommandoene som denne builden inneholder.
> - **Jetson:** pakken er `chloros_1.3.0_arm64_jp6.deb`, for JetPack 6, og GPU-behandlingen kjører på Orin med JetPacks CUDA PyTorch. Se [Linux arm64 — Jetson](#linux-arm64--jetson-jetpack-6).

---

## Konvensjoner

- Alle kommandoer har prefikset `chloros-cli`. På Windows er binærfilen `chloros-cli.exe`; på Linux/Jetson er den `chloros-cli`.
- Valgfrie argumenter vises som `--flag`. Obligatoriske posisjonsargumenter vises uten parenteser.
- Der det er angitt en standardverdi, vil denne verdien brukes dersom flagget utelates.
- CLI-en er en enkel HTTP-klient over Chloros-backenden (serveren på `127.0.0.1:5000` som også skrivebordsappen bruker). **Hvilke kommandoer starter den:**
  - Kjerne-kommandoene — `process`, `login`, `logout`, `status`, `export-status`, `time-sync` og `selftest` — bruker en kjørende backend, eller starter en når ingen kjører, og stopper den igjen når de er ferdige.
  - **Kommandoene `lattice`, `project` og `daq` starter aldri en.** De trenger en backend som allerede kjører: den fra Windows-skrivebordsappen, Linux-[backend-tjenesten](#running-the-backend-as-a-service-headless) eller en du starter selv (se [Start backenden](#start-the-backend)). Uten en slik viser de feilmeldingen `Error: Chloros backend not reachable at http://127.0.0.1:5000`, sammen med kommandoen som starter den, og avsluttes med `1`.
- `CHLOROS_BACKEND_URL=<url>` peker på **`lattice`**,**`project`**og**`daq pool-*`** til en annen backend (`lattice network-analysis` bruker i stedet `--backend-url`). Kjerne-kommandoene låser bevisst `http://127.0.0.1:<port>` og ignorerer den (IPv4-literalet unngår Windows’ `localhost`→`::1`-straff på ca. 2 sekunder per forespørsel). Se [Miljøvariabler](#environment-variables).
- **Relative baner løses i forhold til mappen du kjører `chloros-cli` fra.** Backenden er prosessen som åpner utgangs- og inngangsmapper, så før en bane sendes, gjør CLI-en den absolutt: `-o` / `--output` (inkludert standardverdien, `output`), `--input`, `--burst-dir`, `--profile`, `--save` og `--mask`, en [oppskrift](#recipe-dsl)-trinnets `output` (standard `output`), og standardmappen `output` tilhører `lattice index --live`. `-o captures/` skriver derfor til `<current folder>/captures/`, og de utskrevne stiene er absolutte. `~` utvides. Mot en ekstern backend (`CHLOROS_BACKEND_URL` som angir en annen maskin) sendes banene slik de er skrevet inn og løses opp på den maskinen.
- En Chloros+-konto er påkrevd for alle SDK/CLI -anrop (kjør `chloros-cli login` én gang per maskin; lagres i `~/.chloros/`).
- Eksemplene bruker Linux-stier; på Windows erstatter du `/home/user/...` med `C:/Users/.../...`. En DAQ-U'serielle port er av typen `COM3` på Windows og `/dev/ttyUSB0` / `/dev/ttyACM0` på Linux.
- Oversiktene angir flagg som utelukker hverandre som `{--a | --b}` og valgfrie flagg som `[...]`.

---

## Oversikt på øverste nivå

```
chloros-cli [global options] COMMAND [command options]
```

### Globale alternativer

| Flagg | Beskrivelse |
| --- | --- |
| `--backend-exe PATH` | Overstyr den automatisk oppdagede kjørbare filen for backend. |
| `--port N` | Backend-HTTP-port (standard: `5000`). Brukes kun av kjerne-kommandoene (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`); `lattice`, `project` og `daq` bruker `CHLOROS_BACKEND_URL` (standard `http://127.0.0.1:5000`). |
| `-v, --verbose` | Aktiver detaljert utdata. |
| `--restart` | Tving omstart av backenden (avslutter eventuelle kjørende `backend_server.py`). En backend som kjøres av en systemd-tjeneste (Linux .deb) stoppes aldri: kommandoen skriver ut `sudo systemctl restart <unit>` i stedet og avsluttes `1`. |
| `--version` | Skriv ut versjon (`Chloros CLI 1.3.0`). |
| `--help` | Vis hjelp på øverste nivå. |

### Kommandoindeks

| Kommando | Formål |
| --- | --- |
| [`process`](#chloros-cli-process) | Behandle en mappe med Survey3- eller «LATTICE»-opptak fra start til slutt. |
| [`login`](#chloros-cli-login) | Autentiser denne maskinen med en Chloros+-konto. |
| [`logout`](#chloros-cli-logout) | Tøm bufret påloggingsinformasjon. |
| [`status`](#chloros-cli-status) | Vis gjeldende lisens-/autentiseringsstatus. |
| [`export-status`](#chloros-cli-export-status) | Eksportfremdrift for Live Thread-4 under en `process`-kjøring. |
| [`language`](#chloros-cli-language) | Angi eller vis CLI-visningsspråk (38 støttes). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Standard prosjektmappe (delt med GUI). |
| [`update`](#chloros-cli-update) | Sjekk etter og installer Chloros-oppdateringer (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | Systemdiagnostikk + røykprøver. |
| [`time-sync`](#chloros-cli-time-sync) | PTP-grandmaster-status / -kontroll. |
| [`lattice`](#chloros-cli-lattice) | LATTICE-kamerakontroll og -opptak (45+ underkommandoer). |
| [`daq`](#chloros-cli-daq) | DAQ-spektralsensorstyring (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Åpne og kjør et lagret Chloros-prosjekt (kameraer + DAQ-er). |

---

## Installasjon

`chloros-cli` leveres som en del av Chloros-installasjonsprogrammet for skrivebordet på alle støttede plattformer — det finnes ingen separat nedlasting for CLI. Når du installerer plattformpakken, legges `chloros-cli` til i `PATH` sammen med skrivebordsappen og backend-binærfilen den styrer.

Siste nedlastinger: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Installasjonsprogrammet inneholder også praktiske startskript (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`) som åpner et bruksklart CLI-skall.

### Windows (.exe)

1. Last ned Windows-installasjonsprogrammet fra nedlastingssiden.
2. Kjør `Chloros Installer 1.3.0.exe` og følg veiviseren. Standard installasjonssti er `C:\Program Files\Chloros\` (CLI-en havner i `C:\Program Files\Chloros\cli\`, som installasjonsprogrammet legger til i PATH).
3. Åpne en ny terminal (`cmd.exe`, PowerShell eller Windows-terminalen) slik at den oppdaterte `PATH` blir hentet.

```powershell
chloros-cli --version
```

Installasjonsprogrammet legger automatisk til `chloros-cli.exe` i systemmappen din `PATH` og inkluderer LATTICE-kameradriverne. Backenden er `C:\Program Files\Chloros\backend\chloros-backend.exe` (mappen `backend` i installasjonsmappen din), som ikke finnes på `PATH`.

**Systeminnstillinger.** Installasjonsprogrammets side *Velg tilleggsoppgaver* har to alternativer under **Systeminnstillinger**, begge**merket av som standard**:

| Alternativ (som angitt i installasjonsprogrammet) | Hva det gjør | Avslå stille installasjon |
| --- | --- | --- |
| **Aktiver lange stier i Windows** (lar prosjektmapper ligge dypere enn 260 tegn; anbefalt) | Setter Windows' maskinomfattende innstilling `LongPathsEnabled` til 1. Windows leveres med denne innstillingen deaktivert, og da kan ingen sti være lengre enn 260 tegn; Chloros-produkter ligger opptil ca. 130 tegn under prosjektmappen, så en dypere prosjektmappe avvises (se [Feilsøking](#feilsøkings-tips)). Trer i kraft for programmer som startes etter installasjonen. Forblir aktivert ved avinstallering, fordi andre programmer kan være avhengige av det. | `/MERGETASKS="!longpaths"` |
| **Ekskluder Chloros-backenden fra Microsoft Defender-skanning**(mye raskere kameraopptak; anbefalt) | Legger til et Defender-unntak for**prosessen** `chloros-backend.exe` i installasjonsmappen, slik at sanntidsbeskyttelsen ikke skanner hver fil som backenden skriver, uansett hvor den skriver den. Ingen andre programmer ekskluderes; ekskluderingen fjernes ved avinstallering. Der Defender mangler eller er låst av retningslinjer, mislykkes trinnet uten varsel, og installasjonen fortsetter. | `/MERGETASKS="!defenderexclusion"` |

**Hvorfor Defender-alternativet er viktig:** ved høye opptakshastigheter koster sanntidsskanning av opptaksfilene omtrent 30 % av opptakshastigheten (målt ved kontinuerlig opptak med 9 kameraer: 77,7 opptak/s ekskludert, 52,7 skannet). `lattice array-capture --continuous` viser en melding når det måler at Defender bruker én eller flere CPU-kjerner under opptaket; kjør installasjonsprogrammet på nytt med alternativet merket av.

En stille installasjon (`/SILENT` eller `/VERYSILENT`) bruker begge deler med mindre det velges bort, f.eks. `"Chloros Installer 1.3.0.exe" /VERYSILENT /MERGETASKS="!defenderexclusion"`.

**Windows-brannmur.** Installasjonsprogrammet legger alltid til to innkommende regler for `chloros-backend.exe` i hver nettverksprofil, og avinstallasjonsprogrammet fjerner dem: `Chloros Backend (UDP-In)`, for kamerastrømmer, PTP, mDNS og DAQ-E-datastrømmen, samt `Chloros Backend (TCP-In)`, begrenset til det lokale subnettet, for en DAQ-E-firmwareoppdatering, der sensoren kobler seg tilbake til denne datamaskinen for å hente bildet (se [DAQ-E-enhetsinnstillinger](#daq-e-device-settings)). Backendens HTTP-API forblir bundet til `127.0.0.1`.

### Linux amd64 (.deb)

For Ubuntu 22.04 LTS eller nyere / Debian-baserte x86_64-arbeidsstasjoner.

> **Ubuntu 20.04 støttes ikke.** Pakkens avhengighetsliste er avledet fra
> hva backenden faktisk kobler seg mot, og det inkluderer `libc6 (>= 2.35)`;
> focal leverer glibc 2.31. `apt` avviser installasjonen i stedet for å la den mislykkes under
> kjøring. Listen nevner også `libgl1`, som den medfølgende OpenCV-pakken laster inn og som en minimal
> serverimage kan mangle; `apt install` inkluderer den.

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
```

.deb-filen installerer:
- `chloros-cli` til `/usr/bin/chloros-cli`, og `chloros-backend` (starter backenden) til `/usr/bin/chloros-backend`
- Den kompilerte backenden og CLI-en, utpakket, i `/usr/lib/chloros/backend/` og `/usr/lib/chloros/cli/`
- LATTICE-kameradriverne
- Støyreduksjonsmodeller, kalibreringspakker og konfigurasjon for oppdateringskanalen
- Python-SDK-et for systemet `python3` (`import chloros_sdk`). `apt install` har også sine egne krav, `python3-numpy` og `python3-requests` (anbefalt, ikke påkrevd: programmene ovenfor er selvstendige; `dpkg -i` hopper over dem, og installasjonsutdataene angir hva som mangler). I et virtualenv: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`
– `/etc/sysctl.d/60-chloros-gige.conf`, som øker kjernens mottaksbuffere for kamera strømmer til 32 MB, og bruker dem umiddelbart (en verdi som allerede er satt høyere, forblir uendret). Når pakken fjernes, fjernes også filen. Se [Linux-verter — mottaksbuffere for sokler](#linux-hosts--socket-receive-buffers).
- To valgfrie systemd-tjenester for backenden, ingen av dem er aktivert (neste avsnitt)

#### Kjøre backenden som en tjeneste (headless)

Uten en tjeneste kjører backenden (og med den PTP-grandmasteren) bare så lenge noe har startet den: `chloros-backend` i en terminal, et SDK-skript eller en core-kommando `chloros-cli` så lenge den kjører. For en backend som alltid er på en maskin uten grafisk grensesnitt, aktiver tjenesten **som din bruker**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` kjører backenden som den brukeren. Opptakene den skriver tilhører denne brukeren, den detaljerte loggen finnes i vedkommendes `~/.cache/chloros/logs`, og den bruker dennes `chloros-cli login` (`~/.chloros`), så logg inn én gang som den brukeren først. PTP trenger ikke root-rettigheter. Som alle installerte backender lytter den kun på `127.0.0.1:5000`.

`chloros-backend.service` kjører backenden som **root**. Den er fortsatt installert slik at en maskin som har aktivert den, fortsetter å kjøre, men alt en root-backend skriver tilhører root, så en bruker kan ikke slette sine egne opptak. For å bytte til din bruker:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

Den detaljerte loggen for root-tjenesten finnes i `/var/log/chloros`, og kan leses av medlemmer av gruppen `adm` (tjenesten setter `CHLOROS_LOG_DIR`; se [Miljøvariabler](#environment-variables)). Når du installerer pakken, vises kommandoen `enable` med ditt brukernavn fylt inn, samt bryteren når root-tjenesten er aktivert.

**Aktiver én backend-enhet.** Bare én backend kjører om gangen. En andre aktivert enhet (`chloros-backend.service` ved siden av `chloros-backend@<user>.service`, eller to brukeres `@`-enheter) nekter å starte og forblir som `failed` med avslutningsstatus 98; systemd starter den ikke på nytt. Dens `systemctl status` forklarer hvorfor: `Another instance is already running. Exiting.`, deretter hva som holder på backend-enheten (port 5000, eller låsefilen og brukeren den tilhører), deretter `systemd will not restart <unit> (exit status 98)`. Etter en omstart kan begge enhetene være den som kjører. For å bytte, deaktiver den du ikke bruker, som beskrevet ovenfor.**CLI-en stopper aldri en tjenestes backend.** `chloros-cli` bruker den som den er, slik GUI-en og SDK-en gjør: `process` kjører på den og laster inn sitt eget prosjekt i stedet for det forrige. Der CLI-en ellers ville ha startet backenden på nytt (`--restart`, eller en `process` der importen ikke fant noe), navngir den enheten, skriver ut `sudo systemctl restart <unit>` og avslutter `1`, fordi en backend som stoppes på den måten, vil forbli nede. `GET /api/status` navngir enheten som `service` (`null` for en backend der ingen tjeneste kjører).**Oppgradering og fjerning.** Oppgradering av pakken stopper alle backend-tjenester, og starter deretter hver enkelt som er aktivert på nytt, med `@<user>`-enhetene først (en som er startet manuelt, men aldri aktivert, forblir stoppet, som etter en omstart). Hvis flere enn én er aktivert, kjører bare én, mens de andre forblir i feiltilstand som beskrevet ovenfor; oppgraderingen viser meldingene `Chloros: N backend units are enabled: …` og `Keep one enabled: sudo systemctl disable --now <unit>`. Når du fjerner pakken, stoppes og deaktiveres tjenestene. Hvis du sletter den fullstendig, slettes også `/var/log/chloros`.

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Samme oppsett som amd64 .deb, inkludert mottaksbuffere og [backend-tjenester](#running-the-backend-as-a-service-headless), for Jetson Orin / Orin NX / Orin Nano på **JetPack 6** (L4T R36). PyTorch-versjonen er JetPack 6s CUDA-bygg for Orin, så GPU-behandling, inkludert den teksturbevisst debayeren, kjører på Jetsons GPU. Pakken inneholder ikke CUDA i seg selv: den bruker CUDA- og cuDNN-bibliotekene som JetPack 6 installerer (`sudo apt install nvidia-jetpack` på et bilde som mangler dem). Der det ikke er noen CUDA-GPU tilgjengelig, kjører behandlingen på CPU-en, og teksturbehandlingen bruker standard debayer (se [`process`](#chloros-cli-process)).

### Autentisering én gang per maskin

Hver plattform krever en engangsinnlogging i Chloros+ før SDK- og CLI-kall fungerer:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Påloggingsopplysningene lagres i `~/.chloros/user_session.json`.

### Start backend

Kommandoene `lattice`, `project` og `daq` kommuniserer med en backend som allerede kjører; de starter aldri en (se [Konvensjoner](#konvensjoner)). La en kjøre mens du bruker dem:

| Plattform | Hvordan |
| --- | --- |
| Windows | Åpne Chloros-skrivebordsappen (backenden til denne betjener også CLI), eller kjør `"C:\Program Files\Chloros\backend\chloros-backend.exe"` i en egen terminal (mappen `backend` i installasjonsmappen din) og la den stå åpen. |
| Linux/Jetson | Kjør `chloros-backend` i en egen terminal, eller hold en alltid i gang med [backend-tjenesten](#running-the-backend-as-a-service-headless). |

Det kjører bare én backend om gangen; alle klienter (GUI, CLI, SDK-skript) deler den. Kjerne-kommandoene (`process`, `login`, `status`, …) bruker den også, og starter en når ingen kjører.

**En backend fortsetter å kjøre mellom kommandoer**, slik at den neste ikke må starte den opp på nytt. Den stopper når du logger ut (`chloros-cli logout`, med mindre Chloros-appen er åpen eller en kjøring eller en offline-bygging pågår), når du kjører `chloros-cli backend stop`, eller av seg selv etter 30 minutter uten at noe bruker den: ingen kommando, skript eller app, ingen kjøring eller offline-bygging, ingen tilkoblet kamera eller lyssensor. `backend stop` avviser mens en kjøring eller en offline-bygging pågår, og oppgir årsaken; `--force` stopper den umiddelbart, og den kjøringen eller byggingen går tapt (en burst beholdes, og `array-build-video` bygger den på nytt). Sett `CHLOROS_BACKEND_IDLE_MIN` før backenden starter for å endre de 30 minuttene (`0`: aldri); [backend-tjenesten](#running-the-backend-as-a-service-headless) stopper aldri av seg selv. `chloros-cli backend status` viser om en kjører, hva som bruker den og eventuelle offline-bygg som kjører. `process` åpner prosjektet sitt i den kjørende backenden, med mindre Chloros-appen er åpen med et prosjekt den har åpnet: da nekter den, og du må enten fortsette fra appen eller lukke det prosjektet først. Et prosjekt som en tidligere `process` eller et SDK-skript har latt stå åpent, blir ganske enkelt erstattet.

### Bekreft installasjonen

```bash
chloros-cli --version           # prints "Chloros CLI 1.3.0"
chloros-cli selftest            # 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

`selftest` avsluttes (`0`) først når alle syv sjekkene er bestått. Den siste krever en CUDA-GPU (en NVIDIA-GPU, eller en Jetson Orin) og støyreduksjonsmodellene, så på en datamaskin uten en slik vises `Failed: CUDA + Denoiser` og programmet avsluttes med `1`; alt unntatt den GPU-avhengige teksturbevisst debayeren fungerer fortsatt der.

> **Chloros+-abonnement kreves.**CLI krever et aktivt Chloros+-abonnement.**Copper**er det laveste Chloros+-nivået – alle betalte Chloros+-nivåer har tilgang til CLI/SDK; kun det gratis**Iron**-nivået har ikke det. (Abonnements-ID-kart: `0`=Iron/gratis, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Oppgrader ved [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Denne nedre grensen håndheves av backend-systemet, ikke bare av CLI: en forespørsel med SDK/CLI-flagg uten et betalt abonnement avvises med feilkoden `403 PLAN_UPGRADE_REQUIRED`, uansett om den kommer fra `chloros-cli`, Python-SDK-et eller en håndkodet HTTP-klient. En utlogget bruker får i stedet feilmeldingen `401 AUTH_REQUIRED`. Tilgang fungerer offline i abonnementets overgangsperiode (30 dager for månedsabonnement, frem til utløpet for årsabonnement) og opphører når denne utløper; `chloros-cli status` fortsetter å fungere slik at årsaken er synlig (det er den eneste SDK-/CLI-ruten som er unntatt fra nivåbegrensningen — `GET /api/license-status`).

---

## Fem minutters hurtigstart

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI GNDVI OSAVI

# 3. The lattice / daq commands need a running backend: on Windows open the
#    Chloros desktop app; on Linux start one (or see "Start the Backend")
chloros-backend &

# 4. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 5. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 6. Read a spectrum from a connected DAQ-U (Windows: --port COM3)
chloros-cli daq pool-connect --port /dev/ttyUSB0
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

`process` (trinn 2) starter en backend for sin egen kjøring når ingen er i gang, og stopper den når den er ferdig, så start en for trinn 4 og videre. Se [Start backend](#start-the-backend).

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
| `INPUT` | Sti til inndatamappe som inneholder `.raw + .jpg`- (Survey3), `.tif/.tiff`- (LATTICE) eller `.dng`-filer. |

### Vanlige alternativer

| Flagg | Standard | Beskrivelse |
| --- | --- | --- |
| `-o, --output PATH` | en ny mappe med navnet `-n` i standard prosjektmappen | Prosjektmappen. Prosjektet får navn etter denne, og en mappe som allerede inneholder en `project.json`, får en `_1` / `_2` i stedet for å bli overskrevet. Standard prosjektmappe er den som `set-project-folder` (eller GUI-en) valgte, ellers `Documents/MAPIR/Chloros Projects`. |
| `-n, --project-name NAME` | auto (tidsstempel) | Prosjektnavn, brukes kun **uten** `-o` (med `-o` får prosjektet navnet på mappen). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` bruker en «Chloros+» nevral debayer for Survey3-bilder; tregere, men med høyere kvalitet. Dette gjelder kun for Survey3: LATTICE-opptak bruker alltid standard demosaik, så for en mappe som inneholder LATTICE-opptak, avviser `process` `texture-aware` etter importen, med en feilmelding som angir hvor mange det er, og avslutter `1` før noe blir behandlet (Chloros-appen tilbyr heller ikke «Texture Aware» for disse). Det krever en CUDA-GPU (en NVIDIA-GPU på Windows/Linux x64, Orins GPU på en Jetson): uten en slik bruker kjøringen `standard`, og backenden logger én `[DEBAYER] … needs a CUDA GPU …`-linje som forklarer hvorfor (vist med `--verbose`). |
| `--vignette / --no-vignette` | `--vignette` | Vignettkorreksjon. |
| `--reflectance / --no-reflectance` | `--reflectance` | Refleksjonskalibrering (bruker panelmål hvis det finnes, NIST-kalibrering per serie for LATTICE). For LATTICE multispektral fungerer dette også som **produktet** for refleksjonskalibrering — se [Eksportbrytere per produkt](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | av | Bruk PPK GNSS-korreksjoner fra sidecar-filer. |
| `--exposure-pin-1 CAMERA_MODEL` | ingen | Kameramodellen koblet til eksponeringspinne 1, for PPK-tidssynkronisering. |
| `--exposure-pin-2 CAMERA_MODEL` | ingen | Kameramodellen som er koblet til eksponeringspinne 2, for PPK-tidssynkronisering. |
| `--recal-interval SECONDS` | `0` | Minste rekalibreringsintervall, i sekunder (prosjektets *Minste rekalibreringsintervall*). |
| `--timezone-offset HOURS` | `0` | Lyssensorens tidssoneforskjell fra UTC, i timer (prosjektets *Tidssoneforskjell for lyssensor*). |
| `--format FORMAT` | `TIFF (16-bit)` | En av `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros øker aldri et bildes bitdybde: på bilder som utelukkende består av 8-bit, skrives et format med større dybde som `TIFF (8-bit)`, og dette nevnes én gang i beskrivelsen. I et prosjekt som blander 8-bits og 12/16-bits opptak, eksporteres hvert opptak med sin egen dybde eller lavere. Radiance er alltid 32-bits flytende. |
| `--indices NAME [NAME ...]` | ingen | Vegetasjonsindekser etter forhåndsinnstilt navn (`NDVI`, `GNDVI`, `OSAVI`, `SAVI`, `EVI`, `GCI`, …), uten skille mellom store og små bokstaver. Hver av dem legges til i prosjektet for kameraets bånd, og et nytt prosjekt inkluderes. Et navn som ikke er en forhåndsinnstilling, eller som kameraets bånd ikke kan danne (`NDRE` krever RE og NIR, noe ingen enkeltkamera tilbyr), blir nevnt i en advarsel og får ikke noe indeksbilde. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Tving innføringspunktet i behandlingskjeden for LATTICE TIFF-filer (Survey3.raw påvirkes ikke). Også nødutgangen som gjør at en opptaksserie uten **raw** kan behandles i det hele tatt — se [Hvordan en opptaksmappe ser ut](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | på | Send ut det lineære debayered-produktet (`Debayered_Images`). Se [Eksportbrytere per produkt](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | på | Sender ut forhåndsvisningen (`Preview_Images`): RGB = hvitbalanse (DAQ-lysstyrke når tilgjengelig, ellers gråverden) + gamma; multispektral = strekking til falske farger. |
| `--radiance / --no-radiance` | på | Send ut float32-strålingsintensitet (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referanse for LATTICE-refleksjonsproduktet: `auto` = QA-godkjent mål i rammen er den absolutte referansen (et mål med et QA-flagg konkurrerer med DAQ om målt poengsum), DAQ-nedstråling (ρ = π·L/E) som reserve; `target` = strengt (ingen DAQ-erstatning); `daq` = DAQ-autoritativ. Se [Eksportbrytere per produkt](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | ingen | Katalog over **målte** målrefleksjonsskanninger per enhet (`<serial>.csv`); faller tilbake til de nominelle T3/T4P-spektrene ved manglende data. |
| `--array-alignment / --no-array-alignment` | på | LATTICE-matriser: anvend modul-til-modul-justeringen som er stemplet i hver opptakets `Chloros:Alignment*` XMP på hvert bearbeidet produkt (debayered / forhåndsvisning / radiance / reflektans / indeks). Ingen operasjon for bilder uten taggene. |
| `--array-alignment-crop / --no-array-alignment-crop` | beskjæring | Beskjær justerte eksporter til matrisens felles overlappingsområde, slik at alle moduler deler ett fotavtrykk; `--no-…` beholder hele sensorområdet (svart utfylling utenfor kilden). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Resampling for justeringsforvrengningen. `nearest` bevarer eksakte kilde-DN-er (ingen blanding av radiometriske verdier mellom piksler). |

### Alternativer for måldeteksjon

| Flagg | Beskrivelse |
| --- | --- |
| `--min-target-size PIXELS` | Minste størrelse på kalibreringsmål i piksler for detektoren. |
| `--target-clustering 0-100` | Prosentandel for målgruppering (standard `60`). |
| `--target / --targets` | Søk kun etter kalibreringsmål i en `target`- eller `targets`-undermappe i inndataene (raskere). |

### Eksempler

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI GNDVI OSAVI

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

# Save to a project folder of your choice (the project is named after it)
chloros-cli process "C:/input" -o "C:/projects/Field_A_2026-05-26"

# Name the project and let it go to the default project folder
chloros-cli process "C:/input" -n "Field_A_2026-05-26"
```

### Eksportbrytere per produkt (LATTICE multispektral)

LATTICE-behandlingen fordeler seg på **alle aktuelle produkter i én omgang**. De fire bryterne per type — `--debayered`, `--preview`, `--radiance`, `--reflectance` — er alle**slått PÅ som standard**; bruk skjemaet `--no-<type>` for å deaktivere en. RGB-hovedkameraer sender kun ut debayert + forhåndsvisning (ingen stråling/refleksjon per bånd), så `--radiance`/`--reflectance` har ingen effekt for dem. Bryterne ignoreres for Survey3 `.raw` (som følger standardbanen for reflektans/mål). *(Det gamle `--radiometric-output {reflectance,radiance,sensor-response}`-flagget ble **fjernet** og erstattet av disse bryterne; det finnes ikke lenger noe `sensor-response`-nivå.)*

| Produkt | Utdata | Kreves DAQ-nedstråling? |
| --- | --- | --- |
| `--debayered` | Lineær demosaik (`Debayered_Images`): den samme filen som en opptak lagrer på sitt `debayered`-nivå — kalibreringspakkens mørkeforskyvning (DSNU) fra rådataene, deretter OpenCVs kantbevisste demosaik; ingen linearitets-LUT, flatfelt eller unmix. | Nei. |
| `--preview` | Forhåndsvisning på skjerm (`Preview_Images`): RGB = WB + gamma; multispektral = strekking til falske farger. | Nei. |
| `--radiance` | float32 W/m²/sr/nm fra den fullstendige radiometriske kjeden (`Radiance_Images`). | Nr. |
| `--reflectance` | uint16 reflektans ρ (`32768` = 1,0), Pix4D-klar. | **Ja**, med mindre den forankres av et QA-godkjent mål innenfor rammen (se nedenfor). |

`--reflectance-source` velger reflektansreferansen:**`auto`**(standard) gjør et QA-godkjent mål i bildet til den**absolutte referansen**— de målforankrede empiriske linjekjedene kryssvurderes på utelatte paneler, og den målte vinneren brukes, bånd for bånd: et bånd som DAQ ikke kan kalibrere, eller hvis DAQ-hellingstilpasning mislykkes, tar målets egen linje (`ReflectanceElBandMethods` angir hvilken) — et mål som består QA med en feilmelding (et utelatt panel, et bånd som ikke fungerer) konkurrerer også med DAQ-delingsverdien på den målte poengsummen, og en ramme det taper mot blir merket `ReflectanceChain=daq` — faller tilbake til DAQ-nedstrømningsgrensen (ρ = π·L/E) når det ikke er noe mål til stede eller QA mislykkes;**`target`**er strengt (ingen DAQ-erstatning);**`daq`**velger bort til den DAQ-autoritative oppførselen. Målgeometri (ArUco / fast ROI / stripe) kommer fra prosjektets målkonfigurasjon; `--target-reflectance-dir DIR` lagrer**målte** skanninger (`<serial>.csv`) som slås opp etter målenhetens serienummer/QR, med de nominelle T3/T4P-spektrene som reserve.

DAQ-refleksjonsbanen løser **tidsstempel-matchet nedadgående**automatisk fra en registrert**`.daq`**(DAQ-U/M/E)**eller en DAQ-M-spesifikk `.csv`**som finnes sammen med bildene. Hvis en kalibreringspakke per kamera eller DAQ ikke er lagret lokalt,**henter rørledningen den automatisk fra AWS** ved første bruk (krever internett én gang; lagres under `~/.chloros/`).**Hvem sin måling er det?**En `.daq` som en opptak lagrer, er målingen fra kameraet/kameraene den beskriver — navngitt i en `<file>.daq.cameras.json`-sidecar ved siden av den (`lattice capture`, `multi-capture`, `array-capture` og GUI-en; en rå bildeserie oppgir hvert medlems avlesning i manifestet) — og `process` samsvarer**kun med bilder fra disse kameraene**. Et frittstående kamera uten egne måleverdier låner derfor ikke lenger et annet kameras DAQ; knytt en DAQ til det, eller ta opp en. En DAQs egen opptak (`daq pool-record`, eller en opptak startet i GUI-en), eller ethvert `.daq` uten den sidecaren (inkludert eldre opptak), betjener fortsatt alle kameraene.

#### Avlesning av reflektanspiksler (Pix4D / Metashape / dine egne skript)

Reflektans lagres som heltallig DN, og **den DN-verdien som tilsvarer ρ = 1,0 avhenger av kildekameraet**:

| Kilde | ρ = 1,0 er | Hvordan se det |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (margin opp til ρ 2,0) | XMP-taggen `Chloros:PixelScale=32768` er stemplet på filen. |
| Survey3 | `65535` (avskåret ved ρ 1,0) | Ingen `Chloros:*` XMP-merker — det er nettopp fraværet *som* er signalet. |

**Les `Chloros:PixelScale` og del med det** i stedet for å anta en konstant. Taggen er definert i uint16-domenet, så den forblir `32768` på tvers av utdataformater som skalerer om — `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)` og `TIFF (32-bit, Percent)` er alle selvbeskrivende (normaliser først den lagrede datatypen tilbake til uint16: ×257 fra 8-bit, ×65535 fra float).

> **Filer fra eldre versjoner kan ikke inneholde noen skalering.** En 8-bits reflektans-TIFF-fil blir skalert om og merket `Chloros:PixelScale` på samme måte som de andre. Versjoner før 1.3.0 *klippet* den i stedet til 0..255, slik at alle verdier over ρ≈0,008 ble flatet til 255, og de utelot både `Chloros:PixelScale` og tuplen `MicaSense:RadiometricCalibration`. **Hvis taggen mangler på en LATTICE-refleksjonsfil, må du ikke anta en skala — behandle den på nytt** i stedet for å dele piksler som aldri var delbare.

#### Lesing av indekspiksler (NDVI og de andre vegetasjonsindeksene)

**Et heltallsindeksraster er en kontrastutvidelse per bilde, ikke et −1…1-kart.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` og `JPG (8-bit)` indeksprodukter – inkludert 8-bits TIFF-filen som en 8-bits opptak skrives som – lagrer

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

der `IndexMin` / `IndexMax` er **den aktuelle rammens egne laveste og høyeste endelige indeksverdier**. DN 0 er ikke −1, og full skala er ikke +1.

Chloros merker begge endepunktene slik at avbildningen forblir inverterbar:

| Tag | Betydning |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — konvensjonen nedenfor |
| `Chloros:IndexMin` | indeksverdien lagret som DN 0 |
| `Chloros:IndexMax` | indeksverdien lagret som full skala |

Les dem og inverter, og normaliser den lagrede DN til 0…1 ved først å bruke den lagrede datatypens maksimumsverdi, slik at ett uttrykk dekker alle heltallsformater:

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

Innelasting krever ikke `-config pix4d.config` — exiftool leser disse XMP-egenskapene uten hjelp. Konfigurasjonen er kun nødvendig for å *skrive* dem.

> **`IndexMin == IndexMax`** betyr at hver piksel i det bildet hadde samme indeksverdi. Rasteret er fylt til midt på skalaen, og det er ingenting å dele med — ta endepunktet.

> **Ingen `Chloros:Index*`-tagger betyr at det ikke er angitt noen tilordning.** En LUT-eksport er en fargelagt gjengivelse der pikslene er gradientfarger tilpasset LUT-terskelvinduet ditt, og et bilde uten piksler med endelig indeks har ikke noe område å angi. Ikke anta at det foreligger en tilordning i noen av tilfellene – samme regel som ved manglende `PixelScale`.**Fordi strekkingen er per bilde, er to bilder fra én flyvning ikke sammenlignbare som DN**— hvert har sine egne endepunkter, så den samme NDVI-verdien havner på forskjellige DN-verdier i forskjellige bilder. For kvantitativt arbeid på tvers av en flyvning (sonestatistikk, faste terskler, tidsserier), eksporter**`TIFF (32-bit, Percent)`**: indeksrastere skrives der som float32**indeksverdiene i seg selv**, uten behov for omskalering eller tagger. Til tross for formatets navn er indeksprodukter ikke ×100 – den skaleringen gjelder kun for reflektans med heltallstypen.

#### Unmix-klampemaske (hvilke piksler et bånd ble klemt på)

Den 3×3 spektrale unmix-prosessen kan føre til at et bånd blir negativt — der lyset knapt inneholder det båndet, eller der scenen faller utenfor det kameraets unmix-matrise beskriver. Dette er ikke en reell verdi, så eksporten setter den til 0 – og livevisningen viser den samme 0. Et bånd kan bli klemt på noen få piksler eller på nesten alle; masken registrerer begge deler. `Chloros:Band{R,G,B}UnmixClampFraction` angir hvor stor del av et bilde dette skjedde på; klemmasken angir **hvilke piksler**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— i en `clamp_masks/`-mappe**ved siden av** produktmappen, aldri inne i den (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`; en eksport fra `process`: `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`) — for hver strålings-, refleksjons- og indeksprodukt der minst én piksel er begrenset: både opptak og eksporter (samt TIFF-filene per bilde). Enkeltbånds, på produktets eget pikselrutenett (inkludert orientering, ROI, binning og matrisejustering). Hver maske angir produktets kamera (`Make` / `Model` / serienummer), opptakstid og objektivkoder, og lister opp båndene som bitene står for (`XMP-Camera:BandName`, bit 0 først); dens `ImageDescription` og `Software` markerer den som en klemmaske, og den inneholder ingen kalibrering.
- **Biter**: bit 0 = båndet i produktets R-kanal (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). En satt bit betyr at pikselverdien i det aktuelle båndet ble klemt til 0 — behandle den, og enhver indeks bygget på den, som lavt pålitelig.
- **`Chloros:UnmixClampMask`** er stemplet på hvert produkt som har kjørt unmix: maskens sti i forhold til produktets egen mappe (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, alltid atskilt med `/` – sett den sammen med bildets mappe for å åpne masken), eller `none` når ingen piksler er begrenset. Ingen merkelapp betyr at produktet aldri har kjørt unmix-prosessen (kameraer med RGB- eller monokromopptak, rådata/debayered-nivåer).
- `lattice capture` / `array-capture` skriver ut maskens sti under den lagrede filen (`clamp mask: …`), og i listen over opptaksresultater vises den som rammens `clamp_mask`.
- `lattice index --live --save-multiband` skriver én maske per kildekamera for multibåndstakken — se [`lattice index` — multibåndstakkmasker](#lattice-index--multiband-stack-masks). En video har ingen maske: `array-build-video` rapporterer i stedet klemfraksjonene for hvert bilde (se [`array-build-video`](#array-build-video--offline-reprocess-a-saved-burst)).

Siden ingen produktmappe inneholder en maske, uansett dybde, kan en produktmappe (`radiance/`, `reflectance/`, en `process`-eksports `*_Images/`-mappe) overføres til Pix4D, Metashape eller annen fotogrammetriprogramvare slik den er — inkludert undermapper — og den tar kun imot bilder. Overfør produktmappen, ikke mappen over den (den inneholder også `clamp_masks/` og alle andre eksporttyper). Chloros’ egen import hopper over maskene. Lesing av en maske: `(mask & 1) != 0` betyr «R-kanalbåndet ble begrenset her», `& 2` G, `& 4` B.

#### EXIF overføres til eksporten

`process` kopierer **GPS-blokken og dens ExifIFD** fra kildeopptaket til hvert produkt, slik at en
eksport inneholder `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` og
`CameraSerialNumber` sammen med georeferansen.

**`FocalLength` er ikke valgfritt for fotogrammetri.** Pix4D beregner bakkeprøveavstanden ut fra
brennvidde pluss høyde; uten denne taggen faller programmet tilbake til en helt feil skala. På en
flyvning over en appelsinlund med 49 opptak forvandlet den manglende taggen et område på 411 m × 160 m til et rekonstruert
47,8 km × 13 km – et ortofoto på 455 MP bestående hovedsakelig av «nodata», som deretter ble tolket som et flisleggingsproblem og
et BigTIFF-problem før noen sjekket GSD-en. Hvis ortofotoet ditt ender opp med en usannsynlig
skala, må du først kjøre `exiftool -FocalLength` på det eksporterte produktet.

Kopien er bevisst **ikke** `-all:all`: IFD0's strukturtagger ødelegger LATTICE-utdata når de
kopieres, og `ExifImageWidth` / `ExifImageHeight` er utelatt fordi de beskriver
*kilde*opptaket – en eksport som noen gang har blitt endret i størrelse, ville ellers ha dimensjoner
som strider mot sin egen raster. XMP skrives direkte i stedet for å kopieres, fordi ExifTool
kasserer XMP-tagger fra samme påkalling når XMP-blokken kopieres (noe som ville fjerne MAPIR-
kalibreringstaggene).

### Hvor utdataene havner

Produktene skrives **i prosjektmappen, gruppert etter kamera og deretter etter filformat**:

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

Kameramappen er `LATT-<sensor>-<lens>-F<filter>` for LATTICE (som samsvarer med opptakets EXIF
`Model`) og `<model>_<filter>` for Survey3 — to kameraer som deler sensor og filter, men som skiller seg
fra hverandre når det gjelder objektiv, har separate trær, fordi vignettering, synsfelt og forvrengning varierer. Formatmappen
følger `--format`: `tiff16`, `tiff8`, `png8`, `jpg8`, eller `tiff32` for
`TIFF (32-bit, Percent)`.

> **Hvert eksportert produkt beholder navnet på KILDE-filen.** En radiance-eksport av
> `capture_…_raw.tif` heter fortsatt `capture_…_raw.tif` — den ligger bare i
> `tiff32/Radiance_Images/`. **Mappen identifiserer produktet, ikke filnavnet**, så hvis du søker
> etter `*radiance*.tif`, finner du ingenting; søk i stedet etter mappen.

### Lyssensoropptak — kalibrert `.daq` + `.csv`

`process` håndterer også `.daq`-opptakene i inndatamappen din, og den **trenger ikke**
trenger noen bilder for å gjøre det: en DAQ-U / DAQ-M / DAQ-E som flys alene, utgjør en fullstendig
opptak, og en mappe som kun inneholder `.daq`-filer, er en gyldig inndata.

En DAQ kan tas opp **uten** kalibrering – det er det de offentlige
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) opptakere
(`record_daq.py`) gjør som standard: de skriver rå sensortall og stempler filen slik at
Chloros henter sensorens fabrikkalibrering **via seriell** (først fra lokal cache,
deretter fra MAPIR-skyen) og bruker den. `process` skriver resultatet tilbake:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv` inneholder én rad per avlesning: UTC-tidsstempel, integrasjonstid, total effekt,
fotopisk/skotopisk lux, PPFD (og dens oppdeling i blått/grønt/rødt), toppbølgelengde, deretter
fullt spektrum på sensorens eget bølgelengderaster. `.daq` importeres på nytt uten å bli
kalibrert en gang til.

`<name>` er opptakets eget filnavn. To opptak i forskjellige mapper kan dele
én – måleverdier per opptak (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) lagret i
samme millisekund av to kameraer – og hvert får likevel sine egne produkter: det første beholder
`<name>`, den neste legger til mappene som skiller den ut **etter** navnet
(`<name>__<capture>_daq_calibrated.daq`), slik at opptakets dato og klokkeslett fremdeles står først.

Ved vellykket gjennomføring rapporterer kjøringen `Light-sensor products written: N (calibrated .daq + .csv)`.
Det som står i parentes beskriver hva som faktisk ble skrevet, så det lyder
`(RAW COUNTS — this sensor has no calibration bundle)` for en sensor uten pakke og
`(N calibrated, M raw counts)` for en mappe som inneholder begge deler. Backendens egne
overskrifter `[DAQ-EXPORT]` og `[RUN-SUMMARY]` har fått sin ordlyd på samme måte — ingen av
de tre kan kalle en rå eksport for kalibrert.

En DAQ-U / DAQ-M / DAQ-E-opptak der kalibreringspakken ikke kan hentes – du er
offline, eller sensoren har ingen kalibrering i filen – blir **hoppes over med en begrunnelse** på en
`[DAQ-EXPORT]`-linje, og skrives aldri ut som en «kalibrert» fil som inneholder rå tellinger.
Koble deg til internett og kjør på nytt. Årsaken er den som leseren faktisk
fastslo for den filen (uleselig skjema, ingen pakke, en skrivefeil), og kjøringssammendraget
viser **distinkte** årsaker – tjue filer som ble hoppet over av én årsak, vises som én
årsak, ikke tjue gjentakelser av den. CLI-en navngir hvert hoppet over opptak på slutten av
kjøringen, med årsaken:

```
⚠ Light-sensor recording not exported: <file>.daq: no calibration bundle available for …
```

En mappe med opptak som ikke eksporterer **noen** av dem, mislykkes: se
[En kjøring som ikke skriver noen bilder, mislykkes](#a-run-that-writes-no-images-fails).

En opptak hvis navn ender på `_calibrated` eller `_raw` (`field_raw.daq`) er **ikke
importert**: dette er navnene `process` gir sine egne produkter, og et produkt som leses tilbake
som en opptak vil bli telt to ganger. CLI-en opplyser dette og oppgir filnavnet —
`Not imported as a light-sensor recording: field_raw.daq … Rename the file to import it.`
— i stedet for å rapportere en tom mappe. Gi et slikt opptak nytt navn før du behandler det.

#### DAQ-A-opptak eksporteres som rå tellinger

**DAQ-A** -familien er eldre enn systemet med serienummerbaserte pakker og har ingen kalibreringspakke
å hente – den kalibreres i stedet i felten mot et refleksjonsmål, og det er
grunnen til at den aldri har trengt en. Å avvise disse opptakene gjorde at de ikke hadde noen måte å få ut
tallene sine på i det hele tatt, så de eksporteres under et **annet navn**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Et annet filnavn i stedet for en markør inne i filen, fordi opplysningene må tåle
å bli sendt videre via e-post som et rent navn. `.csv`-overskriften angir
`raw spectral sensor counts (NOT irradiance)` og advarer om at verdiene er sammenlignbare
**innenfor** filen – noe som er nøyaktig hva målbasert kalibrering bruker dem til – og
ikke på tvers av sensorer. De effektavhengige fotometriske kolonnene (total effekt, fotopisk og
skotopisk lux, PPFD) skrives som **NULL** i stedet for å integreres fra tellinger, og kjøringssammendraget
viser `RAW COUNTS`, så «eksportert» i en logg kan ikke tolkes som irradians.

Eldre **v1.01 / v1.02**-opptak (en DAQ-A-SD skriver disse) har ingen epoke per avlesning,
bare filens skrivetid. Bildet↔nedstrålingsmatcher avviser dem fortsatt — å sammenstille et
bilde med et skrivetidspunkt ville gi usynlige feil — men eksportverktøyet leser dem, og
CSV-filen skriver ut `clock=daq_created_on`, slik at produktet angir hvilken klokke det er på.

### Merknader

- `process` oppdager automatisk om mappen din er av typen Survey3, «LATTICE» eller en blanding.
- Fremdriften strømmes via Server-Sent Events; CLI-grensesnittet viser sanntids fremdrift per tråd (Detecting, Analyzing, Processing, Exporting).
- For Linux/Jetson sjekker CLI-en swap-minnet og kan gi en advarsel før store mapper behandles. Den teksturbevisst debayeren setter også automatisk en begrensning på GPU-frekvensen på Jetson-enheter med lav effekt (Nano, Orin Nano).
- Ved vellykket gjennomføring rapporterer kjøringen hvor mange bildeprodukter den har skrevet (`Image products written: N`, med `(plus M clamp masks)` når unmix-prosessen har begrenset noen piksler).
- **`process` deler en kjørende backend på en sikker måte.** Den starter automatisk en allerede kjørende backend på nytt bare når den backenden er inaktiv med et tidligere prosjekt lastet inn og ikke har noen kameraer, matriser eller DAQ-er. En backend som har noen av disse (GUI-en som streamer et array, et SDK-skript, en DAQ-opptak) brukes som den er, og startes aldri på nytt, slik at `process` kan kjøre ved siden av et live-array. Hvis backenden rapporterer at en kjøring pågår, venter `process` venter i opptil 15 sekunder på at den skal fullføres (det tar omtrent ett sekund å rydde opp etter en kjøring som nettopp er fullført), og avviser deretter med feilmeldingen `Backend is mid-run (processing). Wait for it to finish, or pass --restart to force a fresh backend (this kills the in-flight run).`. Det globale flagget `--restart` tvinger fremdeles frem en ny backend og avslutter alle kamera-, matrise- eller DAQ-økter på den. En backend som kjører som en Linux-systemd-tjeneste, startes aldri på nytt: `process` bruker den, og `--restart` skriver ut `sudo systemctl restart <unit>` i stedet (se [Kjøre backenden som en tjeneste](#running-the-backend-as-a-service-headless)). Det samme gjelder Chloros-appens egen backend: når appen er åpen, bruker `process` den og laster inn sitt prosjekt i stedet for appens (en backend som startes i stedet for den, ville låse den åpne appen ute).
- **En kjøring som allerede er i gang, beholder sitt prosjekt.** Mens en Processing-kjøring pågår — startet av appen, `process`, et SDK-skript eller `chloros-mcp` — nekter backenden å opprette, åpne, importere til, legge til eller fjerne filer fra, flytte, slette eller endre prosesseringsinnstillingene for et prosjekt, med HTTP-feil 409 `PROCESSING_IN_PROGRESS` og `A processing run is in progress on project '<name>', so ... now is refused`; en `process` som får dette, avsluttes av `1`. Visningsinnstillinger (miniatyrstørrelse) lagres fortsatt. Behandlingsmodus og prosjektets innstillinger for målreflektans avvises på samme måte (en kjøring beholder modusen den startet i). En opptak som lagres i prosjektet midt i en kjøring (fra appen, CLI, SDK eller `chloros-mcp`) lagres som vanlig, men blir først lagt til i prosjektets filliste når kjøringen avsluttes; dette fremgår av svaret (`project_add`), og det samme gjelder en lyssensoropptak som stoppes og legges inn i prosjektet. Tidligere kunne en annen klient bytte det åpne prosjektet midt i en kjøring, og den pågående kjøringen ble eksportert til det andre prosjektet.
- **Ctrl+C stopper kjøringen i backenden, ikke bare i CLI-en.** CLI-en ber backenden om å stoppe behandlingen, skriver ut `Waiting for the backend to stop (Ctrl+C again to stop waiting)...` og venter i opptil 120 sekunder på `Processing stopped.`, slik at `chloros-cli process … && next` ikke starter `next` på en backend som fortsatt er opptatt med kjøringen. Et nytt trykk på Ctrl+C avslutter ventetiden. `process` avslutter deretter `130`, slik alle avbrutte kommandoer gjør.

#### En kjøring som ikke skriver ut bilder, mislykkes

Hvis du ba om produkter og kjøringen skrev **ingen** — bare `project.json` og
`calibration_data.json` — behandler `process` dette som en feil: den skriver ut
`Processing finished but wrote no image products.` og **avsluttes med en verdi ulik null**, slik at et skript kan
oppdage det. Meldingen angir prosjektmappen og de vanligste årsakene:

- inndatamappen ble ikke gjenkjent som en opptaksserie (sjekk oppsettet og `--input-level`), eller
- alle forespurte produkter ble hoppet over fordi de ikke var relevante for disse kameraene (f.eks. ved forespørsel om
  strålingsintensitet/refleksjonsgrad fra kameraer som kun bruker RGB).

Kjør på nytt med `--verbose` og sjekk backend-loggen for linjer med `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
som forklarer utelatelser per kamera som ellers ikke vises i CLI-utdataene.

En bevisst kjøring kun med metadata – alle produktalternativer slått av og uten `--indices` – er fortsatt en
**suksess**, fordi et tomt bilde som utdata er det riktige resultatet i dette tilfellet.

#### En kjøring der eksportene ikke kunne verifiseres, mislykkes

Etter en kjøring sjekker `process` taggene til de eksporterte filene med ExifTool. Hvis ExifTool ikke kunne kjøres,
skriver den ut `Eksportverifisering MISLYKTES: ExifTool kunne ikke kjøres, så taggene til N av M eksporterte fil(er)
ble ikke sjekket (se [EXIFTOOL]-linjene i backend-loggen)` and `Produktene er skrevet ut og forblir
på plass; kjør på nytt for å verifisere dem.`, does not print `Behandlingen er fullført!`, og **avsluttes med en verdi ulik null**.
Produktene beholdes.

#### Hvilke opptak en kjøring tar med

Opptak som ble fullført før `process` ble kjørt, er inkludert. Opptak som ble lagret under kjøringen, er ikke det; `process`
varsler om hvor mange som ble utelatt, og hvis du kjører det på nytt, blir de inkludert.

#### En kjøring der et bilde gikk tapt, mislykkes

Hvis et bilde mislyktes under en kjøring, skriver `process` ut `N image(s) failed and were not exported` med den
vanligste årsaken, skriver ikke ut `Processing complete!`, og **avsluttes med en verdi ulik null**. De andre produktene blir
skrevet og forblir på plass, og backend-loggen inneholder hver enkelt feil.

Det samme gjelder en **kjøring kun med lyssensor**: en mappe med `.daq`-opptak har per definisjon ingen bilder å eksportere,
og kjøringen vurderes ut fra de kalibrerte `.daq` / `.csv`-filene den skrev i stedet.
Hvis den ikke skrev **noe** — alle opptak ble hoppet over — skriver den ut
`Processing finished but wrote no light-sensor products.`, navngir hvert opptak med
årsaken til at det ble hoppet over, og **avslutter med en verdi ulik null**.

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

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (ved å fjerne eller duplisere deler av passordet). Ved en 401-feil prøver CLI-en automatisk på nytt med `$$`-tilføyd, deretter med en deduplisert halvdel av passordet; hvis et nytt forsøk lykkes, logger den deg inn og viser den riktige syntaksen med enkelt anførselstegn som skal brukes neste gang.

> **Bruk uten grafisk grensesnitt/med skript: ingen bufret økt betyr en påloggingsprompt.** En kjerne-kommando (`process`, `status`, `export-status`, `time-sync`, …) som kjøres uten en lagret økt, ber om `Email:` / `Password:` på stdin før den fortsetter. Hvis stdin er lukket (cron, en systemd-enhet), mislykkes det umiddelbart med feilmeldingen `login cancelled`; hvis stdin er en åpen terminal eller pipe, venter det på inndata. Kjør `chloros-cli login EMAIL PASSWORD` én gang per maskin før du planlegger headless-arbeid.

---

## `chloros-cli logout`

Tømmer den bufret økten og tvinger frem en ny pålogging ved neste oppkall.

Å logge ut mens en behandlingskjøring pågår, stopper ikke denne: kjøringen fullføres i den behandlingsmodusen den startet i, og neste kjøring bruker standardbehandling. `logout` sier dette (`The processing run in progress finishes in <mode> mode; the next run uses standard`), og det samme gjør appen når du logger deg ut der.

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

Hent oppdateringer om eksportfremdriften for Thread-4 i sanntid. Kan trygt påkalles **under** en `process`-kjøring fra et annet skall.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Angi visningsspråket for CLI-en (38 støttes, inkludert CJK, RTL og indiske språk). Faller elegant tilbake til engelsk på eldre konsoller som ikke kan gjengi skriptet.

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

Disse styrer standardplasseringen for prosjektmappen (delt med GUI-en).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

`get-project-folder` viser mappen der nye prosjekter lagres. Hvis den konfigurerte mappen ikke lenger finnes (den er slettet, omdøpt eller ligger på en stasjon som ikke er tilkoblet), faller CLI-en tilbake til standardmappen, akkurat som GUI-en gjør: standardmappen skrives ut på stdout, en advarsel som angir den manglende mappen sendes til stderr, og avslutningskoden er `0`.

---

## `chloros-cli update`

Kun Linux/Jetson. Sjekker `version_url` mot `/etc/chloros/update.conf`, og hvis det finnes en nyere versjon, lastes den tilsvarende `.deb` ned og installeres umiddelbart med `sudo dpkg -i` (ingen bekreftelsesmelding; `sudo` kan be om passordet ditt). På en Jetson tilbys oppdateringen kun til JetPack 6-systemer. På Windows gjør kommandoen ingenting og rapporterer `up to date`: oppdater med et nytt installasjonsprogram.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

På Linux/Jetson kjører CLI-en også en **automatisk oppdateringssjekk ved hver oppstart** (ikke-blokkerende, forsinker aldri kommandoen): den leser `/etc/chloros/update.conf`, lagrer resultatet i hurtigbufferen i 1 time i `~/.chloros/update_cache.json` og viser `Update available: vX.Y.Z / Run: chloros-cli update` når en nyere versjon finnes. Hoppes over uten melding ved eventuelle feil og på Windows; `CHLOROS_DISABLE_UPDATE_CHECK=1` deaktiverer funksjonen. På grunn av den 1-timers cachen kan en utgivelse som publiseres innenfor den timen fremdeles vises som oppdatert.

---

## `chloros-cli selftest`

Kjører en 7-trinns røykprøve: versjon, porttilgjengelighet, oppstart av backend, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), tilstedeværelse av støydempingsmodell, klarhet for CUDA+støyreduksjon. Den bruker en kjørende backend, eller starter en for testen og stopper den etterpå. Den skriver ut `N/7 checks passed` og de mislykkede kontrollene, og avslutter `0` først når alle syv er bestått; CUDA+støyreduksjonskontrollen mislykkes på en datamaskin uten en CUDA-GPU (se [Bekreft installasjonen](#verify-the-install)).

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

PTP-grandmaster-status og -kontroll. Chloros-verten kjører PTP-grandmasteren; LATTICE-kameraer og DAQ-E-enheter fungerer som slaver for den for å oppnå tidsstempler på tvers av enheter (ca. 1 ms mellom enhetene). PTP synkroniserer **tidsstempler**: kameraene i et array eksponerer samtidig fordi M8-hardwareutløseren utløser dem (under 100 µs mellom modulene), ikke på grunn av PTP. `chloros-cli time-sync` uten underkommando kjører `status`.

| Underkommando | Beskrivelse |
| --- | --- |
| `status` | Vis grandmaster-tilstand, BMCA-prioriteter, klokkeidentitet og om pakkene når nettverket: `health` er `OK` (`STOPPED` når grandmasteren ikke kjører), eller `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` sammen med en `health_detail`, og deretter skrives en `WARNING:`-linje ut til stderr (stdout forblir JSON). `sync_sent` / `announce_sent` teller kun meldinger som ble sendt; `*_send_errors` og `last_send_error` teller de som ikke ble sendt. Etter at en kamerakobling brytes og kommer tilbake, gjenoppbygger grandmasteren sine sokler av seg selv (`rebinds`, `last_rebind`). |
| `peers` | Liste over slaver som er sett via Delay_Req (kameraer + DAQ-E-sensorer). |
| `cameras` | PTP-tilstand per kamera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Start grandmaster-prosessen på nytt. |
| `set-priority --priority1 N --priority2 N` | Overstyr BMCA-prioriteringer. |

Grandmasteren sender ut meldinger på kameraenes nettverkskort (ethvert kort med en `169.254.x.x`-adresse inkludert). Hvis ingen er oppe, faller den tilbake til kortet for standardruten bare når det er kablet, aldri Wi-Fi eller et virtuelt kort (Tailscale, WSL), siden ingen kameraer eller DAQ-E som befinner seg bak et slikt. Når det ikke er noe å kunngjøre, kjører det uten sokler (`health` `NO_SOCKETS`), og neste kamera eller DAQ-tilkobling gjenoppbygger det på det nettverkskortet som da er oppe; `time-sync restart` gjør det umiddelbart.

Grandmaster kjører i en egen prosess, ved siden av backenden (`own_process` / `process_pid` i `time-sync status`), med høy prioritet i Windows (`process_priority`), slik at en opptatt backend (en behandlingskjøring, en eksport, flere kommandoer samtidig) ikke kan forsinke tidspunktene den stempler; tidligere kunne en slik forsinkelse forskyve klokken på alle kameraene samtidig. Denne prosessen avsluttes med backend; hvis den ikke kan startes, kjører grandmasteren inne i backend som før (`own_process` false, og backend-loggen viser dette).

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

LATTICE-kamerakontroll. Hver underkommando går via Chloros-backenden; backenden eier kamerapoolen, slik at påfølgende CLI-kall gjenbruker det samme åpne håndtaket.

### Vanlige alternativer (felles for de fleste underkommandoer)

| Flagg | Beskrivelse |
| --- | --- |
| `-d, --device N` | Kameraindeks (standard: 0). |
| `-s, --serial SN` | Spesifikk serienummer; overstyrer `--device`. |
| `--serials SN1,SN2,…` | Kommaseparerte serienumre for drift med flere kameraer. [Kommandoene for midlertidige flerkamerasessioner](#multi-camera-transient-sessions) åpner kun disse kameraene, og det første utløser avtrekkeren. |
| `--all` | Utfør på alle oppdagede kameraer. |
| `--exposure US` | Eksponeringstid i mikrosekunder. |
| `--gain DB` | Forsterkning i dB. |
| `--pixel-format FMT` | f.eks. `BayerRG8`, `BayerRG12`; `Mono8`, `Mono12` på en M3M. Navnet angir bitdybden i kameraets egen familie: på en M3M angir et Bayer-navn monofilformatet for dybden (`BayerRG8` angir `Mono8`), på en M3C angir et mononavn Bayer-navnet (`Mono12` angir `BayerRG12`). |
| `--width N` / `--height N` | Bildedimensjoner. |
| `--preset {default,high_quality,high_speed,triggered}` | Bruk en forhåndsinnstilling: pikselformat, eksponering, forsterkning og bildefrekvens (`high_speed`: `BayerRG8`, 5 ms, 6 dB, 60 fps; `high_quality`: `BayerRG12`, 20 ms, 0 dB; `lattice presets` viser dem). På en M3M er formatet det samme monofilformatet med samme dybde: `high_speed` fanger opp `Mono8`, `high_quality` og `triggered` `Mono12`. Et flagg som følger med (`--exposure`, `--gain`, `--pixel-format`) har forrang fremfor forhåndsinnstillingen, og en innstilling som ikke er nevnt i forhåndsinnstillingen, beholder kameraets egen. Alle frie-kjøring unntatt `triggered`, som aktiverer kameraet for en maskinvareflank på linje 2 — uten noe som driver den linjen vil det vente i all evighet i stedet for å ta opp. |
| `-o, --output DIR` | Utdatakatalog (standard: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | GVSP-pakkestørrelse. `auto` kjører ICMP+GVSP-sonder; `jumbo` = 9000; `standard` = 1500. |

### LATTICE-kameraets arbeidsflyt ved første tilkobling

```bash
# 0. A backend must be running: the Chloros desktop app (Windows),
#    `chloros-backend`, or the Linux service (see "Start the Backend")

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
#      - Full sensor (no binning, no ROI); the frame rate falls to
#        what the host and wire deliver, and the faster options are
#        printed
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
| `lattice info` | Viser en liste over tilkoblede kameraer (leverandør, modell, serienummer, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analyser vertsystemet for optimal kamerakonfigurasjon. `--no-discover` hopper over kamerasøk (raskere, analyse kun av nettverkskort). |
| `lattice network [--fix] [--estimate] [--cameras N] [--pixel-format FMT]` | Kontroller/rett opp NIC-innstillinger (på Linux også kjernens mottaksbuffere for sokler — se [Linux-verter](#linux-hosts--socket-receive-buffers)); estimer båndbredde/FPS. |
| `lattice network-analysis --master SN [--slaves SN1,SN2,…] [--width N] [--height N] [--pixel-format FMT] [--binning {1,2,4}] [--force-tier TIER] [--packed {auto,on,off}] [--backend-url URL] [--json]` | Stable-schema-backend-nettverk-funksjonalitet + anbefaling av array (returnerer `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `--width` / `--height` bruker som standard hele sensoren, `2048` × `1536`. Den avsluttes med `0` for `ok`, `auto_capped_fps` og `auto_shrunk`, og `1` for `needs_force_slip` og `error`, med eller uten `--json`. Den når backenden ved `--backend-url` (standard `http://127.0.0.1:5000`), ikke `CHLOROS_BACKEND_URL`. `auto_capped_fps` beholder den forespurte oppløsningen, men setter en øvre grense for mål-fps — les `recommended.recommended_target_fps` og send den som tilkoblingsmål; behandle det som en suksess, ikke en feil. `auto_shrunk` vises bare når den forespurte rammen ikke kan leveres med noen bildefrekvens; connect reduserer aldri rammer på egen hånd. |
| `lattice analyze-array [--models M1,M2,…] [--binning {1,2,4}] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--packed {auto,on,off}] [--json]` | «Hva-hvis»-analyse uten å åpne kameraer (`--width` / `--height` standard `2048` × `1536`). Både dette og `network-analysis` viser hva connect vil streame: uten `--pixel-format`, gjelder formatene per kamera som «connect» bruker (Mono12 / BayerRG12), og med `--packed auto` (standard) vil [pakket 12-bit](#packed-12-bit-transport---packed) som connect ville valgt på denne datamaskinen — så den oppgitte fps-verdien er hastigheten som matrisen kjører med. Begge viser pakkingsvalget. **`--n-active` er det totale antallet kameraer på nettverket, ikke bare denne matrisens**— øk det når frittstående kameraer strømmer samtidig, eller når ledningskapasiteten beregnes ut fra et behov som undervurderer antallet (standard: `len(--models)`). Viser alltid de samlede linjene `Wire budget:` (MB/s etterspurt vs. kollisjonssikkert tak) og `Max cameras:`, og markerer `** OVER-SUBSCRIBED**` når arrayet overbelaster nettverket — se [Array fps & burst-modell](#array-fps--burst-model). |
| `lattice gpu` | Vis GPU-status. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Sjekk eller oppdater kameraets fastvare. Lokalt `.fwa`-valg er låst: filen i `firmware/<MODEL_PREFIX>/` som samsvarer med buildens `MIN_FIRMWARE_VERSION`, blir flashet når den er til stede (kun den høyeste versjonen som reserve), så et nyere leverandørbilde som er lagret på disken, forblir inaktivt inntil den festede versjonen byttes ut — bevisst nyere utgivelser når enhetene via det signerte AWS-manifestet, noe som foretrekkes når det er nyere. |
| `lattice presets [--apply NAME]` | Vis eller bruk kameraforhåndsinnstillinger. `--apply` angir hva forhåndsinnstillingene heter (pikselformat, eksponering, forsterkning, bildefrekvens, utløser) og skriver ut disse innstillingene slik kameraet leser dem etter: på en M3M, Mono-formatet for forhåndsinnstillingens dybde; bildefrekvensen den kjører med, som kan være lavere enn det forhåndsinnstillingen krever (`high_speed` krever 60 fps; et fullformat-kamera på en 1 Gbps-forbindelse kjører rundt 35). Alle andre innstillinger som «open» skrev, blir gjenopprettet slik de ble funnet. `--apply default` tilbakestiller kameraet til fabrikkstandardene: fabrikkinnstillingene med frittløpende automatisk eksponering på Line2, grensen for koblingens gjennomstrømning og feilretting slått av, samt et RGB-kameras kalibrerte hvitbalanse beholdt (enhet på alle andre). Ingenting lagres i kameraets minne, så ved neste oppstart starter det fra sitt eget minne som før. Som en del av en opptak- eller tilkoblingskommando er `--preset` og `default` ingen forhåndsinnstillinger. |
| `lattice status` | Live-kamerastatus. |

#### Opptak

| Underkommando | Formål |
| --- | --- |
| `lattice capture [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--no-daq] [--daq-tolerance-ms MS]` | Enkeltbilde. **Lagrer alle eksporttyper som standard**(`--processing all`); se [Eksportnivåer for opptak](#capture-export-levels-the-all-default). `--processing` aksepterer `all`, `raw`, `debayered`, `radiance`, `reflectance` eller `display` (`preview` godtas for `display`); ethvert annet navn avvises før kameraet berøres (avslutt `1`). `--levels` lagrer en eksplisitt delmengde (overstyrer `--processing`). Et kamera som Chloros ikke har, åpnes for opptaket, og eksponeringen settes først ut fra scenen (høyst noen få sekunder), slik [midlertidige kommandoer](#multi-camera-transient-sessions) gjør; når opptaket avsluttes, uansett hvordan det avsluttes, settes det kameraet tilbake slik det ble funnet (format, eksponering, fargepipeline, alle innstillinger som opptaket endret — listen under [Multi-Camera (Transient Sessions)](#multi-camera-transient-sessions)). DAQ-en som leser inn opptaket, lagres som en `.daq`-sidecar, med et rådata-utdrag inkludert; `--no-daq` / `--daq-tolerance-ms` justerer det ([DAQ-en som leser inn en opptak lagrer](#the-daq-reading-a-capture-saves)). `--force-daq` godtas fortsatt. `--jpeg-quality` = JPEG-kvalitet 1–100 (standard 95).**Filer som lagres i det åpne prosjektet, blir lagt til i det.** Som i appen vil en opptaksserie der filene havner i mappen til prosjektet som er åpent i Chloros, legge dem til i det prosjektets filliste; filer som lagres andre steder, blir liggende der de er. Kommandoen skriver ut én linje som sier dette (`Project: ...`), eller, mens en behandlingskjøring pågår, at de holdes tilbake og blir lagt til når kjøringen er ferdig. Det samme gjelder `multi-capture`, `array-capture` (`--smart` også) og `project capture`. |
| `lattice continuous [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--queue-depth N] [--no-daq] [--daq-tolerance-ms MS]` | Strømmer til disk inntil Ctrl+C. En `tiff`-kjøringer en matrise som kameraet skriver til filen `raw`: sensorens mosaikk, ett plan med sine egne verdier, ukomprimert, med kalibreringspakken (et 2048 x 1536 12-bits bilde er 6,0 MiB pluss pakken, uansett motiv), så `process` tar utgangspunkt i disse. `png` og `jpg` inneholder 8 bit, så disse sekvensene er demosaiserte bilder for visning, noe `process` ikke tar utgangspunkt i. Med en DAQ-bundet (`daq pool-bind`) eller samlet, får hvert bilde sitt eget `.daq` og `.match.json` i seriens `daq/`-mappe, siden `lattice capture` lagrer ett; `--no-daq` deaktiverer dette, og stoppkommandoen viser hvor mange. Hvert bilde skrives én gang, med fullstendige metadata; ved Ctrl+C skrives de bildene som allerede er hentet inn før kameraet slippes. Stanser seg selv når harddisken fylles, skriver ut `Failed:` / `Disk full:` og avsluttes med `1`; eventuelle bilder som ikke kunne skrives, avsluttes også med `1`. Et kamera som er satt i beredskap for en maskinvareutløser (`TriggerMode` `On`) settes til frikjøring for opptaket. Når opptaket avsluttes, settes kameraet tilbake slik det ble funnet — utløserinnstillingene og alle innstillinger som ble endret under opptaket (eksponering, forsterkning, format, …) — med mindre `--keep-settings`, som lagrer kjøringens innstillinger i kameraets eget minne (frittløpende), eller `--clear-settings`, som tilbakestiller dette minnet til fabrikkinnstillingene. En kjøring som ikke tar opp noen ramme, regnes som mislykket: uten rammer i 10 sekunder stopper den av seg selv, skriver ut `No frames:` med årsaken og avsluttes med `1`. Bilder som skriverne ikke klarte å holde tritt med, regnes som tapt, og når det stopper, skrives `Dropped:` ut med årsaken (kameraets bildefrekvens, skrivernes og køen); en lavere bildefrekvens eller et 8-bits pikselformat beholder dem alle. Kommandoen sjekker inn med backenden hvert 2. sekund; hvis den avsluttes uten å stoppe opptaket (`kill -9`, tomt for minne), stopper backenden den 30 sekunder etter den siste innsjekkingen og setter kameraet tilbake (`stop_reason` `client_gone`, og en linje i backend-loggen). `multi-stream` og `align-stream` gjør det samme. En oppstart der kameraet bruker lengre tid på å åpne seg enn kommandoen venter (60 sekunder; 180 sekunder for `multi-stream` / `align-stream`) — et kamera som fortsatt starter opp — blir avbrutt av backend-systemet: kameraet settes tilbake og lukkes, og ingen opptak fortsetter å kjøre (504 HTTP-feil `CLIENT_DEADLINE_PASSED`). |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Nettleserbasert live MJPEG-forhåndsvisning: kobler til kameraet, skriver ut strømURL-en og åpner den i nettleseren din. `--ae-damping` angir demping av automatisk eksponering (0,4–100). |

##### Live-strømmer (MJPEG; ikke RTSP)

Chloros leverer live-video som **MJPEG over HTTP**(`multipart/x-mixed-replace`, grense `frame`) fra backenden.**RTSP støttes ikke.** Åpne en URL-adresse i en nettleser, eller les den inn med en hvilken som helst MJPEG-klient (OpenCVs `cv2.VideoCapture(url)`, ffmpeg, VLC). Kameraet må først kobles til i backend (`lattice cam-connect`, `lattice array-connect`, `lattice viewer` eller GUI-en).

| URL (`http://127.0.0.1:5000` + …) | Strøm |
| --- | --- |
| `/api/camera/stream/<serial>` | Forhåndsvisning på skjermen, opptil ca. 30 fps. For et array-element, `?max_h=` (standard `720`) og `?jpeg_q=` (standard `80`) angir høyde og JPEG-kvalitet. Dette er URL-en som `lattice viewer` åpner. |
| `/api/camera/<serial>/stream-annotated` | Den samme forhåndsvisningen med GUI-overleggene tegnet inn: `zebra=1`, `crosshair=1`, `grid=3x3` (eller `9x9`), `histogram=1`, `peaking=1`, `spot=1`; uten forespørsel, prosjektets lagrede overleggsinnstillinger for det aktuelle kameraet. |
| `/api/camera/<serial>/stream-reflectance` | 8-biters reflektans per bånd: ρ = piksel ÷ den delen`X-Chloros-Pixel-Scale`-overskriften (255 med mindre `refl_max` er satt). `fps` (standard `10`), `jpeg_q` (standard `95`), `max_h` (standard `0` = native), `band=<name>`, `align=1`. JPEG, altså med tap. Avvist med HTTP-kode 409 når kameraet ikke har noen reflektans (et RGB-kamera, ingen kalibreringspakke, ingen DAQ-avlesning). |

- **Sammenstilling av bilder på tvers av kameraer.**Hver del som et**array-medlem** sender, på alle tre strømmene, inneholder `X-Chloros-Group-Id` (arrayets synkroniserte gruppeteller), `X-Chloros-Frame-Id` og `X-Chloros-Timestamp-Ns`-overskrifter. Deler fra forskjellige kameraer med samme `X-Chloros-Group-Id` kommer fra samme utløser. Et frittstående kamera har ingen gruppe og sender ingen gruppe-ID. Nettlesere ignorerer overskrifter per del; en klient som selv analyserer den flerdelte strømmen, leser dem.
- **Pauset mens matrisevisningen er åpen.** `/api/camera/stream/<serial>` fra et matriseelement sender en statisk «pauset»-ramme mens matrisens samlede direktestrøm er åpen (for eksempel GUI-ens matrisevisning), og gjenopptar sendingen når den lukkes.

#### Sensorjustering

| Underkommando | Formål |
| --- | --- |
| `lattice configure [{--get N1 N2… \| --set N=V N=V… \| --dump}] [--json]` | Les/skriv til en hvilken som helst GenICam-node. `--set ExposureTime=…` (`Gain=…`) uten `ExposureAuto` (`GainAuto`)-bryterne som først slår av automatisk modus, slik som `lattice exposure --set` gjør; en verdi som kameraet avviser, skriver ut `Failed to set <node>: …` på stderr og avslutter `1`. `--get` og den standard gruppevisningen skriver ut en `note:` under `ExposureTime` / `Gain` mens den automatiske modusen kjører — se [Avlesning av eksponering og forsterkning i automatisk modus](#reading-exposure-and-gain-under-auto-mode). |
| `lattice exposure [{--auto \| --auto-once \| --off \| --set US}] [--brightness 0-255] [--brightness-auto] [--damping 0.4-100] [--upper-limit US]` | Eksponering og AE. En `--set`-eksponering eller et `--brightness`-mål blir holdt som ditt (Chloros’ automatiske eksponeringskontroll lar den være i fred); forsterkningen forblir som den var, så i auto-modus fortsetter den å justere seg mot målet inntil du angir `lattice gain`. `--auto` gir eksponeringen tilbake, `--brightness-auto` gir målet tilbake til Chloros’ automatiske mål. Uten flagg leser den eksponeringsnodene, med samme `note:` som `configure --get`. En `--set`-verdi som kameraet avviser, skriver ut `Failed to set ExposureTime: ExposureTime=<value> refused (<camera error>); the camera accepts <min>..<max> now (ExposureAuto put back to <mode>)` på stderr — i automatisk modus gjenopprettes skrivefunksjonen i stedet for å forbli deaktivert — skriver ikke ut noen `Exposure: … us (auto OFF)`-linje, og avslutter `1`. |
| `lattice gain [{--auto \| --off \| --set DB}]` | Forsterkning og automatisk forsterkning. En `--set`-verdi som kameraet avviser, skriver ut `Failed to set Gain: …` på stderr, ingen `Gain: … dB (auto OFF)`-linje, og avslutter `1`. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N]` | Sensor-ROI og binning. Binning er alltid **Gjennomsnitt** — se nedenfor. |
| `lattice format [--set FMT] [--list]` | Pikselformat. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Maskinvare-/programvareutløser. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (ingen flagg = engangs-hvitbalanse) | Hvitbalanseoperasjoner. Kun RGB-/Bayer-kameraer; en no-op (hoppes over) på mono M3M. One-shot beholder hvitbalansen den beregner (`BalanceWhiteAuto` og forsterkningene); alle andre innstillinger som den skriver til, settes tilbake til opprinnelig verdi, og alt dette gjelder når one-shot ikke kan kjøres. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct 2000-10000] [--get]` | Fargebehandlingsrørledning for RGB-visning. `natural` (standard) er den enkle live-behandlingen; `enhanced` legger til defringe + vibrance + CLAHE lokal kontrast for det fullstendige «hub-parity»-utseendet til omtrent 2× behandlingskostnaden per bilde, noe som gir en lavere **live** bildefrekvens — lagrede opptak får uansett alltid full behandling. Kun RGB-/Bayer-kameraer; hoppes over på mono M3M. |
| `lattice color [--saturation 0-200] [--contrast 0-200] [--reset] [--get]` | Metning/kontrast i live-forhåndsvisning (kameraer med RGB-filter); `100` er nøytralt, `--reset` setter begge til 100. Lagrede opptak påvirkes ikke. Hoppes over på mono M3M. |
| `lattice highlights [{--set -100..0 \| --reset \| --get}]` (ingen flagg = les) | Vis **høylys** (kameraer med RGB-filter): demp de lyseste, hviteste pikslene — f.eks. en lys hvit overflate som gjengis som en flat hvit klatt. `0` = av, `-100` = sterkest. Live-forhåndsvisningen og opptakene på skjermnivå følger denne innstillingen; stråling/refleksjon gjør det aldri, og den kan ikke gjenopprette sensordata som er klippet (senk eksponeringen for det). Hoppes over på mono M3M. |
| `lattice filter [--list]` | Viser en liste over filtermodellene Chloros kjenner til (`RGN-IMX265`, `OCN`, `NGB`, …). Kameraets filter hentes fra modellstrengen og kan ikke stilles inn: `--set NAME` søker bare opp en modell, og avslutter deretter `1` med `setting the filter model is not supported`. |
| `lattice power [--sleep]` | Sjekker strøm- og termiske noder; slår lavstrømshvilemodus på/av. `--sleep` lar kameraet være i hvilemodus (grensen for koblingsgjennomstrømning er slått på (`DeviceLinkThroughputLimitMode` `On`) på sitt laveste nivå, strømsparingsmodus er på der kameraet har en) mens alle andre innstillinger gjenopprettes slik de opprinnelig var; en sovemodus som mislykkes, settes tilbake i sin helhet. Oppvåkningen setter koblingen tilbake slik den var da den gikk i sovemodus (grensen, modusen og pakkeforsinkelsen `GevSCPD`, som kameraet holder på 0 mens grensen er aktiv; opptegnelsen lagres på denne datamaskinen, slik at en oppvåkning etter at backend-systemet har startet på nytt også finner den), eller, for et kamera som denne datamaskinen ikke har satt i hvilemodus, men som ser ut til å være i hvilemodus (grensen er på ved minimumsnivået), med grensen av og en `GevSCPD` satt til 0 ved 80 (fabrikkinnstillingene); en grense satt over minimumsnivået forblir uendret. Sonden og oppvåkningen gjenoppretter det som ble skrevet ved åpningen. |

> **Binning er alltid «Gjennomsnitt».** `lattice resolution --binning N` angir «Gjennomsnitt»-binning og tilbyr
> ikke modusbytte. Sum-binning multipliserer hver DN med N², noe som koster omtrent to blenderstopp med
> høylysrom per binningtrinn; Chloros deler denne faktoren ut igjen før den
> radiometriske kjeden, men piksler som Sum har drevet opp til sensorens tak blir klippet og kan ikke
> gjenopprettes. Det gir ingen fordeler som «Gjennomsnitt» ikke gir – en indeks er skalainvariant, og
> strålingsskalarene er kalibrert på «Gjennomsnitt»-skalaen. Hvis et kamera allerede er innstilt på «Sum» (etterlatt
> der av et annet verktøy eller ved en tilbakestilling til fabrikkinnstillinger), oppdager Chloros dette ved en nodelesing og
> kompenserer; å kjøre `lattice resolution --binning N` setter den tilbake til «Average». 1.2.0
> `--binning-mode`-flagget aksepteres fortsatt, slik at eldre skript kjører, men det skriver bare ut en advarsel:
> binning forblir på «Gjennomsnitt».

> **Endring av binning, pikselformat og utløser på et kamera har ingen effekt.** På et kamera der
> backenden holder seg åpen og strømmer (`cam-connect`, GUI-en), trer `lattice resolution --binning`
> trer ikke i kraft (det viser at binning fortsatt er aktivt, `Binning: 1x1`), og
> `lattice format --set` og `lattice trigger --mode` avvises av kameraet
> (`SC_ERR_ERROR -1001`). Frigjør den først (`lattice cam-disconnect -s <SN>`), eller sett
> dem ved tilkobling (`cam-connect --pixel-format …`, `array-connect --binning …`).

##### Avlesning av eksponering og forsterkning i automatisk modus

Mens et kameras `ExposureAuto` (`GainAuto`) kjører, beholder `ExposureTime` (`Gain`) noden **den siste verdien som ble skrevet**, ikke hva kameraet gjør akkurat nå. For et kamera som backenden holder åpent (`cam-connect`, `array-connect`, GUI), viser `lattice configure --get`, standardvisningen `lattice configure` og `lattice exposure` (uten flagg) derfor nodens verdi og, under den, en merknad med kameraets nyeste bilde:

```

ExposureTime = 2000.448
  note: ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448); the camera's newest frame was at 4900 us.
```

Les rammeverdien fra merknaden; nodeverdien er kun den siste oppføringen. Over HTTP holder det samme `POST /api/camera/<serial>/read-nodes`-svaret `nodes` uendret og legger til `frame_values` (`{"ExposureTime": <us>, "Gain": <dB>}`, den nyeste rammen) og `notes` (setningen ovenfor). Begge er kun til stede så lenge automodus ikke er `Off` og en ramme er kjent; et kamera som er åpnet kun for lesing, har ingen ramme og får ingen av delene. `lattice gain` (ingen flagg) skriver ut kun noden — bruk `lattice configure --get Gain` for notatet.

##### Eksponering på et array-element

Mens et array strømmer, kjører hvert element med **`AcquisitionFrameRateEnable` på**, med en bildefrekvens som ligger litt over matrisens utløserfrekvens: perioden er utløserperioden minus 15 % (minst 3 ms), eller kameraets egen maksimale frekvens når utløseren allerede er nær denne. `configure --get` på et element leses derfor som `AcquisitionFrameRateEnable`, `True` og `AcquisitionFrameRate` ved denne innstillingen; aktiveringen slås av igjen når matrisen stopper.**Ikke skriv disse to nodene på et array-medlem** — arrayet holder dem synkronisert med utløseren og overskriver dem.

Denne innstillingen er også medlemmets eksponeringstak: ca. 85 ms ved en utløser på 10 fps (100 ms periode), uansett **ROI eller binning**. Før versjon 1.3.0 begrenset en liten ROI i stedet et elements eksponering til sensorens friløpsperiode (4,9 ms ved 768 × 304), uansett hvor sakte matrisen utløste.**En manuell `ExposureTime` som er lengre enn taket, bremser utløseren for å tilpasse den** når den skrives med `lattice exposure --set` eller `configure --set` (eller fra GUI). Matrisen åpner først det aktuelle elementets tak for verdien og lar den lavere hastigheten kjøre i noen utløsningsperioder, slik at skrivingen kan ta noen sekunder lenger; utløseren utløses deretter ikke raskere enn eksponeringen tillater. På en matrise med automatisk hastighet gjenopprettes den tidligere hastigheten så snart hvert element som ble bremset av manuell eksponering, er tilbake på automatisk eksponering. En verdi som kameraet selv ikke kan ta, blir fortsatt avvist, med det aksepterte området angitt i feilmeldingen, og medlemmets automatiske eksponering gjenopprettes hvis den kun var slått av for den aktuelle skrivingen. En oppskrift med eksponering `apply` bremser ikke utløseren: den holdes på takverdien. Chloros forkorter aldri en manuell eksponering for å tilpasse seg en raskere utløser; en fastsatt hastighet (`array-connect --target-fps F`) som er raskere enn det eksponeringen tillater, gir det aktuelle medlemmet et tak som er langt nok for den, og det medlemmet tar utløsere som ikke er raskere enn det eksponeringen tillater.

#### Kalibrering og sensorer

| Underkommando | Formål |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibrer fra et refleksjonsmål. Opptil `--attempts` rammer (standard `30`) blir søkt etter; en ufullstendig ramme (GVSP-pakketap) koster ett forsøk, ikke hele kjøringen. Det mislykkes med `no target detected in N attempts`, eller ved `no complete frame in N attempts: <last error>` når ingen ramme ankom i sin helhet. |
| `lattice dls {--connect \| --spectrum \| --irradiance} --mac MAC [--filter NAME] [--camera SERIAL] [--json]` | Spektrumet og båndstyrkene til en DAQ-M, avleses via Bluetooth (BLE) gjennom backenden. `--mac` er sensorens egen BLE-MAC-adresse og kreves ved alle de tre handlingene: det finnes ingen standardverdi, siden hver enhet har sin egen (`daq pool-discover --only ble` viser dem); uten den avsluttes kommandoen `1`. Uten noen handling vises hjelpeteksten. `--camera SERIAL` integrerer de utskrevne båndstyrkene gjennom kameraets egen målte spektrale respons i stedet for filtermodellens flate kurve — RGB-kameraer, med kameraets kalibreringspakke lastet inn (tilkoblet) eller lagret i hurtigbufferen på denne datamaskinen. Overskriften angir hvilken kurve som ble brukt: `Band magnitudes (camera <SERIAL>):`, eller `Band magnitudes (model <filter>):` når kameraet ikke har noen (ikke et RGB-kamera, eller ingen pakke lastet inn eller lagret i hurtigbufferen), og standardkurven ble brukt i stedet. Over HTTP tar `POST /api/dls/cli-read` imot `camera_serial` og returnerer `band_magnitudes_source`. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Bruk vignettkorreksjon på eksisterende bilder, hvert med sitt eget objektivs flatfelt (fra EXIF-modellen). `--lens-model` gjelder for bilder der modellen ikke angir noe kamera; et bilde der modellen er i strid med dette, hoppes over. |

#### Flere kameraer (midlertidige økter)

En midlertidig kommando åpner kameraene for én kjøring og lukker dem når den er ferdig: `multi-capture` uten tilkoblet permanent matrise, `multi-stream` og `multi-test` her, og `align-calibrate`, `align-apply`, `align-stream` og `index --live` nedenfor. Disse reglene gjelder for dem alle:

- **`--serials SN1,SN2,…` velger kameraene.** Bare disse åpnes, og det første utløser utløseren (for `align-calibrate`, `--reference` når angitt). En serienummer som ikke er på nettverket, fører til at kjøringen mislykkes før noe kamera åpnes. Uten `--serials` åpnes alle kameraene på nettverket. Et kamera som Chloros allerede har (GUI-en, `cam-connect`, `array-connect`) avviser kjøringen uansett om `--serials` nevner det eller ikke: det deler nettverket som kjøringen er tilpasset for. (`index --live` uten `--serials` åpner i stedet de aktiverte kameraene i profilen sin: se [Indeks / Vegetasjonsmatematikk](#index--vegetation-maths).)
- **Kameraene deler nettverket på samme måte som `array-connect` deler det.** Hvert kamera er tilpasset sin andel av det vertslenken overfører (en forsinkelse mellom pakker, ikke kameraets gjennomstrømningsgrense, hvis nedre grense på 31,25 MB/s allerede overskrides av fire kameraer på 1 GbE), så fire kameraer kjører på en 1 Gbps-forbindelse. Når ikke alle kameraene kan sende samtidig med den laveste hastigheten de oppnår, utløses de ett etter ett, hvert med sin egen forbindelse, og kommandoen skriver ut `Serial emit: the wire cannot carry these cameras at once, so each is triggered <ms> ms after the one before; their frames are not exposed together`. Når selv ett kamera om gangen ikke får plass, avslås kjøringen før noe kamera strømmer: `array over-subscribes the wire: <n> cameras demand <x> MB/s but the collision-safe ceiling is <y> MB/s. …`, HTTP-kode 409, avsluttes med `1`.
- **Hvert kamera må levere, ellers mislykkes kjøringen.** Et kamera som ikke leverer noen ramme, er en feil: kommandoen viser `Error: … no frame from <n> of <total> cameras: SN:<serial> (<why>), …`, der `<why>` er hentefeilen for det aktuelle kameraet, deretter hva årsakene tyder på, og avsluttes med `1`. En ramme som ankom med GVSP-pakketap (`IncompleteBufferError: GVSP packet loss …`) betyr at nettverket ikke klarer å håndtere kameraenes samlede båndbredde (`Every missing frame arrived with GVSP packet loss: …`): sjekk koblingshastigheten til verten og til svitsjens uplink (`lattice network`), bruk jumbo-rammer, eller åpne færre kameraer med `--serials`. Et kamera som ikke leverte noe, mottar vanligvis ikke utløsersignalet fra masteren: sjekk M8-synkroniseringskabelen, og kjør deretter `lattice multi-detect`. Hva hver kommando gjør med en rammegruppe der et kamera mangler, står i den tilhørende raden.
- **Hvert kamera settes tilbake slik det ble funnet.** Når kjøringen avsluttes, får hvert kamera tilbake det det hadde da kjøringen åpnet det: `TriggerMode`, `TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, og `LineMode` / `LineSource` på `Line2` og `Line3` (og deglitching av trigger-inngangen gir en slave-linje, `LineFilterSelector` / `LineFilterWidth`); dets `PtpEnable` (og `PtpSlaveOnly` / `PtpDomainNumber`-settet med det); `ExposureAuto` / `GainAuto` (og `ExposureTime` / `Gain` der det var angitt en manuell verdi); `TargetBrightness`, `ExposureAutoLimitAuto` / `ExposureAutoUpperLimit` / `ExposureAutoLowerLimit`, `GainAutoUpperLimit`, `ExposureAutoDamping`; `BalanceWhiteAuto` (og hver `BalanceRatio` der hvitbalansen var manuell); `BlackLevelAuto` / `BlackLevel`; `PixelFormat`, binning, `Width` / `Height` / `OffsetX` / `OffsetY`; `GammaEnable` (og `Gamma` der gamma var aktivert), `LUTEnable`, `ColorTransformationEnable`; `ReverseX` / `ReverseY`; `UserSetSelector`; pakkestørrelsen (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`); takten, `DeviceLinkThroughputLimitMode` (og `DeviceLinkThroughputLimit` der modusen var `On`), forsinkelsen mellompakkeforsinkelsen `GevSCPD` og startforsinkelsen `GevSCFTD` som en matrise gir for hvert kamera; blokkdataene som hver strøm aktiverer (`ChunkModeActive`, hver `ChunkEnable` under sin `ChunkSelector`); `AcquisitionFrameRateEnable` (og `AcquisitionFrameRate` der aktiveringen var på — når den er av, er hastigheten skrivebeskyttet og leser så mye som takten tillater, så det er ikke en innstilling). Bare det som avviker skrives, kameraet stoppet først, takten og bildefrekvensen før en eksponering som trenger sin bildeperiode (et kamera begrenser `ExposureTime` til én bildeperiode). Med `--keep-settings` kobles utløseren og synkroniseringslinjen tilbake, kameraet kjører fritt, og resten lagres i kameraets eget minne slik kjøringen har angitt det; `--clear-settings` setter fabrikkinnstillingene der.
- **Eksponeringen stilles inn ut fra scenen før det første bildet lagres.** Opptaket tar synkroniserte bildegrupper og forkaster dem mens det stiller inn hvert kameras eksponering og forsterkning ut fra disse bildene. Målet er en gjennomsnittlig lysstyrke på kameraets mål uten at mer enn 1 % av bildet blir klippet; en scene med sterke høylys stopper før målet nås. Dette tar vanligvis 2–6 bildegrupper og høyst 5 s.
  - Hvert kamera retter seg mot målet som en tilkoblet matrise starter det med, via filteret: TargetBrightness **40**på et multispektralt kamera (RGN, OCN, NGB) og**60** på et RGB-kamera. Dette er halvparten av eksponeringsmålet som 80 gir (tre fjerdedeler på RGB), så det er mindre sannsynlig at et lyst refleksjonspanel blir klippet. `--brightness N` (0–255) angir i stedet hvert kameras mål (`multi-test` har ikke `--brightness`: automatisk eksponering er slått av). Backendens logg navngir hvert kameras mål: `[transient-ae] AE target per camera (filter default): SN:<serial> <target>, …`.
  - Eksponeringen kommer først, opp til kameraets egen grense for automatisk eksponering. Forsterkningen dekker bare det som er igjen.
  - Den beregnede eksponeringen og forsterkningen beholdes gjennom hele opptaket, slik at hvert bilde i en `multi-stream` tas med samme eksponering. Et opptak som må følge skiftende lysforhold, hører hjemme i en tilkoblet matrise (`array-connect`), hvor den automatiske eksponeringen justeres kontinuerlig.
  - `--exposure` og `--gain` beholdes nøyaktig slik de er angitt. Når begge er angitt, blir ingen bilder forkastet.
  - Når kjøringen avsluttes, slås den automatiske eksponeringen på hvert kamera på igjen.
  - Backendens logg rapporterer resultatet: `[array-ae] exposure solved in <s> s (<n> warm-up groups): SN:<serial> <ms> ms <dB> dB mean <m>/<target>, …` eller `exposure NOT solved after …` med årsaken, og `(not settled)` for hvert kamera som ikke ble tatt med. Bildene tas likevel i dette tilfellet.
  - `CHLOROS_TRANSIENT_AE_SETTLE_S` endrer 5-sekundersgrensen; `0` slår denne av (se [Miljøvariabler](#environment-variables)).

**Mens Chloros holder et hvilket som helst kamera** (GUI-en, `cam-connect` eller et tilkoblet array), vil `multi-stream`, `multi-test`, `multi-detect`, `align-apply`, `align-stream` og `index --live` nekter å kjøre, fordi de åpner alle kameraer i nettverket, og en ny åpning åpning av et reservert kamera kan føre til at backenden krasjer. (`index --live` avviser når et kamera det skal åpne er reservert: enten `--serials` eller kameraene i profilen, samt ethvert kamera for en profil som ikke angir noen.) Backenden svarer med HTTP-kode 409 `CAMERA_HELD`, og oppgir hvert reservert kamera og frigjøringskommandoene (`chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`), og kommandoen avsluttes med `1`. Når en matrise er tilkoblet, registrerer `array-capture --continuous` den i stedet for `multi-stream`, og `array-capture` (justert som standard) lagrer et justert sett i stedet for `align-apply`; `multi-capture` og `align-calibrate` bruker en tilkoblet matrise som inneholder kameraene deres i stedet for å åpne dem.

| Underkommando | Formål |
| --- | --- |
| `lattice multi-info [--serials SN1,SN2,…]` | Viser en liste over alle kameraer med synkroniseringsroller; med `--serials` vises kun disse, og det første som `MASTER`. |
| `lattice multi-capture [--serials SN1,SN2,…] [--format FMT] [--jpeg-quality N] [--processing LEVEL] [--brightness N] [--no-daq] [--daq-tolerance-ms MS]` | Ett synkronisert bilde fra hvert kamera. Lagrer **alle eksporttyper som standard**når en permanent matrise er tilkoblet, og den DAQ-avlesningen som ble brukt ([Hvilken DAQ-avlesning en opptak lagrer](#the-daq-reading-a-capture-saves)); den midlertidige fallback-løsningen uten matrise er**kun debayert** (kjør `array-connect` først for resten). `--brightness` gjelder kun fallback-modusen: et tilkoblet array beholder sin egen automatiske eksponering. Fallback-modusen lagrer ikke `.daq` (`--no-daq` / `--daq-tolerance-ms` har ingen effekt der), og `multi-capture` tar ikke `--force-daq`. I fallback-modusen vil et kamera som ikke leverer noen ramme mislykkes i opptaket (`Error: multi-capture failed: no frame from …`, avslutt `1`), og ingen fil fra den rammegruppen blir lagret. |
| `lattice multi-stream [--serials SN1,SN2,…] [--fps F] [--count N] [--format FMT] [--jpeg-quality N] [--brightness N]` | Strøm synkroniserte bilder (midlertidig), `--fps` standard `3`, inntil Ctrl+C eller `--count`-grupper. En bildegruppe som et kamera ikke har levert noe til, blir ikke skrevet, og tre på rad avslutter strømmen. Hvis noen gruppe ble hoppet over, avsluttes kjøringen ved å skrive ut `Incomplete: <n> frame group(s) not written, a camera delivered nothing (SN:<serial> x<count>, …)`, pluss `Error: the stream stopped: …` som angir kameraet da strømmen ble avsluttet, og avslutter `1`X. Stanser av seg selv når stasjonen blir full, skriver ut `Failed:` / `Disk full:` og avsluttes med `1`; hvis det er noen filer den ikke klarte å skrive, avsluttes også `1`. |
| `lattice multi-test [--serials SN1,SN2,…] [--count N] [--ptp-settle SECONDS]` | GPIO-synkroniseringstest (forbigående), `--count` standard `10`-utløsere. Skriver ut hvordan nettverket deles (`Wire: each camera paced to <x> MB/s (<n> on a <y> MB/s ceiling, <link> Mbps link)`, og linjen `Serial emit:` når kameraene utløses etter hverandre, noe som tidsstemplene da reflekterer), en `Sync group: master SN:<serial> -> SN:<serial>, …`-linje per synkroniseringsgruppe (`(no slaves: not synced)` for et kamera uten synkroniseringskabel, som utløses på egen hånd), deretter én rad per utløsning: `[  0] SN:<serial>=<timestamp> \| …`, med `SN:<serial>=(no frame)` for et kamera som ikke leverte noe for den utløsningen. Deretter `Offsets from master SN:<serial> (<PTP / device (PTP off)> clock, <n> trigger(s)):` og en `SN:<serial>: mean <±x> us, jitter <y> us`-linje per kamera, hvert kamera i forhold til masteren innenfor én klokkesyklus: en utløser der tidsstemplene blander kameraenes enhetsklokker og PTP (PTP av eller fortsatt låser) utelates, med en `Note:` som teller dem. Ethvert utelatt bilde fører til at testen mislykkes (`Error: multi-test failed: <k> of <n> frame groups incomplete (SN:<serial> missed <k>); …`, avslutt `1`); tre ufullstendige utløsere på rad avslutter den tidlig. Den mislykkes også (avslutt `1`) når færre enn to utløsere har alle kameraene på samme klokke (`… the clocks cannot be compared`), eller når klokkene aldri stemmer overens: PTP-klokker som avviker mer enn 1 ms (pluss eventuelle forsinkelser på serienivå) fra masteren (`… PTP is not locked`), eller enhetsklokker hvis avvik endres med mer enn dette mellom utløserne. Den starter PTP-grandmasteren hvis den ikke kjører, slår på PTP for hvert kamera (slik `array-connect` gjør, uansett hvordan nettverket ser ut), og før den første utløseren venter den på at PTP-klokken til hvert kamera skal låse seg til grandmasteren og stabilisere seg: `PtpStatus` Slave på én grandmaster, innenfor 100 us av den og stabil i 3 s, og hvert kamera låst på denne måten, uten avbrudd, i 120 s — omtrent 2 minutter fra strøm. En klokke som nettopp har låst seg, avviker med opptil ~200 us i omtrent 2 minutter etter at den viser «Slave», og kameraets egen `PtpOffsetFromMaster` viser ikke dette; 120 s bringer kameraene innenfor ~10 us av en stabilisert klokke. Et kamera er fortsatt synkronisert siden en tidligere test der PTP ble stående på, slik denne backenden oppfattet det, teller fra den synkroniseringen, så en test rett etter en annen venter ikke på nytt; et kamera som testen slår på, eller et som oppdages med PTP på og som denne backenden ikke har sett synkronisert (slått på et annet sted, eller før en omstart av backenden), venter sine 120 s, og i en gruppe med begge kameraene som nettopp har synkronisert, venter de fortsatt på sin tur. Et kamera som forlater Slave-modus, går over 100 us eller følger en annen grandmaster i 3 avlesninger på rad (1,5 s) under ventetiden, starter tellingen på nytt; én eller to slike avlesninger logges (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`), og tellingen fortsetter, og en grandmaster-ID som ikke kan leses, blir ansett som ukjent, ikke som en endring. Når tellingen for alle kameraene starter på nytt samtidig (innen ett sekund), skyldes dette at grandmasterens tid beveger seg, ikke kameraenes: loggen til backend viser `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` viser den i `grandmaster_steps`, og en ventetid som deretter går ut, starter feilen med `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Grensen utvides ikke for dette. `--ptp-settle SECONDS` angir avventingstiden for kjøringen (0–600; `0` for ingen avventingstid). Ventetiden er begrenset til stabiliseringstiden pluss 90 s (210 s som standard, minst 90 s), og kommandoen venter på backenden i 240 s pluss stabiliseringstiden (standard 360 s). Den skriver ut `PTP: locked after <s> s (each camera locked <settle> s, already locked: SN:<serial>, …; from the grandmaster: SN:<serial> <±x> us, …)`, og backendens logg har en linje hvert 5. sekund med hvert kameras tilstand og forskyvning, hvor lenge det har vært låst og hvor lenge det har igjen: `[array-sync] PTP at <s> s: SN:<serial> Slave <±x> us for <s> s (<s> s left), … -- <s> s left, the wait gives up at <s> s`. Hvert kamera slås på med PTP slått av; deretter synkroniseres klokkene på omtrent 10 s. Etterpå settes hvert kamera tilbake slik testen fant det – dets PTP, automatisk eksponering og forsterkning holdes av under testkjøringen, og resten (med mindre `--keep-settings`) — så et kamera som står igjen med PTP på (`lattice configure -s <serial> --set PtpEnable=True`, eller en kjøring med `--keep-settings`) forblir synkronisert, og når denne backenden har sett at det har vært synkronisert i stabiliseringstiden, venter ikke en senere test på det. Hvis klokkene ikke stabiliserer seg i ventetiden, mislykkes den før utløsning (`PTP: NOT locked …`, avslutt `1`) og angir hva som skal gjøres: et kamera som aldri har låst seg (`PtpStatus Listening`) betyr at grandmasteren ikke når det (`time-sync status`); et kamera som stadig falt ut, navngis etter hvor lenge det hadde vært låst (`… had not been locked for the 120 s settle time … (SN:<serial> locked <s> s, <s> s to go)`); klokker som fortsatt konvergerer: sjekk `time-sync status`, og for å få dem låst før testen, slå på PTP og la det være på (`lattice configure -s <serial> --set PtpEnable=True` for hvert kamera, eller kjør testen med `--keep-settings`; testen gjenoppretter det den fant, så den forblir på), og kjør den deretter. |
| `lattice multi-detect [--line LINE] [--json]` | Finn kameraene på M8-synkroniseringslinjen og en standardmaster: RGB-kameraet (modell som slutter på `-FRGB`) hvis det er kablet, ellers det første kablede kameraet etter serienummer. M8-kablene møtes i ett knutepunkt, så **ethvert kablet kamera kan være master** — den påtrykte betegnelsen `MASTER` er en standardinnstilling, ikke et faktum om kabling. Hvert kamera driver linjen etter tur, og de andre sjekkes for pulsen fra det; et kamera som allerede er funnet i den kablede gruppen, blir ikke sjekket på nytt, så en fullt kablet oppstilling tar én runde. Uten `--line` søker den etter `Line2`, deretter `Line3`. Skriver ut `MASTER: SN:…`, én `SLAVE: SN:…` per annet kablet kamera og `(probed on <line>)`; `--json` legger til det fullstendige resultatet (`master`, `slaves`, `no_cable` — kameraer som pulsen ikke nådde — `line`, og `results` per kamera). Når prosessen er fullført, gjenoppretter hvert kamera de opprinnelige innstillingene for utløser og synkroniseringslinje (de samme nodene som ovenfor). |

#### Justering

| Underkommando | Formål |
| --- | --- |
| `lattice align-calibrate [--serials SN1,SN2,…] [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N] [--brightness N]` — pluss detektor-/matcher-knapper `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, RANSAC-knapper `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, kombinasjon av flere bilder `[--averaging mean\|median\|inlier_weighted]`, geometriske begrensninger `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, romlig begrensning `[--roi X0,Y0,X1,Y1] [--mask PATH]` ogslave-overstyringer `[--per-cam-override SN:KEY=VALUE]` (kan gjentas) | Beregn justeringsprofil fra live-kameraer. Standardverdier: `--method orb`, `--model rigid`, `--frames 3`, `--save alignment_profile.json` (i mappen du kjører den fra), `--max-features 2000`, `--ratio-threshold 0.75`, `--matcher bf`, `--knn-k 2`, `--ransac-threshold-px 3.0`, `--ransac-iters 2000`, `--ransac-confidence 0.99`, `--averaging mean`. (`project align calibrate` har egne standardinnstillinger; se [tabellen for denne](#project-align-calibrate-options).) `--prefilter` har som standardinnstilling `gradient` (kantkart; samsvarer med GUI/array-justeringsverktøyet — kantene beholdes på tvers av spektralbåndene). `--matcher flann` gir best resultat ved over ~5000 treff; `--averaging median` er robust mot én dårlig opptak, `inlier_weighted` vektlegger etter antall treff; `--lock-scale` projiserer til nærmeste rotasjon (ingen skalering), `--lock-axis` setter én translasjonskomponent til null; `--mask` gjelder for alle kameraer (bruk `--per-cam-override` for innstillinger per kamera, f.eks. `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` nekter å lagre en kalibrering hvis RMS-verdien for reprojeksjonen overskrider grensen. `--method manual` henter punktene sine fra `--points PATH`: et JSON-objekt `{"<serial>": [[x, y], ...], ...}` som angir hvert kameras pikselkoordinater for de samme trekkene, inkludert referansekameraet (minst 2 per kamera); hver av `--frames`-ene beregnes ut fra disse punktene. Når kameraene inngår i en tilkoblet matrise, bruker kalibreringen matrisens live-bilder, og profilen blir også matrisens justering for opptakene. Ellers åpnes en [midlertidig matrise](#multi-camera-transient-sessions): et kamera som ikke leverer noen ramme i noen av `--frames`-ene, mislykkes i kalibreringen (avslutt `1` og gi den et navn), og ingen profil lagres. |
| `lattice align-apply --profile PATH [--serials SN1,SN2,…] [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N] [--brightness N]` | Ta ett justert flerbåndsbild. `--bit-depth` bruker som standard innstillingen for det aktuelle kameraet; `--no-crop` beholder hele bildet (fyller ut med svart); `--interpolation` (standard `linear`) og `--border-mode`/`--border-value` (standard `constant`/0) styrer CPU-warp – GPU-banen er uansett bilineær. Åpner en [transient array](#multi-camera-transient-sessions): et kamera som ikke leverer noen ramme, mislykkes i opptaket (avslutter `1` og navngir det), og ingenting skrives. |
| `lattice align-stream --profile PATH [--serials SN1,SN2,…] [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N] [--brightness N]` | Strømjusterte flerbåndsrammer (samme warp-regulatorer som `align-apply`), `--fps` standard `3`. Åpner en [transient-matrise](#multi-camera-transient-sessions): en bildeserie som et kamera ikke har levert noe til, blir ikke lagret, og tre på rad avslutter strømmen. Hvis en gruppe ble hoppet over, avsluttes kjøringen ved å skrive ut `Incomplete: <n> frame group(s) not saved, a camera delivered nothing (SN:<serial> x<count>, …)` og `Error: …`, og avsluttes med `1`. En strøm som avsluttes på grunn av en hvilken som helst annen feil, skriver også ut `Error: …` og avsluttes med `1`. |
| `lattice align-info --profile PATH [--json]` | Vis profildetaljer. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Endre lagrekkefølge. |

#### Indeks / Vegetasjonsmatematikk

```bash
# Offline: compute NDVI from an aligned multi-band TIFF (an RGN camera's bands)
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: open the cameras, capture one aligned set with a profile made by
# `lattice align-calibrate`, and compute the index from it
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --save-multiband -o output/
```

**`--channel` tilordner symbolene i en forhåndsinnstilling til bånd** (NDVI: `red`, `nir`), én `--channel SYMBOL=BAND` hver.

- **`--live`: kun for et symbol som stakken ikke kan svare på.** Som med `--input`, tar et forhåndsinnstilt symbol du utelater det ene båndet i stakken (båndene til profilens aktiverte kameraer) hvis navn begynner med det. Dette sjekkes før et kamera åpnes: et symbol som ingen bånd, eller flere bånd, svarer på, avvises da, og ved å navngi dem, avsluttes `1`. En tilpasset `--formula` trenger en `--channel` for hvert symbol.
- **`--live`-produkter angir hvem de tilhører.** `index.tif` og `index_LUT.png` inneholder stakkens kamera (merke, modell, serienummer), opptakstidspunktet (referansekameraets eksponeringstid, UTC) og objektiv, og begge angir indeksen som sitt eneste bånd (`XMP-Camera:BandName` `NDVI`; farge-PNG-filen angir indeksen, ikke fargekanalene), slik et `--input`-produkt gjør. En indeks fra flere kameras bånd navngir i stedet hvert enkelt av dem, og dens `Model` er `LATTICE` — se [Filer laget fra flere kameraer](#files-made-from-several-cameras).
- **`--input`: kun for et symbol som filen ikke kan svare på.** Et symbol du utelater, tar det ene båndet i filen hvis navn begynner med det: `red` tar `Red_660` eller `Red_660_RGN`, mens `nir` tar `NIR_850`. Navnene hentes fra selve filen: stakkens egen båndliste, eller, på en Chloros-radianse-/refleksjonsopptak eller et `process`-produkt, dets `Chloros:Band{R,G,B}Name`-/`XMP-Camera:BandName`-koder. Så `lattice index --input <reflectance capture> --preset NDVI` trenger ingen `--channel`.
- **To bånd som svarer til ett symbol blir ikke gjetnet mellom** (en stabel med 9 kameraer har to NIR-bånd): gi det ett navn. Et symbol som ikke samsvarer med noe, avvises med `channel_map missing entries`, avslutt med `1`.**Båndnavn i en stabel** bærer kameraets filter: båndene til et M3C-kamera er `<band>_<filter>` — `NIR_850_RGN`, `Green_550_RGN`, `Red_660_RGN` på en RGN; `NIR_808_OCN`, `Cyan_490_OCN`, `Orange_615_OCN` på en OCN; `Blue_475_RGB`, `Green_550_RGB`, `Red_625_RGB` på en RGB; `Blue_475_NGB`, `Green_550_NGB`, `NIR_850_NGB` på en NGB — og et M3M-kameraeneste bånd er `Mono_<filter>` (`Mono_660`, `Mono_850`). `lattice align-info --profile PATH` viser båndene i en profil.

Fullt flaggsett: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND` (kan gjentas), `--capture-level raw|debayered|radiance|reflectance|unknown` (overstyrer opptaknivået som er registrert i kilde-TIFF-filen; standard: leses fra TIFF-metadata, med tilbakefall til en Chloros-fils `Chloros:ProcessingLevel`, slik at en reflektansopptak rapporterer `reflectance`), `--output PATH` / `-o` (offline: en sti; med en bildefiltype navngis filene ut fra stammen — `-o ndvi_FRGN.png` skriver `ndvi_FRGN.tif` og `ndvi_FRGN_LUT.png` — og uten en filtype er det et prefiks, `-o ndvi` -> `ndvi.tif`, `ndvi_LUT.png`; standard ved siden av inndata, som `<input name>_<preset>`; med `--live` og uten `-o`, en `output/`-mappe under mappen du kjører kommandoen fra), `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Med `--live` gjelder også justeringsknappene: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`, `--brightness N` (kameraenes mål for automatisk eksponering, se reglene for forbigående sesjoner), `--serials SN1,SN2,…`. `--live` åpner en [forbigående matrise](#multi-camera-transient-sessions): et kamera som ikke leverer noen bilde, fører til at kjøringen mislykkes (avslutt `1`, og gi det et navn), og ingen stabel eller indeks skrives. Den åpner `--serials` (det første utløser triggeren), ellers profilens kameraer der `layer_order`-oppføringen er aktivert, med referansen først — referansen selv om båndene er deaktivert, siden de andre er justert etter den og utløser utløseren. Et kamera med deaktiverte bånd blir aldri kalibrert, så et RGB-kamera som er deaktivert i profilen, mislykkes ikke lenger i en reflektansindeks (`no-absolute-radiance-scale`). En profil som ikke navngir noen kameraer, åpner alle kameraene på nettverket, som før.

Hvert skjema kjører på backenden, så den må være i gang: start den fra Chloros-GUI-en, eller med kommandoen som CLI-en viser når den ikke får kontakt med den. Offline leser backenden `--input` og skriver selv float-TIFF-filen og den fargelagte PNG-filen, så en relativ `--input` eller `-o` pekes mot mappen du kjører kommandoen fra (se [Konvensjoner](#conventions)). Begge offline-produktene inneholder kameraopplysningene fra inndataene, slik en `process`-indekseksport gjør: Merke / Modell / serienummer, GPS, objektivmerkene, opptakstidspunktet (inndataenes eget; når EXIF-dataene ikke angir noe, er det eksponeringstidspunktet som ble registrert eller tidsstemplet i filnavnet, i UTC) og MAPIR-kalibreringsidentiteten — aldri raw-DN-kalibreringen, som ikke gjelder for en indeks — pluss `Chloros:ProcessingLevel=index` og indeksuttrykket; float-TIFF-navnet angir ett bånd, forhåndsinnstillingen. `--list-presets` og `--list-gradients` skriver ut backendens forhåndsinnstillings- og gradienttabeller. En formel som backenden avviser, eller et `--channel`-bånd som TIFF-filen ikke har, avsluttes med backendens begrunnelse i `1`.

> **`--channel`-symboler er store og små bokstaver.** Symbolsiden må stemme nøyaktig overens med forhåndsinnstillingens kanalnavn (forhåndsinnstillinger bruker små bokstaver, f.eks. NDVI = `red`,`nir` — sjekk `--list-presets`), og båndsiden må samsvare med et båndnavn i den justerte stabelen (eller være en 0-basert båndindeks i offline-modus). `--channel red=Red_660_RGN --channel nir=NIR_850_RGN` fungerer; `--channel RED=660` mislykkes med en `channel_map missing entries` feil.

#### `lattice index` — flerbåndsstakkmasker

Ved `--capture-level radiance` eller `reflectance` kan hvert kameras 3×3-unmix klemme et bånd til 0 (se [Unmix-klemmaske](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on)). En flerbåndsstabel inneholder bånd fra flere kameraer, og hvert kamera klemmer sitt eget bånd, så `--live --save-multiband` skriver **én maske per kildekamera** ved siden av stabelen:

```
output/
├── index_multiband.tif
└── clamp_masks/
    ├── index_multiband_SN<serial A>-<filter A>_clampmask.tif
    └── index_multiband_SN<serial B>-<filter B>_clampmask.tif
```

- Hver maske er en enkeltbånds-`uint8` på **stakkens** pikselrutenett (gjennom samme justeringsforvrengning og beskjæring), med det aktuelle kameraets egne biter: bit 0 = båndet til R-kanalen, bit 1 = G, bit 2 = B. Et stabelbånd hentet fra kanal *c* på et kamera blir begrenset der kameraets bit *c* (R = 0, G = 1, B = 2) er satt.
- Hvert bånds innebygde opprinnelsesinformasjon angir kameraets maske (`unmix_clamp_mask`, i forhold til stakkens mappe, eller `none`) og biten som skal leses (`unmix_clamp_mask_bit`).
- Et kamera der unmix-prosessen ikke begrenset noe, får ingen fil; et mono- eller RGB-kamera kjører ingen unmix og får heller ingen. CLI-en skriver ut hver maske under stakken (`clamp mask SN…: …`); svaret lister dem opp som `clamp_masks`.
- Indeks-TIFF-/PNG-filen fra samme kjøring beregnes fra samme stabel, på samme rutenett, så maskene beskriver også denne. `--capture-level raw` kjører ingen unmix og skriver ingen maske.
- Stakken og indeksen ligger direkte i `--output`, så maskene havner i `--output`s egen `clamp_masks/` — aldri over mappen du navngav. Den mappen inneholder ulike produkttyper (stabel, indeks, fargelagt rendering): velg filene derfra, ikke overfør hele mappen til fotogrammetriprogramvaren.

#### Vedvarende tilkoblinger (Smart-Prep, GUI-ekvivalent flyt)

Disse kommandoene holder kameraene åpne i backend-poolen på tvers av CLI-kall.

| Underkommando | Formål |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Legg til ett kamera i puljen (enkeltkamera, ikke matrise). De [vanlige alternativene](#common-options-shared-by-most-subcommands) `--exposure`, `--gain`, `--preset`, `--pixel-format`, `--width` / `--height` og `--packet-size` (`standard` = 1500, `jumbo` = 9000, eller byte; `auto`-prober) brukes ved tilkobling (en `--exposure` eller `--gain` slår av automatisk modus). Den returnerer når kameraets strøm har startet (venter ca. 15 s på det, deretter 10 s på den første fullstendige bufferen) og skriver ut `stream: streaming` eller `stream: no_frames -- connected, no frames yet: …` når strømmen ikke leverte noe — for et kamera som bruker jumbo-pakker, med anbefalingen om å koble til på nytt med `--packet-size standard`. Mens ett kamera kobler seg til (en første tilkobling laster ned kalibreringen, ca. 70 sekunder), venter en annen kommando som trenger et kamera på det i maksimalt 5 sekunder, deretter mislykkes den med HTTP-feil 503 `CAMERA_POOL_BUSY`, der kameraet som kobler seg til, nevnes; kjør den på nytt om et øyeblikk. `lattice status` og kameralisten venter aldri: et kamera som er midt i tilkoblingen, vises som «kobler til», med sin fase. |
| `lattice cam-disconnect [--serial SN] [--all] [--keep-settings \| --clear-settings]` | Slipp. Kameraet settes tilbake slik det ble funnet ved tilkoblingen (utløserkobling, PTP, innstillinger); `--keep-settings` lagrer i stedet innstillingene i sitt eget minne, `--clear-settings` tilbakestiller dette minnet til fabrikkinnstillingene (se [Kameraminne](#camera-memory----keep-settings----clear-settings)). Et kamera som er medlem av en matrise frigjør hele matrisen. |
| `lattice cam-list` | Viser kameraer i puljen. |
| **`lattice array-connect`**|**Koble til en permanent synkronisert matrise (DET anbefalte inngangspunktet).** Kjører hele GUI-smart-prep-flyten. Tar omtrent ett minutt for 9 kameraer; rett etter at kameraene har slått seg på, kan det ta flere. Mens den venter, skriver den ut hvert trinn som backenden når, som `    <s> s: <step>` (for eksempel `    41 s: open SN:214000535 (4 of 9)`), og backendens logg har én tidslinje per trinn (`[array-connect <id>] <step>: <s> s …`) og en siste linje med de tregeste trinnene. Den venter i opptil 600 s. En tilkobling som ikke er fullført innen da, blir avbrutt av backenden: hvert kamera den åpnet, settes tilbake slik det ble funnet og lukket, kommandoen skriver ut `Error:` med trinnet den hadde nådd og avsluttes med `1`, og ingen matrise er tilkoblet (HTTP-feil 504 `CLIENT_DEADLINE_PASSED`). Ctrl+C avbryter tilkoblingen på samme måte (`Interrupted: …`); hvis arrayet allerede var tilkoblet, angir linjen dette og oppgir navnet på kommandoen (`array-disconnect`) som frigjør det. En kommando som avbrytes mens den venter (`kill -9`) fører til at tilkoblingen blir avbrutt når backenden ikke har hørt fra den på 30 sekunder. Et array som er oppe, men som ennå ikke har noen synkronisert rammegruppe, rapporteres fortsatt som tilkoblet, med sin strøm som `no_frames`. |
| `lattice array-disconnect [--array-id ID] [--all] [--keep-settings \| --clear-settings]` | Frigjør en matrise. Hvert kamera blir stående i frittløp (`TriggerMode` `Off`), med alt annet slik tilkoblingen fant det – innstillingene for utløser og synkroniseringslinje (masterenss `TriggerSource` tilbake til det den var, synkroniseringslinjen tilbake til en inngang), PTP-innstillingene og kamerainnstillingene som økten endret — slik at en kommando for ett kamera på et medlem virker umiddelbart. `--keep-settings` lagrer hvert kamerass innstillinger slik de var da økten ble avsluttet, i kameraets eget minne; `--clear-settings` tilbakestiller dette minnet til fabrikkinnstillingene. Se [Kameraminne](#camera-memory----keep-settings----clear-settings). |
| `lattice array-list` | Vis tilkoblede matriser. |
| `lattice array-status [--array-id ID]` | Live fps, PTP, siste feil. `ptp_enabled` er kun `True` når alle enheter har PTP aktivert; `ptp_off:` angir en enhet som ikke har det. `state:`-linjen leser `restarting (applying settings)` under en omstart av innstillingene, `recovering (…)` mens et medlem som mistet strømmen åpnes på nytt (se merknaden nedenfor), `stalled (…)` mens grab-sløyfen kjører, men ikke publiserer noen rammegruppe, og `stopped (…)` når grab-sløyfen har gitt opp (koble til arrayet på nytt). |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--orient] [--index\|--no-index] [--no-daq] [--daq-tolerance-ms MS] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Én synkronisert opptak fra live-matrisen — Enkelt / Kontinuerlig / Intervall / Raskest. **Standardinnstilling er `all`**(én fil per gjeldende eksporttype per kamera),**justert** i henhold til appens opptak (`--no-aligned` for native filer). Kameraer som er hoppet over (f.eks. RGB-kameraer ekskludert fra radianse/refleksjon) rapporteres med `Skipped: SN:<serial> (<reason>)`; DAQ-avlesningen som ble brukt til opptaket, lagres sammen med det, et opptak som kun inneholder rådata inkluderes, og rapporteres med `DAQ: <path>` ([DAQ-avlesningen som et opptak lagrer](#the-daq-reading-a-capture-saves)). Se [Opptakmoduser, opptakere og offline-bearbeiding](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only] [--expr EXPR]` | Ta opp den kombinerte indeksvisningen i sanntid som video/GIF (overvåkingskvalitet; uten åpen GUI-visning gjengir backenden matrisens NDVI, eller `--expr`). |
| `lattice array-burst [--duration S] [--max-frames N] [--orient] [--build] [--products …]` | Rå-Bayer-serieopptak med høy bildefrekvens (analyse-kvalitet; etterbehandle offline). I likhet med `array-record` og `array-capture --continuous` sjekker den inn med backenden ved hver avlesning: avsluttes uten å stoppe (`kill -9`), backenden stopper den 30 sekunder etter siste sjekk (`stop_reason` `client_gone`; seriens manifest blir ferdigstilt og oppbyggingen av den kombinerte indeksen startes, slik det skjer ved en stopp), og arrayet forblir tilkoblet. |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Behandle en lagret rå burst på nytt til kalibrert(e) video(er). |

> **Et kamera som mister strøm eller forbindelsen, kommer tilbake av seg selv.** Når et medlem slutter å svare (en strømbryter som slås av og på, starter hvert kamera den forsyner med strøm på nytt), venter matrisen på det, åpner det på nytt når det svarer igjen, og gir det tilbake det som ble angitt av `array-connect`: PTP, utløser- og synkroniseringslinjen, nettverkshastighet, pikselformat og bildestørrelse, samt automatisk eksponering. Deretter strømmer det igjen uten nye innstillinger fra `array-connect`. I mellomtiden viser `array-status` `state: recovering (...)`, og `last_error` angir hvilket kamera det venter på; etterpå skriver det ut `re-opened:`. Et kamera som har startet på nytt, kommer tilbake med PTP slått av, så inntil det åpnes på nytt, er `ptp_enabled` lik `False`, og `ptp_off:` navngir det.

> **Et kamera som holdes, åpnes aldri en gang til.** En ny åpning stopper strømmen til den som holder det og overskriver innstillingene, og kan føre til at backenden krasjer. Så på et kamera som poolen holder (GUI-en eller `cam-connect`), et array-element eller et kamera som en bakgrunnsoppgave holder (en kjørende `lattice continuous`, `multi-stream` eller `align-stream`, enten sin egen eller en annen klients), `lattice continuous`, `lattice capture`, `cam-connect`, `power --sleep`, `calibrate`, en-shot `white-balance` og `presets --apply` avviser med HTTP-kode 409 `CAMERA_HELD`, med angivelse av innehaveren og frigjøringskommandoen (`chloros-cli lattice cam-disconnect -s <SN>`, `chloros-cli lattice array-disconnect --array-id <ID>`, eller for en jobb, ID-en og hvem som startet den), og avslutt `1`. Det samme gjelder et kamera som en annen kommando bruker akkurat nå (en `multi-test` eller `multi-capture` som åpner kameraene sine, en annen engangskommando), for hver kommando her: feilmeldingen sier at man må vente til den kommandoen er ferdig. `lattice power` uten `--sleep` leser gjennom innehaverens eget håndtak der det finnes et (poolen, en matrise, et åpent kamera i en jobb), og blir avvist for et kamera som en annen kommando bruker. Hvert `CAMERA_HELD`-svar inneholder koden både i `code` og `error_code`. For kommandoene som åpner hvert kamera, se [Flerkamera (midlertidige økter)](#multi-camera-transient-sessions).

##### `array-connect`-alternativer

| Flagg | Standard | Beskrivelse |
| --- | --- | --- |
| `--serials SN1,SN2,…` | automatisk oppdage alle LATTICE-kameraer (krever ≥2) | Det første serienummeret er MASTER. Når dette utelates, filtreres oppdagelsen til LATTICE (M3C / M3M)-modeller, og alle kobles til. |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO-synkroniseringslinje. |
| `--target-fps F` | auto | Utløsningsfrekvens for MASTER. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Overstyr nivåvelgeren. |
| `--wire-ceiling-mbps MB_PER_S` | auto-detected | **Vertens vedvarende båndbredde, i MB/s — tallet som hele array-tildelingen avhenger av.** Senk den når arrayet rapporterer GVSP-korrupte rammer: den automatiske verdien er avledet fra nettverkskortets oppgitte koblingshastighet, som overvurderer USB-adaptere, smale PCIe-baner og travle delte nettverk. Lagres i prosjektets array-fangstblokk, slik at en gjenåpning eller en tilkobling via CLI/SDK gjenoppretter den. Se [Array-tilstand](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | ingen (full sensor) | Maskinvarebinning (alltid gjennomsnitt). |
| `--width N` / `--height N` / `--pixel-format FMT` | full sensor; format per kamera | En eksplisitt bildestørrelse eller et eksplisitt pikselformat for hvert kamera, i stedet for valg av full sensor og format per kamera (Mono12 / BayerRG12). |
| `--no-recommend` | av | Hopp over trinnet med nettverksanalyse (hvert kamera beholder da sin nåværende størrelse i stedet for å bli satt til hele sensoren). |
| `--no-ptp` | av | Deaktiver PTP (tidsstempler på tvers av kameraene er da **ikke** sammenlignbare). |
| `--packed {auto,on,off}` | `auto` | Strøm 12-bits rammer pakket og pakk ut på denne datamaskinen — flere rammer per sekund ved full sensor, identiske data. `auto` pakker bare når det er nyttig og denne datamaskinen har kapasitet til det; `off` pakker aldri. Se [Pakket 12-bits transport](#packed-12-bit-transport---packed). |
| `--packet-size {auto,jumbo,standard,N}` | `auto` | GVSP-pakkestørrelse for hvert kamera i matrisen. `auto` lar hvert kamera undersøke nettverket; `jumbo` = 9000, `standard` = 1500, eller en størrelse i byte låser det fast. Før versjon 1.3.0 aksepterte `array-connect` dette flagget og ignorerte det. |

> **Full sensor som standard.** Uten `--width` / `--height` / `--binning` /
> `--pixel-format`, setter `array-connect` hvert kamera til full sensor — ingen
> binning, ingen ROI-beskjæring — og lar bildefrekvensen falle til det denne datamaskinen og
> nettverket klarer å levere, i stedet for å redusere bildene for å opprettholde frekvensen. Den
> nettverksanalyselinjen viser den forventede frekvensen, og neste linje viser
> raskere alternativer (binning eller en mindre ROI) med deres forventede frekvenser; gå
> `--binning` eller `--width`/`--height` for å velge et av dem. Connect utfører aldri binning eller beskjæring
> på egen hånd.

##### Pakket 12-bits transport (`--packed`)

Ved full sensorinnstilling bestemmes bildefrekvensen for et array av hvor raskt hvert kamera kan sende
sine bilder. 12-bits formater (`BayerRG12`, `Mono12`) overføres normalt som 16-bits
ord; **pakket** overføring sender de samme 12 bitene i 1,5 byte per piksel og
pakker dem ut på denne datamaskinen, slik at hvert bilde tar 25 % kortere tid å sende —
omtrent **33 % flere bilder per sekund ved full sensor** (målt: 10,7 → 14,0 fps
på et oppsett med 9 kameraer). **De lagrede dataene er identiske**: samme verdier, samme
16-bits filer, samme metadata.

- `auto` (standard) pakker kun når det er fordelaktig **og** denne datamaskinen har kapasitet til
  å pakke ut. Ved tilkobling måler backenden sin egen utpakkingshastighet, antall CPU-kjerner,
  nåværende CPU-belastning og nettverkskapasitet, slik at en stasjonær PC med ledige kjerner komprimerer
  ved full sensor, mens en opptatt eller strømsparende datamaskin streamer ukomprimert. Små ROI-er og binned bilder gir
  lite.
- `off` komprimerer aldri — bruk den for å utelukke komprimering under feilsøking, eller på en
  datamaskin som må beholde all sin CPU-kapasitet til annet arbeid.
- `on` pakker alltid 12-bits formater.

Connect-utdataene viser beslutningen og begrunnelsen (`pakket 12-bits: på (auto:
projisert 11,0 -> 14,4 fps, …)`); `/api/camera/array/status` rapporterer
`packed_transport` og `packed_decision`. For å angi en standard for hele maskinen
i stedet for å overføre flagget hver gang, sett `CHLOROS_ARRAY_PACKED12=off` (eller
`on`) i backendens miljø — et eksplisitt `--packed on|off` har fortsatt forrang.
8-biters og 16-biters formater pakkes aldri.

### Én prosjektpost, alle moduser

En prosjektmappe inneholder innstillingene for alle enheter: `cameras.json` — hvert kamera (`cameras[].settings`, for et enkelt kamera og for hvert medlem i en matrise) og de matriseomfattende innstillingene for hver matrise (`arrays[].capture`: format, ROI, binning, utløserfrekvens, ledningsbudsjett) — samt `sensors.json`, hver lyssensors (integrasjonstid, ramme-gjennomsnitt, cap). Appen, CLI, SDK og MCP-serveren leser og skriver alle de samme filene på samme måte:

- **En enhet som er koblet til et prosjekt, lagres i dette og registrerer alle endringer der.** «Under et prosjekt» betyr med et prosjekt åpent i backend (appens), eller — fra SDK-et — gjennom en `ChlorosProject`, som angir sin egen mappe. Et kamera, en matrise eller en lyssensor som prosjektet ennå ikke inneholder, legges til i filene når den kobles til (`lattice cam-connect`, `array-connect`, `daq pool-connect`, appen, SDK, `chloros-mcp`), sammen med innstillingene sine. En endring fra en hvilken som helst modus (`lattice exposure`, `gain`, `configure --set`, `resolution`, en matrisesinnstillinger og utløserfrekvens, en lyssensors grenseverdi, appens kontroller) lagres i prosjektets filer et øyeblikk senere. En eksponering, forsterkning eller AE-målverdi som er angitt på et kamera, lagres som en innstilling, slik at den beholdes når prosjektet åpnes på nytt.
- **Når et prosjekt åpnes, hentes enhetene fra det**, uansett modus: et kamera, en matrise eller en lyssensor som er tilkoblet uten egne innstillinger, får prosjektets lagrede innstillinger (`lattice cam-connect` / `array-connect` med et åpent prosjekt, `chloros-cli project connect`, `open_project().connect_all()`, når appen åpnes på nytt). Et array som kobles til under en ny ID, oppdaterer det arrayet prosjektet lagret med de samme kameraene; et array med andre kameraer legges til ved siden av det.
- **Appens egen lagring overskriver aldri en enhets live-innstillinger** med en eldre kopi av dem, og beholder overskrivninger per medlem som er lagret av en CLI- eller SDK-kaller.
- **Pikselformatet du angir, er det formatet prosjektet beholder.** Et format angitt i Array Settings, via `--pixel-format` eller via `connect_array(pixel_format=)`, inkludert 8-bit, lagres og gjenopprettes nøyaktig. Hvis ingenting er angitt, kobles hvert kamera til med sin 12-bits standardinnstilling (`BayerRG12`, `Mono12` på en monosensor). Et formatnavn angir bitdybden; familien tilhører sensoren, så et fargekamera blir aldri satt til et monofargeformat eller et monokamera til et Bayer-format, og linjen `[PixelFormat <serial>] ... written` angir dette når et format ble tilordnet. Eksportene fra 8-bits opptak eksporteres som 8-bits, inkludert refleksjonsdata: omskalert og merket med `PixelScale`.
- En enhet som er koblet til uten å være tilknyttet et prosjekt, tar ikke opp noe.
- **En opptak følger et prosjekt som er opprettet etter den; når prosjektet lukkes, forblir opptaket i prosjektet.**Når et prosjekt opprettes — et SDK-prosjekt `create_project()`, `chloros-mcp`s `create_project` eller appens**Nytt prosjekt**— eller dupliseres (appens**Dupliser**, som åpner kopien) mens en opptak fortsatt skriver til prosjektet som var åpent før, fortsetter opptaket, og det som skrives havner i det nye prosjektet, på samme sted i det (under). Åpning av et eksisterende prosjekt — `chloros-cli process`, `project devices --reconnect`, `ChlorosProject.process()`, appens**Åpne prosjekt** — etterlater en opptak som fortsatt pågår i prosjektet den tar opp i: den fullføres der, og filene blir lagt til i det prosjektets filliste (appen stopper alle opptak før den åpner et prosjekt).
  - `array-record` og `array-burst`: videoen, eller burst-mappen med hvert enkelt bilde, flyttes i sin helhet dit når opptaket stopper (det er `--duration`, Ctrl+C). Den `Burst dir` den skriver ut, er den nye, og `--build` og `array-build-video` bygger videre på den. Hvis en flytting avviser av stasjonen, forblir opptaket der det ble skrevet; loggenloggen for slutten forklarer hvorfor, og det samme gjør HTTP-svaret ved stopp (`relocation_error`). De rå bildene fra en stoppet burst og `.daq` legges til i fillisten til prosjektet den havner i; en video (en `array-record`, en video laget av en burst) legges ikke til i et prosjekts filliste.
  - `array-capture --continuous`: alle opptak etter endringen lagres i det nye prosjektet; de som ble lagret før, forblir der de er.
  - `daq pool-record` og `project sensor log` (samt appens egen lyssensoropptak): `.daq` flyttes til det nye prosjektets`light_sensor`-mappen når opptaket stopper, og blir lagt til i det prosjektets filliste.

  Et opptak i en mappe utenfor det åpne prosjektet (en `-o` et annet sted) forblir der den er, og `lattice continuous` og `multi-stream` beholder mappen de ble tildelt.
- **I appen havner opptakene etter et nytt prosjekt i det.** Når du oppretter eller dupliserer et prosjekt, fortsetter alle pågående opptak og innspillinger, og hvert opptak som tas etter det (Capture All, Start Capturing) lagres i det nye prosjektet og blir lagt til i filisten der. Å åpne et eksisterende prosjekt i appen stopper først alle disse.
- **Et prosjekt flyttes ikke mens det tar opp.**Å flytte det åpne prosjektet til en annen arbeidskatalog (appens**Innstillinger → Arbeidskatalog**; ingen CLI-kommando flytter et prosjekt, men en HTTP-klient som sender til `/api/move-project` gjør det) avslås med HTTP 409 `PROJECT_RECORDING` mens en matriseopptak eller burst, et kontinuerlig matriseopptak, en `lattice continuous` / `multi-stream`-jobb eller en lyssensoropptak skriver til den: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.` Appen avviser på samme måte, før den ber om en mappe, mens dens egen «Capture All», «Start Capturing» eller lyssensoropptak kjører.

Dette er uavhengig av hvordan kameraene står når forbindelsen brytes (se [Kameraminne](#camera-memory----keep-settings----clear-settings)): et prosjekt konfigurerer kameraene selv når det åpnes.

### Kameraminne — `--keep-settings` / `--clear-settings`

Når et array er tilkoblet, overtar kameraene innstillingene til arrayet, og du kan deretter endre dem. Når Chloros kobler fra kameraene — `array-disconnect`, slutten på en midlertidig kommando (`multi-capture` uten matrise, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`), slutten på `lattice continuous`, slutten på en `lattice capture` for et kamera den ikke har — den setter hver enkelt tilbake slik den var før tilkoblingen: utløser- og synkroniseringsledningsforbindelsene, PTP- og kamerainnstillingene (den fullstendige listen finnes under [Flerkamera (midlertidige økter)](#multi-camera-transient-sessions)).

For å bruke kameraene utenfor et prosjekt, skriver to flagg på hvilken som helst av disse kommandoene til kameraets **eget minne** — det det starter opp med, uansett programvare:

- **`--keep-settings`** lagrer hvert kameras innstillinger slik de var da økten ble avsluttet, i kameraets minne (UserSet2, som utgjør oppstartsinnstillingene). Kameraene forblir i frittløp (`TriggerMode` `Off`), med synkroniseringsledningene slik de er. Et kamera uten UserSet2 lagrer i UserSet1; denne datamaskinen husker dette, så Chloros’ matrise med selvreparasjon — noe som tilbakestiller et UserSet1 som har avviket fra Chloros’ referanseverdi — lar det være i fred.
- **`--clear-settings`** tilbakestiller hvert kameras minne til fabrikkinnstillingene: Chloros’ referanseverdi, den som arrayets selvreparasjon innbaker — fabrikkinnstillinger med `TriggerMode` `Off`, `TriggerSource` `Line2`, automatisk eksponering, et RGB-kameras kalibrerte hvitbalanse beholdes — i UserSet1, som utgjør oppstartsettet.
- De to kan ikke kombineres. Hvert kamera får en linje: `SN:…: settings saved to the camera's memory (UserSet2); it powers up with them.` eller `SN:…: camera memory reset to the factory settings.`; svaret `camera_memory` har det samme per kamera.

Appen tilbyr ikke disse flaggene.

**Hver utgang setter kameraene tilbake.**Den samme tilbakestillingen som ved oppstart kjører på hver utgangsvei, ikke bare kommandoene ovenfor: `cam-disconnect` (og appens kamera**X**); appens**X** på en matrise eller på en hvilken som helst rad i et matriseelement (en matrise kan ikke kjøre videre uten et element, så hele matrisen frigjøres); å lukke appen (den gir backenden opptil 20 sekunder på å sette tilbake hvert kamera, matrise og kjørende `lattice continuous` tilbake før den stopper den; en kommando som fortsatt kjører da — en forbigående kommando som åpner kameraene sine eller venter på PTP-klokkene deres, en engangskommando `lattice capture`, `power`, `white-balance` eller `presets --apply` – stopper ved neste sikre punkt, setter først kameraene tilbake, i opptil 8 sekunder, og svarer med HTTP-kode 503 `SHUTTING_DOWN`; en «array connect» som fortsatt kjører — `lattice array-connect`, appens tilkobling — stopper før neste kamera og lukker kameraene den har åpnet, i opptil 10 sekunder, og svarer med HTTP-koden 503 `SHUTTING_DOWN`; og en som kommer inn under nedstengingen, åpner ingenting: HTTP-koden 503 `SHUTTING_DOWN`); og en CLI-kommando som avsluttes eller hvis terminal lukkes (SIGTERM / SIGHUP stopper jobben — og setter kameraene tilbake — før den avsluttes `143` / `129`, som Ctrl+C). En `lattice continuous`, `multi-stream` eller `align-stream` som avsluttes umiddelbart (SIGKILL), og som ikke kan stoppe noe, får jobben sin stoppet og kameraene satt tilbake av backend-systemet 30 sekunder etter den siste innsjekkingen. En backend som krasjer eller blir tvangsavsluttet, kan ikke gjenopprette noe; en strømavbrudd og påslag setter hvert kamera tilbake til sitt minne, og den neste Chloros-tilkoblingen kobler om et kamera den finner aktivert.**Et prosjekt konfigurerer kameraene sine selv.** Når et array kobles til mens et prosjekt er åpent, får hvert medlem innstillingene prosjektet har lagret for det – en eksponering, forsterkning eller AE-mål som er lagret for det (lagret som ditt, slik at automatisk eksponering lar det være i fred) og dets hvitbalansemodus (korrigering fra Chloros; kameraets egen hvitbalanse forblir deaktivert) – uansett hva kameraets minne inneholder.

### Smart-AE / Smart-Capture

LATTICE-oppsett kjører kontinuerlig AE i bakgrunnen så snart de er koblet til, men det tar et øyeblikk før en nyinnrettet scene konvergerer. `array-capture --smart` er den **pakkede løsningen**: den venter på at AE skal stabilisere seg på alle kameraene i matrisen, og utløser deretter opptaket. Bruk den når du bytter motiv midt i en økt.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Standardinnstillingen for stabilisering er konservativ: 5 sekunders tidsavbrudd, 1,5 sekunders stabilitetsvindu, ±5 % toleranse for eksponeringsspredning. Juster via SDK-et (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) hvis du trenger en annen oppførsel fra automatiseringen.

#### Hvordan automatisk eksponering fungerer

- **Eksponering først, forsterkning sist.** Når forsterkningen er på «Auto», forblir den på 0 dB inntil eksponeringen har nådd den lengste tillatte verdien.
- **En manuell eksponering lar forsterkningen forbli på «Auto»** inntil du endrer forsterkningen eller setter den til manuell. Når den går tilbake til «Auto», er forsterkningens grense den samme som før.
- **Små høylys gjør ikke scenen mørkere.** En speilrefleks på opptil 1 % av bildet ignoreres, og i panelene til et kjent kalibreringsmål, opptil 0,5 % av et panel.
- **DAQ-eksponeringstaket gjelder kun mens et kalibreringsmål er i synsfeltet.** Det hindrer at et hvitt panel blir klippet; når det ikke er noe panel synlig, eksponerer kameraet for scenen.
- **Konstant lys forblir stabilt, og endret lys stabiliserer seg i løpet av sekunder.** Ved konstant lys forblir eksponeringen og dens grense uendret, og etter en stor endring i lyset stabiliserer bildet seg i løpet av få sekunder. En utklipping begrenser ikke lenger eksponeringen permanent, og en endring i ROI finner eksponeringen igjen.
- **Lysstyrkemålet holdes innenfor rekkevidde.** Det stiger ikke mens kameraet er ved eksponeringsgrensen og forsterkningen ikke kan økes, slik at et lys som kommer tilbake ikke forårsaker overbelysning i bildet.
- **Et kamera hvis automatiske eksponering slutter å reagere, holdes på nivået for det lyset det befinner seg i.** I sterk sol kan et kamera falle til sin korteste eksponeringstid og forbli der med et mørkt bilde, selv om alle innstillingene viser normale verdier. Chloros overvåker bildet, holder kameraet på det nivået det holdt for det lyset, og slipper det når lyset endres. En mørk eller tildekket lyssensor forstyrrer ikke dette.

### Eksportnivåer for opptak (standardinnstillingen for `all`)

Fra og med denne utgaven, `lattice capture`, `lattice multi-capture` og `lattice array-capture` **er standardinnstilt til `--processing all`** — én lagret fil per eksporttype som gjelder for hvert kamera, i tråd med «Capture All»-funksjonen i brukergrensesnittet. Nivåene er:

| Nivå | Utdata | Gjelder for |
| --- | --- | --- |
| `raw` | Enkanals Bayer (monokameraer: det ene båndet) direkte fra sensoren. | Alle kameraer. |
| `debayered` | 3-kanals farge, lagret i **RGB**-kanalrekkefølge (en TIFF-fil med RGB-fotometrisk tolkning; PNG/JPEG åpnes også som RGB): kalibreringspakkens mørkeforskyvning (DSNU) fjernet fra rådataene, deretter OpenCVs kantdemosaik – ingen linearitets-LUT, flatfelt eller unmix (monokameraer: 1-kanals gråtoner). De debayerte bildene i minnet som Python-SDK-et returnerer (`grab()`, `frame_stream()`) er**BGR**, i OpenCVs rekkefølge; bare den lagrede filen er i RGB-format. | Alle kameraer. |
| `radiance` | float32 W/m²/sr/nm via den fullstendige radiometriske kjeden. | Kun multispektrale (M3C/M3M) — **hoppes over for kameraer med RGB-filter**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), klar for Pix4D. | Kun multispektral, og **kun når en DAQ er tilknyttet + kameraet er kalibrert**; ellers hoppes over. |
| `display` | Full GUI-forhåndsvisningskjede (CCM + WB + gamma i henhold til kameraets profil). Lagret i en `preview/`-mappe. `lattice capture` aksepterer også navnet `preview` for den; `array-capture` / `multi-capture` godtar kun `display`. | Alle kameraer. |

Angi et enkelt nivå for å lagre bare det ene (`--processing debayered`). Når du ber om `all`, hoppes nivåer som ikke gjelder for en gitt kam over (og rapporteres), ikke som feil — et ikke-tilkoblet eller ikke-kalibrert kamera får fortsatt `raw` / `debayered` / `preview`.

#### DAQ-målingen som en opptak lagrer

Hver gang en DAQ er knyttet til kameraet eller matrisen, eller er samlet i en pool, skrives den nedadrettede målingen som ble brukt i et opptak til en **`.daq`**-fil i undermappen `daq/` i opptaksmappen, med sin `.match.json` (hvor langt avlesningen var fra eksponeringen) og sin `.cameras.json` (hvem avlesningen tilhører), og rapporteres på en `DAQ:`-linje. Dette gjelder for hvert nivå, inkludert råopptak og debayerte opptak, slik at ethvert opptak senere kan bearbeides på nytt til radianse, reflektans eller en indeks. Det er den samme filen, med samme navn og sidecars, uansett om opptaket kom fra GUI, CLI eller SDK. Uten noen DAQ-avlesning i det hele tatt er det ingen `.daq`, og en `multi-capture` uten tilkoblet matrise (den midlertidige reserveoppløsningen) lagrer ingenting.

| Flagg | Effekt |
| --- | --- |
| `--no-daq` | Ikke lagre avlesningen på en opptak der nivåene ikke bruker den (`raw`, `debayered`). Et `radiance`-, `reflectance`- eller `display`-nivå skriver den fortsatt, fordi reimport trenger måleverdien som produktet ble laget med. |
| `--daq-tolerance-ms MS` | Hvor langt fra eksponeringen målingen kan ligge og fortsatt regnes som innenfor toleransen (standard 500). Bare vurderingen endres (`.match.json`s `in_tolerance` og `tolerance_ms`, reflektansens `DlsInTolerance`), aldri måleverdien som brukes. Se [En DAQ-måling langt fra eksponeringen](#a-daq-reading-far-from-the-exposure). |
| `--force-daq` | Den eldre måten å be om `.daq` på ved en opptak som kun er i råformat. Dette er nå standard, og `lattice capture` og `array-capture` godtar fortsatt flagget (`multi-capture` gjør det ikke). `--no-daq` har forrang over det. |

Hvilken DAQ: den som er knyttet til kameraet eller matrisen (valg av lyssensor i GUI-en, et prosjekts lagrede tilknytning eller `chloros-cli daq pool-bind`), ellers den nyeste som backenden har. En rå serie (`array-burst`) beholder sin egen `.daq` per enkelt avlesning i burst-mappen, som før.

**PNG og JPEG har 8 bit per kanal, så de bruker kun `display` / `preview` og indeksprodukter.** `--format png` eller `jpg` med et nivå de ikke kan håndtere, avvises før noe blir tatt opp, og nivået og TIFF-formatet som kan håndtere det, angis:

- `radiance` (float32) og `reflectance` (uint16, `32768` = 1,0) — alltid, så `--processing all` (standard) med `--format png`/`jpg` avvises også: bruk `--format tiff` eller `--levels display`.
- `raw` og `debayered` fra et kamera som strømmer mer enn 8 bit (de 12-bits M3C / Mono12 M3M-rammene) — CLI-et kan ikke se kameraets bitdybde, så backenden avviser disse (HTTP-kode 400). Den enkeltståenderammen `lattice capture --processing debayered` henter en 8-bits ramme for PNG/JPEG og blir ikke avvist, og det samme gjelder en `multi-capture` uten en vedvarende matrise (8-bits debayered).

Alle opptaksruter svarer på samme måte (`error_code` `FORMAT_CANNOT_HOLD_LEVEL`): `lattice capture`, `multi-capture`, `array-capture`, `project capture` / `burst`, samt HTTP-rutene bak dem.

### Slik ser en opptaksmappe ut

Hver eksporttype havner i sin **egen undermappe**under `-o`, slik at et opptak på flere nivåer aldri blander typer. Skissen viser**standardopptaket `all` fra ett kamera av `lattice capture`** — flere nivåer per kamera, slik at hver fil har suffikset `_<level>`:

```
output/
├── raw/           capture_<ts>_SN<serial>-<filter>_raw.tiff
├── debayered/     capture_<ts>_SN<serial>-<filter>_debayered.tiff
├── radiance/      capture_<ts>_SN<serial>-<filter>_radiance.tiff
├── reflectance/   capture_<ts>_SN<serial>-<filter>_reflectance.tiff
├── preview/       capture_<ts>_SN<serial>-<filter>_display.tiff
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_radiance_clampmask.tif, reflectance/…, index/… — the unmix clamp
│                  masks, one subfolder per product folder, only when a band clamped
└── daq/           daq_data_<sensor id>_<ts>.daq: the downwelling reading matched to the capture
                   (+ .daq.cameras.json: whose)
```

Hvordan filnavnene varierer:

- **Kamerafelt — serienummer, deretter filter:** hver opptakfil navngir kameraet som tok bildet som `SN<serial>-<filter>`, der `<filter>` er filterdelen av kameramodellen: `SN261902142-F450` for et `M3M-L87-F450`, `SN213800234-FRGN` for et `M3C-L87-FRGN`, `SN214701292-FRGB` for et RGB-kamera. Filteret kobler serienummeret til `-`, ikke `_`, så et navn har de samme `_`-separerte feltene som før (prefiks, dato, klokkeslett, millisekunder, kamera, nivå). Et kamera hvis modell ikke kan leses (en slettet DeviceUserID) får navnet `SN<serial>` alene. Opptak fra tidligere versjoner har ikke noe filter i navnet; Chloros-leser begge deler.
- **Prefiks:** `lattice capture` navngir filene `capture_…`; `array-capture`, `multi-capture` med en tilknyttet matrise, og `project capture` / `burst` / `stream` av en rekke, navngir dem `sync_<ts>_SN<serial>-<filter>…`.
- **Nivåsuffiks kun ved flere nivåer.**Et kamera som lagrer**flere**nivåer i én opptak (`all` eller `--levels a,b`) legger til `_<level>`. Et kamera som lagrer**ett** nivå beholder det vanlige navnet uten suffiks — `--processing reflectance` skriver `reflectance/sync_<ts>_SN<serial>-<filter>.tiff` — slik at skript skrevet for opptak med ett nivå fortsatt fungerer. Nivået angis fortsatt av mappen.
- **Klemmasker følger produktnavnet:** `<product name>_clampmask.tif`. En opptak med ett nivå av refleksjonsmåling skriver `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`; et opptak med `all` skriver `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`.
- **Filendelsen** følger etter `--format`: standardverdien `tiff` gir `.tiff`, mens `--format tif` gir `.tif` (klampemasker er alltid `.tif`).
- **Ett unntak:** `lattice capture --processing debayered` (eller `--levels debayered`) tar en rask enkelt-rammebane og skriver `capture_<ts>_SN<serial>-<filter>.tiff` direkte inn i `-o` — uten undermappe.
- **Strømmer og bursts** har det samme feltet: `lattice continuous` skriver `frame_<seq>_SN<serial>-<filter>.tiff`, `lattice multi-stream`, `stream_<seq>_SN<serial>-<filter>_<frame id>.tiff` og en rå burst `<burst>_f<frame>_SN<serial>-<filter>_raw.tiff`.

`<ts>` er tidsstempelet for opptaket og `<serial>` er kameraets serienummer, slik at én synkronisert gruppe deler et
felles tidsstempel på tvers av kameraene. **Legg merke til den ene asymmetrien:** `display`-nivået lagres i en mappe
med navnet `preview/`, mens filene på flere nivåer beholder `_display` i navnet — mappen og suffikset skiller seg
kun for dette nivået. Ukjente nivåer faller tilbake til en mappe med sitt eget navn, og hvis undermappen
ikke kan opprettes, skrives filen til utdataroten i stedet for å gå tapt.

**Gjennomføring av ny behandling av en opptaksmappe:**pek `chloros-cli process` mot**captures-rotmappen**
(`output/`). `process` importerer normalt bare den mappen du angir, men når den mappen ikke inneholder
bilder, men har undermapper, går den automatisk ned i undermappene — slik at undermappene på hvert nivå og
`daq/`-undermappen blir hentet inn på én gang. Hvert nivå i en opptak importeres som ett enkelt bilde, mens
de andre nivåene er tilgjengelige som moduser, i stedet for ett bilde per nivå.

Det fungerer også å navngi en **nivåundermappe** direkte (f.eks. `output/raw/`). Da blir `daq/`
igjen, så kopier eller pek inn DAQ-avlesningen samtidig når duutleder et radiometrisk
produkt fra `raw/` – ellers har tidsstempelet ingenting å sammenligne seg med.

**Behandlingen starter alltid fra `raw`.** Innenfor hver opptak er råbildet kilden til behandlingskjeden;
`debayered`, `radiance`, `reflectance` og `preview` følger med som visningsmoduser, men føres aldri
tilbake gjennom rørledningen. Å behandle et avledet produkt på nytt ville innebære å påføre vignett, CCM og
strålingsberegninger på nytt – noe som allerede er innbakt i pikslene – så Chloros-programmet avviser dette i stedet for å
utføre dobbel behandling. To konsekvenser det er verdt å kjenne til:

- Renderingene `index/` og `composite/` blir **aldri** behandlet. De er utdata, ikke opptak —
  en NDVI-LUT-rendering har ingen meningsfull tolkning av strålingsdata.
- En opptaksmappe eksportert **uten** `raw` (f.eks. `array-capture --processing reflectance`) har
  ingen gyldig pipeline-kilde. Disse opptakene importeres og vises normalt, men `process` hopper over
  dem og melder følgende:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Hvis du virkelig trenger å sende et avledet produkt videre — en hub-økt som er tatt opp med
  `demosaic` aktivert, eller en eldre mappe — tvinger `--input-level {raw,debayered,processed}` inn
  inngangspunktet og overstyrer hoppet. Dette flagget er en bevisst nødutgang; `auto` (standardinnstillingen)
  behandler aldri en opptak som ikke har rådata.

**Kalibreringsmerker: kun på `raw`.** `raw`-filen til en opptak inneholder kameraets kalibreringspakke: nyttelasten `CalibrationJSON` og de MicaSense-kompatible taggene (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), som beskriver sensorens råverdier. Alle andre nivåers filer (`debayered`, `display`, `radiance`, `reflectance`, `index/`-renderingen) og hver `lattice continuous` / `multi-stream`-ramme inneholder kameraets identitet (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) og bildets egen post, `XMP-mapir:FrameJSON` (eksponering, forsterkning, temperatur, pikselformat, opptakstid, DAQ-samsvar), ettersom hvert `process`-produkt bærer denne identiteten. `process` finner kalibreringen til en slik fil ved hjelp av serienummeret og `CalBundleSha256` i Chloros’ kalibreringsbuffer, og laster ned pakken én gang hvis datamaskinen mangler den.

#### Filer laget av flere kameraer

En fil laget av bånd fra mer enn ett kamera – en multibåndsstabel (`align-apply`, `align-stream`, `index --live --save-multiband`), en live-indeks laget av flere kameraer (`index.tif`, `index_LUT.png`), et bilde fra en array-opptak (`composite/`) — navngir hvert kamera som inngikk i det:

- **Lister per kamera, i båndrekkefølge** (sammensetningen: bakgrunnslag, deretter forgrunn; et indekslag, kameraene som båndene stammer fra): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (kalibreringspakken som hvert kameras bånd gikk gjennom, ellers — en `align-apply` / `align-stream`-stabel, hvis bånd ikke har gjennomgått noen — kameraets pakke som Chloros har lagret i hurtigbufferen, den som opptaksfilene er oppkalt etter), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — parallelle lister, hvor element *n* i hver beskriver det samme kameraet; `-` der et kamera ikke har noen verdi (ingen kalibreringspakke, et ukjent objektiv). Hvert kamera er **én** post: der pakken har flere verdier for et felt — for eksempel en pakkes halogenlampesertifikater — kobles de sammen med `;` (`F-1479-1;F-1480-1;F-1481-1`), aldri med komma, slik at en liste viser én post per kamera uansett hvordan den er utskrevet.
- **EXIF `SerialNumber`** inneholder de samme serienumrene, skilt med komma (`214000535,214000551`).
- Den inneholder **ingen** identitet for et enkelt kamera (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): disse angir ett kamera og dets pakke, og `process` vil beregne filens kalibrering ut fra disse. En indeks beregnet fra flere kameras bånd (`index.tif`, `index_LUT.png`) har EXIF-dataene `Model` `LATTICE` (riggen), ikke én enkelt kameras `LATT-…`-modell.

En fil med båndene fra **ett** kamera — en TIFF-fil (`--per-band`), et `align-apply --per-camera`-bilde, en stabel eller live-indeks fra et enkelt kamera — inneholder det aktuelle kameraets egne identitetsmerker, slik alle andre opptaksfiler gjør. Stakken til `align-apply`, filene per bånd og per kamera, samt hver `align-stream`-stakk og dens filer per bånd, inneholder gruppens opptakstid (referansekameraets eksponering, UTC), og et monokameras `--per-camera`-ramme er ett plan, på `--bit-depth` akkurat som stakken.**Kameras orientering (speilvendt / rotert).** Et kameras orienteringsinnstilling (GUI-ens Mirror H /
Mirror V / rotasjon) bestemmer hva som lagres ved en opptak:

- **`raw` og `debayered` lagres nøyaktig slik sensoren leste dem**, uansett innstilling. Et
  Bayer-mosaikk som er rotert eller speilvendt, vil ha et annet fargefiltermønster, og kalibreringskartene
  (mørkforskyvning, flatfelt) ligger på sensorens eget rutenett.
- **`preview`/`display`, `radiance`, `reflectance` og `index/`-renderingen er orientert** — rotert
  én gang, etter at all kalibreringsberegningen er utført, slik at de åpnes slik kameraet var innrammet. Deres
  `clamp_masks/…_clampmask.tif`-filene ligger på samme rutenett.
- **Hver opptaksfil registrerer innstillingen** i XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, med klokken, etter
  speilvendingen) og `ChlorosOrientationApplied` — `false` på rådata og debayerte (innfødte piksler),
  `true` på de orienterte produktene. `ChlorosProcessing` viser operasjonene som er brukt på filens
  piksler (`none` når det er native). Dette inkluderer hvert bilde som `lattice continuous` strømmer (
  kameraets egen innstilling, ikke anvendt) og hvert bilde i `lattice multi-stream` (identiteten: dets
  medlemsbilder er native, som ved en array-opptak). `lattice align-stream` / `align-apply`
  -stabler registrerer hovedkameraets innstilling, uten at den er anvendt: båndene deres blir forvrengt inn i hovedkameraets
  bilde, men ikke rotert.
- **`process` bruker den registrerte orienteringen** når den behandler et rådata: mørkforskyvning, flat
  feltkorrigering, demosaikering og demiksing i sensorkoordinater først, deretter forvrengning for matrisejustering, og til slutt
  orienteringen til slutt — slik at forhåndsvisning, radianse, reflektans og indeksprodukter blir nøyaktig som
  de fra liveopptaket, og lagrer `ChlorosOrientationApplied=true`. De ukalibrerte reserveproduktene
  (`Sensor_Response_Images`, `Vignette_Corrected_Images`) er orientert på samme måte, etter
  flatfeltet. Det debayerte produktet forblir i opprinnelig format.
- **Chloros-bildeviseren viser en rå eller debayert fil som er rotert i henhold til opptaket** (kun visning —
  filen endres ikke), slik at den ser ut som produktene; miniatyrbildet, visningen i full størrelse,
  markørvisningen og Index/LUT-sandkassen bruker alle samme rotasjon. En fil uten opptak, og
  et produkt (som allerede er orientert), vises slik de er lagret. En sandkasseeksport registrerer orienteringen som
  pikslene har (`ChlorosOrientationApplied=true`; identiteten for et lag uten registrering).
- **Opptak uten registrering behandles som opprinnelige** (uendret oppførsel). Før versjon 1.3.0
  vendte et opptak med ett kamera, der speilvendelse eller rotasjon var angitt, selv sine råfiler og debayerte filer;
  Chloros kan ikke skille mellom slike allerede-rott råfil fra en native, og gjettet ikke, så den
  behandler den slik den er lagret. Kalibreringskartene ligger på sensorens eget rutenett, så for en slik
  opptak bør du bruke en som er tatt med versjon 1.3.0 når kalibreringen er viktig.
- **Arrays roterer ingen medlemsfiler som standard** (justeringsforvrengningen og `Alignment*` XMP-filen ligger i
  kameraenes egne, urotte bilder) og registrerer identiteten. `array-capture --orient` roterer hvert
  medlems produkter etter justeringsforvrengningen: etter **masterens** orientering da de ble
  forvrengt inn i dens ramme (slik at medlemmene forblir samregistrerte), ellers etter deres eget kameras orientering. Hver
  fil registrerer orienteringen til rammen som dens egne piksler befinner seg i — en råfil, fra sitt eget kamera —
  og en råfil som inneholder justeringstransformasjonen registrerer også `ChlorosAlignedFlipHorizontal` /
  `ChlorosAlignedFlipVertical` / `ChlorosAlignedRotation`: masterens, for et produkt som er forvrengt
  inn i masterens ramme. `process` roterer hvert produkt med den verdien som tilhører dets piksler:
  den justerte når den forvrenger RAW-filen (`--array-alignment`, standardverdien – produktene fra den live-justerte
  opptakets produkter), kameraets egen når det ikke gjør det (`--no-array-alignment` – de fra live
  ujusterte opptakene). Det samme alternativet finnes for `array-burst`, `project capture` / `burst` /
  `stream`, oppskrifter og SDK.

### Hoppede opptak i blandede filteroppsett

Når du blander RGB- og multispektrale kameraer i ett oppsett, lagrer `array-capture --processing radiance` (eller `reflectance`) de multispektrale bildene og **hopper over** RGB-kameraene — strålingsintensitet per Bayer-piksel er ikke relevant for en bredbåndssensor. CLI-en viser hver lagrede fil (med eksportnivå), hver `.daq` som er skrevet, og hvert hopp eksplisitt, så antall filer er ikke overraskende:

```
  Saved: …/output/reflectance/sync_…_SN213800234-FRGN.tiff [reflectance] (SN:213800234, fid:1)
  Saved: …/output/reflectance/sync_…_SN214000533-FOCN.tiff [reflectance] (SN:214000533, fid:1)
  Saved: …/output/reflectance/sync_…_SN214701288-FNGB.tiff [reflectance] (SN:214701288, fid:1)
  DAQ:   …/output/daq/daq_data_daq-e-54b5e0_….daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Tokenene for hoppårsaker følger mønsteret `<level>-not-applicable-to-rgb-cam`. Et kamera som ikke har kalibreringspakken lastet inn, hopper over `radiance` / `reflectance` med `<level>-needs-calibration (connect the camera so its cal pack loads)`, og hvis det ikke finnes noen brukbar DAQ-måling av reflektans, hoppes det over med `reflectance-skipped-no-dls`. Reflektans kan også hoppes over med `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, og med `dls-uncalibrated-band-<nm>` når båndet hovedsakelig ligger utenfor DAQ-lyssensorens radiometrisk kalibrerte område (~374–974 nm) — blant de tilgjengelige SKU-ene er det kun F988, som støtter arbeidsflyten med reflektanspanel.

Bruk `--processing debayered` (eller `display`) for å inkludere alle kameraer uavhengig av filtertype, eller standardinnstillingen `all` for å få alle relevante nivåer per kamera i ett opptak.

### En DAQ-avlesning langt fra eksponeringen

Hver `.daq` som en opptak lagrer, er DAQ-avlesningen som samsvarer med den eksponeringen, og `.match.json` ved siden av den registrerer hvor langt fra hverandre de to var (`gap_ms`) og om dette ligger innenfor toleransen (`in_tolerance`, 500 ms med mindre opptakssettet angir `--daq-tolerance-ms`). En måling utenfor toleransegrensen brukes likevel, slik at ingen ramme går tapt, og den merkes: XMP-filen til reflektansfilen inneholder `DlsInTolerance=False` og `DlsMatchGapS`. `lattice capture`, `multi-capture` og `array-capture` skriver den også ut under den aktuelle `.daq`, med DAQ'strømfeil når man forklarer det:

**Hvordan en avlesning tilordnes en eksponering.** En opptak venter på DAQ-enhetens første avlesning etter eksponeringen, slik at den kan blande avlesningene på hver side av den (`method` `interpolated`). En DAQ-E sender en måling hvert 0,4. sekund i dagslys og hvert 1–2. sekund innendørs, så ventetiden tilsvarer omtrent halvannet av disse intervallene for kommandoen, ikke for eksponeringen. En DAQ som har sluttet å sende, blir ikke ventet på. En DAQ-E-måling er lyset gjennomsnittlig beregnet over intervallet siden forrige måling, så den plasseres midt i det intervallet: `gap_ms` måles fra eksponeringen til midten av dette intervallet, og `bracket_stamps_ns` angir målingens egne tidsstempler. `array-capture --continuous` venter uten å bremse opptaket: råfilene skrives umiddelbart, og deres `.daq` oppdateres så snart avlesningen kommer inn. Med `--interval` venter hvert gjennomløp som en enkelt opptakssyklus.

```
  DAQ:   …/output/daq/daq_data_daq-e-54b774_….daq
         DAQ reading 114.9 s from the exposure (tolerance 500 ms): reflectance uses it, labelled DlsInTolerance=False
         DAQ stream fault: no frame accepted from daq-e-54b774.local in 25 s, after 1932 good frame(s). …
```

Når det ikke ble skrevet noen reflektansfil fra den avlesningen, avsluttes den første linjen i stedet med `saved anyway; its .match.json says in_tolerance: false`.

Opptaket avsluttes fortsatt med `0`: alle filene ble skrevet og angir hva de inneholder. En DAQ hvis strøm har stoppet opp, fortsetter å levere sin siste avlesning, så sjekk den med `chloros-cli daq pool-list` før du stoler på refleksjonsverdien fra det opptaket. Backenden kobler på nytt til en samlet DAQ-E som har sluttet å levere (se [En DAQ-E som slutter å levere](#a-daq-e-that-stops-delivering)). En avlesning utenfor toleranseområdet skriver bare ut sin `DAQ:`-linje, med mindre det er en beholdt avlesning (se nedenfor).

**En måling som er beholdt etter at DAQ-en forsvant.** Når ingen DAQ kan levere en måling for en eksponering (den tilknyttede DAQ-en er utilgjengelig, eller ingen samlet DAQ har en, f.eks. etter `daq pool-disconnect`), gjenbruker registreringen den siste målingen Chloros hadde, og dens `.match.json` beskriver denne målingen mot denne eksponeringen: `method` `retained`, `gap_ms` fra denne eksponeringen til prøvene som målingen ble tatt fra, `in_tolerance` fra det gapet, og `retained_from_epoch_ns`, eksponeringen som målingen først ble matchet mot. En registrering tatt lenge etter at DAQ-en ble fjernet, skriver derfor ut linjene ovenfor, og under dem en til, som skrives ut selv om målingen fortsatt ligger innenfor toleransegrensene:

```
         This is the DAQ's last reading: the DAQ is no longer reporting, so each capture reuses it until a new one arrives
```

Dens `.daq` angir fortsatt DAQ-en som tok målingen, sammen med kalibreringen og lokket som målingen ble tatt under, slik at en reimport vet hvilken DAQ, kalibrering og hvilket lokk som produserte den.

### Filer som ikke kan skrives

Et kamera hvis fil ikke kan skrives – en full harddisk, en tillatelsesfeil – hindrer ikke at filene fra de andre kameraene lagres. Hver opptakskommando (`lattice capture`, `multi-capture`, `array-capture`; `project capture`, `burst`, `stream`; samt oppskriftens `capture`, `burst` og `stream`) skriver ut hver fil som mislyktes til stderr, deretter **avsluttes `1`** selv om resten ble lagret:

```
  Failed: SN:214701292 [reflectance] OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

Alle filer som hører til en opptak er dekket, inkludert den eneste-ramme-debayert `lattice capture --levels debayered` inkludert. En `.daq`-avlesning der `.match.json`-sidecar-filen med opprinnelsesinformasjon ikke kunne skrives, er oppført som `[match]` (selve `.daq` ble lagret).

På en full harddisk vil en gjentatt opptak (`array-capture --continuous` / `--interval`, `project burst` / `stream`, en oppskrift `burst` / `stream`) **stopper ved den omgangen** i stedet for å ikke skrive noe til den samme stasjonen resten av kjøringen. Alle andre feil rapporteres ved hver omgang de oppstår, og kjøringen fortsetter til slutten, før den avsluttes med `1`. `array-capture --continuous` (backendens kontinuerlige innsamling) viser ingen filer, inkludert de som mislyktes: den teller dem i fremdrifts- og oppsummeringslinjene (`N failed`, `N file(s) failed to write.`) og avslutter `1` — se [Kontinuerlig modus](#continuous-mode---kontinuerlig).

Opptaksprogrammene — `array-burst` og `array-record` — rapporterer sine egne feil. `array-burst` stopper av seg selv når stasjonen blir full, eller like før den blir det. Begge fortsetter uavhengig av eventuelle andre mislykkede skrivforsøk. Når en av dem rapporterer en feil, skrives den ut etter tellerne, og kommandoen **avslutter `1`**:

```
  Recorder error: disk-full: OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

`Disk low:` i stedet for `Disk full:` betyr at opptaket stoppet før stasjonen. Rammene som ble spilt inn før stoppet, beholdes. `array-burst --build` hopper over byggingen når det ikke er mer plass, fordi videoene ville gå til den samme stasjonen. Sørg for ledig plass først, og kjør deretter `array-build-video --burst-dir …`. Etter eventuelle andre feil fortsetter byggingen likevel.

En `array-record` som ikke skrev noen rammer i det hele tatt, er også en mislykket opptak (`Recorder error: no-frames: the combined view published no frame to record …`, avslutt `1`) — se [`array-record`](#array-record--combined-index-videogif-monitoring-grade).

---

## Opptakmoduser, opptakere og offline-bearbeiding

Alle disse opererer på en **persistent array** (kjør `array-connect` først). De speiler opptaksfeltet i grafisk brukergrensesnitt.

### `array-capture`-moduser

`array-capture` er en enkelt kommando med fire utløsermoduser pluss et sett med eksportbrytere:

| Modus | Flagg | Oppførsel |
| --- | --- | --- |
| **Enkelt** *(standard)* | (ingen) | Én synkronisert opptaksgruppe, deretter avsluttes. |
| **Kontinuerlig** | `--continuous` | Opptak etter hverandre inntil `Ctrl+C`, `--count N`-opptak, `--duration S` eller en full stasjon. Kjører backendens egen kontinuerlige opptak (GUI-ens kontinuerlig modus) — se [Kontinuerlig modus](#continuous-mode---continuous) nedenfor. |
| **Intervall** | `--interval S` | Én gjennomgang hvert `S` sekund (målt fra starten av hver gjennomgang), samme grenser. |
| **Raskest** | `--fastest` | Rå-kun + den tildelte DAQ-avlesningen + den sammensatte indeksen; hopper over beregningene for stråling/refleksjon/visning, slik at bildet vises raskt. Innebærer `--processing raw --force-daq`. Behandle de lagrede `.daq`-dataene på nytt til kalibrerte produkter senere. |

Eksportalternativer (kan kombineres med hvilken som helst modus; alle deler GUI/SDK-endepunktet):

| Flagg | Effekt |
| --- | --- |
| `--array-id ID` | Matrisen som skal hentes fra (utelates når bare én er tilkoblet). |
| `-o, --output DIR` | Utdatamappe (standard `output`, under mappen du kjører kommandoen fra). |
| `--format {tiff,tif,png,jpg,jpeg}` | Filformat (standard `tiff`). PNG/JPEG bruker kun `display` og indeksrendering; se [Eksportnivåer for opptak](#capture-export-levels-the-all-default). |
| `--processing LEVEL` | Enkelt eksportnivå (`raw`, `debayered`, `radiance`, `reflectance`, `display`), eller `all` (standard). |
| `--levels L1,L2,…` | Eksplisitt delmengde av eksporttyper (f.eks. `raw,radiance,reflectance`); **overstyrer `--processing`**. |
| `--aligned` / `--no-aligned` | **Standard: justert, slik som Chloros-appens registrering av den samme matrisen er**(dens**Justert**-innstilling for registrering per matrise, som er aktivert med mindre du deaktiverer den). Hvert medlems ikke-rå eksport (`debayered`, `display`, `radiance`, `reflectance`, indeksrenderingen) blir forvrengt inn i hovedkameraets bilde og beskåret til medlemmene felles overlappingsområde, slik at filene er pikselregistrerte og kommer ut med samme størrelse, beskjæring og `Alignment*`-tagger som appens – mindre enn sensoren (eller ROI) på grunn av medlemmenees forskyvninger; under en ROI blir transformasjonen rebasert til vinduet. `XMP-mapir:AlignmentJSON` angir `"applied": true` og gir `crop_roi`. Raw-filen forblir uforvrengt i opprinnelig størrelse, men bærer transformasjonen i metadataene (`"applied": false`). `--no-aligned` lagrer hver fil i opprinnelig størrelse, uforvrengt. `--aligned` er standardinnstillingen. Uansett om forvrengningen brukes eller ikke, registrerer hver enkelt fil kameraets `Alignment*`-transformasjon så snart matrisen har en justeringsprofil — med unntak av et kamera som profilen ikke justerer (utelatt, eller tvunget til identitet på grunn av en mislykket løsning), og referansekameraet når ingen andre kameraer er justert: disse har ingen `Alignment*`-koder. Uten profil på matrisen lagres opptaket ujustert, og CLI-en viser årsaken på en `Alignment:`-linje, slik den gjør i alle andre tilfeller der forvrengningen ikke kunne brukes (elementer med ulik oppløsning, et array som ikke er kabelsynkronisert, et bilde som profilen ikke kan tilordnes til). Arrayets automatiske justering oppretter profilen så snart eksponeringen har stabilisert seg; `align-calibrate` oppretter en på forespørsel. |
| `--orient` | Slå på hvert elements **produkter**— `display`, `radiance`, `reflectance`, `index/`-renderingen og `composite/` — i henhold til kameraets orientering, etter justeringsforvrengningen: et produkt forvrengt inn i masterrammen (standard) etter**masterens**orientering, slik at elementene forblir samregistrerte; et ujustert produkt (`--no-aligned`, eller et opptak som forvrengningen ikke kunne brukes på) etter sitt eget kameras orientering.**Standard: av — hver enkelt fil forblir i opprinnelig format**, slik opptak i matriseformat alltid har gjort. `raw` og `debayered` forblir i opprinnelig ramme uansett. Hver fil registrerer orienteringen som produktene har (se [Kameraorientering](#hvordan-en-opptaksmappe-ser-ut)); XMP-filen `Alignment*` forblir transformasjonen av den opprinnelige rammen, anvendt før orienteringen, slik at `process` (forvrengning, deretter orientering) gjengir det opprinnelige produktet. |
| `--index` / `--no-index` | Lagre / hopp over vegetasjonsindeks-overlegget per kamera der dette er konfigurert. Standard: gjengi det. |
| `--no-daq` | Hopp over `.daq`-sidecaren når ingen av de valgte nivåene bruker DAQ (et utdrag som kun inneholder rådata). Som standard lagres målingen, slik at bildene kan bearbeides på nytt til reflektans/indeks offline. `--force-daq` godtas fortsatt (standard). |
| `--daq-tolerance-ms MS` | DAQ-samtoleringstoleransen for disse opptakene (standard 500 ms). |
| `--smart` | Vent til AE har stabilisert seg på alle kameraer før utløsning (se [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | TIFF-pikselkomprimering. `none` (standard siden 1.3.0; 1.2.0 skrev `deflate`) = ukomprimert — sensorens piksler nøyaktig slik de leses, ~6,3 MB per bilde i full oppløsning, og den raskeste vedvarende hastigheten; `deflate` = tapsfri zlib L1 + horisontal prediktor, ~4,1 MB per bilde, tregere. Begge er tapsfrie og leses identisk ved import. |

> **TIFF med én-gangs-skriving + modellen med vedvarende hastighet.**Hver fil skrives**én gang**: piksler, XMP-pakken, IFD0 Make/Model/CameraSerialNumber og EXIF-taggene `ExposureTime` / `SerialNumber` i én omgang, uten at noe overskriver det etterpå. `--continuous`X fanger opp påfølgende synkroniserte grupper — hver opptak tar gruppen etter den forrige, inkludert en som ble publisert mens det forrige opptaket fortsatt ble lagret — slik at det kjører med matrisens hastighet. DEFLATE holder Python-GIL, så komprimerte skrivninger**parallelliseres ikke**på tvers av skrivertrådene per kamera; vedvarende opptak i full oppløsning med sensorhastighet krever ukomprimerte skrivninger (standard)**og** en NVMe-klasse disk (~600 MB/s vedvarende for et oppsett med 9 kameraer ved ~10 fps). Den samme innstillingen er tilgjengelig som `compression` på `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (aligned is the default; drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --no-index -o out/

# Each member at its native size, unwarped
chloros-cli lattice array-capture --processing reflectance --no-aligned -o native/

# Continuous raw capture for 60 s, as fast as the array publishes
chloros-cli lattice array-capture --continuous --duration 60 --fastest -o flightline/
```

#### Kontinuerlig modus (`--continuous`)

`array-capture --continuous` (uten `--interval` eller `--smart`) overfører driften til **backendens egen kontinuerlige opptak** — den sesjonen der GUI-ens kontinuerlige modus kjører. Backenden lagrer hver synkronisert gruppe selv, med to opptak på gang samtidig og ingen forespørsler fra CLI-en mellom gruppene, slik at den holder tritt med et raskt oppsett der én forespørsel per opptak henger etter (målt til 73–77 opptak/s på et oppsett med 9 kameraer ved et ROI på 512 piksler). Hver eksportbryter ovenfor gjelder for hver enkelt opptak.

- **Begrensninger:** `--count N` stopper etter N opptak (ett opptak = én synkronisert gruppe: filene til hvert medlem); `--duration S` etter S sekunder. Uten noen av delene kjører den til `Ctrl+C`, en full harddisk, en feil eller at oppsettet slås av. `Ctrl+C` stopper backend-sesjonen, deretter skriver den ut sammendraget.
- **Fremdrift** hvert 2. sekund, med `, N failed` og `, N groups skipped` lagt til når verdien er forskjellig fra null (en hoppet over gruppe er en som arrayet publiserte, men som ingen opptak tok):

  ```
     12.0s  880 captures (73.3/s), 7920 files
  ```

- **Ingen oppføring per fil.** Kjøringen avsluttes med én oppsummeringslinje — fangster, hastighet, filer og utdatamappen — etterfulgt av `N file(s) failed to write.` / `N group(s) skipped.` når verdien er forskjellig fra null:

  ```
  Done — 880 captures (73.3/s), 7920 files in /home/user/flightline.
  ```

- **Stanser** ved full disk med `Continuous capture stopped: the drive is full.`, og ved feil eller når matrisen forsvinner med `Continuous capture stopped: <reason>.` (begge på stderr).
- **Avslutningsstatus:** `0` når opptakene er lagret og alle filer er skrevet, inkludert `Ctrl+C`; `1` ved enhver fil som ikke ble skrevet, en full harddisk, en feil, at matrisen forsvinner eller ingen opptak i det hele tatt.
- **Microsoft Defender:** når Defender bruker en CPU-kjerne eller mer på å skanne filene som opptaket skriver, skrives følgende ut én gang på stderr: `Note: Microsoft Defender is using N.N CPU cores scanning the capture files, CPU the capture needs. Re-run the Chloros installer with "Exclude the Chloros backend from Microsoft Defender scanning" checked.` Se installasjonsprogrammets [Systeminnstillinger](#windows-exe).
- Én kontinuerlig opptakssesjon per matrise om gangen; en ny blir avvist med `a continuous capture is already running for this array`. Over HTTP er sesjonen `POST /api/camera/array/continuous/start` (`/api/camera/array/capture`-brødteksten pluss `max_captures` / `duration_s`), `GET /api/camera/array/continuous/status?array_id=ID` og `POST /api/camera/array/continuous/stop`.

**Interval**(`--interval S`, med eller uten `--continuous`) og**`--continuous --smart`** beholder sløyfen per forespørsel: én innfangingsforespørsel per gjennomgang — på hvert planlagte tidspunkt for Interval, etter at AE har stabilisert seg for `--smart` — med filene for hvert gjennomløp oppført som i enkeltmodus, en `Done — N capture pass(es).`-linje på slutten, og `--count N` som teller gjennomløpene.

### `array-record` — kombinert indeks for video/GIF (overvåkingskvalitet)

Lagrer matrisens **live-visning med kombinert indeks** til en `.avi` (og eventuelt en `.gif`). Når GUI-ets kombinerte visning er åpen, lagrer den den visningen slik den vises. Hvis ingen visning er åpen, gjengir backenden visningen selv: indeksen du sender med `--expr`, eller som standard matrisens NDVI, så snart matrisen er justert. Standard-NDVI bruker de røde og NIR-båndene nærmest 660 nm og 850 nm, slik som live-visningen gjør. En GUI-visning som åpnes under opptaket, blir tatt opp så lenge den forblir åpen. Den sjekker fremdriften hvert 2. sekund og stopper ved `--duration`, `Ctrl+C`, eller når opptakeren avslutter seg selv.

En opptak som ikke skriver noen ramme mislykkes: opptakeren rapporterer `no-frames: …` (matrisen kan fortsatt være i ferd med å justere seg eller stabilisere eksponeringen), og kommandoen **avsluttes med `1`**.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF (no GUI needed)
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# Record a chosen index instead of the default NDVI
chloros-cli lattice array-record --duration 30 \
  --expr "(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)" -o monitoring/
```

| Flagg | Standard | Beskrivelse |
| --- | --- | --- |
| `--array-id ID` | kun matrise | Målmatrise (utelat hvis bare én er tilkoblet). |
| `-o, --output DIR` | `output` | Utdatakatalog. En relativ sti er under mappen du kjører kommandoen fra. |
| `--fps F` | `10` | Opptakets bildefrekvens. |
| `--duration S` | til Ctrl+C | Automatisk stopp etter `S` sekunder. |
| `--gif` | av | Skriv også en animert GIF. |
| `--gif-only` | av | Lagre kun en GIF (ingen `.avi`). |
| `--expr EXPR` | matrisens NDVI | Indeksuttrykk som skal registreres når ingen GUI-visning er åpen, i matrisens båndnavn (f.eks. `'(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)'`; se [båndnavn](#index--vegetation-maths)). Et kort fargenavn (`NIR`, `Red`) fungerer bare når ett bånd i matrisen har det. En åpen GUI-visning registreres i stedet slik den vises. |

### `array-burst` — rå-Bayer-serieopptak med høy bildefrekvens (analyse-kvalitet)

Leser direkte fra synkroniseringsgruppens buffer i opptakssløyfen — **ingen kalibreringskjede, ingen exiftool, ingen live-visning nødvendig** — slik at den kjører med kameraets fulle opptakshastighet. Skriver ut RAW-bilder + et manifest per bilde + én `.daq` per distinkt DLS-avlesning under `<output>/bursts/<base>/`. Bearbeid på nytt offline (neste kommando), eller bruk `--build` for å gjøre det umiddelbart ved stopp.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Flagg | Standard | Beskrivelse |
| --- | --- | --- |
| `--array-id ID` | kun matrise | Målmatrise. |
| `-o, --output DIR` | `output` | Utdatakatalog (burst havner i `<DIR>/bursts/<base>/`). |
| `--duration S` | til Ctrl+C | Automatisk stopp etter `S` sekunder. |
| `--max-frames N` | ubegrenset | Automatisk-stopp etter `N` råbilder. |
| `--orient` | av | Produktene som en bygging lager fra denne serien, roteres etter kameraets orientering — produkter per kamera etter sitt eget kameras, den kombinerte indeksen etter masterens – slik som `array-capture --orient` gjør. Råbildene forblir i opprinnelig format; serien `manifest.json` registrerer hvert medlems orientering og valget. |
| `--build` | av | Etter stopp, bearbeid serien umiddelbart på nytt (samme som `array-build-video`). |
| `--products …` | `combined:index` | Med `--build`: hvilke videoer som skal lages (se nedenfor). |
| `--fps F` | `10` | Med `--build`: bildefrekvens (fps) for utgangsvideoen. |
| `--save-tiffs` | av | Med `--build`: lagre også kalibrerte TIFF-filer per bilde (hver med sin [unmix-klemmaske](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on) når en piksel er klemt; byggeresultatet viser dem som `clamp_masks`). |
| `--gif` | av | Med `--build`: skriv også animerte GIF-filer. |

### `array-build-video` — offline-bearbeiding av en lagret bildeserie

Tidsjusterer hvert råbilde til nærmeste lagrede `.daq`-avlesning og sender det gjennom **den samme strålings-/refleksjons-/indekskjeden som importpipeline**, og renderer én eller flere videoer.

`--products` er en kommaseparert liste over `kind:level`-elementer, der `kind` ∈ `per_cam` | `combined` og `level` ∈ `radiance` | `reflectance` | `index`. En ren `level` (uten `kind:`) har som standardverdi `per_cam`. Standardverdien er `combined:index`.

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
| `--fps F` | `10` | Bildfrekvens (fps) for utgangsvideo. |
| `--save-tiffs` | av | Lagre også kalibrerte TIFF-filer for hvert enkelt bilde sammen med videoene. |
| `--gif` | av | Skriv også animerte GIF-filer. |

**En video inneholder ingen klemmaske** (én bildestrøm, 8-bit). I stedet inneholder byggeresultatet `clamp_fractions`: per videoprodukt, hver bildes andel av piksler som unmix har klemt inn i båndet som dets R/G/B-kanal inneholder — det `Chloros:Band{R,G,B}UnmixClampFraction` som det bildets strålings-/refleksjonsprodukt ville bli stemplet med (for den kombinerte indeksen, for hvert enkelt kamera). Kjøringen skriver ut det dårligste bildet per produkt på en `Clamp:`-linje. `--save-tiffs` legger til maskene per piksel for TIFF-filene, i `clamp_masks/<level>/` ved siden av TIFF-filenes `<level>/`-mappe.**En avbrutt kompilering etterlater ingen ødelagte videoer.** Hver video skrives som `<name>.partial.avi` og får sitt endelige navn først når den er ferdig, så en eldre video med det navnet forblir der inntil erstatningen er fullstendig. Kjør byggingen på nytt for å erstatte en `.partial.avi` som er blitt liggende igjen.

Kjøringen skriver ut hver video den har lagret, deretter eventuelle `Skipped:`-produkter (et forespurt kamera/nivå – eller den kombinerte indeksen – som ikke produserte noen video, med årsaken – f.eks. ingen kalibreringspakke, ingen DLS-avlesning for reflektans, eller ingen rød/NIR-båndpar for den kombinerte indeksen) samt eventuelle `Note:`. Et RGB-kamera som er forespurt for radianse eller reflektans, får i stedet sin visningsvideo, med hvitbalanse når kameraets kalibreringspakke er tilgjengelig; et kamera som gjengis uten dette, blir oppført i en `Note:`. Den **returnerer et tall ulikt null** når et forespurt produkt ble hoppet over eller ikke ble bygget som forespurt (et konfigurert indeksuttrykk blir ikke evaluert offline); videoene som ble bygget, blir fortsatt skrevet ut og oppført.

> **Velg riktig opptaker.** `array-record` er *overvåkingsklasse* — den tar opp den gjengitte live-kompositten: GUI-ens kombinerte visning, eller, hvis ingen GUI er åpen, backendens egen NDVI-visning (eller `--expr`). `array-burst` → `array-build-video` er *analyseklasse* — den lagrer rå sensordata med full hastighet og rekonstruerer kalibrerte strålings-/refleksjons-/indeksvideoer i etterkant, uten at det kreves livevisning.

### Mono (M3M) enkeltbåndskameraer

**M3M**-serien er den monokrome søskenmodellen til Bayer**M3C**: ett smalbåndsinterferensfilter per kamera (`M3M-<lens>-F<wavelength>`, f.eks. `M3M-L87-F685`), slik at sensoren leverer et**enkelt gråtonebånd** uten Bayer-mosaikk. Det er ingenting å demosaikere, ingen kryssforstyrrelser mellom kanalene å skille ut, og ingen hvitbalanse å stille inn – hele fargeprosessen for RGB-visning gjelder ganske enkelt ikke.

Hva dette betyr på CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**oppdager et monokamera og**hopper over med en enlinjemelding** i stedet for å sende meningsløse innstillinger. De fungerer fortsatt normalt med et RGB-/Bayer M3C-kamera i samme økt.
- **Et Bayer-pikselformat angir dybden.** En M3M tilbyr kun monofargede formater, så et Bayer-format fra `--pixel-format` eller `--preset` angir det monofargede formatet med samme dybde: `BayerRG8` og `--preset high_speed` gir `Mono8`; `BayerRG12`, `high_quality` og `triggered` gir `Mono12`; et pakket navn forblir pakket (`BayerRG12p` gir `Mono12p`). Filene og `lattice format` betegner Mono-formatet. En matrise som blander M3C- og M3M-kameraer bruker ett format på på samme måte: Bayer på hver M3C, dens monodybde på hver M3M. Et navn er dets bitdybde: det samme gjelder omvendt, så et M3C-kamera blir aldri satt i et Mono-format (`--pixel-format Mono12` er `BayerRG12` på et M3C-kamera, uansett hvilket kamera som er oppsettets hovedkamera), og `lattice format --set` kartlegges på samme måte og skriver ut det formatet det er satt til.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** fungerer fortsatt — radiance og reflectance er *per-bånd* radiometriske kart og er perfekt definert for ett bånd. Monokromer har en **identitets**-sensorresponsmatrise (ingen 3×3-unmix), så planet går uendret gjennom kalibreringsberegningene.
- **Et enkelt monokamera kan ikke produsere et vegetasjonsindeks.**NDVI/NDRE/osv. krever minst to bånd (f.eks. Red + NIR). For å få et indeks fra monohardware må du rette**flere** M3M-kameraer mot forskjellige bølgelengder, samkjøre dem til én multibåndsstabel og beregne indeksen *for den*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Mono_660 --channel nir=Mono_850 \
  --save-multiband -o output/
```

`index --live` åpner kameraene selv (en [midlertidig økt](#multi-camera-transient-sessions)), så ikke koble til noe først: `--serials`, ellers vil profilens aktiverte kameraer (se [Indeks / Vegetasjonsmatematikk](#index--vegetation-maths)). Så lenge Chloros holder et kamera (GUI-en, `cam-connect`, `array-connect`) blir det avvist med HTTP-kode 409 `CAMERA_HELD` og avsluttes `1`; frigjør dem med `chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`.

`--channel`-symbolene må **nøyaktig** samsvare med forhåndsinnstillingens kanalnavn (skiller mellom store og små bokstaver; NDVI-er er små bokstaver `red`,`nir` — se `--list-presets`), og «band side» angir et bånd i den justerte stakken (offline-modus godtar også 0-baserte båndindekser, f.eks. `--channel red=0 --channel nir=1`).

Det som skiller de ulike stakkene fra hverandre, er tokenet `M3M` i modellstrengen (det forekommer aldri i en `M3C`-streng), som vises i GUI/SDK som `is_mono`.

---

## Konfigurering og justering av verts-NIC (LATTICE-oppsett)

LATTICE-kameraer strømmer GVSP via vertsmaskinens Ethernet-kort, så for oppsett med flere kameraer er kortets **driver**og**mottaksringstørrelsen** like viktig som koblingshastigheten. Feil innstillinger vises som en `FRAMES WILL DROP` / `Reduce ROI to enable`-port i panelet «Array Settings» (og i `lattice network-analysis` / SDK-ets `analyze_array_network()`), selv når kameraene i seg selv fungerer som de skal.

### USB 10GbE-adaptere — Realtek RTL8157 («Realtek USB 10GbE Family Controller»)

Innstillingene nedenfor er egenskaper for Windows-driveren. På Linux får en USB-adapter også en begrensning (400 MB/s på x86_64, 200 MB/s på NVIDIA Jetson) (se [Array fps & burst-modell](#array-fps--burst-model)), og kjernens mottaksbuffere for sokler er av betydning for alle adaptere — se [Linux-verter](#linux-hosts--socket-receive-buffers).

| Element | Nødvendig verdi | Hvorfor det er viktig |
| --- | --- | --- |
| **Driverversjon**|**≥ v10.67 (jan. 2026)**, INF `rtump64x64sta.inf` | Den eldre**2016**-driveren (v10.65, `rtump64x64.inf`) håndterer ikke nedstengning og feilkontroller riktig, noe som fører til**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**ved avslutning/omstart/hvilemodus. Overgangen henger seg opp (~5 minutters tidsavbrudd), brukeren slår av strømmen manuelt, og gjentatte uregelmessige avslutninger**ødelegger WMI-lagringsplassen**(PowerShell/verktøy begynner å svikte med `Invalid class`) og**blokkerer USB-stakken** ved neste oppstart (nettverkskortet aktiveres ikke; USB-enheter slutter å bli opptalt). Oppdater fra realtek.com (eller dongle-leverandøren) før du stoler på rene omstarter. |
| **Mottaksbuffere**— nøkkelord `ReceiveBufferLen` |**256**(maks. i driveren) | Nettverkskortets RX-ring. Driverens standardverdi på**32**etterlater bare ca. 0,26 MB brukbar ring — altfor lite for en multikamera-burst — så array-panelet rapporterer `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` og blokkerer tilkobling. Ved**256**er ringen stor (**~13,5 MB målt på laboratoriets 10GbE-vert**), noe som gir RX-rørledningen reell kapasitet for GVSP-burst med flere kameraer. (Hvorvidt en gitt konfigurasjon faktisk *kobles til*, avgjøres av to kontroller — den **drain-aware**admittanssjekken og**aggregat-overskrivningssjekken** — ikke en rå sammenligning mellom burst og ring; se [Array fps & burst-modell](#array-fps--burst-model).) |
| **Mottak av URB-er**— nøkkelord `PendingReceives` |**64** (maks.) | USB-forespørselsblokker under overføring; økes sammen med mottaksbuffere for å absorbere bursts. |
| **Jumbo Frame** — nøkkelord `*JumboPacket` | **9014** | Nødvendig for 9000-byte GVSP-pakker (6× færre pakker/ramme enn 1500). |

> ⚠️ **En oppdatering av nettverkskortdriveren TILBAKESTILLER disse avanserte egenskapene til standardinnstillingene.**Etter oppdatering eller utskifting av nettverkskortdriveren må du**påføre på nytt** `ReceiveBufferLen=256` og `PendingReceives=64`, ellers vil array-panelet blokkere tilgangen igjen selv om «ingenting er endret i maskinvaren». Dette er den vanligste årsaken til at et system som tidligere fungerte, plutselig nekter å koble seg til.

Bruk fra en **administratorrettighet** PowerShell (erstatt med navnet på nettverkskortet ditt, f.eks. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` gjelder USB 10GbE-kort.** Den oppdager nå korttypen og justerer riktig nøkkelord for mottaksring: `*ReceiveBuffers`→2048 for PCIe-nettverkskort (Intel I219, osv.), eller `ReceiveBufferLen`→256 + `PendingReceives`→64 for Realtek **USB** 10GbE-kontrolleren (som ikke eksponerer `*ReceiveBuffers`). Verdiene begrenses til den maksimale verdien som rapporteres av hver driver (`NumericParameterMaxValue`), slik at det aldri skrives inn en verdi utenfor området. Kjør det fra en **administrator-terminal**; som med alle registerbaserte justeringer trer endringen i kraft etter at nettverkskortet har startet på nytt eller etter en omstart av systemet. De manuelle `Set-NetAdapterAdvancedProperty`-kommandoene ovenfor er fortsatt et godt alternativ — de gjelder umiddelbart (kobler nettverkskortet på nytt) uten omstart.

### Linux-verter — mottaksbuffere for sokler

På Linux, inkludert NVIDIA Jetson, er kjernens mottaksbuffere for sokler like viktige som nettverkskortets mottaksring. Med kjernens standardinnstilling (`net.core.rmem_max` / `net.core.rmem_default` = 212 992 byte) fører en kameras strøm av GVSP-pakker til at strømsokkelen blir overfylt før Chloros-modulen rekker å lese den. Pakker blir droppet i kjernen mens nettverkskortet selv ikke rapporterer noe tap, og et array mister ramme-grupper og reduserer bildefrekvensen. Målt på et array med 4 kameraer koblet til en USB 10 GbE-adapter, økte verdiene begge til 32 MB, noe som førte til at arrayet gikk fra 4,96 til 7,52 fps etter tilkobling, og en kontinuerlig råopptakshastighet fra 4,7 til 7,6 opptak/s, uten at noen pakker ble droppet i kjernen.

- **.deb-filen angir disse verdiene.** Når pakken installeres, skrives `/etc/sysctl.d/60-chloros-gige.conf` (begge til `33554432`) inn og tas i bruk umiddelbart. En verdi som allerede er satt høyere, blir ikke endret.
- **`lattice network` sjekker dem.** Når en av dem er lavere, vises `socket receive buffers too small for GigE Vision (…)` under *Issues found*, og backendens logg inneholder den samme advarselen ved første tilkobling av kamera eller array.
- **For å øke dem manuelt** (frem til neste omstart; pakkens fil bruker dem ved hver oppstart):

  ```bash
  sudo sysctl -w net.core.rmem_max=33554432 net.core.rmem_default=33554432
  ```

  `lattice network --fix` øker og opprettholder dem også, men kun når backenden kjører som root. En backend som kjører som din bruker (en som er startet med `chloros-backend`, eller [`chloros-backend@<user>`](#running-the-backend-as-a-service-headless)) kan ikke endre kjernelinnstillinger, og skriver ut linjen `sudo sysctl` i stedet.

**Den automatiske adapterjusteringen berører kun tilkoblede adaptere.** Ved den første kameratilkoblingen justerer en backend som kjører som root de kablede adapterne som har en liten mottaksring. Den justerer nå kun adaptere med en aktiv forbindelse, slik at en frakoblet innebygd port beholder sin MTU, og den lar mottaksbufferne være uendret. `lattice network` viser ikke lenger MTU-en til en frakoblet adapterMTU som et problem. `lattice network`, `--fix`, `lattice probe` og adapterjusteringen tar kun hensyn til kablede Ethernet-porter, ikke en Jetsons CAN-buss (`can0`) eller koblingene i USB-enhetsmodus (`usb0`, `usb1`) den tilbyr en verts-PC; en USB-Ethernet-adapter for kameraer teller fortsatt.

### Grunnleggende om nettverk (alle LATTICE-koblinger)

- **Adressering:** link-local `169.254.0.0/16` (GigE Vision LLA). Verts-PC-en bruker en statisk adresse `169.254.x.x/16`; kameraer og DAQ-E tildeler seg selv adresser i samme område. Ingen DHCP/gateway nødvendig.
- **Pakke størrelse:**foretrekker jumbo (9000), men la auto-proben finne den — den ser forbi kameraets 1500-byte ICMP-begrensning via en GVSP-probe, slik at den ender opp med jumbo uansett hvor kabelen faktisk klarer å overføre den.**Hvert kamera måles én gang per backend-økt, og igjen når tilkoblingen endres (et annet nettverkskort, en annen hastighet eller en tilkobling som har falt ut, slik det skjer ved bytte av switch); ved gjenoppkobling gjenbrukes resultatet. Etter en endring bak en switch som har vært oppe, må du starte backenden på nytt for å måle igjen.**En første tilkobling kan av og til ta noen minutter ved denne sjekken, og loggen viser dette når det skjer. Bruk `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` kun når du vet bedre enn sonden, og foretrekk per kommando fremfor permanent: en fast innstilling hopper over sonden, så hvis banen faktisk ikke kan overføre 9000, vil**hver** innfanging gå ut på tid med `SC_ERR_TIMEOUT -1011` (se [Miljøvariabler](#environment-variables)).
- **RX-ringen skaleres med `ReceiveBufferLen`:**ved standardverdien `32` er den brukbare ringen ~0,26 MB (for liten for enhver multikamera-burst); ved maksverdien `256` er den stor (~13,5 MB målt på laboratoriets 10GbE-vert), noe som gir reell kapasitetsreserve. Om en konfigurasjon kobles til, avgjøres deretter av den dreneringsbevisste adgangskontrollen**og** den samlede overskrivningskontrollen nedenfor — ikke en rå sammenligning mellom burst og ring.

### Array-fps og burst-modell

Slik leser du panelet «Array Settings» (og `lattice analyze-array` / SDKs `analyze_array_network`):

- **Burst summeres per kamera i hvert kameras faktiske pikselformat.**Mono**M3M**-kameraer strømmer**Mono12 (2 B/px)**, eller med `--pixel-format` dybdeformatet til Mono (`BayerRG8` er `Mono8`, 1 B/px);**M3C**-Bayer-kameraer strømmer 8- eller 12-bit (M3C-firmware sender automatisk BayerRG12 selv når BayerRG8 er forespurt). Så et bilde i full oppløsning fra 4 kameraer er**~12,6 MB hvis alle er 8-bit, men ~25 MB med tre 12-bit mono-kameraer**. Projeksjonen avgjør hvert kameras format ut fra modellen (identitetsbuffer), slik at burst-dataene samsvarer med det som faktisk overføres – ikke en standardantagelse om BayerRG8.
- **En USB-Ethernet-adapter er begrenset til 200 MB/s uavhengig av hva som står på typeskiltet.** Effektivitetstabellen som omregner en koblingshastighet til et vedvarende tall, er avledet fra PCIe; et USB-nettverkskort oppgir sin *Ethernet*-koblingshastighet, men er begrenset av USB-bussen og driveren. En USB 10GbE-dongle oppnådde tidligere ~1063 MB/s «vedvarende» – et tall som aldri ble målt – og den resulterende hastighetsreguleringen ødela 6–18 % av rammene, samtidig som den fortsatt rapporterte en god mål-fps. USB-tilkoblede nettverkskort er nå begrenset til et absolutt tak: **200 MB/s** på Windows og på NVIDIA Jetson, **400 MB/s** på Linux x86_64 (på Linux er et nettverkskort på en USB 2.0-port begrenset til **30 MB/s**, og spesifikasjonene anbefaler å koble det til en USB 3-port) (begrensningen ligger i vertsmaskinens USB-buss og driver, så den skalerer ikke i henhold til spesifikasjonene; et USB 1 GbE-kort oppnår ~80 MB/s og påvirkes ikke). På Linux identifiseres adapteren via sysfs (med tilbakefall til `ethtool -i`s `bus-info`). Hvert tall er målt: på Windows ødela det samme oppsettet med 4 kameraer 6–10 % av bildene over 200 MB/s; på Linux x86_64 fungerte det samme kortet feilfritt ved alle takverdier opp til 480 MB/s (273 MB/s levert, kameraenes egen grense). På NVIDIA Jetson fungerte det samme kortet feilfritt bare ved 200 MB/s og mistet bilder sporadisk over denne hastigheten. `wire_ceiling_source` i kapasitetsloggen beskriver dette med ord, og `nic_is_usb` markerer det. Overstyr dette uansett med `--wire-ceiling-mbps`.
- **Admittans er avhengig av drenering, ikke hel burst mot ring.** En samtidig burst trenger bare å passe inn i den *midlertidige backloggen* = `max(0, Σ per-cam arrival − host drain) × emit_window`, ikke hele bursten. På en struktur med rask vert / treg kamera (en **PCIe**10G-vert + 4× 1 GbE-kameraer: ankomst ≈ 320 MB/s, drenering ≈ 1063 MB/s) tømmes verten raskere enn kameraene fylles, backlog ≈ 0, så full-res sim-emit**slipper gjennom**selv om burst på 25 MB overstiger ringen på 13,5 MB. Plasser de samme fire kameraene bak en**USB** 10GbE-adapter på Windows, og uttaket er 200 MB/s, ikke 1063 — innkommende data overgår det, og tapet vises som ødelagte rammer i stedet for som en lavere bildefrekvens. På en 1 GbE-vert gjør kameraenes DLThr-grense på 31,25 MB/s at ankomsten overgår dreneringen → den **blokkerer** korrekt (for *denne* klassen av blokkeringer, reduser ROI eller bruk binning ≥ 2). Admittans er en av **to** koblingsporter — den andre er den samlede kontrollen for overtegnelse nedenfor.
- **Projisert fps er syklusen til det tregeste kameraet.**Ved samtidig utsending sender hvert kamera sin ramme samtidig, slik at matrisen sykluserer med den lengste av det tregeste medlemmets sensoravlesning og utsendelsestid (rammebytes ÷ utsendingshastigheten: kameraets**tilgangslink**, 1 GbE ≈ 80 MB/s, eller dens andel av vertsmaskinens båndbreddegrense når denne er lavere). Hver eksponering overlapper utlesningen av forrige bilde, så eksponeringen inngår ikke i syklusen: automatisk eksponering fyller perioden som frekvensen gir, øker deretter forsterkningen, og en mørk scene senker ikke lenger frekvensen (i versjon 1.2.0 gjorde den det). Bare en manuell eksponering som varer lenger enn perioden, bremser utløseren (se [Eksponering på et array-medlem](#exposure-on-an-array-member)). Å legge til kameraer bremser ikke matrisen før deres ledningsandel krymper. Ved seriell utsending (`slip-emit-and-capture`) summeres kameraenes utsendingstider i stedet, og hvert elements eksponering innregnes i forskyvningen. Målt ved full sensor, 12-bit, på et array med 9 kameraer med en PCIe 10 GbE-vert: 10,7 fps, eller 14,0 fps med [pakket transport](#packed-12-bit-transport---packed).
- **Etter en kontinuerlig opptaksserie angir projeksjonen hva den holdt.** Et opptak på 30 sekunder eller mer med automatisk hastighet registrerer den hastigheten den stabiliserte seg på ved den oppstillingsstørrelsen på denne datamaskinen; hastighetskontrollerenes korte forsøk på en raskere hastighet utelates. Fra da av angir linjen for `array-connect` og panelet «Array Settings» denne hastigheten når den ligger under modellens, sammen med hva som begrenset den: `clock-bound` (matrisen kjørte med den hastigheten den stabiliserte seg på, gripesløyfen ventet på utløserklokken sin), `loop-bound` (gripesløyfen hadde ikke tid igjen), `capture-bound` (innfangingen kom på etterskudd og hoppet over grupper) eller `measured` (ingenting angir hvilken). Modellens tall finnes i `wire_fps_max`. Arrayet klatrer fortsatt opp til det nettverket tåler.
- **En matrise fungerer på ethvert nettverk – det er bildefrekvensen som avgjør.**Hvert kamera får en jevn andel av**det kollisjonssikre nettverksloftet**(`sustained × sim_emit_factor`), synkronisert med GevSCPD, og alle eksponerer fortsatt ved samme utløser. På en treg kabel er andelen liten, og bildefrekvensen faller tilsvarende: 8 kameraer med full oppløsning og 12-bit på 1 GbE uten jumbo-rammer får 52 / 8 = 6,5 MB/s hver, omtrent**1 fps**(ca. 1,3 fps pakket), mens en 10G-vert kjører dem med 11–14. Hvis kameraene ikke kan synkroniseres så sakte – sjekk hvert kameras GevSCPD-rekkevidde for å finne ut av det – bytter arrayet til**seriell**utsending (`slip-emit-and-capture`): kameraene bruker ledningen etter tur med full koblingshastighet, utløst ett etter ett, slik at hver ramme fremdeles ankommer, men elementene**eksponeres ikke samtidig**. `analyze-array` og connect-utdataene angir modusen; sett `--force-tier sim-capture-sim-emit` til «refuse» for å avvise fallback. Bildestørrelsen påvirker ikke denne delingen, så**binning og et mindre ROI øker bildefrekvensen, men endrer ikke om et array passer inn**. Bare en ledning som ikke kan overføre matrisen selv serielt, er**overbelastet**— f.eks. langt flere kameraer enn én ledning kan håndtere, eller mange andre kameraer som strømmer på den — og der er symptomet GVSP-pakketap, ikke lavere hastighet: `analyze-array` setter tallene for oppnåelig fps til null og skriver ut `** OVER-SUBSCRIBED**`, tilkoblingen fortsetter med en tydelig advarsel, og en HTTP-tilkobling med `pin_resolution: true` (streng modus) **avvises** i stedet. Løsninger er færre kameraer, jumbo-rammer fra ende til ende eller et raskere nettverkskort. `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` nedgraderer avvisningen til en tydelig advarsel for testarbeid — se [Miljøvariabler](#environment-variables).

### Array-tilstand – hvilket delsystem mister rammer

`GET /api/camera/array/<array_id>/capability` for et tilkoblet array inneholder en live
`health`-blokk, som revurderes i et rullende **10-sekunders** vindu. Den deler opp rammetapet
i de to årsakene som krever motsatte løsninger, i stedet for å rapportere én «ufullstendig»
rate som ikke spesifiserer noen av dem:

| Felt | Hva det betyr | Hvilket undersystem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (per seriell) | Rammen **ankom og var strukturelt feil**— GVSP-pakketap. |**Nettverk**: ledningskapasitet, pacing, NIC RX-ring, MTU |
| `never_arrived_rate_pct` (per serienummer) | Rammen **kom aldri i det hele tatt**— kameraet utløste ikke, eller ingenting forlot det. |**Utløser / synkronisering**: M8-kabel, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Det dårligste kameraets andel for hver. | — |
| `per_cam_rate_pct` | Samlet andel ufullstendige bilder per kamera (begge årsakene samlet). | — |
| `stable_for_seconds` | Hvor lenge hvert kamera har ligget under 0,01 %. | — |

Over 5 % logger backenden en `[array-health <id>] WARN`-linje som angir årsaken — ved
første brudd, ved endring av alvorlighetsgrad, en gang i minuttet så lenge det vedvarer, og én gang når
det opphører. Den korrupte halvdelen skriver `[gvsp-corrupt <SN>]` til backend-loggfilen ved det
første treffet per kamera og årsak, deretter en oppsummering hvert 60. sekund; appens loggpanel viser i stedet
`[array-health <id>]`-linjen i stedet. Hver evaluering havner fortsatt i backend-loggfilen;
tellerne beveger seg ved hver buffer uavhengig av hva som skrives ut.

Den samme posten rapporterer tallet som hele tildelingen henger på:

| Felt | Hva det betyr |
| --- | --- |
| `wire_ceiling_mbps` | Vertens gjeldende vedvarende båndbredde, MB/s. |
| `wire_ceiling_source` | Hvor tallet kommer fra, i ord — f.eks. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` (på Linux x86_64, `USB-capped 400 MB/s (…; sysfs: bus=usb (usb2/2-1/2-1:1.0))`) eller `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` når `--wire-ceiling-mbps` (eller feltet **Wire Budget** i GUI-en) angir det. |
| `nic_is_usb` | `true` for et USB-Ethernet-kort — se USB-begrensningen ovenfor. |

**Tolkning:** `gvsp_corrupt_rate_pct` er ikke null og `never_arrived_rate_pct` er 0
betyr at utløsning og kabelsynkronisering er perfekt, og at 100 % av tapet ligger på nettverksbanen
— senk `--wire-ceiling-mbps` og koble til på nytt. Det motsatte mønsteret peker i stedet mot
synkroniseringskabelen eller utløserlinjen.

> **`--target-fps` er ikke avgjørende for korrupte rammer.** GevSCPD-takten skrives
> én gang ved tilkobling, så å senke utløserfrekvensen endrer arbeidscyklussen og ikke
> burstfrekvensen for samtidig utsending. En målt reduksjon av etterspørselen med 5× ga ingen forbedring;
> å senke ledningstaket fra 240 til 200 MB/s reduserte andelen korrupte rammer for samme rigg fra 10,4 %
> til 0,00 %.

> **Et array i drift kan ikke omplanlegge seg selv på LATTICE (M3C / M3M)-firmware.** Den
> kan ikke løse dette selv; koble fra og koble til på nytt, slik at valgverktøyet for tilkoblingstid kan
> planlegge på nytt med den nye grensen.

### Symptom → løsning

| Symptom (Array-innstillinger / tilkobling / `analyze_array_network`) | Årsak | Løsning |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` tilbakestilles til 32 (vanligvis etter en driveroppdatering) | Sett `ReceiveBufferLen`→256, `PendingReceives`→64; åpne panelet på nytt (start backend på nytt hvis den har lagret den gamle ringstørrelsen i hurtigbufferen) |
| Omstart/avslutning henger seg opp; senere `Invalid class` WMI-feil, NIC kan ikke aktiveres, USB-stasjoner mangler | Gammel Realtek USB 10GbE-driver fra 2016 → BSOD `0x9F` → tvungne avstengninger | Oppdater adapterdriveren til ≥ v10.67 (2026), og bruk deretter mottaksringinnstillingene ovenfor på nytt |
| Array kobler seg til med full sensor, men lav bildefrekvens | Auto opprettholder full sensor og lar frekvensen falle til det kabelen tåler | Velg et av de raskere alternativene i tilkoblingsutgangen (`--binning 2` / `4`, eller en mindre `--width`/`--height`), aktiver jumbo-rammer eller oppgrader koblingen |
| Array rapporterer en forventet fps, men leverer bare en brøkdel av den; `health.gvsp_corrupt_rate_pct` ikke-null, `never_arrived_rate_pct` 0 | Vertsmaskinens beregnede båndbreddeoverskudd er høyere enn det den faktisk opprettholder (typisk på et USB-Ethernet-kort, en smal PCIe-bane eller et delt nettverk) | Koble til på nytt med en lavere `--wire-ceiling-mbps` og sjekk helseblokken på nytt. **Ikke** `--target-fps` — GevSCPD-pacing er fastsatt ved tilkobling |
| Linux: en matrise mister rammegrupper og reduserer rammefrekvensen mens nettverkskortet ikke rapporterer tap; `lattice network` viser `socket receive buffers too small` | Kjernens mottaksbuffere for sokler er på standardverdien, så kamerapakker blir droppet i kjernen | Øk `net.core.rmem_max` / `rmem_default` til 32 MB — se [Linux-verter](#linux-hosts--socket-receive-buffers). .deb-pakken gjør dette ved installasjon |
| Kameraer mangler i publiserte grupper; `health.never_arrived_rate_pct` er ikke null, `gvsp_corrupt_rate_pct` 0 | Utløser-/synkroniseringsbane — kameraene utløses ikke, dette er ikke et nettverksproblem | Sjekk M8-synkroniseringskabelen og `--line`; bekreft at alle enheter er aktivert (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` overskredet i `analyze-array`, eller tilkoblingsavslag i streng modus (`pin_resolution`; `array over-subscribes the wire`) | Kabelen kan ikke håndtere matrisen uansett hvilken hastighet kameraene når, verken samtidig eller etter tur — det er langt flere kameraer enn kabelen kan dele, eller andre kameraer som strømmer på den | Færre kameraer, jumbo-rammer fra ende til ende, eller et raskere nettverkskort. **ROI/binning vil IKKE hjelpe** (deling er uavhengig av ramme-størrelse). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` overstyrer på testbenken (aksepterer pakketapet) |
| Array kobles til i `slip-emit-and-capture` (seriell) på et tregt nettverk | Kameraene kan ikke synkroniseres sakte nok til å dele kabelen samtidig, så de går på skift; bildene eksponeres ikke i samme øyeblikk | Jumbo-rammer eller et raskere nettverkskort gjenoppretter samtidig opptak; `--force-tier sim-capture-sim-emit` insisterer på det (og mister pakker i stedet) |

---

## `chloros-cli daq`

Kommandoer for spektralsensorer. `pool-*`-kommandoene er enkle HTTP-klienter som styrer sensoren gjennom backendens permanente pool, så de trenger en kjørende backend (se [Start backenden](#start-the-backend)). Backenden eier transporten, så CLI- og SDK-skriptene deler ett aktivt håndtak i stedet for å kjempe om den serielle porten. `pool-discover` er den eneste måten å finne ut BLE-MAC-adressen til en DAQ-M på.

> **`chloros-cli daq --help`** (eller `-h`, eller `daq help`) viser en liste over `pool-*`-underkommandoene (på Linux sammen med de direkte maskinvarekommandoene som følger med i kompileringen), og `chloros-cli daq <subcommand> --help` (eller `daq help <subcommand>`) viser alternativene for hver enkelt. De dekker tilkobling, stream, innspilling av kalibrerte `.daq`-filer, bytte av cap-profiler og tilkobling av en DAQ til et kamera, samt for en DAQ-E nettverksinnstillinger, PTP, omstart og firmwareoppdateringer. DAQ-en kan også styres fra Python med `chloros_sdk.connect_daq_sensor()`, som bruker den samme samlede banen.
>
> **Bruk `pool-*`-kommandoene på alle plattformer.** De fungerer på samme måte på Windows, Linux og Jetson. På Windows er er `daq discover` og `daq list` aliaser for `pool-discover`, og alle andre `daq`-kommandoer som ikke er `pool-*`, avslutter `1` med `DAQ direct-hardware commands require the full daq package`. På Linux og Jetson er `daq discover`, `daq list` og de andre kommandoene som ikke er av typen `pool-*` og `daq`, DAQ-verktøyets direkte-hardware-kommandoer: de åpner sensoren i CLI-ens egen prosess, utenfor backendens pool (altså ikke på en sensor som backenden, GUI-en eller et skript holder), og der aksepteres `daq discover`, `--only` og `--timeout`, men ikke `--json`. Denne referansen dokumenterer kun kommandoene `pool-*`.

### Arbeidsflyt for første tilkobling av DAQ-sensor

```bash
# 1. Scan every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U serial port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq pool-discover
chloros-cli daq pool-discover --only ble           # BLE only
chloros-cli daq pool-discover --json               # machine-readable

# 2. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect                                   # smart-detect
chloros-cli daq pool-connect --port /dev/ttyUSB0               # DAQ-U on a serial port (Windows: COM3)
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 3. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 4. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

#    Bind it to a camera, so that camera's captures use and save its readings
chloros-cli daq pool-bind --camera 213800234 CB-7C-A8-2E-5F

# 5. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 6. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### `pool-*`-referanse

| Underkommando | Formål |
| --- | --- |
| `daq pool-connect` (smart-detect) | Åpne en sensor i backend-poolen. |
| `daq pool-connect --port PORT` (`-p`) | DAQ-U på en spesifikk seriell port. |
| `daq pool-connect --ble` | DAQ-M over BLE, automatisk skanning av MAC-adresse. |
| `daq pool-connect --mac MAC` (`-m`) | DAQ-M via BLE med kjent MAC-adresse (forutsetter `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E over Ethernet med kjent vertsnavn eller IP-adresse (forutsetter `--eth`). |
| `daq pool-connect --eth` | DAQ-E over Ethernet, verten oppdages automatisk (mDNS + ARP-fallback; fungerer fra en tom ARP-cache på Windows og Linux). |
| `daq pool-connect --integration-time MS (-t) --frame-avg N (-f) --no-ae` | Juster integrasjonsvindu / AE-tilstand. |
| `daq pool-connect --no-stream` | Koble til, men ikke start strømming ennå (fortsett med `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Cap-korreksjonsprofil. Standard på backend er `sunshine_cosine`; `none` er en sensor uten deksel. En [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor) ignorerer den (fast diffusor). **Sensorene leveres uten deksel, så gi dekselet du har montert et navn:** standardverdien registreres som antatt (`cap_id_source=auto_default`), og behandlingen gir en advarsel om dette, siden en uutstyrt DAQ-U under solskinnskorreksjonen viser en verdi som er ca. 30 ganger for høy. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Skann hver transport for sensorer du kan koble til, uten å koble til. **Slik finner du BLE-MAC-adressen til en DAQ-M.** `--timeout` gjelder per transport (standard 5 s, 1–20). Sensorer som allerede er åpne i puljen, vises ikke – en tilkoblet DAQ-M slutter å annonsere – så bruk `pool-list` for disse. `usb` viser kun serielle USB-enheter: datamaskinens egne serielle porter (`COM1`, Linuxs `/dev/ttyS*`) kan ikke være sensorer og utelates. |
| `daq pool-list` | Vis alle sensorer i backend-poolen. |
| `daq pool-bind (--camera SERIAL \| --array ARRAY_ID) {SENSOR \| --clear}` | Knytt et kamera eller en matrise til en DAQ, slik som valg av lyssensor i GUI-en gjør: opptakene bruker, samsvarer med og lagrer målingene fra den DAQ-en. `SENSOR` er en sensor-ID fra `pool-list` eller adressen der DAQ-en nås. `--clear` (ikke `SENSOR`) opphever tilknytningen, og opptakene faller tilbake til den nyeste DAQ-en. Bindingen varer til backenden starter på nytt, eller til det aktuelle kameraet, arrayet eller DAQ-en kobles fra: en frakobling (her, fra SDK-et eller i appen) bryter alle bindinger til maskinvaren som kobles fra, uansett hvem som har satt dem opp, og medlemmene i et array kobles fra sammen med arrayet. Kameraet må være tilkoblet, i det åpne prosjektet, eller ha vært åpnet på denne datamaskinen tidligere, og matrisen må være tilkoblet eller i det åpne prosjektet; alle andre serienumre eller ID-er avvises på grunn av navnet, og ingenting blir bundet. |
| `daq pool-disconnect {--sensor-id ID \| --all}` | Frigjøring — uansett hvem som åpnet sensoren, slik at den også frigjør en sensor som et krasjet skript etterlot i puljen (eventuelle opptak på den fullføres først). En sensor som ikke er i puljen har ingenting å frigjøre: den skriver ut `ID: already released (not in the pool)` og avsluttes med `0`; en frigjøring som backenden avviste eller ikke bekreftet, avsluttes med `1`. SDK: `DAQSensorSession.force_disconnect()`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | De N siste spektrumrammene (standard 1). |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Gjenoppta / pause strømming. |
| `daq pool-record --sensor-id ID [--duration SEC] [-o DIR] [--device-name NAME] [--confirm-after S] [--heartbeat S]` / `--stop` | Start / stopp en .daq-opptak. `-o` er som standard satt til `~/Documents/DAQ Live View` på backend-datamaskinen (en relativ sti under mappen du kjører kommandoen fra); `--duration` (`-d`, standard `0` = til `--stop`). `--confirm-after S` gir en advarsel når det ikke har kommet noen måleverdi S sekunder etter start (standard 20; 0 slår den av); med `--duration` `--heartbeat S` skriver ut antall målinger hvert S. sekund og gir en advarsel når antallet slutter å øke (standard 30; 0 deaktiverer det). En opptak på en DAQ-E som blir stille, fortsetter i samme fil så snart backenden har koblet den til igjen, og `--stop` viser hullene ([En DAQ-E som slutter å levere](#a-daq-e-that-stops-delivering)). Ctrl+C avslutter en `--duration`-opptak før tiden og rapporterer dette; uten `--duration` fortsetter opptaket å kjøre etter Ctrl+C (avslutt `130`) inntil `--stop`. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Bytt kap-korreksjonsprofil under kjøring. `--cap-id` er påkrevd; `--cap-id none` fjerner kap-korreksjonen (en sensor uten deksel). En DAQ-E-S beholder innstillingen for `as_recorded` uansett hva som er angitt. |
| `daq pool-enable-ptp --sensor-id ID` | Slå PTP på igjen for en DAQ-E der PTP er slått av. Innstillingen lagres på enheten, slik at en enhet som er slått av, forblir av — tidsstemplene er ikke PTP-synkronisert — inntil denne kommandoen kjøres. `pool-list` markerer en slik enhet (`!! PTP is switched OFF`) og skriver ut denne kommandoen for den. Chloros slår aldri en enhet på igjen av seg selv, fordi det kan være bevisst at den er slått av. |
| `daq pool-info --sensor-id ID [--json]` | En DAQ-E-enhets egne innstillinger: dens fastvare (og om denne Chloros-utgaven inneholder en nyere versjon), adressen og hvordan den fikk den (DHCP / link-local / statisk), den lagrede nettverksinnstillingen inntil sensoren kjører den (en som er lagret med `--no-reboot` inkludert), strøm ved oppstart og PTP. På en sensor hvis strøm ikke når denne datamaskinen, leser den av sensoren ved en ny tilkobling og melder fra om dette. |
| `daq pool-set-network --sensor-id ID {--auto \| --static IP [--mask M] [--gateway G]} [--no-reboot] [--force] [--timeout S]` | Still inn en DAQ-E til DHCP (`--auto`) eller en statisk adresse (`--mask` standard `255.255.255.0`). Sensoren starter på nytt for å ta innstillingene i bruk og kobles til igjen med de innstillingene den hadde, og venter i opptil `--timeout` sekunder (standard 45) på dette; `--no-reboot` lagrer den bare. Se [DAQ-E-enhetsinnstillinger](#daq-e-device-settings). |
| `daq pool-set-auto-stream --sensor-id ID {--on \| --off}` | Om en DAQ-E skal starte strømming ved oppstart uten tilkoblet programvare (lagret på sensoren). Chloros starter strømmingen uansett. |
| `daq pool-reboot --sensor-id ID [--timeout S]` | Start en DAQ-E på nytt og koble den til igjen med de tidligere innstillingene (venter opptil `--timeout`, standard 45 s). |
| `daq pool-update-firmware --sensor-id ID [--timeout S]` | Oppdater DAQ-E-firmware som følger med denne Chloros-utgivelsen på en sensor som kjører en eldre versjon, og koble den deretter til på nytt (tar omtrent ett minutt; ikke slå den av; `--timeout` standard 45 s før den er tilbake). En sensor som allerede er oppdatert, blir ikke berørt. |

### En DAQ-E som slutter å levere data

En DAQ-E i puljen som ikke leverer noe spektrum i 20 sekunder, kobles til på nytt av backend-systemet. Dette gjøres ved å slå av og på strømbryteren som forsyner sensoren med strøm: sensoren starter på nytt og begynner ikke å streame igjen før den får beskjed om det (med mindre `pool-set-auto-stream --on` er angitt).

- **Den sjekker først at det er den samme sensoren.** Den spør på adressen sensoren hadde, uten å endre noe, om sensorens navn, MAC-adresse og serienummer. Den spør også under sensorens `daq-e-<id>.local`-navn, men stoler kun på det svaret når det kommer fra sensorens eget nettverkskort (MAC). Ingenting annet som svarer blir koblet til på nytt eller skrevet til.
- **Den kobler seg til på nytt med sesjonens innstillinger:** integrasjonstid, rammeutjevning, automatisk eksponering og cap-profil slik de var, ikke sensorens standardinnstillinger ved oppstart. En sensor som hadde PTP slått på og kom tilbake med det slått av, slås på igjen. Deretter starter den strømmen på nytt, samt datamaskinens mottaker for den.
- **Den prøver på nytt med økende intervaller:** umiddelbart, deretter etter 5, 10, 20 og 40 sekunder, og deretter hvert 60. sekund så lenge sensoren er borte. Hvert forsøk spør bare sensoren om hvem den er. En sensor som svarer, men fortsatt ikke sender noe etter tre gjenkoblingsforsøk, blir latt være og rapportert (`auto-reconnect GAVE UP`). `pool-stream --start` prøver på nytt.
- **`pool-list` viser status.** Mens sensoren er stille, angir linjen `STREAM FAULT` hva gjenoppkoblingen gjør. Linjen tømmes med det første spektrumet, og sensoren viser deretter `reconnected automatically N time(s)`.
- **En opptak fortsetter i samme `.daq`.** Filen markerer hvert gap med to rader: `event_type` 5 ved det siste spektrumet før gapet og 6 ved det første etter det, hver med kun en tidsstempel. `pool-record --duration` skriver ut `!! STREAM LOST …` mens sensoren er borte og `stream resumed after …` når den er tilbake. `pool-record --stop` viser hvert gap og gir en advarsel, slik den gjør for en tom fil (en opptak som ikke inneholder noen måling, avsluttes med `1`).
- **Et opptak beholder én kalibrering.** Hvis sensoren kommer tilbake med en annen kalibrering mens et opptak pågår, beholder resten av opptaket den kalibreringen det startet med, og stoppvarselet opplyser om dette. `pool-disconnect` + `pool-connect` tar i bruk den nye.
- **Dine kommandoer har forrang.** En sensor du har brutt forbindelsen til med `pool-disconnect`, eller satt på pause med `pool-stream --stop`, blir aldri koblet til på nytt, selv om en gjenkobling allerede er i gang.
- **`pool-info` på en sensor som ikke svarer** leser av sensoren ved en ny tilkobling og legger til `!! stream NOT reaching this computer`. En sensor som ikke svarer, rapporteres umiddelbart. `pool-list` viser ikke lenger en PTP-status for en sensor som har sluttet å svare.
- DAQ-U og DAQ-M er ikke omfattet. En DAQ-M kobler seg til igjen via Bluetooth av seg selv, og en DAQ-U som er koblet fra USB, trenger `pool-connect` på nytt.

### DAQ-E-S — fast diffusor og vippesensor

**DAQ-E-S** er en DAQ-E med to forskjeller som har betydning for programvaren. Alt annet — oppdagelse, tilkobling, PTP, opptak og enhetsinnstillingene nedenfor — fungerer nøyaktig som for en DAQ-E.**Diffusoren er fastmontert, så den krever ingen kappe-korreksjon.** Diffusoren i Sunshine-stil er innebygd og var på enheten da fabrikkalibreringen ble målt, slik at kalibreringen allerede viser den faktiske nedadgående strålingsintensiteten. Chloros-programmet setter kappen til `as_recorded` (ingen profil) uansett hva som blir forespurt: en cap oppkalt etter den (`sunshine_cosine`, `fov_*`, `none`) ignoreres med en melding, og opptakene lagrer `cap_id_source=model`, slik at advarselen om antatt tak aldri utløses for den. Ikke sett et tak på dataene manuelt heller: `sunshine_cosine` på toppen ville gi en avlesning som er omtrent 11 ganger for høy.

**Den har et 3-akset akselerometer.** Sollysdiffusorens kosinusrespons er kun gyldig når den er i vater, så en skråstilt sensor rapporterer feil innstråling — og refleksjonen arver feilen — uten andre symptomer. Hver DAQ-E-S-avlesning inneholder derfor sin egen orientering:

| Felt | Betydning |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Vinkler i grader. `None` når fastvaren markerer vinkelen som udefinert (fritt fall, en død akse). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Akselerasjon i milli-g. `mag_mg` langt fra ~1000 betyr at enheten var i bevegelse og at vinklene ikke er meningsfulle for den aktuelle prøven. |
| `cal_applied` | Innstilt: aksene og vinklene er i diffusorens referansesystem (enhetens monteringskalibrering er brukt). Uinnstilt: råt referansesystem for kortet, som kan vise opptil ~2,3° helling som ikke er reell. |
| `sample_age_ms` | Alderen på akselerometerprøven da spektrumet ble sendt. |
| `temp_raw` | Ukalibrert **trend** for die-temperatur — kun forskjeller, aldri en termometermåling. |

`daq pool-latest --json` returnerer dem under hver rammens `imu`-nøkkel (`null` på en DAQ-U, DAQ-M eller vanlig DAQ-E). En `.daq` registrert fra en DAQ-E-S lagrer de samme verdiene i sine `imu_*`-kolonner (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) ved siden av hvert spektrum, slik at leseren kan skille mellom en sky og et veltet stativ. På alle andre modeller er disse kolonnene tomme.

Holdningsstrømmen rapporteres separat: `daq pool-list` skriver ut `!! ATTITUDE STREAM FAULT (spectra unaffected)` mens den er nede, og hvor mange feil den gjenopprettet seg fra etterpå. En feil der medfører aldri et gap i spektralregistreringen. `daq pool-discover` oppgir enheten som `DAQ-E-S` (som `DAQ-E` hvis den ikke svarer på identitetskontrollen innen ca. 2 sekunder).

### DAQ-E-enhetsinnstillinger

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330

# A fixed address on the network this computer is on (192.168.1.x)
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1

# Back to the factory setting: DHCP, falling back to a 169.254.x.x
# link-local address when no DHCP server answers
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto

chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

- **`--static` uten `--gateway` lagrer ingen gateway** (eventuelle tidligere angitte gatewayer fjernes), noe som er riktig for et kameranettverk uten ruter.
- **Det må være en statisk adresse på denne datamaskinens nettverk.** Chloros-enheten avviser en enhet hvis undernett ikke inkluderer denne datamaskinens adresse i sensorens nettverk: etter omstarten vil ingenting kunne nå sensoren for å endre det tilbake. Bruk kun `--force` når denne datamaskinen skal omadresseres eller rutes til det nye undernettet.
- **En omstart avslås mens sensoren tar opp.** Stopp opptaket først (`pool-record --stop`). Et opptak som ble startet like før en omstart, avsluttes av denne, og `pool-set-network`, `pool-reboot` og `pool-update-firmware` avsluttes med en `Warning:`-linje som angir opptakets filnavn (også når sensoren da ikke kommer tilbake).
- **Etter en omstart kobler Chloros kun sensoren selv til igjen.** Den venter der sensoren befant seg – og etter `--auto` også under navnet `daq-e-<id>.local` – på et svar med sensorens navn, vertsnavn og modell, fra nettverkskortet (MAC-adressen) den hadde, og kobler den til igjen på adressen som svarte. Et svar under navnet aksepteres kun med den aktuelle MAC-adressen, så fra en datamaskin som når sensoren via en ruter, eller fra WSL2 i dets standard NAT-nettverk, blir en sensor som DHCP har flyttet til en ny adresse ikke koblet til på nytt: kommandoen melder dette og viser kommandoen `pool-connect --eth-host <its address> …`, som henter den tilbake med de innstillingene for lysbegrensning og eksponering den hadde.
- **`pool-update-firmware` avviser en sensor hvis firmwareversjon den ikke kan lese.** En sensor som bruker lengre tid enn ventetiden på å komme tilbake etter blitsen, har ikke sviktet: så snart den svarer, kobles den til igjen med innstillingene fra den nye firmwareversjonen, og kommandoen advarer om at den var treg.
- **En firmwareoppdatering krever en innkommende TCP-forbindelse.** Sensoren kobler seg tilbake til denne datamaskinen for å hente bildet. Chloros lytter på adapteren som videresender til sensoren (på alle adaptere, men kun når den ikke kan skille mellom dem) og betjener kun sensorens egen adresse; alle andre tilkoblinger lukkes uten å bli betjent. I Windows slipper installasjonsprogrammets brannmurregel `Chloros Backend (TCP-In)` den inn (se [Windows (.exe)](#windows-exe)); en brannmur som blokkerer den, fører til at oppdateringen mislykkes med feilmeldingen `… device never connected back on TCP <port> …`.
- **Under WSL2 må du bruke speilet nettverk** (`networkingMode=mirrored` i `%UserProfile%\.wslconfig`). I standard NAT-modus er denne datamaskinens adresse på sensorens nettverk WSLs egen, så alle statiske adresser avvises (med unntak av `--force`), live-strømmen når ikke WSL, og en firmwareoppdatering kan ikke fullføres, fordi sensoren kobler seg tilbake til datamaskinen for å motta bildet.
- **Sensor-ID-en og vertsnavnet endres ikke.** Uansett hvilken adresse den har, er en DAQ-E fortsatt `daq-e-<id>` og `daq-e-<id>.local`, så `--eth-host daq-e-<id>.local` finner den fortsatt.
- **GUI-en har de samme innstillingene**: Fanen «Lyssensorer», sensorens innstillinger (tannhjulet), «Nettverk». GUI-en opprettholder sin egen tilkobling til en sensor, så hvis du endrer en sensor som GUI-en har åpen, må du gjøre det fra GUI-en: en omstart fra CLI-en avslutter GUI-ets økt med den.
- De tilsvarende SDK-ene er `DAQSensorSession.device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` og `update_firmware()`.

---

## `chloros-cli project`

Åpne, koble til og kjør et lagret Chloros-prosjekt (en mappe med `cameras.json` + `sensors.json` + `project.json`). Alt går gjennom backenden, slik at GUI og CLI gir identisk maskinvarestatus.

### Referanse for underkommandoer

| Underkommando | Formål |
| --- | --- |
| `project open PATH [--relink FOLDER]` | Skriver ut prosjektets enhetsmanifest (kameraer, matriser, sensorer), leser fra filene og sjekker kildefilene (se [Et flyttet prosjekt](#a-moved-project---relink)). Den åpner ikke prosjektet i backend: prosjektet som er åpent der, samt enhetene og opptakene, forblir som de er. Avslutter `1` når ingen av prosjektets kildefiler kan finnes. |
| `project devices PATH [--reconnect]` | Viser eller kjører oppdagelsen på nytt. |
| `project connect PATH [--cameras-only] [--sensors-only] [--packed {auto,on,off}]` | Kobler til hvert lagret kamera / array / sensor. Hvert array kobles til på samme måte som `lattice array-connect` gjør: i opptil 600 s, med trinnene skrevet ut til stderr (`[array <id>]    <s> s: <step>`; stdout er JSON-rapporten), og et array som ikke er koblet til innen den tiden, blir tilbakestilt — kameraene settes tilbake til slik de ble funnet — og rapporteres under `errors`, og forblir aldri koblet til. `--packed` angir hvert arrays pakkede 12-bits transport (standard `auto`; et prosjekt lagrer ingen innstillinger per matrise; se [Pakket 12-bits transport](#packed-12-bit-transport---packed)). Hver lyssensor åpnes i backendens DAQ-pool på den lagrede adressen med den lagrede integrasjonstiden, rammeutjevning, automatisk eksponering og cap: en sensor uten valgt cap får `sunshine_cosine`, en som er lagret uten cap (`none`) forblir uten cap, og en DAQ-E-S tar aldri en cap. GUI-ens DAQ-panel er ikke en del av puljen: lukk en sensor der først. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Enkeltopptak fra et navngitt kamera eller en matrise (`--prefix` som standard, `capture`). `--orient` (matriser): matrisen lagrer rådata, som forblir i opprinnelig format — dette registrerer at produktene `process` lager av dem skal orienteres, som `array-capture --orient`. Standard: opprinnelig. `project burst` / `stream` og en oppskrifts `capture` / `burst` / `stream` (`orient: true`) bruker samme alternativ. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | N-ramme-serie fra et navngitt kamera eller en matrise (`-n/--count` standard 5; `-i/--interval` sekunder mellom rammer, standard 0). Seriebilder fra et array fjerner duplikater av gjentatte synkroniserte grupper (foreldelsesvakt), slik at et array med delvis syklus ikke kan levere tilbake N kopier av ett bilde; skriver ut resultater per iterasjon. En opptak som mislykkes underveis, stopper serien og avslutter `1`, med opplysning om hvor mange som ble lagret før det. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--orient]` | Innfanging til disk i en sløyfe: `--fps F` fanger inn en sekund (standard 3), ett bilde per opptak fra et kamera, én synkronisert bildegruppe fra et array (med samme foreldelsesvakt som `project burst`), inntil `-n N` tar opp (standard `0` = til Ctrl+C, som viser hvor mange som ble lagret og avslutter `0`, eller `1` når ingen ble lagret eller en fil mislyktes; et Ctrl+C før `-n N` fanger opp, avslutter `130`). En innspilling som mislykkes underveis, stopper strømmen og avslutter `1`, og oppgir hvor mange bilder som ble lagret før det. `--poll-interval` er utdatert og ignoreres: den aksepteres fortsatt slik at eldre skript kan kjøres, med en merknad på stderr. |
| `project sensor read PATH NAME [--json]` | Siste spektrumramme. |
| `project sensor log PATH NAME [--seconds SEC] [-o DIR] [--device-name NAME]` | Ta opp en .daq-fil for `--seconds` (standard 10). En opptak som ikke inneholder noen målinger, rapporteres på stderr, og  avsluttes`1`. |
| `project run PATH RECIPE.yaml` | Kjør en YAML-/JSON-opptaksopskrift. `--dry-run` validerer uten å kjøre. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Beregn justering for en matrise — se [flaggtabellen nedenfor](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Skriv ut gjeldende justeringsprofil. |
| `project align clear PATH NAME` | Slett den bufret profilen. |
| `project align tweak PATH NAME --serial SN [--dx N] [--dy N] [--rotation-deg N] [--scale N]` | Juster transformasjonen til én slave (standardverdier `0`, `0`, `0`, `1.0`: ingen endring). |
| `project align export PATH NAME --to FILE` | Lagre profilen som JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Last inn en lagret profil. |

#### `project align calibrate` Alternativer

| Flagg | Standard | Beskrivelse |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Justeringsmetode. **Disse stavemåtene skiller seg fra `lattice align-calibrate`**, som bruker kortformene `orb` / `akaze` / `phase`; de to kommandoene kan ikke brukes om hverandre for dette flagget. |
| `--model {translation, rigid, affine, homography}` | `affine` | Transformere modellen for å tilpasse den. |
| `--frames N` | `1` | Synkroniserte bildeutsnitt til gjennomsnittet. |
| `--reference SN` | masteren | Referansekameraets serienummer; hvert annet medlem blir forvrengt til å passe til det. |
| `--max-features N` | `5000` | Øvre grense for antall ORB-trekk. |
| `--ratio-threshold F` | `0.75` | Lowe's forholdstest. |
| `--ransac-threshold-px F` | `3.0` | RANSAC-terskel for inliers. |
| `--min-matches N` | `15` | **Kvalitetsgrense** — avvis løsningen dersom antall inlier-treff er lavere enn dette. |
| `--max-reproj-err-px F` | `4.0` | **Kvalitetsgrense** — avvis løsningen dersom RMS-feilen ved reprojeksjon overstiger denne verdien. |
| `--checkerboard RxC` | — | Brettgeometri for `--method checkerboard`, f.eks. `9x6`. |
| `--name PROFILE` | tom | Profilnavn innebygd i den lagrede JSON-filen. **Ikke matrisenavnet** — det er posisjonsnavnet `NAME`. |

De to kvalitetskontrollene er grunnen til at en kalibrering kan lykkes med å løse oppgaven, men likevel
nekte å lagre: en profil som ikke består en av dem, vil uten forvarsel registrere alle
senere opptak feil, så den avvises i stedet for å lagres.

### Et flyttet prosjekt (`--relink`)

Et prosjekts `project.json` navngir hver kildefil – hvert bilde og hver lyssensor-
opptak – med sin fulle sti. Flytt prosjektmappen, eller hele prosjektmappen,
til en annen stasjon, og disse stiene vil fortsatt peke til den gamle plasseringen.

Når du åpner et prosjekt, retter programmet det det kan på egen hånd: en fil som lå i prosjektets
mappe, blir søkt etter i mappen der prosjektet nå befinner seg, og hver sti som finnes der
blir omdirigert og lagret (`project.json` blir først sikkerhetskopiert som
`project.json.before-relink-<date>-<time>`). Dette skjer ved hver åpning – av appen,
`chloros-cli project open`, `chloros-cli process` inn i et eksisterende prosjekt, SDK-ets
og `chloros-mcp`s. En sti blir kun omdirigert til en fil som faktisk finnes der.

Filer som befant seg utenfor prosjektmappen – en hub-økt, en «Legg til mappe»-import –
kan ikke bli funnet på den måten. Oppgi navnet på mappen de nå befinner seg i:

```bash
chloros-cli project open "N:/Chloros Projects/Field_A" --relink "E:/backup/Hub Projects"
```

`--relink` tar mappen filene ble plassert i, eller en hvilken som helst mappe over den: mappen til hver manglende
fil søkes etter under den ved hjelp av de etterfølgende mappenavnene
(`Hub Projects/<session>/raw/<serial>`, deretter `<session>/raw/<serial>`, ...), og filene i hver
mappe hentes fra det stedet der flest av dem blir funnet. `project open` viser
hva som ble omdirigert, hvor mange kildefiler som ble funnet, og mappene de resterende befant seg
i. Noen mangler fortsatt: en advarsel, avslutt `0` – prosjektet åpnes med resten. Ingen
funnet: en feilmelding som oppgir disse mappene og `--relink`, avslutt `1`; backenden
nekter å åpne et slikt prosjekt for `process` eller `project devices --reconnect` (HTTP 409 `PROJECT_SOURCES_MISSING`).

### Eksempler

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Its hub captures moved to another drive: point it at them
chloros-cli project open "/home/user/Chloros Projects/Field_A" --relink "/media/backup/Hub Projects"

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
        auto_exposure: "Off"
        gain_db: 6.0
        auto_gain: "Off"
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

Støttede handlinger: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. `apply` henter nøklene `cameras.json`, `settings` — `exposure_us`, `auto_exposure`, `gain_db`, `auto_gain`, `target_brightness`, … (de eldre navnene `exposure_time`, `exposure_auto`, `gain`, `gain_auto` godtas som aliaser). En eksponering, forsterkning eller et mål som er angitt på denne måten, beholdes som ditt: Chloros’ automatiske eksponeringskontroll (inkludert eksponeringen målt av DAQ) lar den være i fred. En manuell eksponering lar forsterkningen være som den var: i auto-modus fortsetter kameraet å justere forsterkningen (over hele området) mot mållysstyrken inntil du angir `gain_db` eller `auto_gain: "Off"`. En senere `auto_exposure: "Continuous"` gir eksponeringen tilbake til automatikken, og `target_brightness_auto: true` gir et fastsatt mål tilbake til Chloros’ automatiske mål. Handlingen `burst` krever `name` (påkrevd), `count` (standard 5), `interval` (sekunder, standard 0), `output`, `format`, og `settings` (samme form for innstillinger per kamera som `apply`); array-serier bruker den samme nylig synkroniserte gruppevakten som `project burst`.

A `capture`, `burst` eller `stream`-trinnet, er standardverdien for `output` satt til `output`. En relativ `output` — som er inkludert som standard — blir løst i forhold til mappen du kjører `chloros-cli project run` fra, ikke oppskriftsfilens mappe eller prosjektmappen (i versjon 1.2.0 havnet den i backendens installasjonsmappe).

Kjør den:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

**Hver kjøring sjekkes først opp mot prosjektet.**Hver handling må ha nøyaktig ett av verbene ovenfor som nøkkel — `- capture: {name: FrontLeft}`, ikke `- action: capture` eller `- type: capture` — med et `name` har prosjektets `cameras.json` / `sensors.json` et kamera eller en matrise for `apply` / `capture` / `burst` / `stream` og en lyssensor for `sensor`; en `stream` trenger at `count` > 0 og en `wait` en varighet. En oppskrift med en slik feil utfører**ingen** handling (`[recipe] error: …`, `[recipe] not run: …`, avslutt `1`), selv med `stop_on_error: false`. Taster som et trinn ikke bruker, er en `[recipe] warning:`, det samme gjelder alle matriser eller lyssensorer som oppskriften styrer, men som ikke er tilkoblet nå. En oppskrift kobler aldri til enheter: kjør `chloros-cli project connect PATH` først. `--dry-run` skriver ut oppskriften, deretter de samme feilene og advarslene, og `[recipe] valid: …` / `NOT valid: …`, kjører ingenting og avslutter `1` når oppskriften har en feil.

---

## Miljøvariabler

| Variabel | Effekt |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Overstyrer backend-URL-en (standard `http://127.0.0.1:5000`) — **respekteres kun av kommandofamiliene `lattice`, `project` og `daq pool-*`** (`lattice network-analysis` bruker i stedet `--backend-url`). Kjerne-kommandoene (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) kobler til `http://127.0.0.1:<port>` og ignorerer denne variabelen (IPv4-literalet omgår Windows-straffen på ca. 2 sekunder per forespørsel for `localhost`→`::1`), slik at de alltid retter seg mot den lokale maskinen. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | Angis i backendens miljø. `1` nedgraderer avslaget ved overtegnelse — en array-tilkobling med `pin_resolution`, eller en [kommando for midlertidige flerkamera-økter](#multi-camera-transient-sessions) (`error_code` `ARRAY_OVERSUBSCRIBED`) — til en tydelig advarsel, og aksepterer tap av GVSP-pakker. Kun for testing — se [Array fps & burst-modell](#array-fps--burst-model). |
| `CHLOROS_ARRAY_PACKED12` | Angis i backendens miljø: `on` eller `off` angir standardverdien [pakket 12-biters transport](#packed-12-bit-transport---packed) for hvert array (standard `auto`). En eksplisitt angivelse av `--packed` har fortsatt forrang. |
| `CHLOROS_DISABLE_UPDATE_CHECK` | `1` (eller `true` / `yes`) deaktiverer [automatisk oppdateringssjekk](#chloros-cli-update) i Linux-CLI ved oppstart. |
| `CHLOROS_CLI_MODE` | Angis av selve CLI-en; ber backenden om å aktivere parallellbehandling. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` hopper over GVSP-fallback-sonden (kun ICMP-resultater). **Dette deaktiverer jumbo, det demper ikke bare loggen** — kameraet svarer kun på DF-pinger opp til 1500 på hver bane, så denne sonden er det eneste som kan oppdage jumbo. Sparer ~1 s per kamera per tilkobling; koster ~1,45× ledningens takhastighet hvis nettverket *kunne* ha håndtert jumbo. SDK-et gir en advarsel når du setter dette. |
| `CHLOROS_GVSP_SESSION_REUSE` | Angis i backend-miljøet: `0` måler pakkestørrelsen ved hver tilkobling i stedet for én gang per kamera per backend-økt. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Fester GVSP-pakkestørrelsen til N byte; hopper helt over sondering. Anbefal å bruke per-kommando (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) fremfor å sette den permanent: en fastsatt størrelse slutter å tilpasse seg nettverket foran seg, og å fastsette 9000 på en bane som ikke kan håndtere jumbo-pakker fører til at **hver** innfanging går ut på tid med `SC_ERR_TIMEOUT -1011`. |
| `CHLOROS_TRANSIENT_AE_SETTLE_S` | Angis i backend-miljøet: det maksimale antallet sekunder en [midlertidig flerkamera-kommando](#multi-camera-transient-sessions), eller en `lattice capture` for et kamera som ikke har noe å holde fast ved, bruker på å stille inn hvert kameras eksponering fra scenen før det lagrer et bilde (standard `5`). `0` deaktiverer dette: hvert kameras egen automatiske eksponering kjører da fra det første bildet, med utgangspunkt i eksponeringsgrensen. |
| `CHLOROS_LOG_DIR` | Angis i backendens miljø: mappen der den detaljerte loggen skal lagres. Standard `%LOCALAPPDATA%\Chloros\logs` på Windows, `~/.cache/chloros/logs` på Linux (`$XDG_CACHE_HOME/chloros/logs` når dette er angitt). .deb-filens rottjeneste angir `/var/log/chloros`, som er lesbar for gruppen `adm` — se [Kjøre backenden som en tjeneste](#running-the-backend-as-a-service-headless). |
| `TMPDIR` (Linux) | Hvor midlertidige filer lagres. CLI-en bruker automatisk `/mnt/ssd/tmp` hvis den finnes. |

---

## Avslutningskoder

| Kode | Betydning |
| --- | --- |
| `0` | Vellykket. |
| `1` | Generell feil (de fleste feil i underkommandoer). En opptakskommando avsluttes også med `1` når en fil den ble bedt om å skrive til ikke kunne skrives, selv om de andre ble lagret, og `array-burst` / `array-record` avslutter `1` når opptakeren rapporterer en feil — se [Filer som ikke kan skrives](#files-that-fail-to-write). `process` avsluttes også `1` når eksportene fra en kjøring ikke kunne verifiseres, eller når den mistet et bilde, selv om de andre produktene ble skrevet. |
| `2` | Argumentfeil (et ukjent eller manglende flagg; gjelder også `time-sync set-priority` uten noen av disse prioritetene). |
| `130` | Avbrutt av Ctrl+C: alle kommandoer som avbrytes — `lattice`, `project`, `daq` og `process` (`process` etter at backenden har stoppet kjøringen, se [`process`-merknader](#notes)). En kommando som normalt stoppes med Ctrl+C — `lattice array-capture --continuous` / `--interval`, `lattice viewer`, `lattice continuous`, `multi-stream`, `align-stream`, `array-record`, `array-burst` og `project stream` med `-n 0` — avsluttes av det den faktisk gjorde: `0` når alt var skrevet, `1` når noe mislyktes. |
| `143` / `129` | Avsluttet av SIGTERM / SIGHUP (128 + signalet), etter at kommandoens opprydding ble kjørt: en kjørende jobb ble stoppet og kameraene satt tilbake, en behandlingskjøring ble stoppet i backend. En `lattice continuous` eller annen opptak som normalt stoppes med Ctrl+C, avslutter også `143`, ikke `0`. |

`lattice network-analysis` avslutter `1` for avgjørelsene `needs_force_slip` og `error`, og `selftest` avslutter `1` med mindre alle syv kontroller bestås.

---

## Tips til feilsøking

- **«Pålogging kreves»** → Kjør `chloros-cli login EMAIL PASSWORD` én gang på denne maskinen.
- **«Prosjektmappen er for dyp for denne datamaskinen»** → Lange stier i Windows er deaktivert på denne maskinen (Windows-standard), så ingen sti kan være lengre enn 260 tegn, og produktene ligger opptil ca. 130 tegn under prosjektmappen. Ingenting ble behandlet. Meldingen angir den lengste prosjektmappen som passer: bruk en kortere (`-o` eller `chloros-cli set-project-folder`), eller slå på lange stier i Windows og start Chloros på nytt. Windows-installasjonsprogrammet tilbyr dette som «Aktiver lange Windows-stier», som er merket av som standard; ellers bruk gruppepolicy: *Datamaskinkonfigurasjon > Administrative maler > System > Filsystem > Aktiver lange Win32-stier*.
- **«backend unreachable»** → Start Chloros-skrivebordsappen (Windows), eller start backenden selv: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` på Windows (mappen `backend` i installasjonsmappen din; den finnes ikke i `PATH`), `chloros-backend` på Linux. Eller sjekk `CHLOROS_BACKEND_URL` hvis det er eksternt.
- **`lattice`-kommandoer mislykkes med meldingen «LATTICE-kameradrivere ikke funnet»** → Backenden klarte ikke å laste inn LATTICE-kamerakjøringsmiljøet som den er installert med. Alle `lattice`-kommandoer kjøres på backend; CLI-en i seg selv inneholder ingen kameradrivere. Installer Chloros på nytt: både Windows-installasjonsprogrammet og .deb-filen installerer runtime ved siden av backend.
- **Array connect / Array Settings viser «FRAMES WILL DROP» eller «Reduce ROI to enable»** → Mottakerringen på verts-NIC-en er for liten (blir ofte tilbakestilt til 32 etter en oppdatering av NIC-driveren). Se [Konfigurering og justering av verts-NIC](#host-nic-setup--tuning-lattice-arrays) — sett `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Maskinen henger seg opp ved omstart/avslutning, deretter WMI `Invalid class` / nettverkskortet aktiveres ikke / USB-stasjoner mangler** → Utdatert driver for USB 10GbE-adapter forårsaker `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Oppdater adapterdriveren — se [Konfigurering og justering av verts-NIC](#host-nic-setup--tuning-lattice-arrays).
- **Linux: opptakene tilhører root, så du kan ikke slette dem, og backendens logg finnes ikke i din `~/.cache`** → Backenden kjører som root via `chloros-backend.service`. Bytt til `chloros-backend@<user>.service`, som kjører den som deg — se [Kjøre backenden som en tjeneste](#running-the-backend-as-a-service-headless). Inntil da finnes loggen til root-tjenesten i `/var/log/chloros` (lesbar for gruppen `adm`).
- **Linux: en backend-enhet er `failed` med meldingen «En annen instans kjører allerede» (avslutningsstatus 98)** → En annen backend har allerede porten: bare én kan kjøre om gangen. Hvis to backend-enheter er aktivert, behold én: `sudo systemctl disable --now <unit>` for den du ikke bruker — se [Kjøre backenden som en tjeneste](#running-the-backend-as-a-service-headless). Hvis bare denne enheten er aktivert, er det en backend som er startet utenfor systemd (ved hjelp av `chloros-backend`, `chloros-cli` eller et SDK-skript) som opptar den: stopp den backenden, og deretter `sudo systemctl start <unit>`.
- **Jetson-swap-advarsel** → Legg til filbasert swap; CLI-en viser de nøyaktige `fallocate` / `swapon`-kommandoene.
- **«backend not reachable» fra en `lattice`-, `project`- eller `daq`-kommando** → ingen backend kjører: disse kommandoene starter aldri en. Kjør `chloros-cli backend status`, og se [Start backenden](#start-the-backend).
- **«`daq net ...` er ikke tilgjengelig i Chloros»** (Linux og Jetson) → Noen av DAQ-E-enhetens egne innstillinger er reservert for MAPIR-tjenesten. Nettverksinnstillinger, omstart og firmwareoppdateringer er `pool-set-network`, `pool-reboot` og `pool-update-firmware`; kontakt MAPIR-support for resten.
- **«DAQ-kommandoer direkte til maskinvaren krever hele daq-pakken»** (Windows) → Bruk `pool-*`, som styrer den samme sensoren via backenden, eller `chloros_sdk.connect_daq_sensor()` fra Python.

---

## Se også

- [Python SDK-referanse](sdk-reference.md) — programmatisk ekvivalent til alle CLI-kommandoer.
- [DAQ-lyssensorer](https://mapir.gitbook.io/chloros/daq-light-sensors) i Chloros-håndboken — sensorene, deres grenseverdier og kalibrerte område, opptak, DAQ-E-nettverk.
- [LATTICE-kameraer](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) i Chloros-håndboken — tilkobling, kamerainnstillinger, opptaksmoduser, matriser.
- Online dokumentasjon: `https://mapir.gitbook.io/chloros/cli`
