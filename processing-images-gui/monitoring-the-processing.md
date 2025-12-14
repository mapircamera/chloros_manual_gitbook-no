# Overvåking av behandlingen

Når behandlingen har startet, tilbyr Chloros flere måter å overvåke fremdriften, sjekke for problemer og forstå hva som skjer med datasettet ditt. Denne siden forklarer hvordan du kan spore behandlingen og tolke informasjonen som Chloros gir.

## Oversikt over fremdriftslinje

Fremdriftslinjen øverst i overskriften viser behandlingsstatus og fullføringsprosent i sanntid.

### Fremdriftslinje i gratis modus

For brukere uten Chloros+-lisens:

**2-trinns fremdriftsvisning:**

1. **Måldeteksjon** – Finne kalibreringsmål i bilder
2. **Behandling** – Bruke korreksjoner og eksportere

**Fremdriftslinjen viser:**

* Samlet fullføringsprosent (0–100 %)
* Navn på gjeldende trinn
* Enkel horisontal linjevisualisering

### Chloros+ fremdriftslinje

For brukere med Chloros+-lisens:

**4-trinns fremdriftsvisning:**

1. **Deteksjon** – Finne kalibreringsmål
2. **Analyse** – Undersøke bilder og forberede pipeline
3. **Kalibrering** – Bruke vignett- og refleksjonskorrigeringer
4. **Eksport** – Lagre behandlede filer

**Interaktive funksjoner:**

* **Hold musepekeren over** fremdriftslinjen for å se utvidet 4-trinns panel
* **Klikk** på fremdriftslinjen for å fryse/feste det utvidede panelet
* **Klikk igjen** for å frigjøre og skjule automatisk når musen forlater
* Hvert trinn viser individuell fremdrift (0-100 %)

***

## Forstå hvert behandlingsstadium

### Trinn 1: Deteksjon (målregistrering)

**Hva skjer:**

* Chloros skanner bilder merket med mål-avkrysningsboksen
* Datasynsalgoritmer identifiserer de 4 kalibreringspanelene
* Refleksjonsverdier hentet fra hvert panel
* Mål-tidsstempler registrert for riktig kalibreringsplanlegging

**Varighet:**

* Med merkede mål: 10-60 sekunder
* Uten merkede mål: 5-30+ minutter (skanner alle bilder)

**Fremdriftsindikator:**

* Oppdager: 0 % → 100 %
* Antall skannede bilder
* Antall funnet mål

**Hva du bør se etter:**

* Bør fullføres raskt hvis målene er riktig merket
* Hvis det tar for lang tid, er det mulig at målene ikke er merket
* Sjekk feilsøkingsloggen for meldinger om «Mål funnet»

### Trinn 2: Analyse

**Hva skjer:**

* Leser EXIF-metadata for bilder (tidsstempler, eksponeringsinnstillinger)
* Bestemmer kalibreringsstrategi basert på tidsstempler for mål
* Organiserer kø for bildebehandling
* Forbereder parallelle behandlingsarbeidere (kun Chloros+)

**Varighet:** 5–30 sekunder

**Fremdriftsindikator:**

* Analyserer: 0 % → 100 %
* Rask fase, fullføres vanligvis raskt

**Hva du bør se etter:**

* Bør gå jevnt uten pauser
* Advarsler om manglende metadata vises i feilsøkingsloggen

### Fase 3: Kalibrering

**Hva skjer:**

* **Debayering**: Konvertering av RAW Bayer-mønster til 3 kanaler
* **Vignettkorrigering**: Fjerner mørkningen i kantene av linsen
* **Refleksjonskalibrering**: Normaliserer med målverdier
* **Indeksberegning**: Beregner multispektrale indekser
* Behandler hvert bilde gjennom hele prosessen

**Varighet:** Størstedelen av den totale behandlingstiden (60–80 %)

**Fremdriftsindikator:**

* Kalibrering: 0 % → 100 %
* Gjeldende bilde behandles
* Fullførte bilder / Totalt antall bilder

**Behandlingsatferd:**

* **Fri modus**: Behandler ett bilde om gangen sekvensielt
* **Chloros+ modus**: Behandler opptil 16 bilder samtidig
* **GPU-akselerasjon**: Øker hastigheten betydelig i denne fasen

**Hva du bør se etter:**

* Jevn fremgang gjennom antall bilder
* Sjekk feilsøkingsloggen for meldinger om fullføring per bilde
* Advarsler om bildekvalitet eller kalibreringsproblemer

