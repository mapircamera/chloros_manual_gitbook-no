# Fullføre behandlingen

Når Chloros har fullført behandlingen, er det på tide å gjennomgå resultatene, kontrollere utskriftskvaliteten og gjøre de behandlede bildene klare til bruk i arbeidsflyten din. Denne siden veileder deg gjennom de siste trinnene og de neste handlingene.

## Indikasjon på fullført behandling

Når behandlingen er fullført, vil du se flere indikatorer:

* ✅ **Fremdriftslinje**: Når 100 % fullført
* ✅ **Feilsøkingslogg**: Viser meldingen «Behandlingen er fullført»
* ✅ **Startknapp**: Blir aktivert igjen (klar for neste behandlingsomgang)
* ✅ **Utdatafiler**: Alle behandlede bilder er lagret i undermappen for kameramodellen***

## Finne de behandlede bildene

### Åpne utdatamappen

1. Klikk på **Hovedmeny** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> (øverst til venstre)
2. Velg **«Åpne prosjektmappe»**

3. Filutforskeren åpnes i prosjektkatalogen
4. Finn prosjektet ditt ved å søke etter navnet

***

## Gjennomgå behandlede bilder

### Rask forhåndsvisning i filutforskeren

**Windows innebygd forhåndsvisning:**

1. Naviger til undermappen for kameramodellen
2. Velg en bildefil
3. Forhåndsvisningen vises i Windows Explorer-forhåndsvisningsruten
4. Bruk piltastene til å bla gjennom bildene

### Forhåndsvisning i eksterne bildevisere

**Anbefalte visere:*** **QGIS** – Gratis GIS-programvare (best for georeferert multispektral analyse)
* **IrfanView** – Rask, lett bildeviser (støtter TIFF)
* **Adobe Photoshop** – Profesjonell redigering (støtter TIFF)
* **GIMP** – Gratis alternativ til Photoshop
* **Windows Photos** – Grunnleggende visning (støtter kanskje ikke 16-biters TIFF)

### Forhåndsvisning i Chloros Image Viewer

Bruk Chloross innebygde Image Viewer for avansert visualisering:

1. Klikk på et miniatyrbilde i filbrowseren
2. Bildet åpnes i hovedforhåndsvisningsområdet
3. Klikk på **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> i venstre sidefelt
4. Bruk [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) for interaktiv analyse

Se [Image Viewer](../image-viewer-gui/opening-an-image-full-screen.md) for detaljerte instruksjoner.

***

## Gjennomgang av feilsøkingsloggen

### Se etter advarsler eller feil

1. Åpne **Feilsøkingslogg** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> fanen
2. Bla gjennom meldingene
3. Se etter gule advarsler eller røde feil
4. Gjennomgå eventuelle problemer som er notert
5. Kontakt MAPIR-support for hjelp

### Lagre loggen

For å beholde en oversikt over behandlingen eller for å sende den til MAPIR-support:

1. Klikk på **&quot;Kopier&quot;**- eller**&quot;Last ned&quot;**-knappen
2. Lagre som tekstfil i prosjektmappen
3. Legg den ved prosjektdokumentasjonen
4. Send den til MAPIR-kundestøtte hvis det oppstår problemer

***

## Vanlige utdataproblemer og løsninger

### Problem: Manglende utdatafiler

**Mulige årsaker:**

* Filene oppfylte ikke behandlingskriteriene
* Bilder kun for mål (ekskludert fra eksport)
* Diskplassen tok slutt under eksport
* Filkorrupsjon under behandling

**Løsninger:**

1. Sjekk feillogg for hopp/feilmeldinger
2. Kontroller at det var tilstrekkelig med diskplass
3. Tell filer: Bør stemme overens med (opprinnelig antall – antall mål) × (indekser + 1)
4. Importer og behandle eventuelle manglende filer på nytt

### Problem: Mørke eller lyse kanter (vignettering fortsatt synlig)

**Mulige årsaker:**

