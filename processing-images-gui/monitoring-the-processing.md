# Overvåking av behandlingen

Når behandlingen har startet, tilbyr Chloros flere måter å overvåke fremdriften på, sjekke for problemer og forstå hva som skjer med datasettet ditt. Denne siden forklarer hvordan du kan spore behandlingen og tolke informasjonen som Chloros gir.

## Oversikt over fremdriftsindikatoren

Fremdriftslinjen øverst i toppmenyen viser behandlingsstatus i sanntid og fullføringsprosent. Fremdriften streames direkte fra backenden via Server-Sent Events (SSE), slik at linjen gjenspeiler hva behandlingsrørledningen faktisk gjør.

### Fremdriftslinje i gratisversjonen

For brukere uten Chloros+-lisens:

**Fremdriftsvisning i to trinn:**

1.**Målgjenkjenning** – Finne kalibreringsmål i bilder
2. **Behandling** – Anvendelse av korreksjoner og eksport**Fremdriftslinjen viser:**

* Samlet fullføringsprosent (0–100 %)
* Navn på gjeldende trinn
* Enkel horisontal visning

### Fremdriftslinje for Chloros+

For brukere med Chloros+-lisens:

**4-trinns fremdriftsvisning:**

1.**Deteksjon** – Finne kalibreringsmål
2. **Analyse** – Undersøke bilder og forberede prosesskjeden
3. **Kalibrering** – Anvendelse av vignett- og refleksjonskorrigeringer
4. **Eksport** – Lagring av behandlede filer**Interaktive funksjoner:*** **Hold musepekeren over** fremdriftslinjen for å se det utvidede 4-trinns-panelet
* **Klikk** på fremdriftslinjen for å fryse/feste det utvidede panelet
* **Klikk igjen** for å oppheve frysing og automatisk skjule når musen fjernes
* Hvert trinn viser individuell fremdrift (0–100 %)

{% hint style="info" %}
**CLI paritet**: under en `chloros-cli process`-kjøring rapporterer de samme fire trådene «Detekterer», «Analyserer», «Behandler» og «Eksporterer», mens `chloros-cli export-status` viser eksportfremdriften for tråd 4 i sanntid fra en annen terminal. Se [CLI-referansen](../reference/cli-reference.md).
{% endhint %}

***

## Forståelse av hvert behandlingsstadium

{% hint style="info" %}
**Pipeline-arkitektur**: Disse fire GUI-trinnene tilsvarer [4-tråds behandlingspipeline](../processing-architecture/processing-pipeline.md). På systemer med GPU-akselerasjon drar tråd 3 (kalibrering) nytte av [dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md), som optimaliserer behandlingen for din spesifikke maskinvare.
{% endhint %}

### Trinn 1: Deteksjon (måldeteksjon)

**Hva skjer:**

* Chloros skanner bildene du har merket av med «Mål»-avkrysningsruten (alle bilder hvis ingen er merket av)
* Algoritmer for maskinsyn identifiserer kalibreringspanelene
* Refleksjonsverdier hentes ut fra hvert panel
* Tidsstempler for målene registreres for riktig planlegging av kalibreringen

**Varighet:**

* Med merkede mål: 10–60 sekunder
* Uten merkede mål: 5–30+ minutter (skanner alle bildene)

**Fremdriftsindikator:**

* På jakt etter: 0 % → 100 %
* Antall skannede bilder (teller kun bildene som faktisk skannes)
* Antall funnet mål

**Hva du bør være oppmerksom på:**

* Bør fullføres raskt hvis målene er riktig merket
* Hvis det tar for lang tid, er målene kanskje ikke merket
* Sjekk feilsøkingsloggen for meldinger om «Mål funnet»

### Trinn 2: Analyse

**Hva skjer:**

* Leser EXIF-metadata fra bildene (tidsstempler, eksponeringsinnstillinger)
* Fastsetter kalibreringsstrategi basert på målenes tidsstempler og tilgjengelige DAQ-nedstrømningsdata
* Organiserer køen for bildebehandling
* Forbereder arbeidere for parallellbehandling (kun Chloros+)

**Varighet:** 5–30 sekunder**Fremdriftsindikator:**

