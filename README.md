---
metaLinks: {}
---

# Kom i gang

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros er et program fra [MAPIR](https://www.mapir.camera) for å behandle multispektrale bilder, fjernstyre MAPIR-maskinvare og registrere sensordata. Chloros 1.2.0 støtter hele MAPIR-produktfamilien:

* **Survey3-kameraer** — behandler RAW+JPG-opptak til kalibrerte refleksjons- og vegetasjonsindekskart. Se [Støttede kameraer](supported-cameras.md).
* **LATTICE-kameraer** — koble til GigE-multispektrale kameramoduler i sanntid, enkeltvis eller som synkroniserte flerkamerasystemer: forhåndsvisning, opptak og bearbeiding til kalibrerte strålings- og refleksjonsprodukter. Se [LATTICE-delen](lattice/README.md).
* **DAQ-lyssensorer** — DAQ-U (USB), DAQ-M (Bluetooth) og DAQ-E (Ethernet) spektralsensorer: kalibrerte spektra i sanntid, `.daq`-opptak og nedadrettet belysning for reflektansbehandling. Se [DAQ-delen](daq/README.md).

{% hint style="success" %}
**Hva er nytt i Chloros 1.2.0**: kontroll av LATTICE-kameraer og -matriser i sanntid, integrering av DAQ-lyssensorer, opptaksmoduser og opptakere, en fullstendig radiometrisk behandlingsprosess for LATTICE, prosjektautomatisering fra CLI/SDK og mye mer. Se listen over nyheter nedenfor, og [Last ned](download.md) for endringsloggen.
{% endhint %}

{% hint style="info" %}
**Bruker du Chloros med en AI-assistent?** Denne håndboken er laget for det. Be assistenten din om:

* `https://mapir.gitbook.io/chloros/llms.txt` — maskinlesbar indeks over alle sider.
* Enhver side som rå Markdown — legg til `.md` til dens URL (f.eks. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI-referansen](reference/cli-reference.md) og [SDK-referansen](reference/sdk-reference.md) — komplette referansesider med eksakte verdier, skrevet for bruk i LLM-modeller.

Eksempel på prompt: *«Les https://mapir.gitbook.io/chloros/reference/cli-reference.md,, og skriv deretter et skript som logger inn og behandler mappen ~/flights/flight_001 til GeoTIFF-filer i formatet reflectance + NDVI.»*

Fullstendig veiledning: [Bruk av Chloros med AI-assistenter](ai-assistants.md).
{% endhint %}

***

## Hva er nytt i Chloros 1.2.0

* **Live kamerakontroll — ny «Kameraer»-fane.** Koble til LATTICE-kameraer ett om gangen eller som synkroniserte flerkameraoppsett (PTP-tidssynkronisering, maskinvareutløst opptak), med live-forhåndsvisningsoverlegg, histogrammer per bånd, smart automatisk eksponering, en live-indekskalkulator og firmwareoppdateringer for kameraet direkte i appen.
* **Lyssensorer — ny «Lyssensorer»-fane.** Koble til DAQ-U (USB), DAQ-M (Bluetooth) og DAQ-E (Ethernet)-sensorer; se kalibrerte spektra i sanntid (W/m²/nm), lagre `.daq`-filer i prosjektet ditt, velg profiler for kap-korreksjon og oppdater DAQ-E-firmware over nettverket.
* **Opptaksmoduser og opptakere.** Enkelt-/kontinuerlig-/intervallopptak samt en «Fastest Capture»-modus kun for rådata; valg per prosjekt av hvilke kameraer og eksporttyper «Capture All» skal produsere; array-opptakere for indeksvideo av overvåkingskvalitet og rådata-serier av analysekvalitet med offline videoproduksjon.
* **LATTICE-behandlingsrørledning.** Importer LATTICE-opptaksmapper og del hvert råbilde ut i debayerte bilder, forhåndsvisning, float32-strålingsintensitet (W/m²/sr/nm) og reflektansprodukter med brytere for hvert produkt. Refleksjonsverdien kan hentes fra et kalibreringsmål i bildet eller nedadrettet lys fra datainnsamlingssystemet (DAQ); arrayjustering brukes på eksportene; manglende fabrikkalibrering lastes ned automatisk basert på kameraets serienummer.
* **Prosjekter husker maskinvare.** Tilkoblede kameraer og lyssensorer lagres sammen med prosjektet (`cameras.json` / `sensors.json`) og kobles til igjen med de lagrede innstillingene når du åpner prosjektet på nytt. Se [GUI: Prosjekter](projects.md).
* **Oppgraderinger av bildeviseren.** Avlesning av markørpiksler/indeks med korrekt skalering av reflektans per fil, laghistogrammer, en GSD-binning-glidebryter, rutenettmoduser «Per utløser» / «Per kamera», LATTICE-produktvisninger og eksport av indeks/LUT-sandkasse til disk.
* **CLI og SDK, betydelig utvidet.** Nye kommandofamilier: `lattice`, `daq pool-*`, `project` og `time-sync`; nye `process`-alternativer (`--input-level`, produktvise brytere, `--reflectance-source`, flagg for matrisejustering); SDK smart-connect-håndtak (`connect_camera` / `connect_array` / `connect_daq_sensor`) som starter backenden automatisk; `open_project()`-automatisering; SDK-hjulet er inkludert i installasjonsprogrammene og publisert på PyPI som `chloros-sdk`.
* **Tydelig feilmelding.** En `chloros-cli process`-kjøring som ba om produkter, men ikke skrev noen, mislykkes nå med en tydelig feilmelding og avsluttes med en verdi ulik null; vellykkede kjøringer rapporterer hvor mange bildeprodukter de har skrevet.
* **Nytt utdatalayout.** Produktene plasseres i `<project>/<camera>/<format>/<Product>_Images/`-mapper og beholder kildfilnavnet – det er mappen, ikke et filnavnsuffiks, som identifiserer produktet. Se [Utdata-bildeformater](output-image-formats.md).
* **Flere innganger, planer og språk.** Støtte for `.dng`-innganger; alle 38 grensesnittspråk er fullt utfylt; maskinvarebegrensninger per plan med gratis (uten pålogging) bruk av opptil 4 kameraer og 2 lyssensorer.
* **Pålitelighet.** «Stopp behandling» avsluttes på en ryddig måte med en ærlig kjøresammendrag, prosjekter med flere kameraer eksporterer hvert kamera, og installasjonsoppgraderinger logger deg ikke lenger ut.***

Chloros er tilgjengelig i tre brukergrensesnitt:

## Chloros: GUI-applikasjon for skrivebordet

Frittstående, separat vindu med alle funksjoner, inkludert fanene «Live-kameraer» og «Lyssensorer». _Kun Windows._

## [Chloros CLI: Kommandolinjegrensesnitt](CLI.md)

Batchbehandling via kommandolinjen samt live-kommandoene `lattice`, `daq pool-*`, `project` og `time-sync`. Perfekt for automatisering, skripting og headless-drift. Tilgjengelig på **Windows, Linux amd64 og Linux arm64 (NVIDIA Jetson)**. _CLI krever et betalt Chloros+-abonnement for å få tilgang._

## [Chloros API: Python SDK](api-python-sdk.md)

Programmatisk Python-grensesnitt for automatisering og tilpassede arbeidsflyter: fullstendig rørledningsbehandling, live kamera-/matrisesesjoner, DAQ-sensorsesjoner og automatisering av lagrede prosjekter. Installeres med desktop-/CLI-pakken og publiseres også som `pip install chloros-sdk`. _API krever et betalt Chloros+-abonnement for å få tilgang._

***

## Støttede plattformer

| Plattform | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | Ja | Ja | Ja |
| **Linux amd64 (x86_64)** | Nei | Ja | Ja |
| **Linux arm64 (NVIDIA Jetson)** | Nei | Ja | Ja |

For installasjonsinstruksjoner for Linux, se avsnittet [Linux og Edge Computing](linux/linux-overview.md).

***

## Kom i gang i tre trinn

1. **Installer** — last ned og kjør installasjonsprogrammet for din plattform. Se [Last ned](download.md).
2. **Logg inn (valgfritt for GUI)** — GUI-en behandler bilder gratis uten konto. En [Chloros+-pålogging](chloros+-login.md) gir tilgang til parallellbehandling, GPU-akselerasjon, høyere enhetsgrenser og tilgang til CLI/SDK.
3. **Opprett ditt første prosjekt** — åpne Chloros, opprett et [Nytt prosjekt](projects.md), [legg til bildene dine](processing-images-gui/adding-files-to-a-project.md) og [start behandlingen](processing-images-gui/starting-the-processing.md). For å styre live-maskinvare i stedet, åpner du fanen «Kameraer» eller «Lyssensorer» — se [GUI: Navigering](navigation.md).***

## Chloros+

Selv om Chloros er gratis å bruke til de fleste oppgaver, kan det hende du ønsker mer. Da kan en betalt lisens for Chloros+ være til nytte for deg. Med en Chloros+-lisens kan du få tilgang til nye funksjoner som:

* **Flertrådet behandling**: øker hastigheten på bildebehandlingen betydelig for større prosjekter ved å behandle bilder samtidig gjennom behandlingskjeden.
* **GPU (CUDA)-akselerasjon**: dra nytte av dagens større GPU-minnealternativer for å øke hastigheten på bildebehandlingsprosessen ytterligere. Vi anbefaler 4 GB eller mer VRAM for best resultat.
* **Chloros+**[**CLI**](CLI.md)**Tilgang**: kjør Chloros+ fra kommandolinjen for å automatisere og integrere i din egen programvare. Tilgjengelig på alle betalte abonnementer; håndheves på serversiden.
* **Chloros+**[**API**](api-python-sdk.md)**Tilgang:** Kjør Chloros+ fra Python for programmatisk kontroll, noe som muliggjør sømløs integrasjon med dine forskningsprosesser, arbeidsflyter for dataanalyse og tilpassede applikasjoner. Tilgjengelig på alle betalte abonnementer; håndheves på serversiden.
* **Høyere maskinvaregrenser**: koble til flere kameraer og lyssensorer samtidig. Uten pålogging kobler brukergrensesnittet til opptil 4 kameraer og 2 DAQ-lyssensorer; betalte abonnementer øker begge grensene:

| Abonnement | Kameraer | DAQ-lyssensorer |
| --- | --- | --- |
| Iron (gratis, uten pålogging) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Bruk av flere enheter**: Hver Chloros+-lisens tillater registrering av 2 eller flere enheter. Bruk din MAPIR Cloud-konto til å administrere registrerte enheter. Legg til støtte for flere enheter ved å oppgradere din Chloros+-lisens.
* **Avansert teksturbevisst debayer-metode:** en høykvalitets kantbevisst debayer kombinert med en AI/ML-støyreduksjonsmodell som fjerner nesten all debayering-støy.
* **Tilpassede multispektrale indeksformler:** legg inn tilpassede multispektrale indekser i Chloros-rasterkalkulatorene, både for behandling og i sandkassen for bildevisning.
* **Linux og edge-databehandling:** kjør Chloros på Linux x86_64- og ARM64-plattformer, inkludert NVIDIA Jetson, for felt- og edge-behandling. Se [Linux Oversikt](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Priser og registrering</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.2.0 build so the banner prints "Chloros CLI 1.2.0". -->
