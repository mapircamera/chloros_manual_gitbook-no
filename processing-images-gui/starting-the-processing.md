# Starte behandlingen

Når du har importert bildene dine, merket kalibreringsmålene og konfigurert prosjektinnstillingene, er du klar til å starte behandlingen. Denne siden veileder deg gjennom oppstarten av Chloros-behandlingsprosessen.

## Sjekkliste før behandling

Før du klikker på Start-knappen, må du kontrollere at alt er klart:

* [ ] **Filer importert** – Alle bildene vises i filbrowseren
* [ ] **Målbilder merket** – Målkolonnen er merket for kalibreringsbilder (eller en `.daq`-opptak importert for LATTICE)
* [ ] **Kameramodeller oppdaget** – Kolonnen «Kameramodell» viser riktige kameraer
* [ ] **Innstillinger konfigurert** – Prosjektinnstillingene er gjennomgått og justert
* [ ] **Indekser valgt** – Ønskede multispektrale indekser er lagt til (om nødvendig)
* [ ] **Eksportformat valgt** – Utdataformat som passer til arbeidsflyten din

{% hint style="info" %}
**Tips**: Klikk deg gjennom noen bilder i filbrowseren for å kontrollere at de er lastet inn riktig før behandling.
{% endhint %}

***

## Starte behandlingen

### Finn Start-knappen

Start-/Avspill-knappen finnes i den øverste overskriftslinjen i Chloros:

* Plassering: Øverst i midten av vinduet
* Ikon: **Avspill/Start-knapp** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Status: Knappen er aktivert (lysende) når den er klar til behandling

### Klikk for å starte

1. Klikk på **Avspill/Start-knappen** i toppmenyen
2. Behandlingen starter umiddelbart
3. Knappen blir til en **Stopp**-knapp under behandlingen
4. Fremdriftslinjen oppdateres og viser behandlingsstatus

{% hint style="success" %}
**Behandlingen er startet**: Når du har klikket, håndterer Chloros automatisk alle trinnene i behandlingen – målgjenkjenning, debayering, kalibrering, indeksberegning og eksport. Programmet gjenkjenner automatisk om prosjektet ditt er Survey3, LATTICE eller en blanding, og bruker riktig behandlingsprosess for hvert kamera.
{% endhint %}

***

## Forstå behandlingsmodusene

Chloros opererer i to forskjellige behandlingsmoduser, avhengig av lisensen din:

### Gratis modus (sekvensiell behandling)

**Tilgjengelig for alle brukere**

**Slik fungerer det:**

* Behandler bildene ett om gangen, sekvensielt
* Enkelttrådet drift
* Lavere minneforbruk

**Fremdriftslinjen viser to trinn:**

1.**Målgjenkjenning** – Søker etter kalibreringsmål
2. **Behandling** – Utfører kalibrering og eksporterer bilder**Behandlingstid:**

* Mye tregere enn Chloros+ parallellmodus
* Egnet for små til mellomstore datasett (< 200 bilder)

### Chloros+-modus (parallellbehandling)

**Krever Chloros+-lisens**

**Slik fungerer det:**

* Behandler flere bilder samtidig ved hjelp av en [4-tråds behandlingspipeline](../processing-architecture/processing-pipeline.md)
* [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md) velger automatisk den optimale strategien for maskinvaren din ved oppstart
* GPU (CUDA)-akselerasjon med NVIDIA-grafikkort
* **Antall arbeidere tilpasses maskinvaren**: GPU-strategier kjører**2–4 samtidige arbeidere** (skalert etter VRAM — 3 på en GPU med 2–12 GB, 2 til 4 på en GPU med 12 GB eller mer); Systemer med kun CPU kjører én arbeider per fysisk kjerne, minus én**Fremdriftslinjen viser 4 trinn** (tilsvarende de 4 trådene i pipelinen):

1. **Detektering** (tråd 1) – Finne kalibreringsmål
2. **Analyse** (tråd 2) – Undersøkelse av bildemetadata og beregning av kalibrering
3. **Kalibrering** (tråd 3) – Debayering, vignettkorrigering, kalibrering, indeksberegning
4. **Eksport** (tråd 4) – Lagring av behandlede bilder og indekser

