# Fullføre behandlingen

Når Chloros har fullført behandlingen, er det på tide å gjennomgå resultatene, kontrollere utskriftskvaliteten og forberede de behandlede bildene for bruk i arbeidsflyten. Denne siden veileder deg gjennom de siste trinnene og neste handlinger.

## Indikasjon på at behandlingen er fullført

Når behandlingen er fullført, vil du se flere indikatorer:

* ✅ **Fremdriftslinje**: Når 100 % fullført
* ✅ **Feilsøkingslogg**: Viser meldingen «Behandling fullført»
* ✅ **Startknapp**: Blir aktivert igjen (klar for neste behandlingskjøring)
* ✅ **Utdatafiler**: Alle behandlede bilder lagres i undermappen for kameramodellen

***

## Finne de behandlede bildene dine

### Åpne utdatamappen

1. Klikk på **Hovedmeny** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> (øverst til venstre)
2. Velg **«Åpne prosjektmappe»**
3. Filutforskeren åpnes i prosjektkatalogen
4. Finn prosjektet ditt etter navn

***

## Gjennomgå behandlede bilder

### Rask forhåndsvisning i filutforskeren

**Windows innebygd forhåndsvisning:**

1. Naviger til undermappen for kameramodellen
2. Velg en bildefil
3. Forhåndsvisningen vises i Windows Explorer-forhåndsvisningsruten
4. Bruk piltastene til å bla gjennom bildene

### Forhåndsvisning i eksterne bildevisere

**Anbefalte visere:**

* **QGIS** – Gratis GIS-programvare (best for georeferert multispektral analyse)
* **IrfanView** – Rask, lettvekts bildeviser (støtter TIFF)
* **Adobe Photoshop** – Profesjonell redigering (støtter TIFF)
* **GIMP** – Gratis alternativ til Photoshop
* **Windows Photos** – Grunnleggende visning (støtter kanskje ikke 16-biters TIFF)

### Forhåndsvisning i Chloros Image Viewer

Bruk Chloross innebygde Image Viewer for avansert visualisering:

1. Klikk på et miniatyrbilde i filbrowseren.
2. Bildet åpnes i hovedforhåndsvisningsområdet.
3. Klikk på **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> -fanen i venstre sidefelt.
4. Bruk [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) for interaktiv analyse.

Se [Image Viewer](../image-viewer-gui/opening-an-image-full-screen.md) for detaljerte instruksjoner.

***

## Gjennomgang av feilsøkingsloggen

### Se etter advarsler eller feil

1. Åpne **Feilsøkingslogg** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> -fanen
2. Bla gjennom meldingene
3. Se etter gule advarsler eller røde feilmeldinger
4. Gjennomgå eventuelle problemer som er notert
5. Kontakt MAPIR-kundestøtte for hjelp

### Lagre loggen

For å beholde en oversikt over behandlingen eller for å sende den til MAPIR-kundestøtte:

1. Klikk på **«Kopier»** eller **«Last ned»**-knappen
2. Lagre som tekstfil i prosjektmappen
3. Legg ved prosjektdokumentasjonen
4. Send til MAPIR-kundestøtte hvis det oppstår problemer

***

## Vanlige utdataproblemer og løsninger

### Problem: Manglende utdatafiler

**Mulige årsaker:**

* Filene oppfylte ikke behandlingskriteriene
* Kun målbilder (ekskludert fra eksport)
* Diskplassen ble full under eksporten
* Filkorrupsjon under behandlingen

**Løsninger:**

1. Sjekk feilsøkingsloggen for hopp/feilmeldinger
2. Kontroller at det var tilstrekkelig med diskplass
3. Tell filene: Skal stemme overens med (opprinnelig antall – målantall) × (indekser + 1)
4. Importer og behandle manglende filer på nytt

### Problem: Mørke eller lyse kanter (vignettering fortsatt synlig)

**Mulige årsaker:**

* Vignettkorreksjon deaktivert
* Kamera/objektiv ikke i Chloros-profildatabasen
* Ekstrem vignettering utover korreksjonskapasiteten

