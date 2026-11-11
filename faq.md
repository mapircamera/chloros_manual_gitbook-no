---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Vanlige spørsmål

<details>

<summary>Kan jeg behandle bilder fra kameraer som ikke er av merket MAPIR med Chloros?</summary>

Nei, Chloros støtter kun behandling av bilder fra MAPIR-kameraer — Survey3- og LATTICE-seriene. Se listen over [støttede kameramodeller](supported-cameras.md) for mer informasjon. Vi tilbyr behandling av bilder fra andre kameraer på MAPIR Cloud; se fullstendig liste [her](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Støtter Chloros LATTICE-kameraer?</summary>

Ja. Chloros 1.3.0 støtter LATTICE M3C- og M3M-kameramoduler fra start til slutt: **live-kontroll**— oppdag, koble til, forhåndsvis og ta opp fra GUI-fanen «Kameraer», `chloros-cli lattice`, eller Python-SDK, inkludert synkroniserte flerkameraoppsett med PTP-tidssynkronisering — og**full radiometrisk behandling** av opptakene (rådata → debayering → strålingsintensitet → refleksjonsgrad → indeks). Se [Støttede kameraer](supported-cameras.md) og [LATTICE-veiledningen](lattice/README.md).

</details>

<details>

<summary>Kan jeg kalibrere bildene mine for reflektans uten et kalibreringsmål?</summary>

**Survey3

:** Nei. Uten et bilde av kalibreringsmålet som er tatt samtidig med bildene uten målet, vil du ikke kunne knytte bildets pikselverdier til en kjent refleksjonsprosent. Hvis du heller ikke inkluderer loggen fra en MAPIR

-lyssensor, vil ikke omgivelseslyspektrumet bli målt, og refleksjonsresultatene vil ikke være nøyaktige.

**LATTICE:** Ja. Refleksjonen kan refereres til den nedadgående strålingsintensiteten målt av en DAQ-lyssensor i stedet for et panel (ρ = π·L/E). Når et QA-godkjent mål innenfor rammen *er* til stede, blir det som standard den absolutte referansen (`--reflectance-source auto`), med mindre det godkjennes med et flagg, da målet og DAQ-kjedene konkurrerer om den målte poengsummen. Ett unntak: «F988-refleksjonen kalibreres ved hjelp av et refleksjonspanel i scenen: båndet ligger utenfor DAQ-lyssensorens kalibrerte område, så Chloros

bruker din siste panelopptak og beholder den mellom panelobservasjonene.» Se [Kalibreringsmål](calibration-targets.md).

</details>

<details>

<summary>Trenger jeg en DAQ-lyssensor?</summary>

Ikke for strålingsintensitet: LATTICE-eksport av strålingsintensitet stammer fra hvert kameras fabrikkkalibrering og krever verken en DAQ-sensor eller et mål. For **refleksjonsgrad**trenger du en referanse for omgivelseslyset – enten en nedadrettet måling fra en DAQ-lyssensor eller et kalibreringsmål i bildet. En DAQ-sensor lar deg produsere kalibrert reflektans**uten å plassere noen paneler i scenen**. Innspilte `.daq`-filer blir automatisk matchet med bildene dine ved hjelp av tidsstempel. Se [Kalibreringsmål](calibration-targets.md) og [CLI

-referansen](reference/cli-reference.md).

</details>

<details>

<summary>Kan jeg bruke Chloros med en AI-assistent (Claude, ChatGPT osv.)?</summary>

Ja — denne håndboken og CLI/SDK er laget for dette:

* Den fullstendige håndbokindeksen er tilgjengelig på `https://mapir.gitbook.io/chloros/llms.txt`, slik at AI-assistenter kan finne alle sidene.
* Hver sides rå Markdown-kode er tilgjengelig på sidens URL med små bokstaver, med `.md` lagt til (for eksempel `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI-referansen](reference/cli-reference.md) og [SDK-referansen](reference/sdk-reference.md) er skrevet for bruk av store språkmodeller (LLM): nøyaktige flagg, standardverdier, avslutningssemantikk og kommandoer som kan kopieres og limes inn.

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
* **produktmapper** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (alltid under `tiff32`), `<INDEX>_Index_Images/`
* **`clamp_masks/`** — kun LATTICE multispektral: masker per piksel ved siden av produktmappene, se nedenfor**Eksporterte filer beholder kildfilens navn — mappen identifiserer produktet, ikke et filnavnsuffiks.**Med CLI opprettes prosjektmappen under standardprosjektmappen din (`Documents/MAPIR/Chloros Projects` med mindre du har endret den), med mindre du angir `-o`; en relativ `-o` tolkes relativt til mappen du kjører kommandoen fra. Merk at en `chloros-cli process`-kjøring som ba om produkter, men ikke skrev noen, skriver ut `Processing finished but wrote no image products.` og**avsluttes med en verdi ulik null**, slik at skript kan oppdage det. Se [Utdataformater](output-image-formats.md) og [CLI-referansen](reference/cli-reference.md).

</details>

<details>

<summary>Hva er clamp_masks-mappene og *_clampmask.tif-filene?</summary>

Nytt i versjon 1.3.0. Radianse- og reflektansproduktene fra et multispektralt LATTICE M3C-kamera gjennomgår en 3×3-bånds separasjon, og der det oppstår en negativ verdi i et bånd, klipper Chlorosper denne verdien til null. Et produkt med minst én avklippet piksel får en enkeltbånds `uint8`-maske som markerer hvilke piksler og bånd som ble avklippet (bit 0 = produktets R-kanal, bit 1 = G, bit 2 = B). Masker lagres i en `clamp_masks/`-mappe **ved siden av** produktmappene — for eksempel `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif` — og produktets `Chloros:UnmixClampMask` XMP-tag angir navnet på masken (eller angir `none`).

Dette er ikke bilder av scenen: Chloros’ egen import hopper over dem, og siden de aldri ligger i en produktmappe, kan du overføre en produktmappe til Pix4D eller Metashape slik den er. Se [Clamp-masker](output-image-formats.md#clamp-masks).

</details>

<details>

<summary>Hvorfor sier Chloros: «Prosjektmappen er for dyp for denne datamaskinen»?</summary>

Dette gjelder kun Windows. Windows leveres med lange baner deaktivert, og da kan ingen filbane være lengre enn 260 tegn. Chloros-produkter ligger opptil ca. 130 tegn under prosjektmappen, så Chloros avviser en dypere prosjektmappe før behandlingen starter — i stedet for å kjøre og ikke skrive noe, slik det kunne skje før versjon 1.3.0. Meldingen angir den lengste prosjektmappen som passer.

Løs problemet på en av følgende måter:

* Bruk en kortere prosjektmappe (**Arbeidsmappe** i prosjektinnstillingene, eller `-o` / `chloros-cli set-project-folder` for CLI).
* Slå på lange stier i Windows og start Chloros på nytt. Alternativet **Aktiver lange stier i Windows** i Chloros-installasjonsprogrammet (merket av som standard) gjør dette; kjør installasjonsprogrammet på nytt hvis du har fjernet merket.

Se [Starte behandlingen](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

</details>

<details>

<summary>Kan jeg redigere bildene mine før behandling i Chloros?</summary>

Nei. Chloros forutsetter at inndataene ikke er endret. Ikke endre filnavnene.

</details>

<details>

<summary>Kan jeg stille inn MAPIR-Survey3-kameraene mine på automatisk eksponering og behandle bildene i Chloros?</summary>

Nei. Survey3-bildedatasett må ha en fast/låst eksponering, så ingen automatisk lukkerhastighet eller automatisk ISO. Alle bilder fra samme kameramodell må ha identisk lukkerhastighet og ISO (eksponering).

LATTICE-kameraer har ikke denne begrensningen: Chloros styrer eksponeringen i sanntid (Smart AE), og hver opptak registrerer den eksponeringen og forsterkningen som faktisk ble brukt, noe som den radiometriske behandlingskjeden tar hensyn til.

</details>

<details>

<summary>Kan Chlorose behandle eller analysere ortomosaikkbilder?</summary>

Nei. Kun enkeltbilder fra MAPIR-kameraer støttes, ikke sammensatte bilder som et ortomosaikk-kart.

</details>

<details>

<summary>Hvordan kan jeg få Chloros til å oppdage målene raskere?</summary>

Hvis du i filbrowsertabellen forhåndsvelger målbildene i høyre kolonne, vil Chloros kun lete etter kalibreringsmål i disse bildene, noe som gjør behandlingen betydelig raskere.

</details>

<details>

<summary>Hvis jeg skal laste opp bildene mine til <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> bør jeg da behandle dem i Chloros før opplastingen?</summary>

Hvis du planlegger å laste opp til vår online behandlingsplattform [MAPIR

Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), må du ikke redigere bildene før du laster dem opp. Cloud vil utføre all den samme behandlingen og mer til.

</details>

<details>

<summary>Vil MAPIR noen gang støtte X-funksjonen? Jeg skulle virkelig ønske MAPIR tilbød X.</summary>

Vi er alltid interessert i å motta tilbakemeldinger på produktene våre. Hvis du oppdager et problem med produktene våre, eller har et forslag til hvordan vi kan forbedre dem, kan du [KONTAKTE OSS](https://www.mapir.camera/community/contact) for å dele dine tanker. Det meste av vår FoU er styrt av å lytte til kundenes største behov.

</details>

<details>

<summary>Er Chloros tilgjengelig for Linux?</summary>

Ja! Chloros 1.3.0 støtter Linux amd64 (x86_64) og arm64 (NVIDIA Jetson JetPack 6) via `.deb`-pakker. CLI og Python SDK støttes fullt ut på Linux, inkludert live-kontroll av LATTICE-kameraer og DAQ-sensorer. Det finnes ingen grafisk brukergrensesnitt (GUI) for Linux – all interaksjon skjer via [CLI](CLI.md) eller [Python SDK](api-python-sdk.md). Se [Oversikt over Linux](linux/linux-overview.md) for mer informasjon.

</details>

<details>

<summary>Kan jeg kjøre Chloros på NVIDIA Jetson?</summary>

Ja! Chloros støtter NVIDIA Jetson **Orin-familien** – Orin Nano, Orin NX og AGX Orin – som kjører JetPack 6. Eldre modeller (Jetson Nano, TX2, Xavier NX) kan ikke kjøre JetPack 6 og støttes ikke av den nåværende pakken. Chloros oppdager automatisk Jetson-modellen din og optimaliserer behandlingsstrategien. Jetson-pakken bruker JetPack 6s CUDA-versjon av PyTorch, slik at prosesseringen kjører på Jetsons GPU. Se [NVIDIA Jetson-veiledningen](linux/nvidia-jetson-guide.md) for instruksjoner om oppsett og distribusjon.

</details>

<details>

<summary>Optimerer Chloros automatisk for maskinvaren min?</summary>

Ja! Chloros inkluderer [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) som automatisk gjenkjenner CPU, GPU, RAM og (på Jetson) temperatursensorer. Deretter velger den den optimale behandlingsstrategien — fra `GPU_PARALLEL` på GPU-er med mye minne til `GPU_SINGLE` på mindre GPU-er og til `CPU_PARALLEL` på systemer uten en CUDA-GPU som PyTorch kan bruke. Det er ikke behov for manuell konfigurasjon.

</details>

<details>

<summary>Hva er 4-trådsbehandlingsrørledningen?</summary>

Chloros
bruker en 4-tråds pipelined arkitektur for Chloros
+-brukere: Tråd 1 (deteksjon) laster inn bilder og oppdager kalibreringsmål, tråd 2 (kalibrering) beregner reflektanskalibrering, tråd 3 (behandling) utfører GPU-akselerert debayering og indeksberegning, og tråd 4 (eksport) skriver utdatafiler. Flere bilder kan behandles i forskjellige tråder samtidig for maksimal gjennomstrømning. Se [Behandlingsrørledning](processing-architecture/processing-pipeline.md) for detaljer.

</details>

<details>

<summary>Hvordan kjører jeg diagnostikk på min Chloros-installasjon?</summary>

Bruk kommandoen `selftest` for å kjøre en 7-trinns røykprøve: versjon, porttilgjengelighet, oppstart av backend, API-tilkobling (`/api/test`), systeminformasjon (`/api/system-info` — GPU/CUDA/PyTorch), tilstedeværelse av støydempingsmodell og klarhet for CUDA + støydemping:

```bash
chloros-cli selftest
```

Dette er spesielt nyttig på Linux-systemer for å verifisere GPU- og CUDA-oppsettet. På en Jetson betyr `CUDA: False` vanligvis at JetPack 6s CUDA- og cuDNN-biblioteker mangler: installer dem med `sudo apt install nvidia-jetpack`.

</details>