**Interaksjon med fremdriftslinjen:**

* **Hold musepekeren** over linjen for å se et detaljert nedtrekkspanel med de fire trinnene
* **Klikk** på fremdriftslinjen for å fryse nedtrekkspanelet på plass
* **Klikk igjen** for å oppheve frysingen og skjule panelet**Behandlingstid:**

* Betydelig raskere enn gratis-modus
* GPU-akselerasjon forbedrer hastigheten ytterligere

{% hint style="info" %}
**Chloros+ hastighet**: Parallellbehandling kan være 5–10 ganger raskere enn sekvensiell modus for store datasett. Et prosjekt med 500 bilder som tar 2 timer i gratismodus, kan fullføres på 15–20 minutter med Chloros+.
{% endhint %}

***

## Hva skjer under behandlingen

### Trinn 1: Målgjenkjenning

**Hva Chloros gjør:**

* Skanner bildene du har merket av i «Mål»-kolonnen (alle bildene hvis ingen er merket av)
* Identifiserer kalibreringspanelene i hvert mål
* Henter ut refleksjonsverdier fra målpanelene
* Registrerer tidsstempler for målene for planlegging av kalibrering

**Varighet:** 1–30 sekunder (med merkede mål), 5–30+ minutter (umerkede)

### Trinn 2: Debayering (RAW-konvertering)

**Hva Chloros gjør:**

* Konverterer RAW-data i Bayer-mønster til fullverdige 3-kanalsbilder (LATTICE-monomoduler forblir enkeltbånds – debayering hoppes over for disse med en merknad i loggen)
* Bruker den valgte demosaicing-algoritmen
* Bevarer maksimal bildekvalitet og detaljrikdom

**Varighet:** Varierer avhengig av antall bilder og CPU-/GPU-hastighet

### Trinn 3: Kalibrering

**Hva Chloros gjør:**

* **Vignettkorrigering**: Fjerner mørkningen ved kantene
* **Refleksjonskalibrering**: Normaliserer ved hjelp av målrefleksjonsverdier og/eller DAQ-data for nedadgående stråling
* Anvender korreksjoner på tvers av alle bånd/kanaler
* Bruker passende kalibreringsreferanse for hvert bilde basert på tidsstempel

**Varighet:** Størstedelen av behandlingstiden

### Trinn 4: Indeksberegning

**Hva Chloros gjør:**

* Beregner konfigurerte multispektrale indekser (NDVI, NDRE osv.)
* Anvender båndmatematikk på kalibrerte bilder
* Genererer indeksbilder for hver valgt indeks

**Varighet:** Noen få sekunder per bilde

### Trinn 5: Eksport

**Hva Chloros gjør:**

* Lagrer bearbeidede bilder i det valgte formatet
* **LATTICE fan-out**: hvert rå LATTICE-bilde eksporteres som alle aktiverte produkter i én omgang — debayered, forhåndsvisning, radiance (alltid float32), reflektans
* Skriver filer inn i prosjektets utdatastruktur: `<project>/<camera>/<format>/<Product>_Images/`
* **Beholder kildefilnavnet** — mappen identifiserer produktet, ingen suffiks legges til**Varighet:** Varierer avhengig av eksportformat og filstørrelse***

## Behandlingsatferd

### Automatisk behandlingsrørledning

Når den er startet, kjører hele rørledningen automatisk:

* Ingen brukerinteraksjon nødvendig
* Alle konfigurerte trinn utføres i rekkefølge
* Fremdriftsoppdateringer vises i sanntid
* Eksporterte filer skrives til disken etter hvert som de fullføres — du kan åpne ferdige resultater mens kjøringen fortsetter

### Datamaskinbruk under behandling

**Fri modus:**

* Relativt lav CPU-bruk (enkelttrådet)
* Datamaskinen forblir responsiv for andre oppgaver
* Det er trygt å minimere Chloros og arbeide i andre applikasjoner

**Chloros+ parallellmodus:**

* Høy CPU-bruk i strategiens arbeiderpool
* Med GPU-akselerasjon: Høy GPU-bruk
* Datamaskinen kan være mindre responsiv under behandlingen
* Unngå å starte andre CPU-intensive oppgaver

