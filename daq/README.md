# DAQ-lyssensorer

> **Leter du etter maskinvaren?**Selve sensorene – modeller, montering, deksler, porter, strømforsyning og SCANNER-appen – er beskrevet i**[DAQ-brukerhåndboken](https://mapir.gitbook.io/daq)**. Dette kapittelet omhandler bruk av dem i Chloros.

MAPIRs **DAQ**-lyssensorer måler omgivelseslyset som radiometrisk kalibrerte spektra. I Chloros har de to funksjoner:

* **Et frittstående spektralinstrument** – live spektrumdiagrammer, kolorimetriske data og `.daq`-opptak, alt fra [fanen Lyssensorer](gui.md), [CLI](cli-quick-start.md) eller Python-SDK.
* **En kilde for nedadrettet strålingsintensitet for reflektans** — under behandlingen interpolerer Chloros dine `.daq`-målinger til eksponeringstidspunktet for hvert bilde og bruker det målte nedadrettede lyset til å konvertere kameraets radianse til reflektans (`--reflectance-source daq`); det kreves ikke noe «in-scene»-panel for kalibrerte bånd.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Tre modeller, ett dataformat

| Modell | Overføring | Oppdagelse |
| --- | --- | --- |
| **DAQ-U** | USB (seriell) | skanning via seriell port |
| **DAQ-M** | Bluetooth Low Energy | BLE-skanning etter navn |
| **DAQ-E** | Ethernet (IPv4, PoE-drevet) | mDNS `_daq-e._tcp` (vertsnavn `daq-e-<id>.local`) |

**DAQ-E-S (nytt i 1.3.0).** En DAQ-E med fast diffusor og en vippesensor. Chloros oppfører den som `DAQ-E-S` og kjører den nøyaktig som en DAQ-E, bortsett fra at den ikke tar hensyn til kap-korreksjon og at hver avlesning inneholder enhetens vippe-, rulle- og stigningsvinkel. Se [DAQ-E-S: fast diffusor og tilt](caps-and-range.md#daq-e-s-fixed-diffuser-and-tilt).

Alle tre bruker samme kommunikasjonsprotokoll og leverer identiske data:

* Et **135-punkts spektrum fra 340 til 1010 nm i trinn på 5 nm**, pluss CIE XYZ-tristimulusverdier, i hvert bilde.
* **Radiometrisk kalibrert spektral irradians i W/m²/nm** — hver enhets fabrikkalibreringspakke (pluss den aktive kap-korreksjonsprofilen) blir brukt før dataene når deg.
* Det samme **`.daq`-opptaksformatet** (en SQLite-fil). Viderebehandlingen er identisk uavhengig av hvilken overføringsmetode som produserte filen.

Transportstakkene (USB-seriell, BLE, mDNS/zeroconf) er integrert i Chloros-backenden — det er ingenting å installere for å kommunisere med noen av de tre modellene fra GUI eller CLI-kommandoene `pool-*`.

***

## Kalibrert område: 340–1010 nm rapportert, ~374–974 nm kalibrert

Sensoren rapporterer hele rutenettet på 340–1010 nm, men den NIST-sporbare radiometriske forsterkningen strekker seg over omtrent **374–974 nm**. Chloros-programmet avviser inndelingen etter absolutt reflektans for alle kamerabånd som har mindre enn halvparten av sin spektrale vekt innenfor det kalibrerte området; det utelatte båndet rapporteres med utelatelsesårsaken `dls-uncalibrated-band-<nm>`.

Blant de leverte LATTICE-filtermodellene er kun **F988** berørt:

Refleksjonen for F988 kalibreres ved hjelp av et refleksjonspanel i opptakssituasjonen: båndet ligger utenfor det kalibrerte området til DAQ-lyssensoren, så Chloros bruker den siste panelopptakelsen og beholder den mellom panelmålingene.

Hvis en F988-registrering behandles med kun DAQ-data tilgjengelig, avviser Chloros DAQ-basert reflektans for det aktuelle båndet med utelatelsesårsaken `dls-uncalibrated-band-988` — [arbeidsflyten for reflektanspaneler](../calibration-targets.md) er den støttede fremgangsmåten for F988.

***

## Sensor-ID-er

Hver DAQ rapporterer en stabil sensor-ID. Formen varierer etter modell:

| Modell | ID-form | Eksempel |
| --- | --- | --- |
| DAQ-U | 5-oktett med bindestrek | `CB-7C-A8-2E-5F` |
| DAQ-M | 5 oktetter med bindestrek | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

Sensor-ID-en er:

* innstemplet i hver `.daq`-fil den registrerer,
* nøkkelen Chloros bruker for å hente enhetens fabrikkalibreringspakke,
* verdien du sender til `--sensor-id` i CLI-kommandoene for `pool-*`, og
* for DAQ-E, også dens mDNS-vertsnavn (`daq-e-def330.local`) — verdien som `--eth-host` aksepterer.

***

## Fabrikkkalibrering og skyen

Hver DAQ-enhet er individuelt fabrikkalibrert med en radiometrisk kjede som kan spores tilbake til NIST, og Chloros laster inn hver enhets kalibreringspakke basert på sensor-ID-en. Kalibreringsrapporten for hver enkelt enhet (PDF) kan lastes ned fra sensorens innstillinger under [fanen Lyssensorer](gui.md).

{% hint style="warning" %}
**DAQ-U og DAQ-M krever tilgang til skyen for kalibrering.**Ingen av modellene lagrer noe lokalt: fabrikkalibreringspakkene deres ligger i MAPIRs sky og hentes ut ved hjelp av sensor-ID (og lagres deretter i lokal cache). Chloros trenger en internettforbindelse for å levere kalibrerte W/m²/nm-data fra en DAQ-U eller DAQ-M.**DAQ-E er unntaket** — den har kalibreringen lagret på selve enheten.
{% endhint %}
<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

***

## Hvor opptakene lagres

| Overflate | Standard `.daq`-destinasjon |
| --- | --- |
| GUI — fanen «Lyssensorer» | `<project folder>/light_sensor/` (ferdige opptak legges automatisk til i prosjektet) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` på maskinen som kjører backend (en relativ `-o`-bane løses mot mappen du kjørte kommandoen i når backend-en er på samme datamaskin) |

Hvert `.daq`-filnavn inneholder sensor-ID-en og et tidsstempel.

***

## I dette kapittelet

* [**DAQ-fanen i Chloros**](gui.md) — fullstendig gjennomgang av brukergrensesnittet: tilkobling av hver modell, innstillinger per sensor, spektrumdiagrammer, fargemålingsdata i sanntid, reflektans fra to sensorer og opptak.
* [**Hurtigstart for CLI (pool-\*)**](cli-quick-start.md) — styring av DAQ-sensorer fra `chloros-cli daq pool-*`, den støttede kommandolinjeveien.
* [**Cap-profiler og kalibrert spektralområde**](caps-and-range.md) — hvilke cap-profiler som finnes for hver modell, hvordan man definerer dem, samt det kalibrerte spektralområdet i detalj.
* [**Opptak og .daq-formatet**](recording.md) — `.daq` SQLite-formatet og arbeidsflyter for opptak.
* [**DAQ-E-nettverk og tidssynkronisering**](ethernet-ptp.md) — DAQ-E-transportmoduser og PTP-tidssynkronisering.
* [**Arbeidsflyter for reflektans**](reflectance.md) — bruk av nedadrettede DAQ-data til å beregne reflektans.
* For fullstendig dokumentasjon på flagnivå, se [CLI-referansen](../reference/cli-reference.md) (avsnitt `chloros-cli daq`) og [SDK-referansen](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), begge skrevet for å kunne brukes direkte av AI-assistenter.