* Vignettkorrigering deaktivert
* Kamera/objektiv finnes ikke i Chloros-profildatabasen
* Ekstrem vignettering utenfor korrigeringskapasiteten

**Løsninger:**

1. Kontroller at vignettkorrigering var aktivert i prosjektinnstillingene
2. Kontroller at kameramodellen er riktig gjenkjent
3. Kontakt MAPIR-support hvis vignetteringen vedvarer

### Problem: Feil farger eller verdier

**Mulige årsaker:**

* Ingen kalibreringsmål gjenkjent
* Feil kalibreringsmålmodell valgt
* Refleksjonskalibrering deaktivert
* Målbilder av dårlig kvalitet

**Løsninger:**

1. Kontroller at reflektanskalibrering var aktivert
2. Sjekk «Mål funnet»-meldinger i feilsøkingsloggen
3. Gjennomgå kvaliteten på målbildene
4. Behandle på nytt med riktige mål markert

### Problem: NDVI-verdiene virker feil

**Forventede NDVI-områder:*** **Vann, steiner, jord**: -0,1 til 0,2
* **Sparsom/usunn vegetasjon**: 0,2 til 0,4
* **Moderat vegetasjon**: 0,4 til 0,6
* **Sunn, tett vegetasjon**: 0,6 til 0,9**Hvis verdiene ligger utenfor disse områdene:**

1. Kontroller at reflektanskalibrering ble brukt
2. Kontroller at lyssensorloggen ble inkludert
3. Sjekk at kalibreringsmålene ble oppdaget
4. Sørg for at riktig kameramodell ble oppdaget
5. Gjennomgå tidspunkt og forhold for opptak av målbildet

***

## Bruk av de behandlede bildene

### For fotogrammetri / oppretting av ortomosaikk

**Anbefalt arbeidsflyt:**

1.**Importer kalibrerte reflektansbilder** til fotogrammetriprogramvare:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Behold EXIF-metadata**: Sørg for at GPS-data bevares for geotagging
3. **Kalibrerte arbeidsflyter**: Bruk reflektansbilder for vitenskapelig nøyaktighet
4. **Behandle indeksmosaikker**: Lag NDVI ortomosaikker fra individuelle indeksbilder
5. **Eksporter georefererte GeoTIFF**: For bruk i GIS-applikasjoner

### For GIS-analyse

**Anbefalt arbeidsflyt:**

1.**Last inn i QGIS, ArcGIS eller lignende**

2.**Bruk 16-biters TIFF** refleksjonsbilder for multibåndsanalyse
3. **Bruk indeksbilder** (NDVI, NDRE) som ferdige vegetasjonslag
4. **Rasterkalkulator**: Kombiner bånd for tilpasset analyse
5. **Eksporter**: Lag klassifiseringskart, endringsdeteksjon, kart over vegetasjonens helse

### For direkte analyse / rapportering

**Anbefalt arbeidsflyt:**

1.**Bruk indeksbilder med LUT-farger** for visuelle rapporter
2. **Hent ut statistikk**: Gjennomsnittlig NDVI per felt/tomt
3. **Tidsserier**: Sammenlign indekser på tvers av flere økter
4. **Generer rapporter**: Inkluder kart, statistikk og visualiseringer***

## Arkivering og sikkerhetskopiering

### Anbefalt sikkerhetskopieringsstrategi

**Hva du bør lagre:*** ✅ **Originale RAW/JPG-bilder** – Arkiver på separat stasjon/i skyen
* ✅ **Bearbeidede resultater** – Behold kalibrerte bilder og indekser
* ✅ **Prosjektfil** – Inneholder alle innstillinger for ombearbeiding om nødvendig
* ✅ **Feilsøkingslogg** – Dokumenterer detaljer om behandlingen
* ✅ **Kalibreringsmålbilder** – For verifisering og ombearbeiding**Anbefalinger for lagring:*** **Umiddelbar sikkerhetskopi**: Ekstern harddisk
* **Langtidsarkiv**: Skylagring (Google Drive, Dropbox osv.)
* **Kritiske data**: Oppbevar 2–3 kopier på forskjellige steder***