**Løsninger:**

1. Kontroller at vignettkorrigering er aktivert i prosjektinnstillingene.
2. Kontroller at kameramodellen er riktig oppdaget.
3. Kontakt MAPIR-kundestøtte hvis vignettering vedvarer.

### Problem: Feil farger eller verdier

**Mulige årsaker:**

* Ingen kalibreringsmål oppdaget.
* Feil kalibreringsmålmodell valgt.
* Refleksjonskalibrering deaktivert.
* Målbilder av dårlig kvalitet.

**Løsninger:**

1. Kontroller at refleksjonskalibrering er aktivert.
2. Kontroller meldingene «Mål funnet» i feilsøkingsloggen.
3. Kontroller kvaliteten på målbildene.
4. Behandle på nytt med riktige mål merket.

### Problem: NDVI-verdiene virker feil.

**Forventede NDVI-områder:**

* **Vann, steiner, jord**: -0,1 til 0,2
* **Sparsom/usunn vegetasjon**: 0,2 til 0,4
* **Moderat vegetasjon**: 0,4 til 0,6
* **Sunn, tett vegetasjon**: 0,6 til 0,9

**Hvis verdiene ligger utenfor disse områdene:**

1. Kontroller at refleksjonskalibrering ble brukt.
2. Kontroller at lyssensorloggen ble inkludert.
3. Kontroller at kalibreringsmålene ble oppdaget.
4. Kontroller at riktig kameramodell ble oppdaget.
5. Gjennomgå tidspunktet og forholdene for målbildet.

***

## Bruke de behandlede bildene

### For fotogrammetri/ortomosaikkoppretting

**Anbefalt arbeidsflyt:**

1. **Importer kalibrerte refleksjonsbilder** til fotogrammetriprogramvare:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Behold EXIF-metadata**: Sørg for at GPS-data bevares for geotagging
3. **Kalibrerte arbeidsflyter**: Bruk refleksjonsbilder for vitenskapelig nøyaktighet
4. **Behandle indeksmosaikker**: Lag NDVI ortomosaikker fra individuelle indeksbilder
5. **Eksporter georefererte GeoTIFF**: For bruk i GIS-applikasjoner

### For GIS-analyse

**Anbefalt arbeidsflyt:**

1. **Last inn i QGIS, ArcGIS eller lignende**
2. **Bruk 16-biters TIFF** refleksjonsbilder for multibåndanalyse
3. **Bruk indeksbilder** (NDVI, NDRE) som bruksklare vegetasjonslag
4. **Rasterkalkulator**: Kombiner bånd for tilpasset analyse
5. **Eksporter**: Lag klassifiseringskart, endringsdeteksjon, vegetasjonshelsekart

### For direkte analyse/rapportering

**Anbefalt arbeidsflyt:**

1. **Bruk indeksbilder med LUT-farger** for visuelle rapporter
2. **Ekstraher statistikk**: Gjennomsnittlig NDVI per felt/tomt
3. **Tidsserier**: Sammenlign indekser på tvers av flere økter
4. **Generer rapporter**: Inkluder kart, statistikk og visualiseringer

***

## Arkivering og sikkerhetskopiering

### Anbefalt sikkerhetskopieringsstrategi

**Hva du bør lagre:**

* ✅ **Originale RAW/JPG-bilder** – Arkiver på separat stasjon/sky
* ✅ **Behandlede resultater** – Oppbevar kalibrerte bilder og indekser
* ✅ **Prosjektfil** – Inneholder alle innstillinger for ombehandling om nødvendig
* ✅ **Feilsøkingslogg** – Dokumenterer behandlingsdetaljer
* ✅ **Kalibreringsmålbilder** – For verifisering og ny behandling

**Anbefalinger for lagring:**

* **Umiddelbar sikkerhetskopiering**: Ekstern harddisk
* **Langtidsarkiv**: Skylagring (Google Drive, Dropbox osv.)
* **Kritiske data**: Oppbevar 2–3 kopier på forskjellige steder

