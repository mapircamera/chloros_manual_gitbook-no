---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Vanlige spørsmål

<details>

<summary>Kan jeg behandle bilder fra kameraer som ikke er av merket MAPIR med Chloros?</summary>

Nei, Chloros støtter kun behandling av bilder fra MAPIR-kameraer — Survey3- og LATTICE-seriene. Se listen over [støttede kameramodeller](supported-cameras.md) for mer informasjon. Vi tilbyr behandling av andre kameraer på MAPIR Cloud; se fullstendig liste [her](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Støtter Chloros LATTICE-kameraer?</summary>

Ja. Chloros 1.2.0 støtter LATTICE M3C- og M3M-kameramoduler fra start til slutt: **live-kontroll**— oppdag, koble til, forhåndsvis og ta opp fra fanen «Kameraer» i brukergrensesnittet, `chloros-cli lattice` eller Python SDK, inkludert synkroniserte flerkameraoppsett med PTP-tidssynkronisering — samt**full radiometrisk behandling** av opptak (rådata → debayering → strålingsintensitet → refleksjonsgrad → indeks). Se [Støttede kameraer](supported-cameras.md) og [LATTICE-veiledningen](lattice/README.md).

</details>

<details>

<summary>Kan jeg kalibrere bildene mine for reflektans uten et kalibreringsmål?</summary>

**Survey3:** Nei. Uten et bilde av kalibreringsmålet tatt omtrent samtidig som bildene uten målet tas, vil du ikke kunne knytte bildets pikselverdier til en kjent refleksjonsprosent. Hvis du heller ikke inkluderer loggen fra en MAPIR-lyssensor, vil ikke omgivelseslyspektrumet bli målt, og refleksjonsresultatene vil ikke være nøyaktige.**LATTICE:** Ja. Refleksjonen kan refereres til den nedadgående bestrålingsstyrken målt av en DAQ-lyssensor i stedet for et panel (ρ = π·L/E). Når et QA-godkjent mål *er* til stede i bildet, blir det som standard den absolutte referansen (`--reflectance-source auto`). Ett unntak: «F988-refleksjonen kalibreres ved hjelp av et refleksjonspanel i scenen: båndet ligger utenfor det kalibrerte området til DAQ-lyssensoren, så Chloros bruker din siste panelopptak og beholder den mellom panelobservasjonene.» Se [Kalibreringsmål](calibration-targets.md).

</details>

<details>

<summary>Trenger jeg en DAQ-lyssensor?</summary>

Ikke for strålingsintensitet: LATTICE-eksport av strålingsintensitet er basert på hvert kameras fabrikkkalibrering og krever verken en DAQ-sensor eller et mål. For **refleksjonsgrad**trenger du en referanse for omgivelseslyset – enten en nedadrettet måling fra en DAQ-lyssensor eller et kalibreringsmål i bildet. En DAQ-sensor lar deg produsere kalibrert reflektans**uten å plassere noen paneler i scenen**. Innspilte `.daq`-filer blir automatisk matchet med bildene dine ved hjelp av tidsstempel. Se [Kalibreringsmål](calibration-targets.md) og [CLI-referansen](reference/cli-reference.md).

</details>

<details>

<summary>Kan jeg bruke Chloros med en AI-assistent (Claude, ChatGPT osv.)?</summary>

Ja — denne håndboken og CLI/SDK er laget for dette:

* Den fullstendige håndbokindeksen er tilgjengelig på `https://mapir.gitbook.io/chloros/llms.txt`, slik at AI-assistenter kan finne alle sidene.
* Hver sides rå markdown-kode er tilgjengelig på siden med små bokstaver URL, med `.md` lagt til (for eksempel `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI-referansen](reference/cli-reference.md) og [SDK-referansen](reference/sdk-reference.md) er skrevet for bruk av LLM-er: nøyaktige flagg, standardinnstillinger, avslutningssemantikk og kommandoer som kan kopieres og limes inn.

Se [AI-assistenter](ai-assistants.md) for å finne ut hvordan du kan peke assistenten din mot Chloros.

</details>

<details>

<summary>Hvor havner de bearbeidede utdatafilene mine?</summary>

Produktene lagres i prosjektmappen, gruppert etter kamera og deretter etter filformat:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **kamera-mappe** — `LATT-<sensor>-<lens>-F<filter>` for LATTICE, `<model>_<filter>` (f.eks. `Survey3N_RGN`) for Survey3
* **formatmappe** — `tiff16`, `tiff8`, `png8`, `jpg8` eller `tiff32`
* **produktmapper** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (alltid under `tiff32`), `<INDEX>_Index_Images/`**Eksporterte filer beholder kildefilens navn — mappen identifiserer produktet, ikke et filnavnsuffiks.**Med CLI opprettes prosjektmappen ved siden av inndatamappen, med mindre du angir `-o`. Merk at en kjøring av `chloros-cli process` som ba om produkter, men ikke skrev noen, skriver ut `Processing finished but wrote no image products.` og**avsluttes med en verdi ulik null**, slik at skript kan oppdage det. Se [Utgangsbildformater](output-image-formats.md) og [CLI-referansen](reference/cli-reference.md).

</details>

<details>

<summary>Kan jeg redigere bildene mine før behandling i Chloros?</summary>

Nei. Chloros forutsetter at inndataene ikke er endret. Ikke endre filnavnene.

</details>

<details>

<summary>Kan jeg stille inn MAPIR- og Survey3-kameraene mine på automatisk eksponering og behandle bildene i Chloros?</summary>

Nei. Bildedatasettene til Survey3 må ha en fast/låst eksponering, så det er ikke mulig med automatisk lukkerhastighet eller automatisk ISO. Alle bilder fra samme kameramodell må ha identisk lukkerhastighet og ISO (eksponering).

LATTICE-kameraer har ikke denne begrensningen: Chloros styrer eksponeringen i sanntid (Smart AE), og hvert bilde registrerer den eksponeringen og forsterkningen som faktisk ble brukt, noe som den radiometriske behandlingskjeden tar hensyn til.

</details>

<details>

<summary>Kan Chloros behandle eller analysere ortomosaikkbilder?</summary>

Nei. Kun enkeltbilder fra MAPIR-kameraer støttes, ikke sammensatte bilder som et ortomosaikkart.

</details>

<details>

<summary>Hvordan kan jeg få fart på måldeteksjonstrinnet i Chloros?</summary>

Ved å forhåndsvelge målbildene i høyre kolonne i filbrowsertabellen vil Chloros kun lete etter kalibreringsmål i disse bildene, noe som vil øke behandlingshastigheten betydelig.

</details>

<details>

<summary>Hvis jeg skal laste opp bildene mine til <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> bør jeg da behandle dem i Chloros før opplasting?</summary>

Hvis du planlegger å laste opp til vår online behandlingsplattform [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), må du ikke redigere bildene før du laster dem opp. Cloud vil utføre all den samme behandlingen og mer til.

</details>

<details>

<summary>Vil MAPIR noen gang støtte X-funksjonen? Jeg skulle virkelig ønske at MAPIR tilbød X.</summary>

Vi er alltid interessert i å motta tilbakemeldinger på produktene våre. Hvis du oppdager et problem med produktene våre, eller har et forslag til hvordan vi kan forbedre dem, kan du [KONTAKTE OSS](https://www.mapir.camera/community/contact) for å dele dine tanker. Det meste av vår FoU er basert på å lytte til kundenes viktigste behov.

</details>

<details>

<summary>Er Chloros tilgjengelig for Linux?</summary>

Ja! Chloros 1.2.0 støtter Linux amd64 (x86_64) og arm64 (NVIDIA Jetson JetPack 6) via `.deb`-pakker. CLI og Python SDK støttes fullt ut på Linux, inkludert live-kontroll av LATTICE-kamera og DAQ-sensor. Det finnes ingen grafisk brukergrensesnitt for Linux — all interaksjon skjer via [CLI](CLI.md) eller [Python SDK](api-python-sdk.md). Se [Linux Oversikt](linux/linux-overview.md) for detaljer.

</details>

<details>

<summary>Kan jeg kjøre Chloros på NVIDIA Jetson?</summary>

Ja! Chloros støtter NVIDIA Jetson-plattformer, inkludert Jetson Nano, Orin Nano, Orin NX og AGX Orin som kjører JetPack 6. Chloros gjenkjenner automatisk Jetson-modellen din og optimaliserer behandlingsstrategien. Se [NVIDIA Jetson-veiledningen](linux/nvidia-jetson-guide.md) for instruksjoner om oppsett og distribusjon.

</details>

<details>

<summary>Optimaliserer Chloros automatisk for maskinvaren min?</summary>

Ja! Chloros inkluderer [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) som automatisk gjenkjenner CPU, GPU, RAM og (på Jetson) termiske sensorer. Deretter velger den den optimale behandlingsstrategien – fra `GPU_PARALLEL` på systemer med mye minne til `GPU_SINGLE` på enheter med begrensede ressurser og til `CPU_PARALLEL` på systemer uten en NVIDIA-GPU. Det er ikke behov for manuell konfigurasjon.

</details>

<details>

<summary>Hva er 4-trådsbehandlingsrørledningen?</summary>

Chloros bruker en 4-tråds pipelined-arkitektur for Chloros+-brukere: Tråd 1 (Deteksjon) laster inn bilder og oppdager kalibreringsmål, Tråd 2 (Kalibrering) beregner reflektanskalibrering, Tråd 3 (Behandling) utfører GPU-akselerert debayering og indeksberegning, og Tråd 4 (Eksport) skriver utdatafiler. Flere bilder kan behandles i forskjellige tråder samtidig for å oppnå maksimal gjennomstrømning. Se [Behandlingspipeline](processing-architecture/processing-pipeline.md) for detaljer.

</details>

<details>

<summary>Hvordan kjører jeg diagnostikk på min Chloros-installasjon?</summary>

Bruk kommandoen `selftest` for å kjøre en 7-trinns røykprøve: versjon, porttilgjengelighet, oppstart av backend, API-tilkobling (`/api/test`), systeminformasjon (`/api/system-info` – GPU/CUDA/PyTorch), tilstedeværelse av støyreduksjonsmodell og klarhet for CUDA + støyreduksjon:

```bash
chloros-cli selftest
```

Dette er spesielt nyttig på Linux/Jetson-systemer for å verifisere GPU- og CUDA-oppsettet.

</details>