## Neste behandlingsrunder

### Gjenbruk av prosjektinnstillinger

Hvis du skal behandle lignende datasett i fremtiden:

1. **Lagre prosjektmal** (hvis ikke allerede gjort)
2. **Opprett nytt prosjekt** ved hjelp av den lagrede malen
3. **Importer nye bilder**

4.**Behandle**med identiske innstillinger for konsistens

### Batchbehandling av flere økter

For flere økter/datasett:**Alternativ 1: GUI – Flere prosjekter**

* Opprett et eget prosjekt for hver økt
* Bruk konsistente malinnstillinger
* Behandle ett om gangen

**Alternativ 2: Chloros CLI (kun Chloros+)**

* Automatiser batchbehandling
* Behandle flere mapper med skript
* Se [CLI-dokumentasjon](../CLI.md)

**Alternativ 3: Python SDK (kun Chloros+)**

* Programmatisk kontroll
* Integrasjon med analysepipelines
* Se [API-dokumentasjon](../api-python-sdk.md)

***

## Feilsøking etterbehandling

### Ny behandling med andre innstillinger

Hvis resultatene ikke er tilfredsstillende:

1. Behold originalbildene (slett dem aldri)
2. Åpne det samme prosjektet i Chloros
3. Juster innstillingene i prosjektinnstillingspanelet
4. Behandle på nytt – utdataene vil overskrive tidligere resultater

### Behandling av delmengde av bilder

For å behandle kun bestemte bilder på nytt:

1. Opprett et nytt prosjekt
2. Importer kun bildene som trenger ny behandling
3. Bruk samme innstillingsmal
4. Behandle et mindre datasett

### Få hjelp

Hvis du støter på problemer:

* 📧 **E-post**: info@mapir.camera (inkluder feillogg)
* 🌐 **Support**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ**: [Ofte stilte spørsmål](../faq.md)
* 📖 **Dokumentasjon**: [Chloros-håndbok](../)***

## Oppsummering: Fullført arbeidsflyt

Du har nå fullført hele Chloros-behandlingsarbeidsflyten:

1. ✅ **Opprettet prosjekt** – Se [Prosjekter](../projects.md)
2. ✅ **Lagt til filer** – Se [Legge til filer](adding-files-to-a-project.md)
3. ✅ **Justert innstillinger** – Se [Justere prosjektinnstillinger](adjusting-project-settings.md)
4. ✅ **Merkede mål** – Se [Velge målbilder](choosing-target-images.md)
5. ✅ **Startet behandling** - Se [Starte behandlingen](starting-the-processing.md)
6. ✅ **Overvåket fremdrift** - Se [Overvåke behandlingen](monitoring-the-processing.md)
7. ✅ **Gjennomgått resultater** - Denne siden**Dine kalibrerte, refleksjonskorrigerte multispektrale bilder er klare for analyse!**

***

## Tilleggsressurser

### Avanserte funksjoner

* [**Bildeviser**](../image-viewer-gui/opening-an-image-full-screen.md) - Interaktiv visualisering og analyse
* [**Indeks/LUT-sandkasse**](../image-viewer-gui/index-lut-sandbox.md) – Tilpasset indekstesting
* [**Multispektrale indeksformler**](../project-settings/multispectral-index-formulas.md) – Komplett indeksreferanse

### Automatisering og integrasjon

* [**CLI-dokumentasjon**](../CLI.md) – Batchbehandling fra kommandolinjen
* [**Python SDK**](../api-python-sdk.md) – Programmatisk automatisering
* [**Chloros+ Funksjoner**](../#chloros) – Avanserte behandlingsfunksjoner

### Støtte og opplæring

* [**FAQ**](../faq.md) – Svar på vanlige spørsmål
* [**Kalibreringsmål**](../calibration-targets.md) – Forståelse av reflektanskalibrering
* [**Støttede kameraer**](../supported-cameras.md) – Kompatibel maskinvare
