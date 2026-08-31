# Fullføre behandlingen

Når Chloros har fullført behandlingen, er det på tide å gjennomgå resultatene, kontrollere utskriftskvaliteten og gjøre de behandlede bildene klare til bruk i arbeidsflyten din. Denne siden veileder deg gjennom de siste trinnene og de neste handlingene.

## Indikasjon på fullført behandling

Når behandlingen er fullført, vil du se flere indikatorer:

* ✅ **Fremdriftslinje**: Når 100 % fullført
* ✅ **Feilsøkingslogg**: Viser de siste `[RUN-SUMMARY]`-linjene med antall (bilder, kameragrupper, mål, kalibrerte bilder, filer som er skrevet)
* ✅ **Startknapp**: Blir aktivert igjen (klar for neste behandlingskjøring)
* ✅ **Utdatafiler**: Alle behandlede bilder er lagret i prosjektets utdatatreet (nedenfor)

{% hint style="warning" %}
**En kjøring som ikke skriver ut noen bilder, regnes som mislykket.** Hvis du har forespurt bildeprodukter og kjøringen ikke skrev ut noen, rapporterer Chloros dette som en feil — `[RUN-SUMMARY]` gir i loggnavnet et hint om den sannsynlige årsaken (ingenting importert, ingen mål oppdaget, eller at alle forespurte produkter ble hoppet over som ikke relevante). Den tilsvarende CLI-kjøringen avsluttes med en verdi ulik null. En bevisst kjøring med kun metadata (alle eksportprodukter deaktivert, ingen indekser) regnes fortsatt som vellykket. Se [CLI-referansen](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## Finne de behandlede bildene dine

### Åpne utdatamappen

1. Klikk på **Hovedmeny**-ikonet <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> (øverst til venstre)
2. Velg **«Åpne prosjektmappe»**

3. Filutforskeren åpnes i prosjektkatalogen
4. Finn prosjektet ditt ved å søke etter navnet

### Utdatatreet

Produktene lagres **i prosjektmappen, gruppert etter kamera og deretter etter filformat**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per selected index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

* **Kameramappe**: `LATT-<sensor>-<lens>-F<filter>` for LATTICE (samsvarer med opptakets EXIF-data: `Model`), `<model>_<filter>` for Survey3 (f.eks. `Survey3N_RGN`). To kameraer som deler sensor og filter, men har forskjellige objektiver, har separate trær — vignettering, synsfelt og forvrengning er forskjellige.
* **Formatmappe**: følger innstillingen for eksportformat — `tiff16`, `tiff8`, `png8`, `jpg8`, eller `tiff32` for TIFF (32-bit, prosent). Radiance er alltid float32 og havner alltid under `tiff32`.
* **Produktmapper**:
  * `Reflectance_Calibrated_Images/` — kalibrert reflektans
  * `Debayered_Images/` — lineær debayering (LATTICE)
  * `Preview_Images/` — forhåndsvisning på skjerm (LATTICE)
  * `Radiance_Images/` — spektral strålingsintensitet i float32, W/m²/sr/nm (LATTICE multispektral)
  * `Vignette_Corrected_Images/` **eller** `Sensor_Response_Images/` — den ukalibrerte reserveverdien for bilder uten refleksjonsreferanse; nøyaktig én av de to finnes per kjøring, valgt av innstillingen for Vignette-korreksjon
  * `<INDEX>_Index_Images/` — én mappe per valgt indeks (f.eks. `NDVI_Index_Images`)

{% hint style="info" %}
**Hvert eksportert produkt beholder navnet på KILDE-filen.**En radiance-eksport av `capture_..._raw.tif` heter fortsatt `capture_..._raw.tif` — den ligger bare i `tiff32/Radiance_Images/`.**Mappen identifiserer produktet, ikke filnavnet**, så hvis du søker etter `*radiance*.tif`, finner du ingenting; søk i stedet i katalogen.
{% endhint %}



<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-… camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images) and tiff32 (Radiance_Images) subfolders visible -->### Hvor mange filer skal det være?

Ikke regn ut antallet ved hjelp av en formel – antall utdata avhenger av hvilke produkter som er aktivert og hvilke som gjelder for hvert kamera (f.eks. får RGB-kameraer ingen radianse/reflektans). Det offisielle antallet står i loggen: den siste `[RUN-SUMMARY]`-linjen rapporterer nøyaktig hvor mange filer som ble skrevet, og hintlinjer forklarer alt som ble hoppet over.

***

## Gjennomgang av behandlede bilder

### Rask forhåndsvisning i Filutforsker

**Windows innebygd forhåndsvisning:**

1. Naviger til en produktmappe (f.eks. `tiff16/Reflectance_Calibrated_Images/`)
2. Velg en bildefil
3. Forhåndsvisningen vises i forhåndsvisningsruten i Windows Utforsker
4. Bruk piltastene til å bla gjennom bildene

