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
| **Grafikkort**    | DirectX 11-kompatibelt                                | NVIDIA-GPU med 4 GB+ VRAM                            |
| **Lagringsplass**          | 6 GB ledig plass                                       | SSD med 10 GB+ ledig plass                            |
| **Skjerm**          | 1920x1080                                            | 2560x1440 eller høyere                                  |
| **Internett**         | Nødvendig for \[valgfri] Chloros+ lisensaktivering | Nødvendig for \[valgfri] aktivering av Chloros+-lisensen |

#### Linux amd64 (x86\_64)

| Krav       | Minimum                    | Anbefalt               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribusjon**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Prosessor**     | x86_64 (Intel/AMD)        | Intel Core i7 eller bedre   |
| **Minne (RAM)**  | 8 GB                        | 16 GB eller mer              |
| **Grafikkort** | Ingen (CPU-behandling)      | NVIDIA-GPU med 4 GB+ VRAM |
| **Lagringsplass**       | 2 GB ledig plass             | SSD med 10 GB+ ledig       |
| **Python**        | Python 3.7+ (for SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Krav      | Minimum                      | Anbefalt                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Plattform**     | NVIDIA Jetson med JetPack 6 | Jetson Orin NX 16 GB eller AGX Orin |
| **Minne (RAM)** | 8 GB (delt mellom GPU og CPU)         | 16 GB+ delt                    |
| **Lagringsplass**      | 2 GB ledig plass               | NVMe SSD med 10 GB+ ledig        |
| **Python**       | Python 3.7+ (for SDK)        | Python 3,10+                    |

{% hint style="info" %}
**GPU-akselerasjon**: Brukere av Chloros+ med NVIDIA-GPU-er kan bruke CUDA-akselerasjon for betydelig raskere behandling. Dette fungerer både på Windows (stasjonære GPU-er) og Linux (stasjonære GPU-er og NVIDIA Jetson). Chloros+-brukere får også flertrådet behandling for maksimal hastighet.
{% endhint %}

***

## Last ned Chloros

### Siste stabile utgave: Versjon 1.2.0

<!-- NOLAN: replace installer links + release date for 1.2.0 — the three download buttons below still point at the 1.1.0 Google Drive files, and the release date needs to be added to the heading above. -->



### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Last ned Chloros for Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Last ned Chloros for Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Last ned Chloros for Linux arm64 / Jetson (.deb)</a>

#### Windows-installasjonsprogram (GUI + CLI + Backend)

* **Filtype**: .exe (Windows-installasjonsprogram)**Installasjonstrinn:**

1. Last ned .exe-filen ovenfor
2. Dobbeltklikk på installasjonsprogrammet for å starte installasjonen
3. Følg instruksjonene i installasjonsveiviseren
4. Velg installasjonskatalog (standard: `C:\Program Files\MAPIR\Chloros\`)
5. Fullfør installasjonen og start Chloros eller Chloros CLI
6. Logg inn med din [MAPIR Cloud Chloros+-konto](https://cloud.mapir.camera/pricing) (eller fortsett med gratisversjonen)

{% hint style="success" %}
Installasjonsprogrammet legger automatisk til `chloros-cli` i systemets PATH for tilgang via kommandolinjen.
{% endhint %}

#### Linux amd64 (.deb-pakke — CLI + Backend)

* **Filtype**: .deb (Debian/Ubuntu-pakke)
* **Arkitektur**: x86_64 (amd64)

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

Se [Linux-installasjon](linux/linux-installation.md) for detaljerte instruksjoner om oppsett og [NVIDIA Jetson-veiledning](linux/nvidia-jetson-guide.md) for Jetson-spesifikk veiledning.

#### Python SDK (alle plattformer)

Hvert installasjonsprogram inneholder et tilhørende `chloros_sdk`-hjul, slik at SDK-versjonen alltid samsvarer med den installerte GUI/CLI/backend. På Windows installerer installasjonsprogrammet det automatisk i systemet ditt Python; på Linux plasserer `.deb` hjulet i `/usr/lib/chloros/sdk/` og viser installasjonskommandoen:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

For verter som kun bruker pip (uten Chloros-pakken installert), finnes SDK også på PyPI:

```bash
pip install chloros-sdk
```

Se [API : Python SDK](api-python-sdk.md) og [SDK-referansen](reference/sdk-reference.md) for dokumentasjon.

{% hint style="info" %}
**Linux-brukere**: `.deb`-pakken installerer CLI og backend. Det finnes ingen grafisk brukergrensesnitt for Linux — all interaksjon skjer via CLI eller SDK.
{% endhint %}

***

## Ytterligere ressurser

### Python SDK

For utviklere og automatiseringsarbeidsflyter, installer Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentasjon**: [API: Python SDK](api-python-sdk.md)**Krav**: Chloros må være installert (Windows-installasjonsprogrammet eller Linux `.deb`-pakken), Chloros+ lisensinnlogging kreves***

## Hva er inkludert

### Windows-installasjonsprogram

* ✅ **Chloros GUI** – Grafisk brukergrensesnitt med full funksjonalitet
* ✅ **Chloros CLI** - Kommandolinjegrensesnitt (krever Chloros+-lisens)
* ✅ **Chloros Backend** – Behandlingsmotor
* ✅ **Kameraprofiler** – Forhåndskonfigurerte MAPIR-kameramaler

### Linux .deb-pakke

* ✅ **Chloros CLI** – Kommandolinjegrensesnitt (krever Chloros+-lisens)
* ✅ **Chloros Backend** – Behandlingsmotor
* ✅ **Kameraprofiler** – Forhåndskonfigurerte MAPIR-kameramaler
* ❌ Ingen grafisk brukergrensesnitt — Linux er kun tilgjengelig som headless CLI/SDK

### Python SDK (PIP, alle plattformer)

* ✅ **Chloros SDK** – Python API (krever Chloros+-lisens)***

## Oppgrader til Chloros+

Få tilgang til avanserte funksjoner med et Chloros+-abonnement:

* 🚀 **Flertrådet behandling** – Behandle bilder parallelt
* ⚡ **GPU (CUDA)-akselerasjon** – Utnytt kraften fra NVIDIA-GPU-er
* 💻 **CLI-tilgang** - Automatiser med kommandolinjeverktøy
* 🐍 **Python SDK** – Programmatisk API-tilgang
* 📱 **Flere enheter** – Bruk på 2–10+ enheter (avhengig av abonnement)
* **🐻 Avansert teksturbevisst debayer-metode** – en høykvalitets kantbevisst debayer kombinert med en AI/ML-støyreduksjonsmodell som fjerner nesten all debayer-støy.
* 🧮 **Egendefinerte formler** – Lag egendefinerte multispektrale indekser

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Se Chloros+-abonnementer og priser</a></p>***

## Hjelp til installasjon

### Feilsøking

**Installasjonen mislykkes med følgende feilmelding:**

* Sørg for at du har administratorrettigheter
* Deaktiver antivirusprogrammet midlertidig
* Sjekk at du oppfyller minimumskravene til systemet

**Programmet starter ikke (Windows):**

* Kontroller at Windows 10/11 (64-bit) er installert
* Oppdater grafikkdrivere
* Sjekk feilinformasjonen i Windows Hendelsesvisning
* Kontakt support med feilloggene

**CLI starter ikke (Linux):**

* Kontroller at `.deb`-pakken er riktig installert: `dpkg -l | grep chloros`
* Kontroller tilgangsrettigheter: `sudo chmod +x /usr/bin/chloros-cli`
* Kjør diagnostikk: `chloros-cli selftest`
* Sjekk om det mangler biblioteker: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Problemer med lisensaktivering:**

* Sørg for at internettforbindelsen er aktiv
* Bekreft påloggingsopplysningene på [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Sjekk at brannmuren ikke blokkerer Chloros
* Se [Chloros+ Innlogging](chloros+-login.md) for detaljerte instruksjoner

### Få støtte

Trenger du hjelp med installasjon eller oppsett?

* 📧 **E-post**: info@mapir.camera
* 🌐 **Nettsted**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentasjon**: [Kom i gang](./)
* ❓ **FAQ**: [Ofte stilte spørsmål](faq.md)***

## Programvareoppdateringer

Chloros søker etter oppdateringer, varsler når en ny versjon er tilgjengelig og lenker til denne nedlastingssiden — du oppdaterer ved å kjøre det nye signerte installasjonsprogrammet. Innstillingene og prosjektene dine beholdes etter oppdateringer. På Linux og Jetson søker `chloros-cli update` etter en nyere versjon og tilbyr å laste ned og installere den tilhørende `.deb` (denne kommandoen er kun tilgjengelig i Linux).

***

## Endringslogg**Versjon 1.2.0 (siste)**— se**Hva er nytt i Chloros 1.2.0** på siden [Kom i gang](./) for den fullstendige funksjonslisten.

<details>

<summary>Versjon 1.0.5</summary>

**Utgivelsesdato: 10. februar 2026**

**Nye funksjoner*** **Teksturbevisst debayer-metode \[Kun Chloros+] –** Teksturbevisst bruker en kantbevisst debayer av høy kvalitet kombinert med en AI/ML-støyreduksjonsmodell som fjerner nesten all debayering-støy.
* **Støtte for T4P-kalibreringsmål*** **Raskere GPU-behandling i Chloros+, bedre minnestyring**

**Feilrettinger*** Helt nytt brukergrensesnitt (GUI), bør nå fungere på alle Windows-datamaskiner.

</details>

<details>

<summary>Versjon 1.0.4</summary>

**Utgivelsesdato: 5. januar 2026**

**Nye funksjoner*** **Veksling mellom bilde og metadata**: Lagt til veksling i filutforskeren for å vise metadata for det valgte bildet i en tabell i stedet for i bildeoversikten
* **Zoomglidebryter for bildeoversikt**: Ny glidebryter i brukergrensesnittet for å justere miniatyrstørrelsen (støtter også CTRL + mushjul)
* **Eksportknapper for bildeoversikten**: Knapper i øverste rad for å bytte miniatyrbilder fra JPG til bearbeidede eksporter (Targets, Reflectance, Index, LUT)
* **Kartfanen**: Nytt interaktivt 2D-kart som viser GPS-posisjonsmarkører for bildene
  * Støtter Google Maps og ESRI-kartfliser (velger automatisk den beste flisetjenesten basert på tilgjengelighet ved ulike zoomnivåer)
  * Forhåndsvisning av miniatyrbilder ved å holde musepekeren over kartmarkørene

**Feilrettinger*** Forbedret støtte for installasjon av Chloros på datamaskiner med andre språk enn engelsk

</details>

<details>

<summary>Versjon 1.0.3</summary>

**Utgivelsesdato: 20. desember 2025**

**Nye funksjoner*** Første utgivelse

**Forbedringer*** Første utgivelse

**Feilrettinger*** Første utgivelse

**Kjente problemer*** Første utgivelse

</details>***

## Lisensavtale**Proprietær programvare** – Copyright (c) 2026 MAPIR Inc.

Uautorisert bruk, distribusjon eller modifisering er forbudt.

**Gratisversjon**: Tilgjengelig for personlig og kommersiell bruk med begrensede funksjoner**Chloros+**: Abonnementsbasert lisens for avanserte funksjoner og kommersiell bruk
