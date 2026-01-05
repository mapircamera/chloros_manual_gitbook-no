# Starte behandlingen

Når du har importert bildene, merket kalibreringsmålene og konfigurert prosjektinnstillingene, er du klar til å starte behandlingen. Denne siden veileder deg gjennom oppstarten av Chloros-behandlingsprosessen.

## Sjekkliste før behandling

Før du klikker på Start-knappen, må du kontrollere at alt er klart:

* [ ] **Filer importert** – Alle bildene vises i filbrowseren
* [ ] **Målbilder merket** – Målkolonnen er sjekket for kalibreringsbilder
* [ ] **Kameramodeller oppdaget** – Kameramodellkolonnen viser riktige kameraer
* [ ] **Innstillinger konfigurert** – Prosjektinnstillinger gjennomgått og justert
* [ ] **Indekser valgt** – Ønskede multispektrale indekser lagt til (om nødvendig)
* [ ] **Eksportformat valgt** – Utdataformat som passer for arbeidsflyten din

{% hint style=&quot;info&quot; %}
**Tips**: Klikk gjennom noen bilder i filbrowseren for å kontrollere at de er lastet inn riktig før behandling.
{% endhint %}

***

## Starte behandlingen

### Finn startknappen

Start-/avspillingsknappen finnes i den øverste overskriftslinjen i Chloros:

* Plassering: Øverst i midten av vinduet
* Ikon: **Avspillings-/startknapp** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
* Status: Knappen er aktivert (lys) når den er klar til behandling

### Klikk for å starte

1. Klikk på **Spill av/Start-knappen** i den øverste overskriften
2. Behandlingen starter umiddelbart
3. Knappen blir deaktivert (grå) under behandlingen
4. Fremdriftslinjen oppdateres og viser behandlingsstatus

{% hint style=&quot;success&quot; %}
**Behandling startet**: Når du har klikket, håndterer Chloros automatisk alle behandlingstrinn – måldeteksjon, debayering, kalibrering, indeksberegning og eksport.
{% endhint %}

***

## Forstå behandlingsmoduser

Chloros fungerer i to forskjellige behandlingsmoduser, avhengig av lisensen din:

### Gratis modus (sekvensiell behandling)

**Tilgjengelig for alle brukere**

**Slik fungerer det:**

* Behandler bilder ett om gangen, sekvensielt
* Enkeltrådet drift
* Lavere minnebruk

**Fremdriftslinjen viser to trinn:**

1.**Måldeteksjon** – Skanner etter kalibreringsmål
2. **Behandling** – Bruker kalibrering og eksporterer bilder**Behandlingstid:**

* Mye tregere enn Chloros+ parallellmodus
* Egnet for små til mellomstore datasett (&lt; 200 bilder)

### Chloros+ modus (parallellbehandling)

**Krever Chloros+ lisens**

**Slik fungerer det:**

* Behandler flere bilder samtidig
* Multitrådet drift (opptil 16 parallelle arbeidere)
* Utnytter flere CPU-kjerner
* Valgfri GPU (CUDA)-akselerasjon med NVIDIA-grafikkort

**Fremdriftslinjen viser 4 trinn:**

1.**Deteksjon** – Finne kalibreringsmål
2. **Analyse** – Undersøke bildemetadata og forberede pipeline
3. **Kalibrering** – Bruk av korreksjoner og kalibreringer
4. **Eksport** – Lagring av behandlede bilder og indekser**Interaksjon med fremdriftslinjen:*** **Hold musen** over linjen for å se et detaljert nedtrekkspanel med 4 trinn
* **Klikk** på fremdriftslinjen for å fryse nedtrekkspanelet på plass
* **Klikk igjen** for å frigjøre og skjule panelet**Behandlingstid:**

* Betydelig raskere enn gratis modus
* Skaleres med antall CPU-kjerner
* GPU-akselerasjon forbedrer hastigheten ytterligere

{% hint style=&quot;info&quot; %}
**Chloros+ Hastighet**: Parallell behandling kan være 5-10 ganger raskere enn sekvensiell modus for store datasett. Et prosjekt med 500 bilder som tar 2 timer i gratis modus, kan fullføres på 15-20 minutter med Chloros+.
{% endhint %}

***

## Hva skjer under behandlingen

### Trinn 1: Målregistrering

**Hva Chloros gjør:**

* Skanner merkede målbilder (eller alle bilder hvis ingen er merket)
* Identifiserer de 4 kalibreringspanelene i hvert mål
* Ekstraherer refleksjonsverdier fra målpanelene
* Registrerer måltidsstempler for kalibreringsplanlegging