### Forhåndsvisning i eksterne bildevisere

**Anbefalte visere:*** **QGIS** – Gratis GIS-programvare (best egnet for georeferert multispektral analyse)
* **IrfanView** – Rask, lettvektig bildeviser (støtter TIFF)
* **Adobe Photoshop** – Profesjonell redigering (støtter TIFF)
* **GIMP** – Gratis alternativ til Photoshop
* **Windows Photos** – Grunnleggende visning (støtter kanskje ikke 16-biters TIFF)

### Forhåndsvisning i Chloros-bildeviser

Bruk Chloross innebygde bildeviser for avansert visualisering:

1. Klikk på et miniatyrbilde i filbrowseren
2. Bildet åpnes i hovedforhåndsvisningsområdet
3. Klikk på **Bildeviser**-<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">-fanen i venstre sidefelt
4. Bruk [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) for interaktiv analyse

Se [Bildeviser](../image-viewer-gui/opening-an-image-full-screen.md) for detaljerte instruksjoner.

***

## Avlesning av refleksjonsverdier for piksler (GIS / Pix4D / Skript)

Refleksjonsverdien lagres som et heltall (DN), og **den DN-verdien som tilsvarer ρ = 1,0, avhenger av kildekameraet**:

| Kilde          | ρ = 1,0 er | Hvordan se det                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (margin opp til ρ 2,0) | XMP-taggen `Chloros:PixelScale=32768` er stemplet på filen |
| Survey3         | **65535** (avskåret ved ρ 1,0)     | Ingen `Chloros:*` XMP-tagger — fraværet er selve signalet |

**Les `Chloros:PixelScale`-taggen og del med den** i stedet for å anta en generell verdi på 65535 — å dele LATTICE-reflektansen med 65535 halverer stille hver verdi. Ett spesielt tilfelle har ingen skalering etter design: en opptak fra en 8-bit-kilde som er skrevet ut som 8-bit-utdata, blir beskåret, ikke omskalert, og får bevisst ingen skaleringstag — eksporter på nytt i 16-bit eller 32-bit i stedet for å dele den. Se [Utgangsbildeformater](../output-image-formats.md) for hele historien.***

## Metadata som overføres til eksportene

Hvert produkt beholder kildeopptakets **GPS-blokk**og dets**EXIF-sub-IFD**, slik at en
eksport inneholder `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` og
`CameraSerialNumber` samt georeferansen.

{% hint style="warning" %}
**Hvis en ortomosaikk vises i en absurd målestokk, må du først sjekke `FocalLength`.**
Pix4D beregner avstanden mellom bildepunkter ut fra brennvidde pluss høyde. Uten denne taggen
faller den tilbake til en helt feil skala — på en målt flytur med 49 opptak ble en 411 m × 160 m
appelsinlund rekonstruert som 47,8 km × 13 km, noe som resulterte i en ortomosaikk på 455 megapiksler med hovedsakelig
tomt rom. Treg flislegging og en uventet stor fil er symptomer på dette, ikke separate
problemer.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Ikke *alle* tagger kopieres. IFD0s strukturelle tagger blir bevisst utelatt (å kopiere
dem ødelegger LATTICE-utdataene), og `ExifImageWidth` / `ExifImageHeight` er ekskludert
fordi de beskriver den opprinnelige opptaket – en eksport som har blitt endret i størrelse, ville ellers
angitt dimensjoner som står i strid med rasteret selv.

***

## Gjennomgang av feilsøkingsloggen

### Se etter advarsler eller feil

1. Åpne **Feilsøkingslogg**-<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">-fanen
2. Bla gjennom meldingene
3. Se etter gule advarsler eller røde feil
4. Les `[RUN-SUMMARY]`-linjene og eventuelle tips
5. Kontakt MAPIR-kundestøtte for hjelp

### Lagre loggen

For å beholde en oversikt over behandlingen eller for å sende den til MAPIR-kundestøtte:

1. Klikk på **«Kopier»**- eller**«Last ned»**-knappen
2. Lagre som tekstfil i prosjektmappen
3. Legg den ved prosjektdokumentasjonen
4. Send den til MAPIR-kundestøtte hvis det oppstår problemer

***

## Vanlige problemer med utdata og løsninger

### Problem: Manglende utdatafiler

**Mulige årsaker:**

* Produktet gjelder ikke for det aktuelle kameraet (f.eks. radianse/reflektans for RGB-kameraer – dette fremgår av loggen)
* En nødvendig referanse manglet (f.eks. reflektans uten mål og uten `.daq` nedadgående stråling)
* Avkryssingsruten for eksport av produktet var deaktivert i prosjektinnstillingene
* Det ble tomt for diskplass under eksporten

