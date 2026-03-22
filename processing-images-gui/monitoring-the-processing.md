# Overvåking av behandlingen

Når behandlingen er i gang, tilbyr Chloros flere måter å overvåke fremdriften på, sjekke om det er problemer og få oversikt over hva som skjer med datasettet ditt. Denne siden forklarer hvordan du kan følge med på behandlingen og tolke informasjonen som Chloros gir.

## Oversikt over fremdriftslinjen

Fremdriftslinjen øverst i toppmenyen viser behandlingsstatus og fullføringsprosent i sanntid.

### Fremdriftslinje i gratisversjonen

For brukere uten Chloros+-lisens:

**2-trinns fremdriftsvisning:**

1.**Målregistrering** – Finne kalibreringsmål i bilder
2. **Behandling** – Anvendelse av korreksjoner og eksport**Fremdriftslinjen viser:**

* Samlet fullføringsprosent (0–100 %)
* Navn på gjeldende trinn
* Enkel horisontal linjevisualisering

### Fremdriftslinje for Chloros+

For brukere med Chloros+-lisens:

**4-trinns fremdriftsvisning:**

1.**Detektering** – Finne kalibreringsmål
2. **Analyse** – Undersøke bilder og forberede prosesskjeden
3. **Kalibrering** – Anvende vignett- og refleksjonskorrigeringer
4. **Eksport** – Lagre behandlede filer**Interaktive funksjoner:*** **Hold musepekeren over** fremdriftslinjen for å se det utvidede 4-trinns panelet
* **Klikk** på fremdriftslinjen for å fryse/feste det utvidede panelet
* **Klikk igjen** for å oppheve frysing og automatisk skjule når musen fjernes
* Hvert trinn viser individuell fremdrift (0–100 %)

***

## Forståelse av hvert behandlingstrinn

{% hint style="info" %}
**Pipeline-arkitektur**: Disse 4 GUI-trinnene tilsvarer [4-tråds behandlingspipeline](../processing-architecture/processing-pipeline.md). På systemer med GPU-akselerasjon drar tråd 3 (Kalibrering) nytte av [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md), som optimaliserer behandlingen for din spesifikke maskinvare.
{% endhint %}

### Trinn 1: Deteksjon (målgjenkjenning)

**Hva skjer:**

* Chloros skanner bilder merket med avkrysningsruten «Mål»
* Datasynsalgoritmer identifiserer de 4 kalibreringspanelene
* Refleksjonsverdier hentes ut fra hvert panel
* Mål-tidsstempler registreres for riktig kalibreringsplanlegging

**Varighet:**

* Med merkede mål: 10–60 sekunder
* Uten merkede mål: 5–30+ minutter (skanner alle bilder)

**Fremdriftsindikator:**

* Detektering: 0 % → 100 %
* Antall skannede bilder
* Antall funnet mål

**Hva du bør være oppmerksom på:**

* Bør fullføres raskt hvis målene er riktig merket
* Hvis det tar for lang tid, er målene kanskje ikke merket
* Sjekk feilsøkingsloggen for meldinger om «Mål funnet»

### Fase 2: Analyse

**Hva skjer:**

* Leser EXIF-metadata fra bildet (tidsstempler, eksponeringsinnstillinger)
* Bestemmer kalibreringsstrategi basert på målets tidsstempler
* Organiserer køen for bildebehandling
* Forbereder parallelle behandlingsarbeidere (kun Chloros+)

**Varighet:** 5–30 sekunder**Fremdriftsindikator:**

* Analyserer: 0 % → 100 %
* Rask fase, fullføres vanligvis raskt

**Hva du bør se etter:**

* Skal gå jevnt fremover uten pauser
* Advarsler om manglende metadata vil vises i feilsøkingsloggen

### Fase 3: Kalibrering

**Hva skjer:*** **Debayering**: Konvertering av RAW-Bayer-mønster til 3 kanaler
* **Vignettkorrigering**: Fjerner mørkere kanter på bildet
* **Refleksjonskalibrering**: Normaliserer med målverdier
* **Indeksberegning**: Beregner multispektrale indekser
* Behandler hvert bilde gjennom hele prosessen

**Varighet:** Størstedelen av den totale behandlingstiden (60–80 %)**Fremdriftsindikator:**

