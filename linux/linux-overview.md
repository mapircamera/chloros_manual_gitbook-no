# Linux Oversikt

Chloros 1.1.0 gir innebygd Linux-støtte for **CLI**og**Python SDK**, noe som muliggjør headless multispektral bildebehandling på Linux-arbeidsstasjoner, servere og NVIDIA Jetson-kant-enheter.

{% hint style="info" %}
**Ingen GUI på Linux.** Chloros Desktop GUI er kun tilgjengelig på Windows. Linux-brukere kommuniserer med Chloros via [CLI](../CLI.md) og [Python SDK](../api-python-sdk.md).
{% endhint %}

***

## Plattformstøttematrise

| Funksjon | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Desktop GUI** | Ja | N/A | Nei | Nei |
| **CLI** | Ja | Ja | Ja | Ja |
| **Python SDK** | Ja | Ja | Ja | Ja |
| **GPU-akselerasjon (CUDA)** | Ja | Ja | Ja | Ja (JetPack 6) |
| **Teksturbevisst debayer** | Ja (Chloros+) | Ja (Chloros+) | Ja (Chloros+) | Ja (Chloros+) |
| **Dynamisk beregningsadaptasjon** | Ja | Ja | Ja | Ja |***

## Støttede arkitekturer

| Arkitektur | Beskrivelse | Installasjonsmetode |
| --- | --- | --- |
| **amd64 (x86_64)** | Standard prosessorer for stasjonære datamaskiner/servere (Intel, AMD) | `.deb`-pakke |
| **arm64 (aarch64)** | ARM-baserte prosessorer, hovedsakelig NVIDIA Jetson | `.deb`-pakke (JetPack 6) |

## Støttede Linux-distribusjoner

* **Ubuntu 20.04+** (amd64)
* **Debian 11+** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson-plattformer)***

## Hva Linux-brukere får

* **Chloros CLI** — Fullt kommandolinjegrensesnitt for batchbehandling, automatisering og skripting
* **Chloros Python SDK** — Programmatisk Python-grensesnitt (`pip install chloros-sdk`) for integrering i forskningspipelines og tilpassede verktøy
* **GPU-akselerasjon** — CUDA-akselerert behandling på NVIDIA-GPUer (stasjonære og Jetson)
* **Dynamisk beregningsadaptasjon** — Automatisk maskinvaredeteksjon og optimalisering av behandlingsstrategi
* **Alle behandlingsfunksjoner** — Samme multispektrale behandlingspipeline som Windows (kalibrering, vignettkorrigering, vegetasjonsindekser, alle eksportformater)
* **Chloros+-funksjoner** — Multitrådet prosessering, teksturbevisst debayer, tilpassede indekser (med Chloros+-lisens)

## Hva Linux-brukere ikke får

* **Desktop GUI** — Ingen grafisk grensesnitt; all interaksjon skjer via CLI eller Python SDK
* **Bildevisning** — Ingen interaktiv bildevisning, rutenettvisning eller kartmarkører
* **Visuell prosjektstyring** — Prosjekter styres via CLI-kommandoer og SDK-kall***

## Komme i gang med Linux

1. **Installer Chloros** — Se [Linux Installasjon](linux-installation.md) for installasjon av `.deb`-pakken
2. **Installer Python SDK** (valgfritt) — `pip install chloros-sdk`
3. **Aktiver lisensen din** — `chloros-cli login your@email.com 'password'`
4. **Behandle ditt første datasett** — `chloros-cli process ~/datasets/flight001`

For NVIDIA Jetson-brukere, se den dedikerte [NVIDIA Jetson-veiledningen](nvidia-jetson-guide.md) for plattformspesifikk oppsett og optimalisering.

***

## Neste trinn

* [Linux Installasjon](linux-installation.md) — Detaljerte installasjonsinstruksjoner for amd64 og arm64
* [NVIDIA Jetson-veiledning](nvidia-jetson-guide.md) — Jetson-spesifikk konfigurasjon, termisk styring og feltutrulling
* [CLI : Kommandolinje](../CLI.md) — Fullstendig CLI-referanse
* [API : Python SDK](../api-python-sdk.md) — Fullstendig SDK-referanse
* [Dynamisk databehandlingstilpasning](../processing-architecture/dynamic-compute-adaptation.md) — Hvordan Chloros tilpasser seg maskinvaren din
