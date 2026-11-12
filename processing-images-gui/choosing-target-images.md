# Velge målbilder

Når du merker hvilke bilder som inneholder kalibreringsmål, forteller du Chloros nøyaktig hvor de skal lete etter dem. Når minst ett bilde er merket av i kolonnen «Mål», skanner Chloros **kun de merkede bildene** — så ved å merke målene kan du både få raskere behandling og unngå at kartleggingsbilder forveksles med et mål.

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

## Hvorfor merke målbilder?

### Merkingen styrer skanningen

Når du merker av i kolonnen «Mål» for bestemte bilder:

* Chloros skanner kun de merkede bildene for mål
* Målgjenkjenningen fullføres mye raskere
* Kartleggingsbilder kan ikke gi falske målgjenkjenninger

Hvis **ingen** bilder er merket av, går Chloros tilbake til å skanne hvert eneste bilde i prosjektet:

* Målgjenkjenningsalgoritmer kjøres på hvert bilde
* Hundrevis eller tusenvis av bilder blir sjekket unødvendig
* Behandlingen tar betydelig lengre tid, spesielt for store datasett

{% hint style="success" %}
**Hastighetsforbedring**: Å merke 2–3 målbilder i et datasett på 500 bilder kan redusere måldeteksjonstiden fra over 30 minutter til under 1 minutt.
{% endhint %}

***

## Slik merker du målbilder

### Trinn 1: Identifiser målbildene dine

Gå gjennom de importerte bildene i filutvalget og identifiser hvilke bilder som inneholder kalibreringsmål.

**Vanlige scenarier:**

* **Mål før opptak**: Opptatt før sesjonen starter
* **Mål etter opptak**: Opptatt etter at økten er fullført
* **Mål i feltet**: Mål plassert innenfor opptaksområdet
* **Flere mål**: 2–3 målbilder per økt (anbefalt)

### Trinn 2: Merk av i «Target»-kolonnen <img src="../.gitbook/assets/image (33).png" alt="" data-size="original">

For hvert bilde som inneholder et kalibreringsmål:

1. Finn bildet i tabellen i filbrowseren
2. Finn **Target**-kolonnen (kolonnen helt til høyre)
3. Merk av i avmerkingsboksen i «Target»-kolonnen for det aktuelle bildet
4. Gjenta for alle bilder som inneholder mål

### Trinn 3: Kontroller valget ditt

Før behandling, dobbeltsjekk:

* [ ] Alle bilder med kalibreringsmål er merket av
* [ ] Ingen bilder som ikke er mål er merket av ved en feiltakelse
* [ ] Målene er tydelig synlige i de merkede bildene

***

## LATTICE: Mål er valgfrie når en DAQ tar opp

For LATTICE-multispektralkameraer er et kalibreringsmål i bildet **en av to** mulige refleksjonsreferanser:

* **Mål i bildet**: når et merket målbilde oppfyller kvalitetskravene (QA) til «Chloros», blir målet den**absolutte reflektansreferansen** for bildene rundt det.
* **Nedadrettet lys fra DAQ**: når det ikke er noe mål til stede (eller QA mislykkes), beregner Chloros i stedet reflektansen ut fra den nedadrettede strålingsintensiteten fra DAQ-lyssensoren (ρ = π·L/E). Hvis en `.daq`- eller DAQ-M `.csv`-opptak dekker opptakene dine, får du kalibrert reflektans**uten noen målbilder i det hele tatt**.

