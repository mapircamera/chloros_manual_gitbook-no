# Overvåking av behandlingen

Når behandlingen har startet, tilbyr Chloros

flere måter å overvåke fremdriften på, sjekke om det er problemer og forstå hva som skjer med datasettet ditt. Denne siden forklarer hvordan du kan følge med på behandlingen og tolke informasjonen Chloros

gir.

## Oversikt over fremdriftsindikatoren

Fremdriftslinjen øverst i toppmenyen viser behandlingsstatus i sanntid og fullføringsprosent. Fremdriften streames direkte fra backend via Server-Sent Events (SSE), slik at linjen gjenspeiler hva behandlingsrørledningen faktisk gjør.

### Fremdriftslinje i gratisversjonen

For brukere uten Chloros

+-lisens:

**2-trinns fremdriftsvisning:**

1.**Målgjenkjenning** – Finne kalibreringsmål i bilder
2. **Behandling** – Anvende korreksjoner og eksportere**Fremdriftslinjen viser:**

* Samlet fullføringsprosent (0–100 %)
* Navn på gjeldende trinn
* Enkel horisontal visning med stolpe

### Chloros

+-fremdriftsindikator

For brukere med Chloros

+-lisens:

**4-trinns fremdriftsvisning:**

1.**Deteksjon** – Finne kalibreringsmål
2. **Analyse** – Undersøke bilder og forberede pipelinen
3. **Kalibrering** – Anvendelse av vignett- og refleksjonskorrigeringer
4. **Eksport** – Lagring av behandlede filer**Interaktive funksjoner:*** **Hold musepekeren over** fremdriftslinjen for å se det utvidede 4-trinns-panelet
* **Klikk** på fremdriftslinjen for å fryse/feste det utvidede panelet
* **Klikk igjen** for å oppheve frysing og automatisk skjule når musen fjernes
* Hvert trinn viser individuell fremdrift (0–100 %)

{% hint style="info" %}
**Paritet med CLI**: under en `chloros-cli process`-kjøring rapporterer de samme fire trådene «Detecting», «Analyzing», «Processing» og «Exporting», og `chloros-cli export-status` viser eksportfremdriften for tråd 4 i sanntid fra en annen terminal. Se [CLI

-referansen](../reference/cli-reference.md).
{% endhint %}

***

## Forståelse av hvert behandlingsstadium

{% hint style="info" %}
**Pipeline-arkitektur**: Disse fire GUI-trinnene tilsvarer [4-tråds behandlingspipeline](../processing-architecture/processing-pipeline.md). På systemer med GPU-akselerasjon drar tråd 3 (kalibrering) nytte av [dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md), som optimaliserer behandlingen for din spesifikke maskinvare.
{% endhint %}

### Trinn 1: Deteksjon (måldeteksjon)

**Hva skjer:**

* Chloros

skanner bildene du har merket av med avkrysningsruten «Mål» (alle bilder hvis ingen er merket av)
* Algoritmer for maskinsyn identifiserer kalibreringspanelene
* Refleksjonsverdier hentes ut fra hvert panel
* Tidsstempler for målene registreres for riktig planlegging av kalibreringen

**Varighet:**

* Med merkede mål: 10–60 sekunder
* Uten merkede mål: 5–30+ minutter (skanner alle bildene)

**Fremdriftsindikator:**

* Gjenkjenning: 0 % → 100 %
* Antall skannede bilder (teller kun bildene som faktisk skannes)
* Antall funnet mål

**Hva du bør være oppmerksom på:**

* Bør fullføres raskt hvis målene er riktig merket
* Hvis det tar for lang tid, er målene kanskje ikke merket
* Sjekk feilsøkingsloggen for meldinger om «Mål funnet»

### Trinn 2: Analyse

**Hva skjer:**

* Leser EXIF-metadata fra bildene (tidsstempler, eksponeringsinnstillinger)
* Fastsetter kalibreringsstrategi basert på målobjektenes tidsstempler og tilgjengelige DAQ-data
* Organiserer køen for bildebehandling
* Forbereder parallelle prosesseringsarbeidere (Kun Chloros

+)

**Varighet:** 5–30 sekunder**Fremdriftsindikator:**

* Analyserer: 0 % → 100 %
* Rask fase, fullføres vanligvis raskt

**Hva du bør være oppmerksom på:**