* Kalibrering: 0 % → 100 %
* Bildet som behandles for øyeblikket
* Fullførte bilder / Totalt antall bilder

**Behandlingsatferd:*** **Fri modus**: Behandler ett bilde om gangen sekvensielt
* **Chloros+ modus**: Behandler opptil 16 bilder samtidig
* **GPU-akselerasjon**: Fremskynder denne fasen betydelig**Hva du bør følge med på:**

* Jevn fremgang gjennom bildetellingen
* Sjekk feilsøkingsloggen for fullføringsmeldinger per bilde
* Advarsler om bildekvalitet eller kalibreringsproblemer

### Fase 4: Eksport

**Hva skjer:**

* Skriver kalibrerte bilder til disk i valgt format
* Eksporterer multispektrale indeksbilder med LUT-farger
* Oppretter undermapper for kameramodeller
* Bevarer originale filnavn med passende suffikser

**Varighet:** 10–20 % av total behandlingstid**Fremdriftsindikator:**

* Eksport: 0 % → 100 %
* Filer som skrives
* Eksportformat og destinasjon

**Hva du bør følge med på:**

* Advarsler om diskplass
* Feil ved filskriving
* Fullføring av alle konfigurerte utdata

***

## Fanen Debug Log

Debug Log gir detaljert informasjon om behandlingsfremdriften og eventuelle problemer som oppstår.

### Åpne Debug Log

1. Klikk på **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> i venstre sidefelt
2. Loggpanelet åpnes og viser behandlingsmeldinger i sanntid
3. Ruller automatisk for å vise de nyeste meldingene

### Forstå loggmeldinger

#### Informasjonsmeldinger (hvit/grå)

Normale behandlingsoppdateringer:

```
[INFO] Processing started
[INFO] Target detected in IMG_0015.RAW - 4 panels found
[INFO] Calibrating IMG_0234.RAW
[INFO] Exported NDVI image: IMG_0234_NDVI.tif
[INFO] Processing complete
```

#### Advarselsmeldinger (gul)

Ikke-kritiske problemer som ikke stopper behandlingen:

```
[WARN] No GPS data found in IMG_0145.RAW
[WARN] Target image timestamp gap > 30 minutes
[WARN] Low contrast in calibration panel - results may vary
```

**Handling:** Gjennomgå advarsler etter behandlingen, men ikke avbryt

#### Feilmeldinger (Red)

Kritiske problemer som kan føre til at behandlingen mislykkes:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Tiltak:** Stopp behandlingen, løse feilen, start på nytt

### Vanlige loggmeldinger

| Melding                          | Betydning                                | Nødvendig tiltak                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| &quot;Mål oppdaget i \[filnavn]&quot; | Kalibreringsmål funnet  | Ingen – normalt                                         |
| &quot;Behandler bilde X av Y&quot;        | Oppdatering av gjeldende fremdrift                | Ingen – normalt                                         |
| &quot;Ingen mål funnet&quot;               | Ingen kalibreringsmål oppdaget        | Merk målbilder eller deaktiver reflektanskalibrering |
| &quot;Utilstrekkelig diskplass&quot;        | Ikke nok lagringsplass for utdata          | Frigjør diskplass                                    |
| &quot;Hopper over ødelagt fil&quot;        | Bildefilen er skadet                  | Kopier filen på nytt fra SD-kortet                             |
| &quot;PPK-data brukt&quot;               | GPS-korreksjoner fra .daq-fil brukt | Ingen - normalt                                         |

### Kopiere loggdata

For å kopiere loggen for feilsøking eller support:

1. Åpne Debug Log-panelet
2. Klikk på **&quot;Copy Log&quot;**-knappen (eller høyreklikk → Velg alt)
3. Lim inn i tekstfil eller e-post
4. Send til MAPIR-support om nødvendig

***

## Overvåking av systemressurser

### CPU-bruk

**Fri modus:**

* 1 CPU-kjerne på ~100 %
* Andre kjerner er inaktive eller tilgjengelige
* Systemet forblir responsivt

**Chloros+ Parallellmodus:**

* Flere kjerner på 80–100 % (opptil 16 kjerner)
* Høy samlet CPU-utnyttelse
* Systemet kan føles mindre responsivt