### Fase 4: Eksport

**Hva skjer:**

* Skriver kalibrerte bilder til disk i valgt format
* Eksporterer multispektrale indeksbilder med LUT-farger
* Oppretter undermapper for kameramodeller
* Bevarer originale filnavn med passende suffikser

**Varighet:** 10–20 % av total behandlingstid

**Fremdriftsindikator:**

* Eksporterer: 0 % → 100 %
* Filer som skrives
* Eksportformat og destinasjon

**Hva du bør være oppmerksom på:**

* Advarsler om diskplass
* Filskrivefeil
* Fullføring av alle konfigurerte utdata

***

## Fanen Debug Log (Feilsøkingslogg)

Feilsøkingsloggen gir detaljert informasjon om behandlingsfremdriften og eventuelle problemer som oppstår.

### Tilgang til feilsøkingsloggen

1. Klikk på **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> i venstre sidefelt
2. Loggpanelet åpnes og viser behandlingsmeldinger i sanntid
3. Ruller automatisk for å vise de nyeste meldingene

### Forstå loggmeldinger

#### Informasjonsmeldinger (hvite/grå)

Normale behandlingsoppdateringer:

```
[INFO] Processing started
[INFO] Target detected in IMG_0015.RAW - 4 panels found
[INFO] Calibrating IMG_0234.RAW
[INFO] Exported NDVI image: IMG_0234_NDVI.tif
[INFO] Processing complete
```

#### Advarselsmeldinger (gule)

Ikke-kritiske problemer som ikke stopper behandlingen:

```
[WARN] No GPS data found in IMG_0145.RAW
[WARN] Target image timestamp gap > 30 minutes
[WARN] Low contrast in calibration panel - results may vary
```

**Handling:** Gjennomgå advarsler etter behandling, men ikke avbryt

#### Feilmeldinger (Red)

Kritiske problemer som kan føre til at behandlingen mislykkes:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Handling:** Stopp behandlingen, løse feilen, start på nytt.

### Vanlige loggmeldinger

| Meldinger                          | Betydning                                | Nødvendig handling                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| «Mål oppdaget i \[filnavn]» | Kalibreringsmål funnet  | Ingen – normalt                                         |
| «Behandler bilde X av Y»        | Oppdatering av gjeldende fremdrift                | Ingen – normalt                                         |
| «Ingen mål funnet»               | Ingen kalibreringsmål oppdaget        | Merk målbilder eller deaktiver refleksjonskalibrering |
| «Utilstrekkelig diskplass»        | Ikke nok lagringsplass for utdata          | Frigjør diskplass                                    |
| «Hopper over ødelagt fil»        | Bildefilen er skadet                  | Kopier filen på nytt fra SD-kortet                             |
| «PPK-data brukt»               | GPS-korreksjoner fra .daq-fil brukt | Ingen – normalt                                         |

### Kopiere loggdata

Slik kopierer du loggen for feilsøking eller support:

1. Åpne Debug Log-panelet.
2. Klikk på **«Copy Log»**-knappen (eller høyreklikk → Select All).
3. Lim inn i tekstfil eller e-post.
4. Send til MAPIR-support om nødvendig.

***

## Overvåking av systemressurser

### CPU-bruk

**Fri modus:**

* 1 CPU-kjerne på \~100 %
* Andre kjerner er inaktive eller tilgjengelige
* Systemet forblir responsivt

**Chloros+ Parallell modus:**

* Flere kjerner på 80–100 % (opptil 16 kjerner)
* Høy total CPU-bruk
* Systemet kan føles mindre responsivt

**For å overvåke:**

* Windows Oppgavebehandling (Ctrl+Shift+Esc)
* Fanen Ytelse → CPU-delen
* Se etter prosessene «Chloros» eller «chloros-backend»

### Minnebruk (RAM)

**Typisk bruk:**

* Små prosjekter (&lt; 100 bilder): 2–4 GB
* Mellomstore prosjekter (100–500 bilder): 4–8 GB
* Store prosjekter (500+ bilder): 8–16 GB
* Chloros+ parallellmodus bruker mer RAM

**Hvis minnet er lavt:**

* Behandle mindre batcher
* Lukk andre applikasjoner
* Oppgrader RAM hvis du regelmessig behandler store datasett

### GPU-bruk (Chloros+ med CUDA)

Når GPU-akselerasjon er aktivert:

* NVIDIA GPU viser høy utnyttelse (60-90 %)
* VRAM-bruk øker (krever 4 GB+ VRAM)
* Kalibreringsfasen er betydelig raskere

