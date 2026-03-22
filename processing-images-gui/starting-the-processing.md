# Starte behandlingen

Når du har importert bildene, merket kalibreringsmålene og konfigurert prosjektinnstillingene, er du klar til å starte behandlingen. Denne siden veileder deg gjennom oppstarten av Chloros-behandlingsprosessen.

## Sjekkliste før behandling

Før du klikker på Start-knappen, må du kontrollere at alt er klart:

* [ ] **Filer importert** – Alle bildene vises i filbrowseren
* [ ] **Målbilder merket** – Målkolonnen er merket for kalibreringsbilder
* [ ] **Kameramodeller oppdaget** – Kameramodellkolonnen viser riktige kameraer
* [ ] **Innstillinger konfigurert** – Prosjektinnstillinger gjennomgått og justert
* [ ] **Indekser valgt** – Ønskede multispektrale indekser lagt til (om nødvendig)
* [ ] **Eksportformat valgt** – Utdataformat som passer for arbeidsflyten din

{% hint style="info" %}
**Tips**: Klikk deg gjennom noen bilder i filbrowseren for å kontrollere at de er lastet inn riktig før behandling.
{% endhint %}

***

## Starte behandlingen

### Finn Start-knappen

Start/Spill av-knappen finnes i den øverste overskriftslinjen i Chloros:

* Plassering: Øverst i midten av vinduet
* Ikon: **Spill av/Start-knapp** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
* Status: Knappen er aktivert (lysende) når den er klar til behandling

### Klikk for å starte

1. Klikk på **Avspill/Start-knappen** i toppmenyen
2. Behandlingen starter umiddelbart
3. Knappen blir deaktivert (grå) under behandlingen
4. Fremdriftslinjen oppdateres og viser behandlingsstatus

{% hint style="success" %}
**Behandlingen er startet**: Når du har klikket, håndterer Chloros automatisk alle behandlingstrinnene – måldeteksjon, debayering, kalibrering, indeksberegning og eksport.
{% endhint %}

***

## Forstå behandlingsmodusene

Chloros opererer i to forskjellige behandlingsmoduser avhengig av lisensen din:

### Gratis modus (sekvensiell behandling)

**Tilgjengelig for alle brukere**

**Slik fungerer det:**

* Behandler bilder ett om gangen, sekvensielt
* Enkelt-trådet drift
* Lavere minnebruk

**Fremdriftslinjen viser to trinn:**

1.**Målregistrering** – Skanner etter kalibreringsmål
2. **Behandling** – Anvender kalibrering og eksporterer bilder**Behandlingstid:**

* Mye tregere enn Chloros+ parallellmodus
* Egnet for små til mellomstore datasett (&lt; 200 bilder)

### Chloros+-modus (parallellbehandling)

**Krever Chloros+-lisens**

**Slik fungerer det:**

* Behandler flere bilder samtidig ved hjelp av en [4-tråds behandlingspipeline](../processing-architecture/processing-pipeline.md)
* [Dynamisk beregningsadaptasjon](../processing-architecture/dynamic-compute-adaptation.md) velger automatisk den optimale strategien for maskinvaren din
* GPU (CUDA)-akselerasjon med NVIDIA-grafikkort (stasjonær og Jetson)
* Skalerer fra en Jetson Nano (1 arbeider) til en stasjonær PC med 12 GB+ GPU (3–4 arbeidere)

**Fremdriftslinjen viser 4 trinn** (tilsvarende de 4 pipeline-trådene):

1. **Detektering** (Tråd 1) – Finne kalibreringsmål
2. **Analyse** (Tråd 2) – Undersøke bildemetadata og beregne kalibrering
3. **Kalibrering** (Tråd 3) – GPU-debayering, vignettkorrigering, indeksberegning
4. **Eksport** (Tråd 4) – Lagring av behandlede bilder og indekser**Interaksjon med fremdriftslinjen:*** **Hold musen** over linjen for å se et detaljert nedtrekkspanel med 4 trinn
* **Klikk** på fremdriftslinjen for å fryse nedtrekkspanelet på plass
* **Klikk igjen** for å oppheve frysing og skjule panelet**Behandlingstid:**

* Betydelig raskere enn gratis-modus
* Skalerer med antall CPU-kjerner
* GPU-akselerasjon forbedrer hastigheten ytterligere

{% hint style="info" %}
**Chloros+ Hastighet**: Parallellbehandling kan være 5–10 ganger raskere enn sekvensiell modus for store datasett. Et prosjekt med 500 bilder som tar 2 timer i gratis modus, kan fullføres på 15–20 minutter med Chloros+.
{% endhint %}

***

## Hva skjer under behandlingen

### Trinn 1: Målgjenkjenning

**Hva Chloros gjør:**