**Løsninger:**

1. Sjekk `[RUN-SUMMARY]`-hint og `[EXPORT-CHECK]`-linjene i feilsøkingsloggen – de forklarer utelatelser per kamera
2. Kontroller avmerkingsboksene for eksportprodukter i [Prosjektinnstillinger](adjusting-project-settings.md)
3. Kontroller at det var tilstrekkelig med diskplass
4. Kjør prosessen på nytt etter å ha løst årsaken

### Problem: Mørke eller lyse kanter (vignettering fortsatt synlig)

**Mulige årsaker:**

* Vignettkorrigering deaktivert
* Kamera/objektiv finnes ikke i Chloros-profildatabasen
* Ekstrem vignettering som overstiger korrigeringskapasiteten

**Løsninger:**

1. Kontroller at vignettkorrigering var aktivert i Prosjektinnstillinger
2. Kontroller at kameramodellen er riktig gjenkjent
3. Kontakt MAPIR-support hvis vignetteringen vedvarer

### Problem: Feil farger eller verdier

**Mulige årsaker:**

* Ingen kalibreringsmål oppdaget
* Feil modell for kalibreringsmål valgt
* Refleksjonskalibrering deaktivert
* Målbilder av dårlig kvalitet

**Løsninger:**

1. Kontroller at reflektanskalibrering var aktivert
2. Sjekk «Mål funnet»-meldingene i feilsøkingsloggen
3. Gjennomgå kvaliteten på målbildene
4. Behandle på nytt med riktige mål merket

### Problem: NDVI-verdiene virker feil

**Forventede NDVI-verdier:*** **Vann, stein, jord**: -0,1 til 0,2
* **Sparsom/usunn vegetasjon**: 0,2 til 0,4
* **Moderat vegetasjon**: 0,4 til 0,6
* **Sunn, tett vegetasjon**: 0,6 til 0,9**Hvis verdiene ligger utenfor disse intervallene:**

1. Kontroller at reflektanskalisering ble brukt
2. Kontroller at lyssensorloggen ble inkludert
3. Kontroller at kalibreringsmålene ble oppdaget
4. Sørg for at riktig kameramodell ble oppdaget
5. Gjennomgå tidspunkt og forhold for opptak av målbilde
6. Hvis du beregner indeksene selv fra reflektansfiler, må du bekrefte at du har dividert med filens `Chloros:PixelScale` (se ovenfor)

***

## Bruk av de bearbeidede bildene

### For fotogrammetri / oppretting av ortomosaikk

**Anbefalt arbeidsflyt:**

1.**Importer kalibrerte reflektansbilder** til fotogrammetriprogramvare:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Behold EXIF-metadata**: Sørg for at GPS-data bevares for geotagging
3. **Kalibrerte arbeidsflyter**: Bruk reflektansbilder for vitenskapelig nøyaktighet — LATTICE-reflektans inneholder XMP-kalibreringstagger som Pix4D leser
4. **Behandle indeksmosaikker**: Lag ortomosaikker med filnavnet NDVI fra individuelle indeksbilder
5. **Eksporter georeferert GeoTIFF**: For bruk i GIS-applikasjoner

### For GIS-analyse

**Anbefalt arbeidsflyt:**

1.**Last inn i QGIS, ArcGIS eller lignende**

2.**Bruk 16-bits TIFF**-refleksjonsbilder til multibåndsanalyse (del med filens `Chloros:PixelScale`)
3. **Bruk indeksbilder** (NDVI, NDRE) som ferdige vegetasjonslag
4. **Rasterkalkulator**: Kombiner bånd for tilpasset analyse
5. **Eksporter**: Lag klassifiseringskart, endringsdeteksjon og kart over vegetasjonens helsetilstand

### For direkte analyse/rapportering

**Anbefalt arbeidsflyt:**

1.**Bruk indeksbilder med LUT-farger** til visuelle rapporter
2. **Hent ut statistikk**: Gjennomsnittlig NDVI per felt/parsell
3. **Tidsserier**: Sammenlign indekser på tvers av flere måleomganger
4. **Generer rapporter**: Inkluder kart, statistikk og visualiseringer***

## Arkivering og sikkerhetskopiering

### Anbefalt sikkerhetskopieringsstrategi

