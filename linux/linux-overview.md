# Oversikt over Linux

Chloros 1.3.0 tilbyr innebygd Linux-støtte for **CLI**og**Python SDK** – multispektral bildebehandling uten skjerm, samt styring av LATTICE-kameraer og DAQ-lyssensorer i sanntid – på Linux-arbeidsstasjoner, servere og NVIDIA Jetson-kant-enheter.

{% hint style="info" %}
**Ingen GUI for skrivebordet på Linux.**Chloros Desktop GUI er kun tilgjengelig for Windows. Linux-brukere kommuniserer med Chloros via [CLI](../CLI.md) og [Python SDK](../api-python-sdk.md). `.deb` legger til en**Chloros CLI**-post i applikasjonsmenyen din — den åpner ganske enkelt en terminalemulator som kjører `chloros-cli`.
{% endhint %}

***

## Plattformstøttematrise

| Funksjon | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **GUI for skrivebord** | Ja | Ikke aktuelt | Nei | Nei |
| **CLI** (`chloros-cli`) | Ja | Ja | Ja | Ja |
| **Python SDK** (`chloros-sdk`) | Ja | Ja | Ja | Ja |
| **Bildbehandlingsrørledning** | Ja | Ja | Ja | Ja |
| **LATTICE-kamerakontroll (live)** | Ja (fanen «Kameraer») | Ja (`chloros-cli lattice`, SDK) | Ja | Ja |
| **DAQ-lyssensorer (live)** | Ja (fanen «Lyssensorer») | Ja (`chloros-cli daq pool-*`, SDK) | Ja | Ja |
| **PTP-tidssynkronisering (vert er grandmaster)** | Ja | Ja (`chloros-cli time-sync`) | Ja | Ja |
| **GPU-akselerasjon (CUDA)** | Ja | Ja | Ja | Ja (JetPack 6) |
| **Teksturbevisst debayer** (krever en CUDA-GPU) | Ja (Chloros+) | Ja (Chloros+) | Ja (Chloros+) | Ja (Chloros+) |
| **Dynamisk beregnings tilpasning** | Ja | Ja | Ja | Ja |
| **Backend som systemtjeneste** (`chloros-backend@<user>.service`) | Nei | Nei | Ja (valgfritt) | Ja (valgfritt) |
| **Oppdatering på stedet** (`chloros-cli update`) | Nei (kjør installasjonsprogrammet) | Nei (kjør installasjonsprogrammet) | Ja | Ja |

***

## Støttede arkitekturer

| Arkitektur | Beskrivelse | Pakke |
| --- | --- | --- |
| **amd64 (x86_64)** | Standard prosessorer for stasjonære datamaskiner/servere (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | ARM-prosessorer — NVIDIA Jetson Orin-familien (JetPack 6) | `chloros_<version>_arm64_jp6.deb` |

## Støttede Linux-distribusjoner

* **Ubuntu 22.04 LTS eller nyere** (amd64)
* **Debian 12 eller nyere** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson Orin-plattformer)

***

## Hva Linux-brukere får