**For å overvåke:**

* NVIDIA-ikonet i systemstatusfeltet
* Oppgavebehandling → Ytelse → GPU
* GPU-Z eller lignende overvåkingsverktøy

### Disk I/O

**Hva du kan forvente:**

* Høy disklesing under analyseringsfasen
* Høy diskskriving under eksporteringsfasen
* SSD er betydelig raskere enn HDD

**Ytelsestips:**

* Bruk SSD for prosjektmappen når det er mulig
* Unngå nettverksstasjoner for store datasett
* Sørg for at disken ikke er nær kapasitet (påvirker skrivehastigheten)

***

## Oppdage problemer under behandling

### Advarselstegn

**Fremdriften stopper opp (ingen endring i mer enn 5 minutter):**

* Sjekk feilsøkingsloggen for feil
* Kontroller at det er ledig diskplass
* Sjekk Oppgavebehandling for å sikre at Chloros kjører

**Feilmeldinger vises ofte:**

* Stopp behandlingen og gjennomgå feilene
* Vanlige årsaker: diskplass, ødelagte filer, minneproblemer
* Se avsnittet Feilsøking nedenfor

**Systemet svarer ikke:**

* Chloros+ parallellmodus bruker for mange ressurser
* Vurder å redusere antall samtidige oppgaver eller oppgradere maskinvaren
* Fri modus er mindre ressurskrevende

### Når du skal stoppe behandlingen

Stopp behandlingen hvis du ser:

* ❌ Feilmeldinger om «Disk full» eller «Kan ikke skrive fil»
* ❌ Gjentatte feil med ødelagte bildefiler
* ❌ Systemet er helt frosset (svarer ikke)
* ❌ Oppdaget at feil innstillinger ble konfigurert
* ❌ Feil bilder importert

**Slik stopper du:**

1. Klikk på **Stopp/Avbryt-knappen** (erstatter Start-knappen)
2. Behandlingen stoppes, fremdriften går tapt
3. Løs problemene og start på nytt fra begynnelsen

***

## Feilsøking under behandling

### Behandlingen er veldig treg

**Mulige årsaker:**

* Umerkede målbilder (skanner alle bilder)
* HDD i stedet for SSD-lagring
* Utilstrekkelige systemressurser
* Mange indekser konfigurert
* Tilgang til nettverksstasjon

**Løsninger:**

1. Hvis du nettopp har startet og er i detekteringsfasen: Avbryt, merk mål, start på nytt
2. For fremtiden: Bruk SSD, reduser indekser, oppgrader maskinvare
3. Vurder CLI for batchbehandling av store datasett

### Advarsler om «diskplass»

**Løsninger:**

1. Frigjør diskplass umiddelbart
2. Flytt prosjektet til en stasjon med mer plass
3. Reduser antall indekser som skal eksporteres
4. Bruk JPG-format i stedet for TIFF (mindre filer)

### Hyppige meldinger om «korrupte filer»

**Løsninger:**

1. Kopier bildene på nytt fra SD-kortet for å sikre integriteten
2. Test SD-kortet for feil
3. Fjern korrupte filer fra prosjektet
4. Fortsett behandlingen av de gjenværende bildene

### Systemet blir overopphetet/bremset

**Løsninger:**

1. Sørg for tilstrekkelig ventilasjon.
2. Rengjør støv fra datamaskinens ventilasjonsåpninger.
3. Reduser behandlingsbelastningen (bruk Free-modus i stedet for Chloros+).
4. Behandle på kjøligere tidspunkter av døgnet.

***

## Meldingen «Behandling fullført»

Når behandlingen er fullført:

* Fremdriftslinjen når 100 %
* Meldingen **«Behandling fullført»** vises i feilsøkingsloggen
* Startknappen blir aktivert igjen
* Alle utdatafiler ligger i undermappen for kameramodellen

***

## Neste trinn

Når behandlingen er fullført:

1. **Gjennomgå resultatene** – Se [Fullføre behandlingen](finishing-the-processing.md)
2. **Kontroller utdatamappen** – Kontroller at alle filene er eksportert riktig
3. **Gjennomgå feilsøkingsloggen** – Kontroller om det er advarsler eller feil
4. **Forhåndsvis behandlede bilder** – Bruk Image Viewer eller ekstern programvare

For informasjon om gjennomgang og bruk av behandlede resultater, se [Fullføre behandlingen](finishing-the-processing.md).