Denne automatiske oppførselen er standardinnstillingen. I CLI / SDK tilsvarer dette `--reflectance-source auto`; du kan også tvinge frem `target` (streng — ingen DAQ-erstatning) eller `daq` (DAQ-autoritativ). Se [CLI-referansen](../reference/cli-reference.md#per-product-export-toggles-lattice-multispectral).

**LATTICE-målgeometrier**: i tillegg til den klassiske paneldeteksjonen som brukes for Survey3, støtter LATTICE-behandlingen**ArUco-merkede mål**,**mål med fast ROI**og**stripemål**, konfigurert per prosjekt.**Målte** refleksjonsskanninger per enhet kan leveres etter serienummer (CLI: `--target-reflectance-dir`, én `<serial>.csv` per målenhet), med de nominelle T3/T4P-spektrene som reserve.

{% hint style="info" %}
**F988-modul**: F988-refleksjonen kalibreres ved hjelp av et refleksjonspanel i scenen: båndet ligger utenfor det kalibrerte området til DAQ-lyssensoren, så Chloros bruker din siste panelopptak og beholder den mellom panelobservasjonene. Hvis en F988-modul behandles kun med DAQ, avviser Chloros DAQ-basert reflektans for det båndet (hopp over grunn `dls-uncalibrated-band-988`) – panelarbeidsflyten er den støttede fremgangsmåten.
{% endhint %}

***

## Anbefalte fremgangsmåter for målbilder

### Retningslinjer for opptak av målbilder

**Tidspunkt:**

* Ta bilder av målet umiddelbart før og gjennom hele opptakssesjonen
* Under de samme lysforholdene som DAQ-lyssensoren
* Ta helst målbilder så ofte som mulig for å oppnå de beste resultatene. Ellers vil dataene fra lyssensoren brukes til å justere kalibreringen over tid.

**Kameraposisjon:**

* Hold kameraet over målet slik at det er sentrert og fyller omtrent 40–60 % av bildets sentrum.
* Hold kameraet parallelt med/rett under måloverflaten

**Belysning:**

* Samme omgivelsesbelysning som for DAQ-lyssensoren
* Unngå skygger på måloverflatene
* Ikke blokker lyskilden med kroppen, kjøretøyet eller vegetasjon
* Overskyet vær gir de mest konsistente resultatene

**Målets tilstand:**

* Hold målpanelene rene og tørre
* Alle panelene på målet (f.eks. alle 4 på en T4) skal være tydelig synlige og uten hindringer
* Målene skal om mulig være vinkelrett på/rett under lyskilden

### Hvor mange målbilder?

**Minimum:**1 målbilde per økt.**Anbefalt:** 3–5 målbilder per økt.**Anbefalt tidsplan:**

* Ta 3–5 bilder kort tid etter at lyssensoren har startet opptaket
* Roter kameraet mellom opptakene for å oppnå best mulig resultat
* Valgfritt: med jevne mellomrom midt i økten hvis lysforholdene endrer seg kontinuerlig

***

## Arbeide med flere kameraer

### Oppsett med to kameraer

Hvis du bruker to MAPIR-kameraer samtidig (f.eks. Survey3W RGN + Survey3N OCN):

1. Ta målbilder med **begge kameraene** samtidig
2. Bruk **samme fysiske mål** for begge kameraene
3. Merk målbildene for **begge kameratypene** i filutvalget
4. Chloros vil bruke passende mål for kalibrering av hvert kamera

### Kolonnen «Kameramodell»

Kolonnen **«Kameramodell»** hjelper deg med å identifisere hvilke bilder som kommer fra hvilket kamera:

* Survey3W\_RGN
* Survey3N\_OCN
* LATT-M3M-L41-F550
* LATT-M3C-L87-FRGN
* osv.

Bruk denne kolonnen til å kontrollere at du har merket mål for hver kameratype i prosjektet ditt.

***

## Innstillinger for måldeteksjon

### Justere deteksjonsfølsomhet

Hvis Chloros ikke oppdager målene dine riktig, må du justere disse innstillingene i [Prosjektinnstillinger](adjusting-project-settings.md):

**Minimalt kalibreringsprøveområde (px):**

* **Standard**: 25 piksler
* **Øk** hvis du får falske deteksjoner på små artefakter
* **Reduser** hvis målene ikke blir oppdaget

**Minimum målgruppering (0–100):**

* **Standard**: 60
* **Øk** hvis målene deles opp i flere registreringer
* **Reduser** hvis mål med fargevariasjoner ikke registreres fullstendig

{% hint style="info" %}
**Tips fra CLI**: `chloros-cli process` aksepterer de samme innstillingene (`--min-target-size`, `--target-clustering`), og flagget `--target`/`--targets` merker en hel inngangsmappe som kun for målpanelet. Se [CLI-referansen](../reference/cli-reference.md).
{% endhint %}

***

## Vanlige problemer med målbilder

### Problem: Ingen mål oppdaget

**Mulige årsaker:**

* Målbilder er ikke merket i filutvalget
* Målet er for lite i bildet (< 30 % av bildet)
* Dårlig belysning (skygger, gjenskinn)
* Innstillingene for måldeteksjon er for strenge

**Løsninger:**

1. Kontroller at «Mål»-kolonnen er merket av for de riktige bildene
2. Kontroller kvaliteten på målbilde i forhåndsvisning
3. Ta nye bilder av målene hvis kvaliteten er dårlig
4. Juster innstillingene for måldeteksjon om nødvendig

### Problem: Feilaktige måldeteksjoner

**Mulige årsaker:**

* Hvite bygninger, kjøretøy eller bakgrunnsvegetasjon forveksles med mål
* Lyse flekker i vegetasjonen
* Deteksjonsfølsomheten er for lav

**Løsninger:**

1. Merk kun faktiske målbilder – kun merkede bilder blir skannet
2. Øk minimumsarealet for kalibreringsprøven
3. Øk minimumsverdien for målgruppering
4. Sørg for at målbildene kun viser målet (minimalt med forstyrrende elementer i bakgrunnen)

***

## Sjekkliste for verifisering

Før du starter behandlingen, må du verifisere utvalget av målbilder:

* [ ] Minst 1 målbilde merket per økt (eller, for LATTICE, en `.daq`/`.csv`-opptak som dekker økten)
* [ ] Avmerkingsboksene i målkolonnen er merket av for alle målbilder
* [ ] Målbilder er tatt innenfor samme tidsramme som undersøkelsen
* [ ] Målene er tydelig synlige i forhåndsvisningen når du klikker på dem
* [ ] Alle kalibreringspaneler er synlige i hvert målbilde
* [ ] Ingen skygger eller hindringer på målene
* [ ] For dobbeltkamera: Målene er merket for begge kameratypene

***

## Behandling uten mål

### LATTICE: Med en DAQ-opptak

Hvis en DAQ-lyssensor registrerte nedadrettet strålingsintensitet under LATTICE-opptakene dine, trengs det ikke noe mål:

1. Importer `.daq`-filen (eller DAQ-M `.csv`) sammen med bildene
2. La «Target»-kolonnen være avmerket
3. Refleksjonen beregnes automatisk ut fra DAQ-referansen for nedadrettet stråling
4. Strålingsintensitet trenger aldri et mål eller en DAQ – den kommer utelukkende fra kameraets fabrikkkalibrering

### Behandling uten noen referanse

Du kan også behandle uten mål og uten en DAQ:

1. La alle avmerkingsboksene i «Mål»-kolonnen være avmerket
2. **Deaktiver** «Reflektanskalibrering / hvitbalanse» i prosjektinnstillingene – da hoppes måldeteksjonen helt over
3. Vignettkorreksjon vil fortsatt bli brukt
4. Utdataene vil ikke bli kalibrert for absolutt reflektans (LATTICE multispektral eksporterer fortsatt debayered-, forhåndsvisnings- og radiance-produkter)

{% hint style="warning" %}
**Anbefales ikke for vitenskapelig arbeid med «Survey3»**: Uten reflektanskalibrering representerer pikselverdiene i «Survey3» kun relativ lysstyrke, ikke vitenskapelige reflektansmålinger. Bruk kalibreringsmål (eller, for LATTICE, en DAQ-lyssensor) for nøyaktige, repeterbare resultater.
{% endhint %}

***

## Neste trinn

Når du har merket målbildene dine:

1. **Gjennomgå innstillingene dine** – Se [Justere prosjektinnstillinger](adjusting-project-settings.md)
2. **Start behandlingen** – Se [Starte behandlingen](starting-the-processing.md)
3. **Følg med på fremdriften** – Se [Overvåke behandlingen](monitoring-the-processing.md)

For mer informasjon om selve kalibreringsmålene, se [Kalibreringsmål](../calibration-targets.md).
