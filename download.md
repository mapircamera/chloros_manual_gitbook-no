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
| **Grafikkort**    | Kompatibelt med DirectX 11                                | NVIDIA-GPU med 4 GB eller mer VRAM                            |
| **Lagringsplass**          | 6 GB ledig plass                                       | SSD med 10 GB eller mer ledig plass                            |
| **Skjerm**          | 1920x1080                                            | 2560x1440 eller høyere                                  |
| **Internett**         | Nødvendig for \[valgfri] aktivering av Chloros+-lisens | Nødvendig for \[valgfri] aktivering av Chloros+-lisens |

#### Linux amd64 (x86\_64)

| Krav       | Minimum                    | Anbefalt               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribusjon**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Prosessor**     | x86\_64 (Intel/AMD)        | Intel Core i7 eller bedre   |
| **Minne (RAM)**  | 8 GB                        | 16 GB eller mer              |
| **Grafikkort** | Ingen (behandles av CPU)      | NVIDIA-GPU med 4 GB+ VRAM |
| **Lagringsplass**       | 2 GB ledig plass             | SSD med 10 GB+ ledig       |
| **Python**        | Python 3.7+ (for SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Krav      | Minimum                      | Anbefalt                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Plattform**     | NVIDIA Jetson med JetPack 6 | Jetson Orin NX 16 GB eller AGX Orin |
| **Minne (RAM)** | 8 GB (delt GPU/CPU)         | 16 GB+ delt                    |
| **Lagringsplass**      | 2 GB ledig plass               | NVMe SSD med 10 GB+ ledig        |
| **Python**       | Python 3.7+ (for SDK)        | Python 3.10+                    |

{% hint style="info" %}
**GPU-akselerasjon**: Chloros+-brukere med NVIDIA-GPU-er kan bruke CUDA-akselerasjon for betydelig raskere behandling. Dette fungerer med NVIDIA-GPU-er for stasjonære datamaskiner på Windows og Linux amd64, samt på NVIDIA Jetson med JetPack 6. Chloros+-brukere får også flertrådet behandling for maksimal hastighet.
{% endhint %}

***

## Last ned Chloros


### Siste stabile utgave: Versjon 1.3.0 (11. november 2026)

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Last ned Chloros for Windows (.exe)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Last ned Chloros for Linux amd64 (.deb)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Last ned Chloros for Linux arm64 / Jetson (.deb)</a>


#### Windows-installasjonsprogram (GUI + CLI + Backend)

* **Filtype**: .exe (Windows-installasjonsprogram)

**Installasjonsfremgangsmåte:**

1. Last ned filen .exe ovenfor
2. Dobbeltklikk på installasjonsprogrammet for å starte installasjonen
3. Følg instruksjonene i installasjonsveiviseren
4. Velg installasjonskatalog (standard: `C:\Program Files\Chloros\`)
5. På siden **Velg tilleggsoppgaver**må du la begge alternativene under**Systeminnstillinger** være merket av (anbefalt – se nedenfor)
6. Fullfør installasjonen og start Chloros eller Chloros CLI
7. Logg inn med din [MAPIR Cloud Chloros+-konto](https://cloud.mapir.camera/pricing) (eller fortsett med gratisversjonen)

**Alternativer for systeminnstillinger** (begge merket av som standard):

| Alternativ, slik det er formulert i installasjonsprogrammet | Hva det gjør | Mulighet for å velge bort stille installasjon |
| --- | --- | --- |
| **Aktiver lange stier i Windows** (lar prosjektmapper ligge dypere enn 260 tegn; anbefalt) | Setter Windows’ systemomfattende innstilling `LongPathsEnabled` til 1. Windows leveres med denne innstillingen deaktivert, og da kan ingen filstier være lengre enn 260 tegn; Chloros-produkter ligger opptil ca. 130 tegn under prosjektmappen, så når lange stier er deaktivert, avslås en dypere prosjektmappe. Trer i kraft for programmer som startes etter installasjonen. Innstillingen forblir aktiv etter avinstallering, da andre programmer kan være avhengige av den. | `/MERGETASKS="!longpaths"` |
| **Ekskluder Chloros-backenden fra Microsoft Defender-skanning** (mye raskere kameraopptak; anbefalt) | Legger til et unntak for Microsoft Defender-prosessen for `chloros-backend.exe` i installasjonsmappen, slik at sanntidsbeskyttelsen ikke skanner hver eneste fil som backenden skriver, uansett hvor den skriver den. Ved høye opptakshastigheter utgjør skanningen omtrent 30 % av opptakshastigheten (77,7 mot 52,7 opptak per sekund ved kontinuerlig opptak med 9 kameraer). Ingen andre programmer ekskluderes, og unntaket fjernes ved avinstallering. Der Defender mangler eller er låst av retningslinjer, mislykkes dette trinnet uten melding, og installasjonen fortsetter. | `/MERGETASKS="!defenderexclusion"` |

En stille installasjon (`/SILENT` eller `/VERYSILENT`) bruker begge alternativene, med mindre du velger bort det med verdien `/MERGETASKS` som vises.

{% hint style="success" %}
Installasjonsprogrammet legger til CLI-mappen (`C:\Program Files\Chloros\cli\`) på systemet ditt `PATH` én gang — ved reinstallering eller oppgradering legges det ikke lenger til en duplikatoppføring, og duplikater som tidligere installasjonsprogrammer har lagt til, fjernes. Åpne en ny terminal etter installasjonen for å bruke `chloros-cli`. Installasjonsprogrammet legger også til to Windows-brannmurregler for backenden, på hver nettverksprofil: **Chloros Backend (UDP-In)**, for kamerastrømmer, PTP, mDNS og DAQ-E-datastrømmen, og**Chloros Backend (TCP-In)**, begrenset til det lokale undernettet, for en DAQ-E-firmwareoppdatering, der sensoren kobler seg tilbake til denne datamaskinen for å hente bildet. Begge fjernes ved avinstallering.
{% endhint %}

#### Linux amd64 (.deb-pakke — CLI + backend)

* **Filtype**: .deb (Debian/Ubuntu-pakke)
* **Arkitektur**: x86_64 (amd64)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

#### Linux arm64 — NVIDIA Jetson (.deb-pakke — CLI + backend)

* **Filtype**: .deb (JetPack 6)
* **Arkitektur**: aarch64 (arm64)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

Bruk det filnavnet du faktisk lastet ned, og behold `./` slik at `apt` installerer den filen i stedet for å lete etter en pakke etter navn. Installer med `apt install`, ikke `dpkg -i`: apt installerer også kravene til Python-SDK, som pakken anbefaler snarere enn krever, og `dpkg -i` hopper over dem. Siden versjon 1.3.0 installeres backend og CLI, utpakket, under `/usr/lib/chloros/`, slik at de starter uten å måtte pakke seg ut først.

Jetson-pakken bruker JetPack 6s CUDA-versjon av PyTorch, så processing, inkludert Texture Aware-debayeren, kjører på Jetsons GPU. Den bruker CUDA- og cuDNN-bibliotekene som JetPack 6 installerer (`sudo apt install nvidia-jetpack` på et bilde som mangler dem).

Se [Linux-installasjon](linux/linux-installation.md) for detaljerte konfigurasjonsinstruksjoner og [NVIDIA Jetson-veiledning](linux/nvidia-jetson-guide.md) for Jetson-spesifikk veiledning.

#### Python SDK (alle plattformer)

Hvert installasjonsprogram inneholder et tilhørende `chloros_sdk`-wheel, slik at SDK-versjonen alltid samsvarer med den installerte GUI/CLI/backend. På Windows installerer installasjonsprogrammet det automatisk i systemets Python. På Linux installerer `.deb` SDK-en for selve systemet `python3` (i `/usr/lib/python3/dist-packages`), slik at `import chloros_sdk` fungerer umiddelbart etter installasjonen, uten pip og uten nettverk. Kun en virtualenv, som ikke har tilgang til systemets pakker, trenger den medfølgende wheel-filen installert i seg:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

For verter som kun bruker pip (ingen Chloros-pakke installert; SDK driver da en backend på en annen maskin), finnes SDK også på PyPI:

```bash
pip install chloros-sdk
```

Se [API: Python SDK](api-python-sdk.md) og [SDK-referansen](reference/sdk-reference.md) for dokumentasjon.

{% hint style="info" %}
**Linux-brukere**: Pakken `.deb` installerer CLI, backend og Python-SDK. Det finnes ingen grafisk brukergrensesnitt for Linux – all interaksjon skjer via CLI eller SDK.
{% endhint %}

***

## Ytterligere ressurser

### Python-SDK


For utviklere og automatiseringsarbeidsflyter følger Chloros-Python-SDK med alle installasjonsprogrammene: Windows-installasjonsprogrammet installerer det i din Python, og Linux-pakken `.deb` installerer det for systemet `python3`. På en maskin uten Chloros-pakke (en som driver en backend på en annen maskin), installerer du den fra PyPI:

```bash
pip install chloros-sdk
```

**Dokumentasjon**: [API: Python SDK](api-python-sdk.md)

**Krav**: en Chloros-backend å kjøre (Windows-installasjonsprogrammet eller Linux-pakken `.deb` installerer en), og en Chloros+-pålogging på et betalt nivå

***

## Hva er inkludert

### Windows-installasjonsprogram

* ✅ **Chloros GUI** – Grafisk brukergrensesnitt med full funksjonalitet
* ✅ **Chloros CLI** – Kommandolinjegrensesnitt (krever Chloros+-lisens)
* ✅ **Chloros-backend** – Behandlingsmotor
* ✅ **Python SDK** – installeres i Python hvis dette finnes (krever Chloros+-lisens)
* ✅ **Kameraprofiler** – forhåndskonfigurerte MAPIR-kameramaler

### Linux.deb-pakke

* ✅ **Chloros CLI** – Kommandolinjegrensesnitt (krever Chloros+-lisens)
* ✅ **Chloros Backend** – Behandlingsmotor
* ✅ **Python SDK** – installert for systemet `python3` (krever Chloros+-lisens)
* ✅ **Kameraprofiler** – Forhåndskonfigurerte MAPIR-kameramaler
* ❌ Ingen grafisk brukergrensesnitt — Linux er kun «headless» CLI/SDK


### Python-SDK (PyPI, for maskiner uten Chloros-pakke)

* ✅ **Chloros-SDK** – Python-API (krever Chloros+-lisens)

***

## Oppgrader til Chloros+

Få tilgang til avanserte funksjoner med et Chloros+-abonnement:

* 🚀 **Flertrådet behandling** – Behandle bilder parallelt
* ⚡ **GPU (CUDA)-akselerasjon** - Utnytt kraften i NVIDIA-GPU-er
* 💻 **Tilgang via CLI** – Automatiser med kommandolinjeverktøy
* 🐍 **Python SDK** – Programmatisk API-tilgang
* 📱 **Flere enheter** – Bruk på 2–10+ enheter (avhengig av abonnement)
* **🐻 Avansert teksturbevisst debayer-metode** – en høykvalitets, kantbevisst debayer kombinert med en AI/ML-støyreduksjonsmodell som fjerner nesten all debayer-støy.
* 🧮 **Egendefinerte formler** – Lag egendefinerte multispektrale indekser

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Se Chloros+-abonnementer og priser</a></p>

***

## Hjelp til installasjon

### Feilsøking

**Installasjonen mislykkes med følgende feilmelding:**

* Sørg for at du har administratorrettigheter
* Deaktiver antivirusprogrammet midlertidig
* Sjekk at du oppfyller minimumskravene til systemet

**Programmet starter ikke (Windows):**

* Kontroller at Windows 10/11 (64-bit) er installert
* Oppdater grafikkdriverne
* Sjekk Windows Hendelsesvisning for feildetaljer
* Kontakt support med feillogger

**CLI starter ikke (Linux):**

* Kontroller at `.deb`-pakken er riktig installert: `dpkg -l | grep chloros`
* Sjekk tilgangsrettigheter: `sudo chmod +x /usr/bin/chloros-cli`
* Kjør diagnostikk: `chloros-cli selftest`
* Sjekk om det mangler biblioteker: `ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"` (siden versjon 1.3.0 er `/usr/lib/chloros/chloros-backend` et lite startskript; selve programmet ligger i `/usr/lib/chloros/backend/`)

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
* ❓ **FAQ**: [Ofte stilte spørsmål](faq.md)

***

## Programvareoppdateringer

Chloros søker etter oppdateringer, varsler når en ny versjon er tilgjengelig og lenker til denne nedlastingssiden — du oppdaterer ved å kjøre det nye signerte installasjonsprogrammet. Innstillingene, prosjektene og påloggingsopplysningene dine beholdes etter oppdateringer. På Linux og Jetson søker `chloros-cli update` etter en nyere versjon og tilbyr å laste ned og installere den tilsvarende `.deb` (denne kommandoen er kun tilgjengelig på Linux).

***

## Endringslogg**Versjon 1.3.0 (Siste, 11. november 2026)**— se**Hva er nytt i Chloros

1.3.0** på siden [Kom i gang](./) for den fullstendige listen.**Versjon 1.2.0**— se**Hva er nytt i Chloros 1.2.0** på siden [Kom i gang](./).

<details>

<summary>Versjon 1.0.5</summary>

**Utgivelsesdato: 10. februar 2026**

**Nye funksjoner**

* **Teksturbevisst debayer-metode \[Kun Chloros+] -** Teksturbevisst bruker en kantbevisst debayer av høy kvalitet kombinert med en AI/ML-støyreduksjonsmodell som fjerner nesten all debayer-støy.
* **Støtte for T4P-kalibreringsmål**
* **Raskere Chloros+ GPU-behandling, bedre minnestyring**

**Feilrettinger**

* Helt nytt brukergrensesnitt (GUI), bør nå fungere på alle Windows-datamaskiner.

</details>

<details>

<summary>Versjon 1.0.4</summary>

**Utgivelsesdato: 5. januar 2026**

**Nye funksjoner**

* **Veksling mellom bilde og metadata**: Lagt til veksling i filbrowseren for å vise metadata for det valgte bildet i en tabell i stedet for i bildeoversikten
* **Zoomglidebryter for bildeoversikt**: Ny glidebryter i brukergrensesnittet for å justere størrelsen på miniatyrbildene (støtter også CTRL + mushjul)
* **Eksportknapper for bildegitter**: Knapper i øverste rad for å bytte miniatyrbilder fra JPG til bearbeidede eksporter (Targets, Reflectance, Index, LUT)
* **Kartfanen**: Nytt interaktivt 2D-kart som viser GPS-posisjonsmarkører for bildene
  * Støtter Google Maps og ESRI-kartfliser (velger automatisk den beste flisetjenesten basert på tilgjengelighet på zoomnivå)
  * Forhåndsvisning av miniatyrbilder ved å holde musepekeren over kartmarkører

**Feilrettinger**

* Forbedret støtte for installasjon av Chloros på datamaskiner med andre språk enn engelsk

</details>

<details>

<summary>Versjon 1.0.3</summary>

**Utgivelsesdato: 20. desember 2025**

**Nye funksjoner**

* Første lansering

**Forbedringer**

* Første lansering

**Feilrettinger**

* Første lansering

**Kjente problemer**

* Første lansering

</details>

***

## Lisensavtale**Proprietær programvare** – Copyright (c) 2026 MAPIR Inc.

Uautorisert bruk, distribusjon eller modifisering er forbudt.

**Gratisversjon**: Tilgjengelig for personlig og kommersiell bruk med begrensede funksjoner**Chloros+**: Abonnementsbasert lisens for avanserte funksjoner og kommersiell bruk