{% hint style="warning" %}
**Ytelsestips**: For best mulig ytelse fra Chloros+, lukk andre programmer og la Chloros bruke alle systemressursene.
{% endhint %}

### Behandlingen kan ikke settes på pause (men avslutningen er ren)

* Når behandlingen er startet, kan den ikke settes på pause og gjenopptas senere
* Ved å klikke på **Stopp** avsluttes kjøringen fullstendig ved første klikk
* Produkter som allerede er eksportert før stoppet, forblir på disken
* En stoppet kjøring rapporterer nøyaktig hva den har fullført (se linjene `[RUN-SUMMARY]` i loggen)
* En ny kjøring starter prosesskjeden fra begynnelsen

**Planleggingstips:** For svært store prosjekter bør du vurdere å behandle i batcher eller bruke CLI for bedre kontroll.

***

## Overvåke behandlingen

Mens behandlingen pågår, kan du:

* **Følge med på fremdriftslinjen** – Se den samlede fullføringsprosenten
* **Se gjeldende trinn** – Detektering, analyse, kalibrering eller eksport
* **Sjekke loggfanen** – Se detaljerte behandlingsmeldinger og advarsler
* **Forhåndsvise ferdige bilder** – Eksportfiler vises på disken under behandlingen

For detaljert informasjon om overvåking, se [Overvåking av behandlingen](monitoring-the-processing.md).

***

## Stoppe behandlingen

Hvis du trenger å stoppe behandlingen:

### Slik stopper du

1. Finn **Stopp-knappen** (erstatter Start-knappen under behandlingen)
2. Klikk på den én gang – linjen viser **«Stoppes...»** mens det pågående bildet fullføres
3. Kjøringen avsluttes i en endelig stoppet tilstand, og loggen skriver ut en detaljert oversikt over hva som ble fullført

### Når du bør stoppe

**Gyldige grunner til å stoppe:**

* Du oppdaget at feil innstillinger ble brukt
* Du glemte å merke målbildene
* Feil bilder ble importert
* Systemet kjører for sakte eller svarer ikke

**Etter at du har stoppet:**

* Produkter som ble eksportert før stoppet, forblir på disken
* Gjennomgå og rett opp eventuelle problemer, juster innstillingene etter behov
* Start behandlingen på nytt — kjøringen starter fra begynnelsen

***

## Estimert behandlingstid

Den faktiske behandlingstiden varierer sterkt avhengig av:

* Antall bilder
* Bildeoppløsning
* Inndataformat: RAW eller JPG
* Behandlingsmodus (Free vs. Chloros+)
* CPU-hastighet og antall kjerner
* Tilgjengelighet av GPU (kun Chloros+)
* Antall indekser som skal beregnes
* Antall aktiverte eksportprodukter (LATTICE)

### Grove anslag (Chloros+, 12 MP-bilder, moderne CPU)

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
* Backend er ikke fullstendig startet
* Forrige behandling pågår fortsatt
* Prosjektet er ikke fullstendig lastet inn

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

### Kjøringen fullføres, men det skrives ikke ut noen bilder

En kjøring som ba om bildeprodukter, men som ikke skrev ut noen, behandles som en **feil, ikke en suksess** — Chloros rapporterer dette tydelig:

* GUI-loggen viser `[RUN-SUMMARY]`-meldinger som angir den sannsynlige årsaken — ingen bilder importert, ingen mål oppdaget, eller at alle forespurte produkter ble hoppet over som uanvendelige (f.eks. å be om strålingsintensitet/refleksjonsgrad fra kameraer som kun tar RGB-bilder)
* Tilsvarende CLI-melding (`chloros-cli process`) viser `Processing finished but wrote no image products.` og **avsluttes med en verdi ulik null**, slik at skript kan oppdage feilen
* En bevisst kjøring med kun metadata (alle eksportprodukter deaktivert, ingen indekser) regnes fortsatt som en suksess