* Analyserer: 0 % → 100 %
* Rask fase, fullføres vanligvis raskt

**Hva du bør være oppmerksom på:**

* Fremdriften bør være jevn uten pauser
* Advarsler om manglende metadata vises i feilsøkingsloggen

### Fase 3: Kalibrering

**Hva skjer:*** **Debayering**: Konverterer RAW-Bayer-mønster til 3 kanaler (hoppes over for LATTICE-monomoduler, med en merknad)
* **Vignettkorrigering**: Fjerner mørkningen ved kanten av objektivet
* **Refleksjonskalibrering**: Normalisering med målverdier og/eller DAQ-nedstråling
* **Indeksberegning**: Beregning av multispektrale indekser
* Behandling av hvert bilde gjennom hele prosesskjeden

**Varighet:** Størstedelen av den totale behandlingstiden (60–80 %)**Fremdriftsindikator:**

* Kalibrerer: 0 % → 100 %
* Bildet som behandles for øyeblikket
* Fullførte bilder / Totalt antall bilder

**Behandlingsatferd:*** **Fri modus**: Behandler ett bilde om gangen sekvensielt
* **Chloros+ modus**: Kjører en maskinvareadaptiv arbeiderpool – 1–4 samtidige arbeidere på GPU-systemer (avhengig av VRAM), én arbeider per fysisk kjerne (minus én) på rene CPU-systemer. Se [Dynamisk beregningsadaptasjon](../processing-architecture/dynamic-compute-adaptation.md)
* **GPU-akselerasjon**: Fremskynder denne fasen betydelig**Hva du bør holde øye med:**

* Jevn fremgang gjennom antall bilder
* Sjekk feilsøkingsloggen for fullføringsmeldinger per bilde
* Advarsler om bildekvalitet eller kalibreringsproblemer

### Trinn 4: Eksport

**Hva skjer:**

* Skriver bearbeidede bilder til disken i det valgte formatet etter hvert som de fullføres
* **LATTICE**: hvert bilde spres ut til alle aktiverte produkter (debayered / forhåndsvisning / radiance / reflektans)
* Eksportering av multispektrale indeksbilder med LUT-farger
* Opprettelse av utdatatreet `<project>/<camera>/<format>/<Product>_Images/` — eksporterte filer beholder kildfilnavnet; mappen identifiserer produktet

**Varighet:** 10–20 % av total behandlingstid**Fremdriftsindikator:**

* Eksport: 0 % → 100 %
* Filer blir skrevet
* Eksportformat og destinasjon

**Hva du bør være oppmerksom på:**

* Advarsler om diskplass
* Feil ved skriving av filer
* Fullføring av alle konfigurerte utdata

***

## Fanen «Debug Log»

Debug Log gir detaljert informasjon om fremdriften i behandlingen og eventuelle problemer som oppstår. Oppstartsmeldinger fra backend vises også i loggkonsollen, slik at loggen gir et fullstendig bilde selv om du åpner den i etterkant.

### Åpne feilsøkingsloggen

1. Klikk på **Feilsøkingslogg**-<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

ikonet i venstre sidefelt
2. Loggpanelet åpnes og viser behandlingsmeldinger i sanntid
3. Ruller automatisk for å vise de nyeste meldingene

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written) -->

### Forstå loggmeldinger