* **Chloros CLI** — det fullstendige kommandolinjegrensesnittet for batchbehandling, automatisering og skripting
* **Chloros Python SDK** — programmatisk Python-grensesnitt for forskningspipelines og tilpassede verktøy, installert av `.deb` for systemet `python3` (et versjonsmatchet wheel for virtualenvs finnes i `/usr/lib/chloros/sdk/`; PyPI er for maskiner uten et Chloros-pakke)
* **LATTICE-kamerakontroll** — oppdag, koble til, konfigurer og ta opp fra LATTICE-kameraer og synkroniserte flerkamerasystemer via `chloros-cli lattice` og SDK; `.deb` inneholder Arena SDK-kjøringsmiljøet som kameraene krever
* **DAQ-lyssensorkontroll** — koble til DAQ-U/M/E-sensorer, strøm kalibrerte spektra og ta opp `.daq`-filer via `chloros-cli daq pool-*` og SDK
* **PTP-tidssynkronisering** — Chloros-backenden kjører PTP-grandmasteren som LATTICE-kameraer og DAQ-E-sensorer er slaver til; kontroller den med `chloros-cli time-sync`, og hold den i gang uten brukergrensesnitt med systemd-enheten `chloros-backend@<user>.service` (se [Linux-installasjon](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Prosjektautomatisering** — kjør lagrede prosjekter uten brukergrensesnitt med `chloros-cli project` og SDK-ets `open_project`
* **GPU-akselerasjon** — CUDA-akselerert behandling på NVIDIA-GPU-er (stasjonære GPU-er på amd64 og Jetsons GPU med JetPack 6)
* **Dynamisk beregnings tilpasning** — automatisk maskinvaredeteksjon og valg av behandlingsstrategi, med `CHLOROS_STRATEGY`-overstyring som en «ekspert-utvei»
* **Alle behandlingsfunksjoner** — samme prosess som i Windows: kalibrering, vignettkorrigering, vegetasjonsindekser og alle eksportformater
* **Chloros+-funksjoner** — flertrådet (pipelinert) behandling, Texture Aware-debayer (på en CUDA-GPU) og tilpassede indekser, med et betalt Chloros+-abonnement

## Hva Linux-brukere ikke får

* **GUI for skrivebordet** — ingen grafisk brukergrensesnitt; all interaksjon skjer via CLI eller Python-SDK
* **Bildeviser** — ingen interaktiv bildeviser, rutenettvisning eller kartmarkører
* **Visuell prosjektstyring** — prosjekter opprettes og styres via CLI-kommandoer og SDK-kall (selve maskinvaren — kameraer, sensorer, opptak — kan fortsatt styres fullt ut fra terminalen)

***

## Lisenskrav

Tilgang til CLI og SDK krever et **betalt Chloros+-abonnement — Copper eller høyere**(Copper, Bronze, Silver, Gold). Det gratis**Iron**-abonnementet har ingen tilgang til CLI/SDK. Begrensningen håndheves av backenden, ikke bare av CLI:

| Situasjon | Svar fra backenden |
| --- | --- |
| Ikke logget inn | `401` med `error_code: AUTH_REQUIRED` |
| Logget inn på det gratis Iron-nivået | `403` med `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` fungerer på alle nivåer — det er den eneste ruten som er unntatt fra porten — så årsaken til et avslag er alltid synlig.

***

## Kom i gang på Linux

1. **Installer Chloros** — se [Linux-installasjon](linux-installation.md) for installasjonen av `.deb`
2. **Kontroller** — `chloros-cli --version` skriver ut `Chloros CLI 1.3.0`; `chloros-cli selftest` kjører den 7-trinns diagnosen
3. **Sjekk Python-SDK** — `.deb` installerer den: `python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"` (i en virtualenv, `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`)
4. **Logg inn** — `chloros-cli login your@email.com 'your-password'` (én gang per maskin; pakkeoppgraderinger beholder påloggingen, mens fjerning av pakken sletter den)
5. **Behandle ditt første datasett** — `chloros-cli process ~/datasets/flight001`

For NVIDIA Jetson, se den dedikerte [NVIDIA Jetson-veiledningen](nvidia-jetson-guide.md) for plattformspesifikk oppsett, termisk oppførsel og feltutrulling.

***

## Neste trinn

* [Installasjon på Linux](linux-installation.md) — detaljert installasjon, filplasseringer og feilsøking for amd64 og arm64
* [NVIDIA Jetson-veiledning](nvidia-jetson-guide.md) — Jetson-spesifikk konfigurasjon, minne- og termisk oppførsel, feltutrulling
* [CLI: Kommandolinje](../CLI.md) — CLI-veiledningen
* [API: Python SDK](../api-python-sdk.md) — SDK-veiledningen
* [CLI-referanse](../reference/cli-reference.md) og [SDK-referanse](../reference/sdk-reference.md) — uttømmende oversikt over kommandoer og API for versjon 1.3.0
* [Dynamisk databehandlings tilpasning](../processing-architecture/dynamic-compute-adaptation.md) — hvordan Chloros tilpasser seg maskinvaren din

{% hint style="info" %}
**Lesing av denne håndboken programmatisk.** Hver side leveres også som rå Markdown på sin egen URL pluss `.md` (for eksempel `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), og en indeks over hele håndboken er publisert på [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