***

## Neste behandlingskjøringer

### Gjenbruk av prosjektinnstillinger

Hvis du skal behandle lignende datasett i fremtiden:

1. **Lagre prosjektmal** (hvis ikke allerede gjort)
2. **Opprett nytt prosjekt** ved hjelp av lagret mal
3. **Importer nye bilder**
4. **Behandle** med identiske innstillinger for konsistens

### Batchbehandling av flere økter

For flere økter/datasett:

**Alternativ 1: GUI – flere prosjekter**

* Opprett separate prosjekter for hver sesjon
* Bruk konsistente malinnstillinger
* Behandle én om gangen

**Alternativ 2: Chloros CLI (kun Chloros+)**

* Automatiser batchbehandling
* Behandle flere mapper med skript
* Se [CLI-dokumentasjon](../CLI.md)

**Alternativ 3: Python SDK (kun Chloros+)**

* Programmatisk kontroll
* Integrasjon med analysepipelines
* Se [API-dokumentasjon](../api-python-sdk.md)

***

## Feilsøking etter behandling

### Behandle på nytt med andre innstillinger

Hvis resultatene ikke er tilfredsstillende:

1. Behold originalbildene (slett dem aldri)
2. Åpne samme prosjekt i Chloros
3. Juster innstillingene i panelet Prosjektinnstillinger
4. Behandle på nytt – resultatene vil overskrive tidligere resultater

### Behandling av delmengde av bilder

For å behandle bare bestemte bilder på nytt:

1. Opprett nytt prosjekt
2. Importer bare bildene som må behandles på nytt
3. Bruk samme innstillingsmal
4. Behandle mindre datasett

### Få hjelp

Hvis du støter på problemer:

* 📧 **E-post**: info@mapir.camera (inkluder feilsøkingslogg)
* 🌐 **Support**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ**: [Ofte stilte spørsmål](../faq.md)
* 📖 **Dokumentasjon**: [Chloros-håndbok](../)

***

## Oppsummering: Fullfør arbeidsflyten

Du har nå fullført hele arbeidsflyten for Chloros-behandlingen:

1. ✅ **Opprettet prosjekt** – Se [Prosjekter](../projects.md)
2. ✅ **Lagt til filer** – Se [Legge til filer](adding-files-to-a-project.md)
3. ✅ **Justert innstillinger** – Se [Justere prosjektinnstillinger](adjusting-project-settings.md)
4. ✅ **Merkede mål** – Se [Velge målbilder](choosing-target-images.md)
5. ✅ **Startet behandling** – Se [Starte behandlingen](starting-the-processing.md)
6. ✅ **Overvåket fremgang** - Se [Overvåke behandlingen](monitoring-the-processing.md)
7. ✅ **Gjennomgått resultater** - Denne siden

**Dine kalibrerte, refleksjonskorrigerte multispektrale bilder er klare for analyse!**

***

## Tilleggsressurser

### Avanserte funksjoner

* [**Bildeselger**](../image-viewer-gui/opening-an-image-full-screen.md) – Interaktiv visualisering og analyse
* [**Indeks/LUT Sandbox**](../image-viewer-gui/index-lut-sandbox.md) – Tilpasset indeks testing
* [**Multispektrale indeksformler**](../project-settings/multispectral-index-formulas.md) – Komplett indeksreferanse

### Automatisering og integrering

* [**CLI-dokumentasjon**](../CLI.md) – Kommandolinjebasert batchbehandling
* [**Python SDK**](../api-python-sdk.md) – Programmatisk automatisering
* [**Chloros+ Funksjoner**](../#chloros) – Avanserte behandlingsfunksjoner

### Støtte og læring

* [**FAQ**](../faq.md) – Svar på vanlige spørsmål
* [**Kalibreringsmål**](../calibration-targets.md) – Forstå refleksjonskalibrering
* [**Støttede kameraer**](../supported-cameras.md) – Kompatibel maskinvare