Chloros-logglinjer har et prefiks med tagger i parentes som angir navnet på delsystemet – for eksempel `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Det viktigste å kjenne til er **kjøringssammendraget**, som skrives ut på slutten av hvert kjøringsløp (inkludert avbrutte kjøringsløp):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Ekstra `[RUN-SUMMARY]`-hintlinjer følger når noe trenger forklaring — for eksempel en kjøring som ikke ga noe resultat, eller et kamera der det forespurte produktet ble hoppet over fordi det ikke var relevant. `[EXPORT-CHECK]`-linjer forklarer utelatelser per kamera (f.eks. hvorfor et RGB-kamera ikke fikk noe strålingsprodukt).

De generelle alvorlighetsgradene for meldinger (eksemplene nedenfor er illustrative, ikke ordrette):

#### Informasjonsmeldinger (hvit/grå)

Normale oppdateringer om behandlingen: behandlingen startet, mål oppdaget (med antall paneler), fremdrift i kalibrering per bilde, filer eksportert, behandlingen fullført.

#### Advarselsmeldinger (Gul)

Ikke-kritiske problemer som ikke stopper behandlingen – f.eks. manglende GPS-data i et bilde, et stort tidsstempelgap mellom målbilder eller lav kontrast i et kalibreringspanel.

**Tiltak:** Gjennomgå advarsler etter behandlingen, men ikke avbryt

#### Feilmeldinger (Red)

Kritiske problemer som kan føre til at behandlingen mislykkes – f.eks. full disk, en ødelagt bildefil eller ingen mål oppdaget mens refleksjonskalibrering ble forespurt.

**Tiltak:** Stopp behandlingen, løs feilen, start på nytt

### Vanlige logg-situasjoner

| Situasjon                             | Betydning                                       | Nødvendig tiltak                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Mål oppdaget i \[filnavn]            | Kalibreringsmål funnet         | Ingen – normalt                                         |
| Fremdriftslinjer per bilde              | Oppdatering av gjeldende fremdrift                       | Ingen – normalt                                         |
| Ingen mål funnet                      | Ingen kalibreringsmål oppdaget               | Merk målbilder eller deaktiver reflektanskalibrering |
| Utilstrekkelig diskplass               | Ikke nok lagringsplass for utdata                 | Frigjør diskplass                                    |
| Hopper over ødelagt fil               | Bildefilen er skadet                         | Kopier filen på nytt fra SD-kortet                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | En opptak uten råramme kan ikke behandles | Ta opptaket på nytt med råramme, eller bruk CLI `--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | Kjøringen ga ingen bildeprodukter — rapportert som en feil med tips | Les tipslinjene; sjekk hva som ble hoppet over og hvorfor |

### Kopiere loggdata

Slik kopierer du loggen for feilsøking eller support:

1. Åpne panelet «Debug Log»
2. Klikk på **«Kopier logg»**-knappen (eller høyreklikk → Velg alt)
3. Lim inn i en tekstfil eller e-post
4. Send til MAPIR-support om nødvendig

***

## Overvåking av systemressurser

### CPU-bruk

**Fri modus:**

* 1 CPU-kjerne på ~100 %
* Andre kjerner er inaktive eller tilgjengelige
* Systemet forblir responsivt

**Chloros+ Parallellmodus:**

* Flere kjerner med høy utnyttelse — hvor mange avhenger av strategien valgt av [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md)
* Systemet kan føles mindre responsivt

**Slik overvåker du:**

* Windows Oppgavebehandling (Ctrl+Shift+Esc)
* Fanen Ytelse → CPU-delen
* Se etter «Chloros»- eller «chloros-backend»-prosesser

### Minnebruk (RAM)

**Typisk bruk:**

* Små prosjekter (&lt; 100 bilder): 2–4 GB
* Mellomstore prosjekter (100–500 bilder): 4–8 GB
* Store prosjekter (500+ bilder): 8–16 GB
* Parallellmodus Chloros+ bruker mer RAM

**Hvis minnet er lavt:**

* Behandle mindre batcher
* Lukk andre applikasjoner
* Oppgrader RAM hvis du regelmessig behandler store datasett

### GPU-bruk (Chloros+ med CUDA)

Når GPU-akselerasjon er aktivert:

* NVIDIA-GPU viser høy utnyttelse (60–90 %)
* VRAM-forbruket øker (krever 4 GB+ VRAM; 7 GB+ for samtidig «Texture Aware»-debayering)
* Kalibreringsfasen går betydelig raskere

**Overvåk:**

* NVIDIA-ikonet i systemstatusfeltet
* Oppgavebehandling → Ytelse → GPU
* GPU-Z eller lignende overvåkingsverktøy

### Disk-I/O

**Hva du kan forvente:**

* Høy disklesing under analysefasen
* Høy disk-skriving under eksportfasen
* SSD er betydelig raskere enn HDD

**Ytelsestips:**

* Bruk SSD til prosjektmappen når det er mulig
* Unngå nettverksstasjoner for store datasett
* Sørg for at disken ikke er nesten full (påvirker skrivehastigheten)

***

## Oppdage problemer under behandling

### Advarselstegn

