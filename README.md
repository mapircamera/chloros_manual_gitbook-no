---
metaLinks: {}
---

# Kom i gang

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>

Chloros
er et program fra [MAPIR
](https://www.mapir.camera) for behandling av multispektrale bilder, live-kontroll av MAPIR
-maskinvare og registrering av sensordata. Chloros
1.3.0 støtter hele MAPIR
-produktfamilien:

* **Survey3
-kameraer** — behandler RAW+JPG-opptak til kalibrerte refleksjons- og vegetasjonsindekskart. Se [Støttede kameraer](supported-cameras.md).
* **LATTICE-kameraer** — koble til GigE-multispektrale kameramoduler i sanntid, enkeltvis eller som synkroniserte flerkamerasystemer: forhåndsvisning, opptak og bearbeiding til kalibrerte strålings- og refleksjonsprodukter. Se [LATTICE-delen](lattice/README.md).
* **DAQ-lyssensorer** — DAQ-U (USB), DAQ-M (Bluetooth) samt DAQ-E og DAQ-E-S (Ethernet) spektralsensorer: kalibrerte spektra i sanntid, `.daq`-opptak og nedadrettet belysning for refleksjonsbehandling. Se [DAQ-delen](daq/README.md).

{% hint style="success" %}
**Hva er nytt i Chloros
1.3.0**: AI-assistenter kan styre Chloros
via `chloros-mcp`, og CLI
, SDK
og `chloros-mcp` tilbyr de samme funksjonene. LATTICE-indeksproduktene er forbedret, fullsensor-matriser er raskere og fungerer på alle nettverk, og backenden fortsetter å kjøre mellom kommandoene. Se listen over nyheter nedenfor, og [Last ned](download.md) for installasjonsprogrammene.
{% endhint %}

{% hint style="info" %}
**Bruker du Chloros
med en AI-assistent?** Denne håndboken er laget for det. Be assistenten din om:

* `https://mapir.gitbook.io/chloros/llms.txt` — maskinlesbar indeks over alle sider.
* Enhver side som rå Markdown — legg til `.md` i URL
-en (f.eks. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI
-referansen](reference/cli-reference.md) og [SDK
-referansen](reference/sdk-reference.md) — komplette referansesider med eksakte verdier, skrevet for bruk av store språkmodeller (LLM).

Eksempel på kommando: *«Les https://mapir.gitbook.io/chloros/reference/cli-reference.md,, og skriv deretter et skript som logger inn og behandler mappen ~/flights/flight_001 til GeoTIFF-filer med reflektans og NDVI
.»*

Eller la den kjøre Chloros
direkte: `chloros-mcp`, en MCP-server installert med SDK
, fungerer med Claude Desktop, Claude Code, Cursor og andre MCP-klienter.

Fullstendig veiledning: [Bruk av Chloros
med AI-assistenter](ai-assistants.md).
{% endhint %}

***

## Hva er nytt i Chloros
1.3.0

* **AI-assistenter kan styre Chloros
.** `chloros-mcp`, en MCP-server installert med Python
-SDK
, lar Claude Desktop, Claude Code, Cursor og andre MCP-klienter behandle bilder og betjene LATTICE-kameraer, matriser og DAQ-lyssensorer. Se [Bruk av Chloros
med AI-assistenter](ai-assistants.md).
* **CLI
, SDK
og `chloros-mcp` tilbyr de samme funksjonene.** Av sikkerhetshensyn forblir kun pålogging, DAQ-E-konfigurering og de direkte DAQ-serverne i CLI
. Noen 1.2.0-skript må endres: se **Endret i 1.3.0** øverst i [CLI
-referansen](reference/cli-reference.md) og [SDK
-referansen](reference/sdk-reference.md).
* **Forbedrede LATTICE-indeksprodukter.** NDVI
og de andre indeksproduktene som eksporteres fra LATTICE-opptak, er forbedret i alle behandlingsmoduser.
* **Windows
-installasjonsalternativer.**Under**Systeminnstillinger**tilbyr installasjonsprogrammet**Aktiver lange Windows
-stier**og**Ekskluder Chloros
-backend fra Microsoft Defender-skanning**, begge merket av som standard. Det første alternativet tillater at prosjektmapper har navn med mer enn 260 tegn; det andre hindrer at sanntidsskanning bremser kameraopptaket (ca. 30 % av opptakshastigheten ved høye hastigheter). Installasjonsprogrammet legger nå også til CLI
-mappen i `PATH` bare én gang, og fjerner de dupliserte oppføringene som tidligere installasjonsprogrammer etterlot seg. Se [Nedlasting](download.md).
* **Dype prosjektmapper avvises på forhånd.** Når lange stier i Windows
er deaktivert (Windows
-standard), avvises en prosjektmappe som er for dyp for stigrensen på 260 tegn før noe kjøres, og det vises informasjon om den lengste prosjektmappen som passer, samt hvordan man aktiverer lange stier. Tidligere skrev en slik kjøring ingen produkter. Når lange baner er slått på, beholder produkter i svært dype mapper nå metadataene sine (GPS- og XMP-tagger).
* **Klippemasker.** Når båndseparasjonen av et LATTICE-strålings- eller refleksjonsprodukt gir en negativ båndverdi, blir denne verdien klippet til null, og en maske per piksel i en `clamp_masks/`-mappe ved siden av produktmappene registrerer hvilke piksler og bånd som ble berørt. Produktet navngir masken sin i `Chloros:UnmixClampMask`-taggen. Live-visninger klipper nå nøyaktig på samme måte som eksporten gjør. Se [Utdata-bildeformater](output-image-formats.md).
* **Produktene beskriver seg selv.** Indeksprodukter har egne behandlingskoder (`Chloros:ProcessingLevel=index`, indeksuttrykket og om de ble beregnet ut fra radianse eller reflektans) i stedet for kildens rådata. Survey3
-produkter har Chloros
-behandlingskoder i alle behandlingsmoduser. LATTICE-råfiler og debayered-filer beholder sensorens opprinnelige orientering og registrerer orienteringen i XMP; Image Viewer roterer dem kun for visning.
* **Linux
-pakker leveres ukomprimert.** Backenden og CLI
-en installeres som mapper under `/usr/lib/chloros/`, slik at de ikke lenger pakker seg ut ved hver oppstart. På x86_64 bruker backenden ikke lenger rundt 100 sekunder på å starte, og `.deb` er omtrent halvparten så stor som før. Det medfølgende ExifTool kjører nå på en ren Ubuntu-installasjon. `.deb` installerer nå også Python
-SDK
for systemet `python3` (installer den med `sudo apt install ./chloros_1.3.0_<arch>.deb`), øker kjernens mottaksbuffere for sokler som kamerastrømmer trenger, og legger til en backend-tjeneste som kjører som din bruker. Jetson-pakken, `chloros_1.3.0_arm64_jp6.deb`, er bygget for JetPack 6 og behandler data på Jetsons GPU med JetPacks CUDA-versjon av PyTorch. På Linux
når hvert eksportert bilde nå disken én gang (skrevet og merket i minnet, deretter kopiert til sin plass): på en Jetson, der disken bestemmer tempoet, tar en LATTICE-kjøring med 288 bilder 14 minutter i stedet for 24. Se [Linux
-installasjon](linux/linux-installation.md).
* **LATTICE OCN
-kameraer leser mer nøyaktig i sollys.** Radiance, reflektans og indekser får en hurtig korreksjon etter båndseparasjonen, slik at et grått mål leses som nær nøytral NDVI
i sollys. RGN
-kameraer er uendret.
* **Feilrapporter i appen.** Et feilikon øverst på fanen «Debug Log» sender en beskrivelse, opptil 10 bilder, loggene og systemdetaljer til MAPIR
. Se [GUI: Navigasjon](navigation.md).
* **LATTICE-matriser.**Matriser beholder full sensor som standard, og pakket 12-bits transport (`--packed`, standard `auto`) øker bildefrekvensen for full sensor med omtrent en tredjedel; et array fungerer på ethvert nettverk med den bildefrekvensen nettverket tillater (en treg forbindelse senker bildefrekvensen i stedet for å avvise arrayet eller redusere bildefrekvensen); et array der kameraene har blitt slått av og på, gjenoppretter seg selv, med PTP på igjen; automatisk eksponering senker ikke lenger frekvensen i svakt lys (den fyller perioden, og øker deretter forsterkningen); Kontinuerlig opptak holder tritt med raske matriser; en**Highlights**-kontroll for RGB
-kameraer; og SDK
-funksjonene `ArraySession`, `grab()`, `frame_stream()` og `continuous()`. Se [LATTICE-delen](lattice/README.md).
* **Stabilere automatisk eksponering.** Eksponeringen holdes konstant ved jevnt lys og stabiliserer seg innen få sekunder etter en endring; små høylys gjør ikke lenger scenen mørkere; ved manuell eksponering forblir forsterkningen på «Auto», og et kamera som slutter å reagere i sterkt sollys holdes på det nivået som passer for lyset, målt av lyssensoren eller, hvis ingen er tilkoblet, av kameraets egne bilder. Se [Kamerainnstillinger](lattice/camera-settings.md).
* **Lyssensorer.** **DAQ-E-S-støtte:** den faste diffusoren krever ingen dekselkorreksjon, og hver avlesning inneholder enhetens tilt, roll og pitch. En opptak angir om dekselet ble valgt eller antatt, og behandlingen gir en advarsel ved et antatt deksel: DAQ-U, DAQ-M og DAQ-E leveres uten deksel, så oppgi navnet på det som er montert. Live `.daq`-opptak overlever et strømbrudd (behold en `.daq-journal`-fil sammen med dens `.daq`), og en DAQ-E som slutter å levere data, kobles til igjen med gapet markert i opptaket. Et prosjekt som kun inneholder opptak fra lyssensorer, viser en liste over disse og behandler dem fra appen og CLI
-en, og et opptak laget med `chloros_scripts` vises som den DAQ-en som laget det, med en begrensning du kan angi. Hvert opptak lagrer DAQ-avlesningen som ble brukt. Se [DAQ-delen](daq/README.md).
* **Behandling.**LATTICE-rammer starter behandlingen under måldeteksjon. Mål- og DAQ-referanser bestemmes per bånd. Et flyttet prosjekt kobler filene sine på nytt. Eksportene fra en 8-bits opptak er 8-bits, med et nytt**TIFF
-format (8-bits)**. En Survey3
-kjøring uten kalibreringsmål lagres som et reserveprodukt (`Vignette_Corrected_Images` eller `Sensor_Response_Images`), ikke som reflektans. DLS-taggene per bånd for et reflektansbilde tilhører alltid det aktuelle bildet. «Texture Aware» avvises for LATTICE-opptak, som alltid får standard debayer.
* **CLI
og SDK
.** Relative utdatastier havner nå i mappen der du kjører kommandoen eller skriptet fra (en installert backend pleide å løse dem i sin egen installasjonsmappe og mislyktes). Backenden fortsetter å kjøre mellom kommandoer og skript og stopper seg selv etter 30 minutter uten bruk; `chloros-cli backend status` og `backend stop` håndterer dette. Innlesingskommandoer oppgir navnene på alle filene de ikke klarte å skrive til, og avsluttes med kode 1, og gjentatte innlesinger stopper når harddisken er full. En vellykket kjøring av `process` rapporterer `Image products written: N (plus M clamp masks)`. På Windows
finner SDK
s `ChlorosLocal` / `process_folder` finner den installerte backenden (og bruker en som allerede kjører), og `read_image_audit_tags` fungerer fra en vanlig `pip install`.

***

## Hva er nytt i Chloros
1.2.0

* **Live kamerakontroll — ny «Kameraer»-fane.** Koble til LATTICE-kameraer ett om gangen eller som synkroniserte flerkamerasystemer (PTP-tidssynkronisering, maskinvareutløst opptak), med live forhåndsvisningsoverlegg, histogrammer per bånd, smart automatisk eksponering, en live indekskalkulator og oppdateringer av kameraets fastvare direkte i appen.
* **Lyssensorer — ny «Lyssensorer»-fane.** Koble til DAQ-U (USB), DAQ-M (Bluetooth) og DAQ-E (Ethernet)-sensorer; se kalibrerte spektra i sanntid (W/m²/nm), lagre `.daq`-filer i prosjektet ditt, velg profiler for kap-korreksjon og oppdater DAQ-E-firmware via nettverket.
* **Opptaksmoduser og opptakere.** Enkelt-/kontinuerlig-/intervallopptak samt en «Fastest Capture»-modus kun for rådata; valg per prosjekt av hvilke kameraer og eksporttyper «Capture All» skal produsere; array-opptakere for indeksvideo av overvåkingskvalitet og rådata-serier av analysekvalitet med offline videokonstruksjoner.
* **LATTICE-behandlingsrørledning.** Importer LATTICE-opptaksmapper og del hvert råbilde ut i debayered-, forhåndsvisnings-, float32-radiance- (W/m²/sr/nm) og reflektansprodukter med brytere per produkt. Refleksjonsverdien kan hentes fra et kalibreringsmål i bildet eller fra nedadgående lys fra DAQ; array-justering brukes på eksportene; manglende fabrikkalibrering lastes ned automatisk basert på kameraets serienummer.
* **Prosjekter husker maskinvare.** Tilkoblede kameraer og lyssensorer lagres sammen med prosjektet (`cameras.json` / `sensors.json`) og kobles til igjen med de lagrede innstillingene når du åpner prosjektet på nytt. Se [GUI: Prosjekter](projects.md).
* **Oppgraderinger av bildeviseren.** Avlesning av markørpiksler/indeks med korrekt skalering av reflektans per fil, laghistogrammer, en GSD-binning-glidebryter, rutenettmoduser «Per utløser» / «Per kamera», LATTICE-produktvisninger og eksport av indeks/LUT-sandkasse til disk.
* **CLI
og SDK
, betydelig utvidet.** Nye kommandofamilier `lattice`, `daq pool-*`, `project` og `time-sync`; nye `process`-alternativer (`--input-level`, produktvise brytere, `--reflectance-source`, flagg for matrisejustering); SDK
-smart-connect-håndtak (`connect_camera` / `connect_array` / `connect_daq_sensor`) som starter backenden automatisk; `open_project()`-automatisering; SDK
-hjulet er inkludert i installasjonsprogrammene og publisert på PyPI som `chloros-sdk`.
* **Tydelig feilmelding.** En `chloros-cli process`-kjøring som ba om produkter, men ikke skrev noen, mislykkes nå tydelig og avsluttes med en verdi ulik null; vellykkede kjøringer rapporterer hvor mange bildeprodukter de skrev.
* **Nytt utdataoppsett.** Produktene plasseres i `<project>/<camera>/<format>/<Product>_Images/`-mapper og beholder kildefilnavnet — mappen, ikke et filnavnsuffiks, identifiserer produktet. Se [Utdata-bildeformater](output-image-formats.md).
* **Flere innganger, planer og språk.** Støtte for `.dng`-innganger; alle 38 grensesnittspråk er fullstendig tilgjengelige; maskinvarebegrensninger per plan med gratis (uten pålogging) bruk av opptil 4 kameraer og 2 lyssensorer.
* **Pålitelighet.** «Stopp behandling» avsluttes på en ryddig måte med en ærlig kjøresammendrag, prosjekter med flere kameraer eksporterer hvert kamera, og installasjonsprogrammet logger deg ikke lenger ut ved oppgraderinger.***

Chloros
er tilgjengelig i tre brukergrensesnitt:

## Chloros
: GUI-applikasjon for skrivebordet

Frittstående, separat vindu med alle funksjoner, inkludert fanene for live-kameraer og lyssensorer. _Kun Windows._

## [Chloros
CLI
: Kommandolinjegrensesnitt](CLI.md)