* Skanner merkede målbilder (eller alle bilder hvis ingen er merket)
* Identifiserer de 4 kalibreringspanelene i hvert mål
* Henter ut refleksjonsverdier fra målpanelene
* Registrerer tidsstempler for målene for kalibreringsplanlegging

**Varighet:** 1–30 sekunder (med merkede mål), 5–30+ minutter (umerkede)

### Trinn 2: Debayering (RAW-konvertering)

**Hva Chloros gjør:**

* Konverterer RAW-data med Bayer-mønster til fullstendige RGB-bilder
* Bruker en demosaicing-algoritme av høy kvalitet
* Bevarer maksimal bildekvalitet og detaljrikdom

**Varighet:** Varierer etter antall bilder og CPU-hastighet

### Trinn 3: Kalibrering

**Hva Chloros gjør:*** **Vignettkorrigering**: Fjerner mørkere kanter på bildet
* **Refleksjonskalibrering**: Normaliserer ved hjelp av målverdier for refleksjon
* Bruker korreksjoner på tvers av alle bånd/kanaler
* Bruker passende kalibreringsmål for hvert bilde basert på tidsstempel

**Varighet:** Størstedelen av behandlingstiden

### Trinn 4: Indeksberegning

**Hva Chloros gjør:**

* Beregner konfigurerte multispektrale indekser (NDVI, NDRE, osv.)
* Bruker båndmatematikk på kalibrerte bilder
* Genererer indeksbilder for hvert valgt indeks

**Varighet:** Noen få sekunder per bilde

### Trinn 5: Eksport

**Hva Chloros gjør:**

* Lagrer kalibrerte bilder i valgt format
* Eksporterer indeksbilder med konfigurerte LUT-farger
* Skriver filer til undermapper for kameramodeller
* Bevarer originale filnavn med suffikser

**Varighet:** Varierer etter eksportformat og filstørrelse***

## Behandlingsatferd

### Automatisk behandlingsrørledning

Når den er startet, kjører hele rørledningen automatisk:

* Ingen brukerinteraksjon nødvendig
* Alle konfigurerte trinn utføres i rekkefølge
* Fremdriftsoppdateringer vises i sanntid

### Databruk under behandling

**Fri modus:**

* Relativt lav CPU-bruk (enkelt-trådet)
* Datamaskinen forblir responsiv for andre oppgaver
* Det er trygt å minimere Chloros og arbeide i andre applikasjoner

**Chloros+ Parallellmodus:**

* Høy CPU-bruk (fler-trådet, opptil 16 kjerner)
* Med GPU-akselerasjon: Høy GPU-bruk
* Datamaskinen kan være mindre responsiv under behandlingen
* Unngå å starte andre CPU-intensive oppgaver

{% hint style="warning" %}
**Ytelsestips**: For best mulig ytelse, lukk andre programmer og la Chloros bruke alle systemressursene.
{% endhint %}

### Behandlingen kan ikke settes på pause

**Viktige begrensninger:**

* Når behandlingen er startet, kan den ikke settes på pause
* Du kan avbryte behandlingen, men fremdriften går tapt
* Delvise resultater lagres ikke
* Må startes på nytt fra begynnelsen hvis den avbrytes

**Planleggingstips:** For svært store prosjekter bør du vurdere å behandle i batcher eller bruke CLI for bedre kontroll.***

## Overvåke behandlingen

Mens behandlingen pågår, kan du:

* **Se fremdriftslinjen** – Se den samlede fullføringsprosenten
* **Se gjeldende trinn** – Oppdage, analysere, kalibrere eller eksportere
* **Sjekke loggfanen** – Se detaljerte behandlingsmeldinger og advarsler
* **Forhåndsvise ferdige bilder** – Noen eksportfiler kan vises under behandlingen

For detaljert informasjon om overvåking, se [Overvåking av behandlingen](monitoring-the-processing.md).

***

## Avbryte behandlingen

Hvis du trenger å stoppe behandlingen:

### Slik avbryter du

1. Finn **Stopp/Avbryt-knappen** (erstatter Start-knappen under behandlingen)
2. Klikk på Stopp-knappen
3. Behandlingen stoppes umiddelbart
4. Delvise resultater blir forkastet

### Når du bør avbryte

**Gyldige grunner til å avbryte:**

* Du oppdaget at feil innstillinger ble brukt
* Du glemte å merke målbilder
* Feil bilder importert
* Systemet kjører for sakte eller svarer ikke

**Etter avbrytelse:**

* Gjennomgå og løse eventuelle problemer
* Juster innstillingene etter behov
* Start behandlingen på nytt fra begynnelsen
* For best mulig opplevelse, lukk Chloros helt og start på nytt