**Fremdriften stopper opp (ingen endring i over 5 minutter):**

* Sjekk feilloggen for feil
* Kontroller tilgjengelig diskplass
* Sjekk Oppgavebehandling for å sikre at Chloros kjører

**Feilmeldinger vises ofte:**

* Stopp behandlingen og gå gjennom feilene
* Vanlige årsaker: diskplass, ødelagte filer, minneproblemer
* Se avsnittet om feilsøking nedenfor

**Systemet reagerer ikke:**

* Chloros+ i parallellmodus bruker for mange ressurser
* Vurder å redusere antall samtidige oppgaver eller oppgradere maskinvaren
* Fri modus er mindre ressurskrevende

### Når du bør stoppe behandlingen

Stopp behandlingen hvis du ser:

* ❌ Feilmeldinger som «Disk full» eller «Kan ikke skrive fil»
* ❌ Gjentatte feil med ødelagte bildefiler
* ❌ Systemet er helt frosset (svarer ikke)
* ❌ Oppdager at feil innstillinger er konfigurert
* ❌ Feil bilder er importert

**Slik stopper du:**

1. Klikk på**Stopp-knappen** (erstatter Start-knappen) – én gang er nok
2. Linjen viser «Stoppes...» mens det pågående bildet fullføres, deretter avsluttes kjøringen i stoppet tilstand
3. Produkter som allerede er eksportert, forblir på disken; loggen viser en detaljert oversikt over hva som ble fullført
4. Løs problemene og start på nytt – kjøringen starter fra begynnelsen

***

## Feilsøking under behandling

### Behandlingen går veldig tregt

**Mulige årsaker:**

* Umerkede målbilder (skanner alle bilder)
* HDD i stedet for SSD-lagring
* Utilstrekkelige systemressurser
* Mange konfigurerte indekser
* Tilgang til nettverksstasjon

**Løsninger:**

1. Hvis du nettopp har startet og er i «Detecting»-fasen: Stopp, merk målene, start på nytt
2. For fremtiden: Bruk SSD, reduser antall indekser, oppgrader maskinvaren
3. Vurder å bruke CLI for batchbehandling av store datasett

### Advarsler om «diskplass»

**Løsninger:**

1. Frigjør diskplass umiddelbart
2. Flytt prosjektet til en stasjon med mer plass
3. Reduser antall indekser som skal eksporteres
4. Deaktiver LATTICE-eksportprodukter du ikke trenger (Prosjektinnstillinger → Behandling)
5. Bruk JPG-format i stedet for TIFF (mindre filer)

### Hyppige meldinger om «korrupte filer»

**Løsninger:**

1. Kopier bildene på nytt fra SD-kortet for å sikre at de er intakte
2. Test SD-kortet for feil
3. Fjern ødelagte filer fra prosjektet
4. Fortsett behandlingen av de gjenværende bildene

### Overoppheting / ytelsesbegrensning av systemet

**Løsninger:**

1. Sørg for tilstrekkelig ventilasjon
2. Fjern støv fra datamaskinens ventilasjonsåpninger
3. Reduser prosesseringsbelastningen (bruk Free-modus i stedet for Chloros+)
4. Utfør prosessering på kjøligere tidspunkter av døgnet

***

## Melding om fullført prosessering

Når prosesseringen er fullført:

* Fremdriftslinjen når 100 %
* `[RUN-SUMMARY]`-linjene vises i feilsøkingsloggen med endelige tall
* Startknappen blir aktivert igjen
* Alle utdatafiler ligger i prosjektets utdatastruktur per kamera: `<project>/<camera>/<format>/<Product>_Images/`

***

## Neste trinn

Når behandlingen er fullført:

1. **Gjennomgå resultatene** – Se [Fullføre behandlingen](finishing-the-processing.md)
2. **Sjekk utdatamappen** – Kontroller at alle filer er eksportert riktig
3. **Gjennomgå feillogg** – Se etter eventuelle advarsler eller feil
4. **Forhåndsvis behandlede bilder** – Bruk Image Viewer eller ekstern programvare

For informasjon om hvordan du gjennomgår og bruker de behandlede resultatene, se [Fullføre behandlingen](finishing-the-processing.md).
