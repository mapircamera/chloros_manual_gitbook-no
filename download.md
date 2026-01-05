---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Last ned

Last ned den nyeste versjonen av Chloros for å komme i gang med multispektral bildebehandling.

### Systemkrav

| Krav          | Minimum                         | Anbefalt                     |
| -------------------- | ------------------------------- | ------------------------------- |
| **Operativsystem** | Windows 10 (64-bit)             | Windows 11 (64-bit)             |
| **Prosessor**        | Intel Core i5 eller tilsvarende     | Intel Core i7 eller bedre         |
| **Minne (RAM)**     | 8 GB                             | 16 GB eller mer                    |
| **Grafikkort**    | DirectX 11-kompatibelt           | NVIDIA GPU med 4 GB+ VRAM       |
| **Lagring**          | 6 GB ledig plass                  | SSD med 10 GB+ ledig plass       |
| **Skjerm**          | 1920x1080                       | 2560x1440 eller høyere             |
| **Internett**         | Nødvendig for lisensaktivering | Nødvendig for lisensaktivering |

{% hint style=&quot;info&quot; %}
**GPU-akselerasjon**: Chloros+-brukere med NVIDIA GPU-er (4 GB+ VRAM) kan bruke CUDA-akselerasjon for betydelig raskere behandling. Chloros+-brukere får også flertrådet behandling for maksimal hastighet.
{% endhint %}

***

## Last ned Chloros

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Last ned Chloros her</a>

### Siste stabile utgivelse

**Chloros Installasjonsprogram for Windows*** **Versjon**: 1.0.4
* **Utgivelsesdato**: 5. januar 2026
* **Filstørrelse (nedlasting)**: 1,8 GB
* **Filstørrelse (installert)**: 5,7 GB
* **Filtype**: .exe (Windows Installer)

#### **Installasjonsfremgangsmåte:**

1. Last ned filen `CHLOROS INSTALLER - CURRENT VERSION.exe`
2. Dobbeltklikk på installasjonsprogrammet for å starte installasjonen
3. Følg instruksjonene i installasjonsveiviseren
4. Velg installasjonskatalog (standard: `C:\Program Files\[USER]\Chloros\`)
5. Fullfør installasjonen og start Chloros, Chloros (nettleser) eller Chloros CLI
6. Logg inn med din [MAPIR Cloud Chloros+ konto](https://cloud.mapir.camera/pricing) (eller fortsett med gratisversjonen)

{% hint style=&quot;success&quot; %}
Installasjonsprogrammet legger automatisk til `chloros-cli` i systemets PATH for kommandolinjetilgang.
{% endhint %}

***

## Tilleggsressurser

### Python SDK

For utviklere og automatiseringsarbeidsflyter, installer Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentasjon**: [API: Python SDK](api-python-sdk.md)**Krav**: Chloros Desktop må være installert, Chloros+ lisensinnlogging kreves.***

## Hva er inkludert

Chloros-installasjonen inkluderer:

* ✅ **Chloros** – Fullverdig grafisk grensesnitt
* ✅ **Chloros (nettleser)** – Nettbasert grensesnitt for systemer med lavere spesifikasjoner
* ✅ **Chloros CLI** – Kommandolinjegrensesnitt (krever Chloros+ lisens)
* ✅ **Chloros SDK** - Python API (krever Chloros+ lisens)
* ✅ **Kameraprofiler** - Forhåndskonfigurerte MAPIR kameramaler***

## Oppgrader til Chloros+

Få tilgang til avanserte funksjoner med et Chloros+-abonnement:

* 🚀 **Multitrådet behandling** – Behandle bilder parallelt
* ⚡ **GPU (CUDA)-akselerasjon** – Utnytt kraften i NVIDIA GPU
* 💻 **CLI-tilgang** – Automatiser med kommandolinjeverktøy
* 🐍 **Python SDK** – Programmatisk API-tilgang
* 📱 **Flere enheter** – Bruk på 2–10+ enheter (avhengig av abonnement)
* 🧮 **Tilpassede formler** – Lag tilpassede multispektrale indekser

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Vis Chloros+ Planer og priser</a></p>***

## Hjelp til installasjon

### Feilsøking

**Installasjonen mislykkes med feilmelding:**

* Sørg for at du har administratorrettigheter
* Deaktiver antivirusprogramvaren midlertidig
* Kontroller at du oppfyller minimumssystemkravene

**Programmet starter ikke:**

* Prøv Chloros (nettleser) versjonen
* Kontroller at Windows 10/11 (64-bit) er installert
* Oppdater grafikkdrivere
* Sjekk Windows Hendelseslogg for feildetaljer
* Kontakt support med feillogger

**Problemer med lisensaktivering:**

* Kontroller at internettforbindelsen er aktiv
* Kontroller påloggingsinformasjonen på [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Kontroller at brannmuren ikke blokkerer Chloros
* Se [Chloros+ Login](chloros+-login.md) for detaljerte instruksjoner

### Få support

Trenger du hjelp med installasjon eller oppsett?

* 📧 **E-post**: info@mapir.camera
* 🌐 **Nettsted**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentasjon**: [Komme i gang](./)
* ❓ **FAQ**: [Ofte stilte spørsmål](faq.md)***

## Endringslogg

<details>

<summary>Versjon 1.0.4</summary>

#### **Utgivelsesdato**: 5. januar 2026**Nye funksjoner*** **Veksling mellom bilde/metadata**: Veksling lagt til i filbrowseren for å vise metadata for valgt bilde i en tabell i stedet for i bilgeristen
* **Zoomglidebryter for bilgeristen**: Ny glidebryter i brukergrensesnittet for å justere miniatyrbildestørrelsen (støtter også CTRL + mushjul)
* **Eksportknapper for bildegitter**: Knapper i øverste rad for å bytte miniatyrbilder fra JPG til bearbeidede eksporter (mål, refleksjonsgrad, indeks, LUT)
* **Kart-fanen**: Nytt interaktivt 2D-kart som viser GPS-posisjonsmarkører for bilder
  * Støtter Google Maps og ESRI-kartfliser (velger automatisk den beste flisetjenesten basert på tilgjengelig zoomnivå)
  * Musepekeren viser forhåndsvisning av miniatyrbilder på kartmarkører

**Feilrettinger*** Forbedret støtte for installering av Chloros på datamaskiner med andre språk enn engelsk

</details>

<details>

<summary>Versjon 1.0.3</summary>

#### **Utgivelsesdato**: 20. desember 2025**Nye funksjoner*** Første lansering

**Forbedringer*** Første lansering

**Feilrettinger*** Første lansering

**Kjente problemer*** Første lansering

</details>***

## Lisensavtale**Proprietær programvare** – Copyright (c) 2025 MAPIR Inc.

Uautorisert bruk, distribusjon eller modifisering er forbudt.

**Gratis versjon**: Tilgjengelig for personlig og kommersiell bruk med funksjonsbegrensninger.**Chloros+**: Abonnementsbasert lisens for avanserte funksjoner og kommersiell bruk.