**Varighet:** 1–30 sekunder (med merkede mål), 5–30+ minutter (umerkede)

### Trinn 2: Debayering (RAW-konvertering)

**Hva Chloros gjør:**

* Konverterer RAW-Bayer-mønsterdata til fullstendige RGB-bilder
* Bruker demosaicing-algoritme av høy kvalitet
* Bevarer maksimal bildekvalitet og detaljrikdom

**Varighet:** Varierer avhengig av antall bilder og CPU-hastighet

### Trinn 3: Kalibrering

**Hva Chloros gjør:*** **Vignettkorreksjon**: Fjerner mørkningen av linsen i kantene
* **Refleksjonskalibrering**: Normaliserer ved hjelp av målrefleksjonsverdier
* Bruker korreksjoner på tvers av alle bånd/kanaler
* Bruker passende kalibreringsmål for hvert bilde basert på tidsstempel

**Varighet:** Størstedelen av behandlingstiden

### Trinn 4: Indeksberegning

**Hva Chloros gjør:**

* Beregner konfigurerte multispektrale indekser (NDVI, NDRE osv.)
* Bruker båndmatematikk på kalibrerte bilder
* Genererer indeksbilder for hver valgt indeks

**Varighet:** Noen få sekunder per bilde

### Trinn 5: Eksport

**Hva Chloros gjør:**

* Lagrer kalibrerte bilder i valgt format
* Eksporterer indeksbilder med konfigurerte LUT-farger
* Skriver filer til undermapper for kameramodeller
* Bevarer originale filnavn med suffikser

**Varighet:** Varierer avhengig av eksportformat og filstørrelse***

## Behandlingsatferd

### Automatisk behandlingsprosess

Når prosessen er startet, kjører hele prosessen automatisk:

* Ingen brukerinteraksjon nødvendig
* Alle konfigurerte trinn utføres i rekkefølge
* Fremdriftsoppdateringer vises i sanntid

### Datamaskinbruk under behandling

**Fri modus:**

* Relativt lav CPU-bruk (enkeltrådet)
* Datamaskinen forblir responsiv for andre oppgaver
* Det er trygt å minimere Chloros og arbeide i andre applikasjoner

**Chloros+ Parallell modus:**

* Høy CPU-bruk (flertrådet, opptil 16 kjerner)
* Med GPU-akselerasjon: Høy GPU-bruk
* Datamaskinen kan være mindre responsiv under behandlingen
* Unngå å starte andre CPU-intensive oppgaver

{% hint style=&quot;warning&quot; %}
**Ytelsestips**: For best mulig ytelse av Chloros+, lukk andre applikasjoner og la Chloros bruke alle systemressursene.
{% endhint %}

### Behandlingen kan ikke pauses

**Viktige begrensninger:**

* Når behandlingen er startet, kan den ikke pauses.
* Du kan avbryte behandlingen, men fremdriften går tapt.
* Delvise resultater lagres ikke.
* Du må starte på nytt fra begynnelsen hvis du avbryter.

**Planleggingstips:** For svært store prosjekter bør du vurdere å behandle i batcher eller bruke CLI for bedre kontroll.***

## Overvåke behandlingen

Mens behandlingen pågår, kan du:

* **Se fremdriftslinjen** – Se total fullføringsprosent
* **Se gjeldende trinn** – Oppdage, analysere, kalibrere eller eksportere
* **Sjekke loggfanen** – Se detaljerte behandlingsmeldinger og advarsler
* **Forhåndsvise fullførte bilder** – Noen eksportfiler kan vises under behandlingen

For detaljert informasjon om overvåking, se [Overvåking av behandlingen](monitoring-the-processing.md).

***

## Avbryte behandlingen

Hvis du trenger å stoppe behandlingen:

### Slik avbryter du

1. Finn **Stopp/Avbryt-knappen** (erstatter Start-knappen under behandlingen)
2. Klikk på Stopp-knappen
3. Behandlingen stoppes umiddelbart
4. Delvise resultater forkastes

### Når du bør avbryte

**Gyldige grunner til å avbryte:**

* Du oppdaget at feil innstillinger ble brukt
* Du glemte å merke målbildene
* Feil bilder ble importert
* Systemet kjører for sakte eller svarer ikke

**Etter avbrudd:**

* Gjennomgå og løse eventuelle problemer
* Juster innstillingene etter behov
* Start behandlingen på nytt fra begynnelsen
* For å få en optimal opplevelse, lukk Chloros helt og start på nytt

{% hint style=&quot;warning&quot; %}
**Ingen delvise resultater**: Avbrytelse forkaster all fremgang. Chloros lagrer ikke delvis behandlede bilder.
{% endhint %}