**For å overvåke:**

* Windows Oppgavebehandling (Ctrl+Shift+Esc)
* Fanen Ytelse → CPU-delen
* Se etter «Chloros»- eller «chloros-backend»-prosesser

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

* NVIDIA GPU viser høy utnyttelse (60–90 %)
* VRAM-bruk øker (krever 4 GB+ VRAM)
* Kalibreringsfasen er betydelig raskere

**For å overvåke:**

* NVIDIA-ikonet i systemstatusfeltet
* Oppgavebehandling → Ytelse → GPU
* GPU-Z eller lignende overvåkingsverktøy

### Disk-I/O

**Hva du kan forvente:**

* Høy disklesing under analysefasen
* Høy diskinnskriving under eksportfasen
* SSD er betydelig raskere enn HDD

**Ytelsestips:**

* Bruk SSD til prosjektmappen når det er mulig
* Unngå nettverksstasjoner for store datasett
* Sørg for at disken ikke er nær kapasitetsgrensen (påvirker skrivehastigheten)

***

## Oppdage problemer under behandling

### Advarselstegn

**Fremdriften stopper opp (ingen endring i 5+ minutter):**

* Sjekk feilloggen for feil
* Kontroller tilgjengelig diskplass
* Sjekk Oppgavebehandling for å sikre at Chloros kjører

**Feilmeldinger vises ofte:**

* Stopp behandlingen og gjennomgå feilene
* Vanlige årsaker: diskplass, ødelagte filer, minneproblemer
* Se avsnittet Feilsøking nedenfor

**Systemet reagerer ikke:**

* Chloros+ parallellmodus bruker for mange ressurser
* Vurder å redusere antall samtidige oppgaver eller oppgradere maskinvaren
* Fri modus er mindre ressurskrevende

### Når du bør stoppe behandlingen

Stopp behandlingen hvis du ser:

* ❌ Feilmeldinger som «Disk full» eller «Kan ikke skrive fil»
* ❌ Gjentatte feil med ødelagte bildefiler
* ❌ Systemet er helt frosset (svarer ikke)
* ❌ Oppdaget at feil innstillinger ble konfigurert
* ❌ Feil bilder importert

**Slik stopper du:**

1. Klikk på**Stopp/Avbryt-knappen** (erstatter Start-knappen)
2. Behandlingen stoppes, fremdriften går tapt
3. Løs problemene og start på nytt fra begynnelsen

***

## Feilsøking under behandling

### Behandlingen går veldig tregt

**Mulige årsaker:**

* Umarkerte målbilder (skanner alle bilder)
* HDD i stedet for SSD-lagring
* Utilstrekkelige systemressurser
* Mange indekser konfigurert
* Tilgang til nettverksstasjon

**Løsninger:**

1. Hvis du nettopp har startet og er i deteksjonsfasen: Avbryt, merk mål, start på nytt
2. For fremtiden: Bruk SSD, reduser antall indekser, oppgrader maskinvare
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

### Systemoveroppheting / Throttling

**Løsninger:**

1. Sørg for tilstrekkelig ventilasjon
2. Fjern støv fra datamaskinens ventilasjonsåpninger
3. Reduser prosesseringsbelastningen (bruk Free-modus i stedet for Chloros+)
4. Behandle på kjøligere tider av døgnet

***

## Melding om fullført behandling

Når behandlingen er ferdig:

* Fremdriftslinjen når 100 %
* **Meldingen «Behandlingen er fullført»** vises i feilsøkingsloggen
* Startknappen blir aktivert igjen
* Alle utdatafiler ligger i undermappen for kameramodellen

***

## Neste trinn

Når behandlingen er fullført:

1. **Gjennomgå resultatene** – Se [Fullføre behandlingen](finishing-the-processing.md)
2. **Sjekk utdatamappen** – Kontroller at alle filer er eksportert riktig
3. **Gjennomgå feilsøkingsloggen** – Sjekk om det er advarsler eller feil
4. **Forhåndsvis behandlede bilder** – Bruk bildeviser eller ekstern programvare

For informasjon om hvordan du gjennomgår og bruker de behandlede resultatene, se [Fullføre behandlingen](finishing-the-processing.md).