* Fremdriften bør være jevn uten pauser
* Advarsler om manglende metadata vises i feilsøkingsloggen

### Trinn 3: Kalibrering

**Hva skjer:*** **Debayering**: Konvertering av RAW-Bayer-mønster til 3 kanaler (hoppes over for LATTICE-monomoduler, med en merknad)
* **Vignettkorrigering**: Fjerner mørkningen ved kanten av objektivet
* **Refleksjonskalibrering**: Normalisering med målverdier og/eller DAQ-nedjustering
* **Indeksberegning**: Beregning av multispektrale indekser
* Behandling av hvert bilde gjennom hele prosesskjeden

**Varighet:** Størstedelen av den totale behandlingstiden (60–80 %)**Fremdriftsindikator:**

* Kalibrerer: 0 % → 100 %
* Bildet som behandles for øyeblikket
* Fullførte bilder / Totalt antall bilder

**Behandlingsatferd:*** **Fri modus**: Behandler ett bilde om gangen sekvensielt
* **«Chloros

+»-modus**: Kjører en maskinvareadaptiv arbeiderpool – 1–4 samtidige arbeidere på GPU-systemer (avhengig av VRAM), én arbeider per fysisk kjerne (minus én) på rene CPU-systemer. Se [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md)
* **GPU-akselerasjon**: Fremskynder denne fasen betydelig**Hva du bør være oppmerksom på:**

* Jevn fremgang gjennom bildetallet
* Sjekk feilsøkingsloggen for fullføringsmeldinger per bilde
* Advarsler om bildekvalitet eller kalibreringsproblemer

### Trinn 4: Eksport

**Hva skjer:**

* Skriver bearbeidede bilder til disk i det valgte formatet etter hvert som de fullføres
* **LATTICE**: hvert bilde spres ut til alle aktiverte produkter (debayered / forhåndsvisning / radiance / reflectance)
* Eksportering av multispektrale indeksbilder med LUT-farger
* Opprettelse av utdatatreet `<project>/<camera>/<format>/<Product>_Images/` — eksporterte filer beholder kildens filnavn; mappen identifiserer produktet

**Varighet:** 10–20 % av total behandlingstid**Fremdriftsindikator:**

* Eksport: 0 % → 100 %
* Filer blir skrevet
* Eksportformat og destinasjon

**Hva du bør være oppmerksom på:**

* Advarsler om diskplass
* Feil ved filskriving
* Fullføring av alle konfigurerte utdata

***

## Fanen «Feilsøkingslogg»

Feilsøkingsloggen gir detaljert informasjon om fremdriften i behandlingen og eventuelle problemer som oppstår. Oppstartsmeldinger fra backend vises også i loggkonsollen, slik at loggen gir et fullstendig bilde selv om du åpner den i ettertid.

### Åpne feilsøkingsloggen

1. Klikk på **Feilsøkingslogg**-ikonet <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

i venstre sidefelt
2. Loggpanelet åpnes og viser behandlingsmeldinger i sanntid
3. Ruller automatisk for å vise de nyeste meldingene

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written), with the 1.3.0 bug icon (Report a bug) visible at the top of the tab -->

### Forstå loggmeldinger

Chloros