**Hva du bør lagre:*** ✅ **Originale RAW/JPG-bilder eller LATTICE-råopptak** – Arkiver på separat harddisk/i skyen; rådataene er kilden til behandlingsprosessen, og alt annet kan gjenskapes ut fra disse
* ✅ **`.daq` / `.csv` lyssensorfiler** – Nødvendig for å beregne reflektans på nytt senere
* ✅ **Bearbeidede resultater** – Behold kalibrerte bilder og indekser
* ✅ **Prosjektmappe** (`project.json` og tilhørende filer) – Inneholder alle innstillinger for ny behandling ved behov
* ✅ **Feilsøkingslogg** – Dokumenterer detaljer om behandlingen
* ✅ **Kalibreringsmålbilder** – For verifisering og ny behandling**Anbefalinger for lagring:*** **Umiddelbar sikkerhetskopiering**: Ekstern harddisk
* **Langtidsarkiv**: Skylagring (Google Drive, Dropbox osv.)
* **Kritiske data**: Oppbevar 2–3 kopier på forskjellige steder***

## Neste behandlingsrunder

### Gjenbruk av prosjektinnstillinger

Hvis du skal behandle lignende datasett i fremtiden:

1. **Lagre prosjektmal** (hvis ikke allerede gjort)
2. **Opprett nytt prosjekt** ved hjelp av den lagrede malen
3. **Importer nye bilder**

4.**Behandle**med identiske innstillinger for å sikre konsistens

### Batchbehandling av flere økter

For flere økter/datasett:**Alternativ 1: GUI – Flere prosjekter**

* Opprett et eget prosjekt for hver økt
* Bruk konsistente malinnstillinger
* Behandle ett om gangen

**Alternativ 2: Chloros CLI (kun Chloros+)**

* Automatiser batchbehandling
* Behandle flere mapper med skript
* Se [CLI-dokumentasjon](../CLI.md) og [CLI-referanse](../reference/cli-reference.md)

**Alternativ 3: Python SDK (kun Chloros+)**

* Programmatisk kontroll
* Integrasjon med analysepipelines
* Se [API-dokumentasjon](../api-python-sdk.md) og [SDK-referanse](../reference/sdk-reference.md)

***

## Feilsøking av etterbehandling

### Ny behandling med andre innstillinger

Hvis resultatene ikke er tilfredsstillende:

1. Behold de originale bildene (slett dem aldri)
2. Åpne det samme prosjektet i Chloros
3. Juster innstillingene i panelet «Prosjektinnstillinger»
4. Behandle på nytt — resultatene lagres i de samme produktmappene, så filer med samme navn fra forrige kjøring blir erstattet

### Behandling av en delmengde av bilder

For å behandle kun bestemte bilder på nytt:

1. Opprett et nytt prosjekt
2. Importer kun bildene som trenger ny behandling
3. Bruk samme innstillingsmal
4. Behandle et mindre datasett

### Få hjelp

Hvis du støter på problemer:

* 📧 **E-post**: info@mapir.camera (legg ved feillogg)
* 🌐 **Support**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ**: [Ofte stilte spørsmål](../faq.md)
* 📖 **Dokumentasjon**: [Chloros-håndbok](../)***

## Oppsummering: Fullstendig arbeidsflyt

Du har nå fullført hele behandlingsarbeidsflyten for Chloros:

1. ✅ **Opprettet prosjekt** – Se [Prosjekter](../projects.md)
2. ✅ **Lagt til filer** – Se [Legge til filer](adding-files-to-a-project.md)
3. ✅ **Justert innstillinger** – Se [Justere prosjektinnstillinger](adjusting-project-settings.md)
4. ✅ **Merkte mål** – Se [Velge målbilder](choosing-target-images.md)
5. ✅ **Startet behandling** – Se [Starte behandlingen](starting-the-processing.md)
6. ✅ **Overvåket fremdrift** – Se [Overvåke behandlingen](monitoring-the-processing.md)
7. ✅ **Gjennomgåtte resultater** – Denne siden**De kalibrerte, refleksjonskorrigerte multispektrale bildene dine er klare for analyse!**

***

## Ytterligere ressurser

### Avanserte funksjoner

* [**Bildevisning**](../image-viewer-gui/opening-an-image-full-screen.md) – Interaktiv visualisering og analyse
* [**Indeks/LUT-sandkasse**](../image-viewer-gui/index-lut-sandbox.md) – Testing av tilpassede indekser
* [**Multispektrale indeksformler**](../project-settings/multispectral-index-formulas.md) – Fullstendig indeksreferanse

### Automatisering og integrasjon

* [**CLI-dokumentasjon**](../CLI.md) - Batchbehandling via kommandolinjen
* [**Python SDK**](../api-python-sdk.md) – Programmatisk automatisering
* [**Chloros+ Funksjoner**](../#chloros) – Avanserte behandlingsfunksjoner

### Støtte og opplæring

* [**Ofte stilte spørsmål**](../faq.md) - Svar på vanlige spørsmål
* [**Kalibreringsmål**](../calibration-targets.md) - Forståelse av reflektanskalibrering
* [**Støttede kameraer**](../supported-cameras.md) - Kompatibel maskinvare
