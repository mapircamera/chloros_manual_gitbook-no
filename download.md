---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Last ned

Last ned den nyeste versjonen av Chloros for å komme i gang med multispektral bildebehandling.

### Systemkrav

#### Windows

| Krav          | Minimum                                              | Anbefalt                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Operativsystem** | Windows 10 (64-bit)                                  | Windows 11 (64-bit)                                  |
| **Prosessor**        | Intel Core i5 eller tilsvarende                          | Intel Core i7 eller bedre                              |
| **Minne (RAM)**     | 8 GB                                                  | 16 GB eller mer                                         |
| **Grafikkort**    | Kompatibelt med DirectX 11                                | NVIDIA GPU med 4 GB+ VRAM                            |
| **Lagringsplass**          | 6 GB ledig plass                                       | SSD med 10 GB+ ledig plass                            |
| **Skjerm**          | 1920x1080                                            | 2560x1440 eller høyere                                  |
| **Internett**         | Nødvendig for \[valgfri] Chloros+ lisensaktivering | Nødvendig for \[valgfri] Chloros+ lisensaktivering |

#### Linux amd64 (x86\_64)

| Krav       | Minimum                    | Anbefalt               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribusjon**  | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+             |
| **Prosessor**     | x86\_64 (Intel/AMD)        | Intel Core i7 eller bedre   |
| **Minne (RAM)**  | 8 GB                        | 16 GB eller mer              |
| **Grafikkort** | Ingen (CPU-behandling)      | NVIDIA GPU med 4 GB+ VRAM |
| **Lagring**       | 2 GB ledig plass             | SSD med 10 GB+ ledig       |
| **Python**        | Python 3.7+ (for SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Krav      | Minimum                      | Anbefalt                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Plattform**     | NVIDIA Jetson med JetPack 6 | Jetson Orin NX 16 GB eller AGX Orin |
| **Minne (RAM)** | 8 GB (delt GPU/CPU)         | 16 GB+ delt                    |
| **Lagring**      | 2 GB ledig plass               | NVMe SSD med 10 GB+ ledig        |
| **Python**       | Python 3.7+ (for SDK)        | Python 3.10+                    |

{% hint style="info" %}
**GPU-akselerasjon**: Chloros+-brukere med NVIDIA-GPU-er kan bruke CUDA-akselerasjon for betydelig raskere behandling. Dette fungerer både på Windows (stasjonære GPU-er) og Linux (stasjonære GPU-er og NVIDIA Jetson). Chloros+-brukere får også flertrådet behandling for maksimal hastighet.
{% endhint %}

***

## Last ned Chloros

### Siste stabile utgivelse (23. mars 2026): Versjon 1.1.0

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Last ned Chloros for Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Last ned Chloros for Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Last ned Chloros for Linux arm64 / Jetson (.deb)</a>

#### Windows Installer (GUI + CLI + Backend)

* **Filtype**: .exe (Windows Installer)**Installasjonstrinn:**

1. Last ned filen .exe ovenfor
2. Dobbeltklikk på installasjonsprogrammet for å starte installasjonen
3. Følg instruksjonene i installasjonsveiviseren
4. Velg installasjonskatalog (standard: `C:\Program Files\[USER]\Chloros\`)
5. Fullfør installasjonen og start Chloros eller Chloros CLI
6. Logg inn med din [MAPIR Cloud Chloros+-konto](https://cloud.mapir.camera/pricing) (eller fortsett med gratisversjonen)

{% hint style="success" %}
Installasjonsprogrammet legger automatisk til `chloros-cli` i systemets PATH for kommandolinjetilgang.
{% endhint %}

#### Linux amd64 (.deb-pakke — CLI + Backend)

* **Filtype**: .deb (Debian/Ubuntu-pakke)
* **Arkitektur**: x86\_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (.deb-pakke — CLI + Backend)

* **Filtype**: .deb (JetPack 6)
* **Arkitektur**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Se [Linux Installasjon](linux/linux-installation.md) for detaljerte oppsettinstruksjoner og [NVIDIA Jetson-veiledning](linux/nvidia-jetson-guide.md) for Jetson-spesifikk veiledning.

#### Python SDK (Alle plattformer)

```bash
pip install chloros-sdk
```

Se [API : Python SDK](api-python-sdk.md) for dokumentasjon.

{% hint style="info" %}
**Linux-brukere**: `.deb`-pakken installerer CLI og backend. Python SDK installeres separat via pip. Det finnes ingen GUI for Linux — all interaksjon skjer via CLI eller SDK.
{% endhint %}

***

## Tilleggsressurser

### Python SDK

For utviklere og automatiseringsarbeidsflyter, installer Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentasjon**: [API: Python SDK](api-python-sdk.md)**Krav**: Chloros må være installert (Windows-installasjonsprogram eller Linux `.deb`-pakke), Chloros+ lisensinnlogging kreves***

## Hva er inkludert

### Windows-installasjonsprogram

* ✅ **Chloros GUI** – Grafisk grensesnitt med alle funksjoner
* ✅ **Chloros CLI** – Kommandolinjegrensesnitt (krever Chloros+ lisens)
* ✅ **Chloros Backend** – Behandlingsmotor
* ✅ **Kameraprofiler** - Forhåndskonfigurerte MAPIR-kameramaler

### Linux .deb-pakke

* ✅ **Chloros CLI** - Kommandolinjegrensesnitt (krever Chloros+ lisens)
* ✅ **Chloros Backend** - Behandlingsmotor
* ✅ **Kameraprofiler** - Forhåndskonfigurerte MAPIR kameramaler
* ❌ Ingen GUI — Linux er kun headless CLI/SDK

### Python SDK (pip, alle plattformer)

* ✅ **Chloros SDK** - Python API (krever Chloros+-lisens)***

## Oppgrader til Chloros+

Få tilgang til avanserte funksjoner med et Chloros+-abonnement:

* 🚀 **Multitrådet behandling** – Behandle bilder parallelt
* ⚡ **GPU (CUDA)-akselerasjon** – Utnytt kraften i NVIDIA-GPU-er
* 💻 **CLI-tilgang** – Automatiser med kommandolinjeverktøy
* 🐍 **Python SDK** – Programmatisk API-tilgang
* 📱 **Flere enheter** – Bruk på 2–10+ enheter (avhengig av abonnement)
* **🐻 Avansert teksturbevisst debayer-metode** – en høykvalitets kantbevisst debayer kombinert med en AI/ML-støyreduksjonsmodell som fjerner nesten all debayering-støy.
* 🧮 **Tilpassede formler** – Lag tilpassede multispektrale indekser

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Se Chloros+-planer og priser</a></p>***

## Hjelp til installasjon

### Feilsøking

**Installasjonen mislykkes med feilmelding:**

* Sørg for at du har administratorrettigheter
* Deaktiver antivirusprogramvare midlertidig
* Sjekk at du oppfyller minimumssystemkravene

**Programmet starter ikke (Windows):**

* Kontroller at Windows 10/11 (64-bit) er installert
* Oppdater grafikkdrivere
* Sjekk Windows Hendelsesvisning for feildetaljer
* Kontakt support med feillogger

**CLI starter ikke (Linux):**

* Kontroller at `.deb`-pakken er riktig installert: `dpkg -l | grep chloros`
* Kontroller tillatelser: `sudo chmod +x /usr/bin/chloros-cli`
* Kjør diagnostikk: `chloros-cli selftest`
* Sjekk om det mangler biblioteker: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Problemer med lisensaktivering:**

* Sørg for at internettforbindelsen er aktiv
* Kontroller påloggingsinformasjonen på [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Sjekk at brannmuren ikke blokkerer Chloros
* Se [Chloros+ Innlogging](chloros+-login.md) for detaljerte instruksjoner

### Få support

Trenger du hjelp med installasjon eller oppsett?

* 📧 **E-post**: info@mapir.camera
* 🌐 **Nettsted**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentasjon**: [Kom i gang](./)
* ❓ **FAQ**: [Ofte stilte spørsmål](faq.md)***

## Endringslogg

<details>

<summary>Versjon 1.1.0 (Siste)</summary>

**Utgivelsesdato: mars 2026**

**Nye funksjoner*** **Linux-støtte** — Innbygget CLI og SDK for Linux amd64 (x86\_64) og arm64 (NVIDIA Jetson JetPack 6). Installer via `.deb`-pakker.
* **NVIDIA Jetson-støtte** — Optimalisert prosessering for Jetson Nano-, Orin Nano-, Orin NX- og AGX Orin-kant-enheter.
* **Dynamisk databehandlings tilpasning** — Automatisk maskinvaredeteksjon og optimalisering av behandlingsstrategi. Chloros tilpasser seg maskinvaren din, fra en Jetson Nano til en arbeidsstasjon med flere GPU-er.
* **4-tråds behandlingspipeline** — Samtidige tråder for deteksjon, kalibrering, behandling og eksport med dynamisk tildeling av GPU-minne.
* **Nye CLI-kommandoer** — `selftest` (systemdiagnostikk) og `update` (Linux oppdateringsadministrasjon).
* **Nye CLI-prosessflagg** — `--debayer` (standard/teksturbevisst), `--indices` (spesifiser indekser), `--target` (søk først i målundermappen for raskere deteksjon).
* **Nye GUI-menyelementer** — Legg til filer, Legg til mappe og Start/Stopp behandling er nå tilgjengelig fra rullegardinmenyen i hovedmenyen.**Forbedringer**

* Automatisk gjenkjenning av plattformuavhengig backend (banene Windows og Linux)
* Forbedret SDK `get_status()` med fremdriftssporing per tråd
* Nye SDK-unntak: `ChlorosConfigurationError`, `ChlorosAuthenticationError`
* Termisk styring og adaptiv struping for NVIDIA Jetson
* Automatisk minnestyring med OOM-fallback til flislagt GPU-behandling

</details>

<details>

<summary>Versjon 1.0.5</summary>

**Utgivelsesdato: 10. februar 2026**

**Nye funksjoner*** **Teksturbevisst debayer-metode \[Kun Chloros+] -** Teksturbevisst bruker en kantbevisst debayer av høy kvalitet kombinert med en AI/ML-støyreduksjonsmodell som fjerner nesten all debayering-støy.
* **Støtte for T4P-kalibreringsmål*** **Raskere Chloros+ GPU-behandling, bedre minnestyring**

**Feilrettinger*** Helt nytt frontend (GUI), bør nå fungere på alle Windows-datamaskiner.

</details>

<details>

<summary>Versjon 1.0.4</summary>

**Utgivelsesdato: 5. januar 2026**

**Nye funksjoner*** **Veksling mellom bilde og metadata**: Lagt til veksling i filbrowseren for å vise metadata for valgt bilde i en tabell i stedet for i bildeoversikten
* **Zoomglidebryter for bildeoversikt**: Ny glidebryter i brukergrensesnittet for å justere størrelsen på miniatyrbildene (støtter også CTRL + mushjul)
* **Eksportknapper for bildeoversikt**: Knapper i øverste rad for å bytte miniatyrbilder fra JPG til bearbeidede eksporter (Mål, Refleksjon, Indeks, LUT)
* **Kart-fanen**: Nytt interaktivt 2D-kart som viser GPS-posisjonsmarkører for bildene
  * Støtter Google Maps og ESRI-kartfliser (velger automatisk den beste flisetjenesten basert på tilgjengelighet for zoomnivå)
  * Forhåndsvisning av miniatyrbilder ved å holde musen over kartmarkører

**Feilrettinger*** Forbedret støtte for installasjon av Chloros på datamaskiner med andre språk enn engelsk

</details>

<details>

<summary>Versjon 1.0.3</summary>

**Utgivelsesdato: 20. desember 2025**

**Nye funksjoner*** Første lansering

**Forbedringer*** Første lansering

**Feilrettinger*** Første lansering

**Kjente problemer*** Første lansering

</details>***

## Lisensavtale**Proprietær programvare** – Copyright (c) 2026 MAPIR Inc.

Uautorisert bruk, distribusjon eller modifisering er forbudt.

**Gratisversjon**: Tilgjengelig for personlig og kommersiell bruk med begrensede funksjoner**Chloros+**: Abonnementsbasert lisens for avanserte funksjoner og kommersiell bruk