Batchbehandling via kommandolinjen samt live-kommandoene `lattice`, `daq pool-*`, `project` og `time-sync`. Perfekt for automatisering, skripting og headless-drift. Tilgjengelig på **Windows
, Linux
amd64 og Linux
arm64 (NVIDIA Jetson)**. _CLI
krever et betalt Chloros
+-abonnement for å få tilgang._

## [Chloros
API
: Python
SDK
](api-python-sdk.md)

Programmatisk Python
-grensesnitt for automatisering og tilpassede arbeidsflyter: fullstendig rørledningsbehandling, live kamera-/matrisesesjoner, DAQ-sensorsesjoner og automatisering av lagrede prosjekter. Installeres sammen med desktop-/CLI-pakken og er også publisert som `pip install chloros-sdk`. _API krever et betalt Chloros
+-abonnement for å få tilgang._

***

## Støttede plattformer

| Plattform | GUI | CLI
| Python
SDK
|
| --- | --- | --- | --- |
| **Windows
10/11 (x64)** | Ja | Ja | Ja |
| **Linux
amd64 (x86_64)** | Nei | Ja | Ja |
| **Linux
arm64 (NVIDIA Jetson)** | Nei | Ja | Ja |

For installasjonsinstruksjoner for Linux
, se avsnittet [Linux
og Edge Computing](linux/linux-overview.md).

