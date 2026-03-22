---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Vanlige spørsmål

<details>

<summary>Kan jeg behandle bilder fra kameraer som ikke er av merket MAPIR med Chloros?</summary>

Nei, Chloros støtter kun behandling av bilder fra MAPIR-kameraer. Se listen over [støttede kameramodeller](supported-cameras.md) for mer informasjon. Vi tilbyr behandling av bilder fra andre kameraer på MAPIR Cloud, se fullstendig liste [her](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Kan jeg kalibrere bildene mine for reflektans uten et kalibreringsmål?</summary>

Nei. Uten et bilde av kalibreringsmålet som er tatt samtidig med bildene uten målet, vil du ikke kunne knytte bildets pikselverdier til en kjent refleksjonsprosent. Hvis du heller ikke inkluderer loggen fra en MAPIR-lyssensor, vil ikke omgivelseslyspektrumet bli målt, og refleksjonsresultatene vil ikke være nøyaktige.

</details>

<details>

<summary>Kan jeg redigere bildene mine før behandling i Chloros?</summary>

Nei. Chloros forutsetter at inndataene ikke er endret. Ikke endre filnavnene.

</details>

<details>

<summary>Kan jeg stille inn MAPIR- og Survey3-kameraene mine på automatisk eksponering og behandle bildene i Chloros?</summary>

Nei. Survey3-bildedatasett må ha en fast/låst eksponering, så ingen automatisk lukkerhastighet eller automatisk ISO. Alle bilder fra samme kameramodell må ha identisk lukkerhastighet og ISO (eksponering).

</details>

<details>

<summary>Kan Chloros behandle eller analysere ortomosaikkbilder?</summary>

Nei. Kun individuelle MAPIR-kamera-bilder støttes, ikke sammensatte bilder som et ortomosaikk-kart.

</details>

<details>

<summary>Hvordan kan jeg øke hastigheten på måldeteksjonstrinnet i Chloros?</summary>

I filbrowsertabellen vil forhåndsvalg av målbildene i høyre kolonne fortelle Chloros å bare lete etter kalibreringsmål i disse bildene, noe som øker hastigheten på behandlingen betydelig.

</details>

<details>

<summary>Hvis jeg skal laste opp bildene mine til <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> bør jeg da behandle dem i Chloros før opplasting?</summary>

Hvis du planlegger å laste opp til vår online behandlingsplattform [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), må du ikke redigere bildene før du laster dem opp. Cloud vil utføre all den samme behandlingen og mer.

</details>

<details>

<summary>Vil MAPIR noen gang støtte X-funksjonen? Jeg skulle virkelig ønske at MAPIR tilbød X.</summary>

Vi er alltid interessert i å motta tilbakemeldinger på produktene våre. Hvis du oppdager et problem med produktene våre, eller har et forslag til hvordan vi kan forbedre dem, kan du [KONTAKTE OSS](https://www.mapir.camera/community/contact) for å dele dine tanker. Det meste av vår FoU er styrt av å lytte til kundenes største behov.

</details>

<details>

<summary>Er Chloros tilgjengelig for Linux?</summary>

Ja! Chloros 1.1.0 støtter Linux amd64 (x86\_64) og arm64 (NVIDIA Jetson JetPack 6) via `.deb`-pakker. CLI og Python SDK støttes fullt ut på Linux. Det finnes ingen GUI for Linux — all interaksjon skjer via [CLI](CLI.md) eller [Python SDK](api-python-sdk.md). Se [Linux Oversikt](linux/linux-overview.md) for detaljer.

</details>

<details>

<summary>Kan jeg kjøre Chloros på NVIDIA Jetson?</summary>

Ja! Chloros 1.1.0 støtter NVIDIA Jetson-plattformer, inkludert Jetson Nano, Orin Nano, Orin NX og AGX Orin som kjører JetPack 6. Chloros oppdager automatisk Jetson-modellen din og optimaliserer behandlingsstrategien. Se [NVIDIA Jetson-veiledningen](linux/nvidia-jetson-guide.md) for instruksjoner om oppsett og distribusjon.

</details>

<details>

<summary>Optimaliserer Chloros automatisk for maskinvaren min?</summary>

Ja! Chloros 1.1.0 inkluderer [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) som automatisk oppdager CPU, GPU, RAM og (på Jetson) termiske sensorer. Deretter velger den den optimale behandlingsstrategien — fra `GPU_PARALLEL` på systemer med mye minne til `GPU_SINGLE` på enheter med begrensninger til `CPU_PARALLEL` på systemer uten en NVIDIA GPU. Ingen manuell konfigurasjon er nødvendig.

</details>

<details>

<summary>Hva er 4-tråds prosesseringspipeline?</summary>

Chloros 1.1.0 bruker en 4-tråds pipelined arkitektur for Chloros+ brukere: Tråd 1 (deteksjon) laster inn bilder og oppdager kalibreringsmål, tråd 2 (kalibrering) beregner reflektanskalibrering, tråd 3 (behandling) utfører GPU-akselerert debayering og indeksberegning, og tråd 4 (eksport) skriver utdatafiler. Flere bilder kan være i forskjellige tråder samtidig for maksimal gjennomstrømning. Se [Behandlingspipeline](processing-architecture/processing-pipeline.md) for detaljer.

</details>

<details>

<summary>Hvordan kjører jeg diagnostikk på min Chloros-installasjon?</summary>

Bruk kommandoen `selftest` til å kjøre 7 systemdiagnostikker, inkludert versjonssjekk, porttilgjengelighet, oppstart av backend, API-tilkobling, systeminformasjon, støydempingsmodeller og CUDA-tilgjengelighet:

```bash
chloros-cli selftest
```

Dette er spesielt nyttig på Linux/Jetson-systemer for å verifisere GPU- og CUDA-oppsettet.

</details>