***

## Estimert behandlingstid

Den faktiske behandlingstiden varierer sterkt avhengig av:

* Antall bilder
* Bildeoppløsning
* RAW vs JPG-inndataformat
* Behandlingsmodus (Free vs Chloros+)
* CPU-hastighet og antall kjerner
* GPU-tilgjengelighet (kun Chloros+)
* Antall indekser som skal beregnes
* Eksportformatets kompleksitet

### Grove estimater (Chloros+, 12 MP-bilder, moderne CPU)

| Antall bilder | Gratis modus | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 bilder   | 15-20 min | 5-8 min        | 3-5 min        |
| 100 bilder  | 30-40 min | 10-15 min      | 5-8 min        |
| 200 bilder  | 1-1,5 timer | 20-30 min      | 10-15 min      |
| 500 bilder  | 2-3 timer   | 45-60 min      | 20-30 min      |
| 1000 bilder | 4-6 timer   | 1,5-2 timer      | 40-60 min      |

{% hint style=&quot;info&quot; %}
**Første kjøring**: Den første behandlingen kan ta lengre tid, da Chloros bygger opp cache og profiler. Senere behandling av lignende datasett vil gå raskere.
{% endhint %}

***

## Vanlige problemer ved oppstart

### Startknappen er deaktivert (grå)

**Mulige årsaker:**

* Ingen bilder importert
* Backend er ikke fullstendig startet
* Forrige behandling kjører fortsatt
* Prosjektet er ikke fullstendig lastet inn

**Løsninger:**

1. Vent til backend er fullstendig initialisert (sjekk hovedmenyikonet)
2. Kontroller at bildene er importert i filbrowseren
3. Start Chloros på nytt hvis knappen fortsatt er deaktivert
4. Sjekk feilsøkingsloggen for feilmeldinger

### Behandlingen starter, men mislykkes umiddelbart

**Mulige årsaker:**

* Ingen gyldige bilder i prosjektet
* Korrupte bildefiler
* Utilstrekkelig diskplass
* Utilstrekkelig minne (RAM)

**Løsninger:**

1. Sjekk feilsøkingsloggen <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> for feilmeldinger
2. Kontroller at det er tilstrekkelig diskplass
3. Prøv å behandle en mindre delmengde av bildene
4. Kontroller at bildene ikke er ødelagte

### Advarsel «Ingen mål oppdaget»

**Mulige årsaker:**

* Glemte å merke målbildene
* Målbildene inneholder ikke synlige mål
* Innstillingene for måldeteksjon er for strenge

**Løsninger:**

1. Gå gjennom [Velge målbilder](choosing-target-images.md)
2. Merk passende bilder i kolonnen Mål
3. Kontroller at målene er synlige i merkede bilder
4. Juster innstillingene for måldeteksjon om nødvendig

***

## Tips for vellykket behandling

### Før du starter

1. **Test med et lite delsett først** – Behandle 10–20 bilder for å kontrollere innstillingene
2. **Kontroller tilgjengelig diskplass** – Sørg for at du har 2–3 ganger så mye ledig plass som datasettets størrelse
3. **Lukk unødvendige programmer** – Frigjør systemressurser
4. **Kontroller målbildene** – Forhåndsvis merkede mål for å sikre kvaliteten
5. **Lagre prosjektet** – Prosjektet lagres automatisk, men det er lurt å lagre manuelt.

### Under behandlingen

1. **Unngå at systemet går i dvale** – Deaktiver strømsparingsmodus.
2. **Hold Chloros i forgrunnen** – Eller i det minste synlig i oppgavelinjen.
3. **Overvåk fremdriften av og til** – Sjekk om det er advarsler eller feil.
4. **Ikke last inn andre tunge applikasjoner** – Spesielt med Chloros+ parallellmodus

### Chloros+ GPU-akselerasjon

Hvis du bruker NVIDIA GPU-akselerasjon:

1. Oppdater NVIDIA-drivere til den nyeste versjonen
2. Sørg for at GPU har 4 GB+ VRAM
3. Lukk GPU-intensive programmer (spill, videoredigering)
4. Overvåk GPU-temperaturen (sørg for tilstrekkelig kjøling)

***

## Neste trinn

Når behandlingen har startet:

1. **Overvåk fremdriften** – Se [Overvåke behandlingen](monitoring-the-processing.md)
2. **Vent til behandlingen er fullført** – Behandlingen kjører automatisk
3. **Gjennomgå resultatene** – Se [Fullføre behandlingen](finishing-the-processing.md)

For informasjon om hva du skal gjøre under behandlingen, se [Overvåke behandlingen](monitoring-the-processing.md).