***

## Kom i gang i tre trinn

1. **Installer** — last ned og kjør installasjonsprogrammet for din plattform. Se [Nedlasting](download.md).
2. **Logg inn (valgfritt for GUI)** — GUI-en behandler bilder gratis uten konto. En [Chloros
+-pålogging](chloros+-login.md) gir tilgang til parallellbehandling, GPU-akselerasjon, høyere enhetsgrenser og tilgang til CLI
og SDK
.
3. **Opprett ditt første prosjekt** — åpChloros
, opprett et [nytt prosjekt](projects.md), [legg til bildene dine](processing-images-gui/adding-files-to-a-project.md) og [start behandlingen](processing-images-gui/starting-the-processing.md). Hvis du i stedet vil styre live-maskinvare, åpner du fanen «Kameraer» eller «Lyssensorer» — se [GUI: Navigasjon](navigation.md).

***

## Chloros


Selv om Chloros
er gratis å bruke til de fleste oppgaver, kan det hende du ønsker mer. Da kan en betalt lisens for Chloros
+ være til nytte for deg. Med en Chloros
+-lisens kan du få tilgang til nye funksjoner som:

* **Flertrådet behandling**: øk hastigheten på bildebehandlingen betydelig for større prosjekter ved å behandle bilder samtidig gjennom behandlingskjeden.
* **GPU (CUDA)-akselerasjon**: dra nytte av dagens større GPU-minnealternativer for å øke hastigheten på bildebehandlingskjeden ytterligere. Vi anbefaler 4 GB eller mer VRAM for best resultat.
* **Chloros
+**[**CLI**](CLI.md) **Tilgang**: kjør Chloros
+ fra kommandolinjen for å automatisere og integrere det i din egen programvare. Tilgjengelig på alle betalte nivåer; håndheves på serversiden.
* **Chloros
+**[**API**](api-python-sdk.md) **Tilgang:** Kjør Chloros
+ fra Python
for programmatisk kontroll, noe som muliggjør sømløs integrasjon med dine forskningsprosesser, arbeidsflyter for dataanalyse og tilpassede applikasjoner. Tilgjengelig på alle betalte abonnementer; håndheves på serversiden.
* **Høyere maskinvaregrenser**: koble til flere kameraer og lyssensorer samtidig. Uten pålogging kobler brukergrensesnittet til opptil 4 kameraer og 2 DAQ-lyssensorer; betalte abonnementer øker begge grensene:

| Abonnement | Kameraer | DAQ-lyssensorer |
| --- | --- | --- |
| Iron (gratis, uten pålogging) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Bruk på flere enheter**: hver Chloros
+-lisens tillater registrering av 2 eller flere enheter. Bruk din MAPIR
Cloud-konto til å administrere registrerte enheter. Få støtte for flere enheter ved å oppgradere din Chloros
+-lisens.
* **Avansert teksturbevisst debayer-metode:** en høykvalitets kantbevisst debayer kombinert med en AI/ML-støyreduksjonsmodell som fjerner nesten all debayer-støy.
* **Tilpassede multispektrale indeksformler:** Legg inn tilpassede multispektrale indekser i Chloros
-rasterkalkulatorene, både for behandling og i sandkassen for bildevisning.
* **Linux
og edge-databehandling:** kjør Chloros
på Linux
x86_64- og ARM64-plattformer, inkludert NVIDIA Jetson, for felt- og edge-behandling. Se [Linux
-oversikt](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Priser og registrering for Chloros</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.3.0 build so the banner prints "Chloros CLI 1.3.0". -->