{% hint style="warning" %}
**Ingen delvise resultater**: Avbrytelse sletter all fremgang. Chloros lagrer ikke delvis behandlede bilder.
{% endhint %}

***

## Estimert behandlingstid

Faktisk behandlingstid varierer sterkt avhengig av:

* Antall bilder
* Bildeoppløsning
* RAW- eller JPG-inndataformat
* Behandlingsmodus (Free vs Chloros+)
* CPU-hastighet og antall kjerner
* Tilgjengelighet av GPU (kun Chloros+)
* Antall indekser som skal beregnes
* Kompleksitet i eksportformat

### Grove estimater (Chloros+, 12 MP-bilder, moderne CPU)

| Antall bilder | Gratis modus | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 bilder   | 15–20 min | 5–8 min        | 3–5 min        |
| 100 bilder  | 30–40 min | 10–15 min      | 5–8 min        |
| 200 bilder  | 1–1,5 timer | 20–30 min      | 10–15 min      |
| 500 bilder  | 2–3 timer   | 45–60 min      | 20–30 min      |
| 1000 bilder | 4–6 timer   | 1,5–2 timer      | 40–60 min      |

{% hint style="info" %}
**Første kjøring**: Den første behandlingen kan ta lengre tid, da Chloros bygger opp hurtigbuffer og profiler. Senere behandling av lignende datasett vil gå raskere.
{% endhint %}

***

## Vanlige problemer ved oppstart

### Startknappen er deaktivert (gråtonet)

**Mulige årsaker:**

* Ingen bilder importert
* Backend ikke fullstendig startet
* Forrige behandling kjører fortsatt
* Prosjektet ikke fullstendig lastet inn

**Løsninger:**

1. Vent til backend er fullstendig initialisert (sjekk ikonet i hovedmenyen)
2. Kontroller at bildene er importert i filbrowseren
3. Start Chloros på nytt hvis knappen fortsatt er deaktivert
4. Sjekk feilloggen for feilmeldinger

### Behandlingen starter, men mislykkes umiddelbart

**Mulige årsaker:**

* Ingen gyldige bilder i prosjektet
* Ødelagte bildefiler
* Utilstrekkelig diskplass
* Utilstrekkelig minne (RAM)

**Løsninger:**

1. Sjekk feilloggen <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> for feilmeldinger
2. Kontroller tilgjengelig diskplass
3. Prøv å behandle et mindre utvalg av bilder
4. Kontroller at bildene ikke er ødelagte

### Advarsel «Ingen mål oppdaget»

**Mulige årsaker:**

* Glemte å merke målbilder
* Målbildene inneholder ikke synlige mål
* Innstillingene for måldeteksjon er for strenge

**Løsninger:**

1. Gå gjennom [Valg av målbilder](choosing-target-images.md)
2. Merk de aktuelle bildene i målkolonnen
3. Kontroller at målene er synlige i de merkede bildene
4. Juster innstillingene for måldeteksjon om nødvendig

***

## Tips for vellykket behandling

### Før du starter

1. **Test med et lite utvalg først** – Behandle 10–20 bilder for å kontrollere innstillingene
2. **Sjekk ledig diskplass** – Sørg for at det er 2–3 ganger så mye ledig plass som datasettets størrelse
3. **Lukk unødvendige programmer** – Frigjør systemressurser
4. **Kontroller målbildene** – Forhåndsvis merkede mål for å sikre kvaliteten
5. **Lagre prosjektet** – Prosjektet lagres automatisk, men det er god praksis å lagre manuelt

### Under behandlingen

1. **Unngå at systemet går i dvale** – Deaktiver strømsparingsmodusene
2. **Hold Chloros i forgrunnen** – Eller i det minste synlig i oppgavelinjen
3. **Overvåk fremdriften av og til** – Se etter advarsler eller feil
4. **Ikke last inn andre ressurskrevende applikasjoner** – Spesielt med Chloros+ parallellmodus

### Chloros+ GPU-akselerasjon

Hvis du bruker NVIDIA GPU-akselerasjon:

1. Oppdater NVIDIA-driverne til den nyeste versjonen
2. Sørg for at GPU-en har 4 GB+ VRAM
3. Lukk GPU-intensive programmer (spill, videoredigering)
4. Overvåk GPU-temperaturen (sørg for tilstrekkelig kjøling)

***

## Neste trinn

Når behandlingen har startet:

1. **Overvåk fremdriften** – Se [Overvåke behandlingen](monitoring-the-processing.md)
2. **Vent til behandlingen er fullført** – Behandlingen kjører automatisk
3. **Gjennomgå resultatene** – Se [Fullføre behandlingen](finishing-the-processing.md)

For informasjon om hva du skal gjøre under behandlingen, se [Overvåke behandlingen](monitoring-the-processing.md).