-logglinjer har et prefiks med tagger i parentes som angir navnet på delsystemet — for eksempel `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Det viktigste å kjenne til er **kjøringssammendraget**, som vises på slutten av hver kjøring (inkludert stoppede kjøringer):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Ekstra `[RUN-SUMMARY]`-hintlinjer følger når noe trenger forklaring — for eksempel et kjøringsløp som ikke ga noe resultat, eller et kamera der det forespurte produktet ble hoppet over fordi det ikke var relevant. `[EXPORT-CHECK]`-linjene forklarer utelatelser per kamera (f.eks. hvorfor et RGB

-kamera ikke fikk noe strålingsprodukt).

De generelle alvorlighetsgradene for meldinger (eksemplene nedenfor er illustrative, ikke ordrette):

#### Informasjonsmeldinger (hvit/grå)

Normale oppdateringer om behandlingen: behandling startet, mål oppdaget (med antall paneler), fremdrift i kalibrering per bilde, filer eksportert, behandling fullført.

#### Advarselsmeldinger (Gul)

Ikke-kritiske problemer som ikke stopper behandlingen — f.eks. manglende GPS-data i et bilde, et stort tidsstempelgap mellom målbilde, eller lav kontrast i et kalibreringspanel.

**Tiltak:** Gjennomgå advarslene etter behandlingen, men avbryt ikke

#### Feilmeldinger (rRed

)

Kritiske problemer som kan føre til at behandlingen mislykkes – f.eks. full disk, en ødelagt bildefil eller ingen mål oppdaget mens refleksjonskalibrering ble forespurt.

**Tiltak:** Stopp behandlingen, løse feilen, start på nytt

### Vanlige loggsituasjoner

| Situasjon                             | Betydning                                       | Nødvendig tiltak                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Mål oppdaget i \[filnavn]        | Kalibreringsmål funnet                                 | Ingen – normalt                                         |
| Fremdriftslinjer per bilde              | Oppdatering av gjeldende fremdrift                       | Ingen – normalt                                         |
| Ingen mål funnet                      | Ingen kalibreringsmål oppdaget               | Merk målbilder eller deaktiver reflektanskalibrering |
| Utilstrekkelig diskplass               | Ikke nok lagringsplass for utdata                 | Frigjør diskplass                                    |
| Hopper over ødelagt fil               | Bildefilen er skadet                         | Kopier filen på nytt fra SD-kortet                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | En opptaksserie uten råramme kan ikke behandles | Ta opptaket på nytt med råramme, eller bruk CLI

`--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | Kjøringen ga ingen bildeprodukter — rapportert som en feil med tips | Les tipslinjene; sjekk hva som ble hoppet over og hvorfor |
| `[PROCESS] refused: The project folder is too deep for this computer…` | Lange baner i Windows

er deaktivert, og prosjektmappen ligger for dypt i strukturen til å overholde grensen på 260 tegn for banen; ingenting ble behandlet | Bruk en kortere prosjektmappe (meldingen angir den lengste som passer), eller slå på lange stier i Windows

og start Chloros

på nytt — se [Starte behandlingen](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer) |
| `[unmix-clamp-mask] …` | Det var ikke mulig å skrive klemmasken til et LATTICE-produkt; selve produktet er i orden og viser `Chloros:UnmixClampMask=unavailable` | Sjekk diskplass og mappetillatelser, og kjør deretter behandlingen på nytt hvis du trenger masken |

### Kopiere loggdata

Slik kopierer du loggen for feilsøking eller support:

1. Åpne panelet «Debug Log»
2. Klikk på **Kopier**-knappen (kopierer loggen til utklippstavlen), eller**Last ned** for å lagre den som en tekstfil
3. Lim inn i en tekstfil eller e-post
4. Send til MAPIR

-support om nødvendig

### Rapportere en feil fra fanen «Debug Log»

Nytt i 1.3.0: feilikonet (**Rapporter en feil**) øverst på fanen «Debug Log» sender en rapport direkte til MAPIR

.

1. Klikk på feilikonet øverst på fanen «Debug Log»
2. Beskriv problemet: hva du gjorde, hva du forventet, og hva som skjedde i stedet (påkrevd)
3. Du kan eventuelt merke av for **Legg til noen bilder (opptil 10)**og velge bilder fra det åpne prosjektet, eller bruke**Bla gjennom...** for andre bildefiler
4. Klikk på **Send**

Rapporten inneholder beskrivelsen din, eventuelle bilder du legger ved, Chloros

-loggene og systemdetaljer (Chloros

-versjon, operativsystem, GPU og prosjektinnstillinger). Du må være logget inn på Chloros

(fanen «Bruker») for å kunne sende en rapport. Du kan lukke vinduet mens rapporten sendes; når overføringen er fullført, skrives en referanse som for eksempel `Bug report sent. Reference: BR-…` inn i loggen – oppgi denne når du kontakter MAPIR

. Hvis rapporten ikke kan lastes opp (for eksempel på grunn av manglende internettforbindelse), lagrer Chloros

den på datamaskinen din og viser stien, slik at du kan sende den via e-post til support@mapir.camera.

***

## Overvåking av systemressurser

### CPU-bruk

**Fri modus:**

* 1 CPU-kjerne på ~100 %
* Andre kjerner er inaktive eller tilgjengelige
* Systemet forblir responsivt

**Parallellmodus i Chloros

+:**

* Flere kjerner med høy utnyttelse – hvor mange avhenger av strategien valgt av [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md)
* Systemet kan oppleves som mindre responsivt

**For å overvåke:**