Se [CLI-referansen](../reference/cli-reference.md#a-run-that-writes-no-images-fails) for fullstendig semantikk.

### Prosjektmappen ligger for dypt i katalogstrukturen for denne datamaskinen

**Kun Windows, nytt i versjon 1.3.0.** Windows leveres med lange baner deaktivert, og da kan ingen filbane være lengre enn 260 tegn. Chloros-produkter ligger opptil ca. 130 tegn under prosjektmappen, så en prosjektmappe som ligger dypere enn det, kan ikke motta produktene sine. I stedet for å kjøre og ikke skrive noe, sjekker Chloros nå før behandlingen starter:

* Å klikke på Start starter ikke en kjøring: fremdriftslinjen viser årsaken, og feilloggen viser `[PROCESS] refused: The project folder is too deep for this computer…`. Meldingen oppgir prosjektmappens lengde og den lengste prosjektmappen som passer.
* Å opprette eller åpne et prosjekt i en mappe som ligger for dypt, selv for prosjektets egne filer, blir også avvist. En meldingsboks forklarer hvorfor, med den lengste prosjektmappen som passer og hvordan man aktiverer lange baner, og ingenting blir opprettet eller åpnet; feilloggen registrerer dette som `[PROJECT] refused: …`. Å duplisere et prosjekt under et lengre navn kan også føre til at grensen overskrides: kopien blir laget, men ikke åpnet, og den samme meldingen forklarer hvorfor.

**Løsninger:**

1. Oppbevar prosjekter i en kortere mappe – endre**Arbeidskatalog** i [Prosjektinnstillinger](../project-settings/project-settings.md) og opprett prosjektet på nytt, eller
2. Slå på lange stier i Windows og start Chloros på nytt: kjør Chloros-installasjonsprogrammet på nytt med **Aktiver lange stier i Windows** merket av (det er merket av som standard), eller bruk gruppepolicy: *Datamaskinkonfigurasjon > Administrative maler > System > Filsystem > Aktiver lange Win32-stier*.

### Advarsel «Ingen mål oppdaget»

**Mulige årsaker:**

* Glemte å merke målbilder
* Målbildene inneholder ikke synlige mål
* Innstillingene for måldeteksjon er for strenge

**Løsninger:**

1. Gå gjennom [Velge målbilder](choosing-target-images.md)
2. Merk de aktuelle bildene i «Mål»-kolonnen
3. Kontroller at målene er synlige i de merkede bildene
4. Juster innstillingene for måldeteksjon om nødvendig

***

## Tips for vellykket behandling

### Før du starter

1. **Test først med et lite utvalg** – Behandle 10–20 bilder for å kontrollere innstillingene
2. **Sjekk ledig diskplass** – Sørg for at det er 2–3 ganger datasettets størrelse ledig (mer hvis alle LATTICE-produkter er aktivert)
3. **Lukk unødvendige programmer** – Frigjør systemressurser
4. **Kontroller målbildene** – Forhåndsvis merkede mål for å sikre kvaliteten
5. **Lagre prosjektet** – Prosjektet lagres automatisk, men det er god praksis å lagre manuelt

### Under behandlingen

1. **Unngå at systemet går i dvale** – Deaktiver strømsparingsmodusene
2. **Hold Chloros i forgrunnen** – Eller i det minste synlig i oppgavelinjen
3. **Følg med på fremdriften av og til** – Se etter advarsler eller feil
4. **Ikke last inn andre ressurskrevende programmer** – Spesielt ikke i Chloros+ parallellmodus

### Chloros+ GPU-akselerasjon

Hvis du bruker NVIDIA GPU-akselerasjon:

1. Oppdater NVIDIA-driverne til den nyeste versjonen
2. Sørg for at GPU-en har minst 4 GB VRAM (minst 7 GB for samtidig «Texture Aware»-debayering)
3. Lukk GPU-krevende programmer (spill, videoredigering)
4. Overvåk GPU-temperaturen (sørg for tilstrekkelig kjøling)

***

## Neste trinn

Når behandlingen har startet:

1. **Overvåk fremdriften** – Se [Overvåking av behandlingen](monitoring-the-processing.md)
2. **Vent til behandlingen er fullført** – Behandlingen kjører automatisk
3. **Gjennomgå resultatene** – Se [Fullføre behandlingen](finishing-the-processing.md)

For informasjon om hva du skal gjøre under behandlingen, se [Overvåke behandlingen](monitoring-the-processing.md).