* Windows

Oppgavebehandling (Ctrl+Shift+Esc)
* Fanen Ytelse → CPU-delen
* Se etter «Chloros

»- eller «chloros-backend»-prosesser

### Minne (RAM)-bruk

**Typisk bruk:**

* Små prosjekter (&lt; 100 bilder): 2–4 GB
* Mellomstore prosjekter (100–500 bilder): 4–8 GB
* Store prosjekter (500+ bilder): 8–16 GB
* Chloros

i parallellmodus bruker mer RAM

**Hvis minnet er lavt:**

* Behandle mindre batcher
* Lukk andre programmer
* Oppgrader RAM hvis du regelmessig behandler store datasett

### GPU-bruk (Chloros

+ med CUDA)

Når GPU-akselerasjon er aktivert:

* NVIDIA-GPU viser høy utnyttelse (60–90 %)
* VRAM-bruken øker (krever 4 GB+ VRAM; 7 GB+ for samtidig «Texture Aware»-debayering)
* Kalibreringsfasen går betydelig raskere

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
* Sørg for at disken ikke er nesten full (påvirker skrivehastigheten)

***

## Oppdage problemer under behandlingen

### Advarselstegn

**Fremdriften stopper opp (ingen endring i 5+ minutter):**

* Sjekk feilloggen for feil
* Kontroller tilgjengelig diskplass
* Sjekk Oppgavebehandling for å sikre at Chloros

kjører

**Feilmeldinger vises hyppig:**

* Stopp behandlingen og gå gjennom feilene
* Vanlige årsaker: diskplass, ødelagte filer, minneproblemer
* Se avsnittet om feilsøking nedenfor

**Systemet reagerer ikke:**

* Chloros

i «+» parallellmodus bruker for mange ressurser
* Vurder å redusere antall samtidige oppgaver eller oppgradere maskinvaren
* «Free»-modus er mindre ressurskrevende

### Når du bør stoppe behandlingen

Stopp behandlingen hvis du ser:

* ❌ Feilmeldinger som «Disk full» eller «Kan ikke skrive fil»
* ❌ Gjentatte feil på grunn av ødelagte bildefiler
* ❌ Systemet er helt frosset (svarer ikke)
* ❌ Du oppdager at feil innstillinger er konfigurert
* ❌ Feil bilder er importert

**Slik stopper du:**

1. Klikk på**Stopp-knappen** (erstatter Start-knappen) – én gang er nok
2. Linjen viser «Stopper...» mens den pågående bildebehandlingen fullføres, deretter avsluttes kjøringen i stoppet tilstand
3. Produkter som allerede er eksportert, forblir på disken; loggen viser en detaljert oversikt over hva som er fullført
4. Løs problemene og start på nytt — kjøringen starter fra begynnelsen

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

1. Hvis du nettopp har startet og er i «Detecting»-fasen: Stopp, merk målene, start på nytt
2. For fremtiden: Bruk SSD, reduser antall indekser, oppgrader maskinvaren
3. Vurder å bruke CLI

-kommandoer for batchbehandling av store datasett

### Advarsler om «diskplass»

**Løsninger:**

1. Frigjør diskplass umiddelbart
2. Flytt prosjektet til en stasjon med mer plass
3. Reduser antall indekser som skal eksporteres
4. Deaktiver LATTICE-eksportprodukter du ikke trenger (Prosjektinnstillinger → Behandling)
5. Bruk JPG-format i stedet for TIFF

-format (mindre filer)

### Hyppige meldinger om «korrupte filer»

**Løsninger:**

1. Kopier bildene på nytt fra SD-kortet for å sikre integriteten
2. Test SD-kortet for feil
3. Fjern ødelagte filer fra prosjektet
4. Fortsett behandlingen av de gjenværende bildene

### Overoppheting / ytelsesbegrensning i systemet

**Løsninger:**

1. Sørg for tilstrekkelig ventilasjon
2. Fjern støv fra datamaskinens ventilasjonsåpninger
3. Reduser behandlingsbelastningen (bruk «Free»-modus i stedet for «Chloros

+»)
4. Behandle bildene på kjøligere tidspunkter av døgnet

***

## Varsel om fullført behandling

Når behandlingen er fullført:

* Fremdriftslinjen når 100 %
* Linjene `[RUN-SUMMARY]` vises i feilsøkingsloggen med endelige tall
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
