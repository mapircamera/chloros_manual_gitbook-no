# Velge målbilder

Å merke hvilke bilder som inneholder kalibreringsmål er et viktig trinn som øker hastigheten på Chloros-behandlingsprosessen betydelig. Ved å forhåndsvelge målbilder eliminerer du behovet for at Chloros skal skanne hvert bilde i datasettet ditt for kalibreringsmål.

## Hvorfor merke målbilder?

### Behandlingshastighet

Uten å merke målbilder må Chloros:

* Skanne hvert eneste bilde i prosjektet ditt
* Kjøre måldeteksjonsalgoritmer på hvert bilde
* Sjekke hundrevis eller tusenvis av bilder unødvendig

**Resultat**: Behandlingen kan ta betydelig lengre tid, spesielt for store datasett.

### Med merkede målbilder

Når du merker av for bestemte bilder i kolonnen Mål:

* Chloros skanner bare de merkede bildene for mål
* Målregistreringen fullføres mye raskere
* Den totale behandlingstiden reduseres betydelig

{% hint style=&quot;success&quot; %}
**Hastighetsforbedring**: Å merke 2-3 målbilder i et datasett med 500 bilder kan redusere måldeteksjonstiden fra over 30 minutter til under 1 minutt.
{% endhint %}

***

## Hvordan merke målbilder

### Trinn 1: Identifiser målbildene dine

Se gjennom de importerte bildene i filbrowseren og identifiser hvilke bilder som inneholder kalibreringsmål.

**Vanlige scenarier:**

* **Mål før opptak**: Opptatt før sesjonen startet
* **Mål etter opptak**: Opptatt etter at sesjonen var fullført
* **Mål i feltet**: Mål plassert innenfor opptaksområdet
* **Flere mål**: 2–3 målbilder per sesjon (anbefalt)

### Trinn 2: Kontroller målkolonnen

For hvert bilde som inneholder et kalibreringsmål:

1. Finn bildet i filbrowser-tabellen.
2. Finn kolonnen **Mål** (kolonnen helt til høyre).
3. Klikk på avmerkingsboksen i målkolonnen for det bildet.
4. Gjenta for alle bilder som inneholder mål.

### Trinn 3: Kontroller valget ditt

Før du fortsetter, må du dobbeltsjekke følgende:

* [ ] Alle bilder med kalibreringsmål er merket av
* [ ] Ingen bilder uten mål er merket av ved en feiltakelse
* [ ] Målene er tydelig synlige i de merkede bildene

***

## Beste praksis for målbilder

### Retningslinjer for målopptak

**Tidspunkt:**

* Ta målbilder umiddelbart før og under opptakssesjonen
* Under de samme lysforholdene som DAQ-lyssensoren
* Ideelt sett bør du ta målbilder så ofte som mulig for å oppnå best mulig resultat. Ellers vil lyssensordataene brukes til å justere kalibreringen over tid.

**Kameraposisjon:**

* Hold kameraet over målet slik at det er sentrert og fyller rundt 40–60 % av bildets sentrum.
* Hold kameraet parallelt/nadir med måloverflaten

**Belysning:**

* Samme omgivelsesbelysning som DAQ-lyssensoren
* Unngå skygger på målflatene
* Ikke blokker lyskilden med kroppen, kjøretøyet eller vegetasjonen
* Overskyet vær gir de mest konsistente resultatene

**Målbetingelser:**

* Hold målpanelene rene og tørre
* Alle 4 panelene skal være tydelig synlige og uten hindringer
* Målene skal være vinkelrette/nadir i forhold til lyskilden hvis mulig

### Hvor mange målbilder?

**Minimum:** 1 målbilde per økt. **Anbefalt:** 3–5 målbilder per økt.

**Beste praksis:**

* Ta 3–5 bilder kort tid etter at lyssensoren begynner å ta opp.
* Roter kameraet mellom opptakene for å få best mulig resultat.
* Valgfritt: Ta bilder med jevne mellomrom midt i økten hvis lysforholdene endrer seg kontinuerlig.

***

## Arbeide med flere kameraer

### Oppsett med to kameraer

Hvis du bruker to MAPIR-kameraer samtidig (f.eks. Survey3W RGN + Survey3N OCN):

1. Ta bilder av målet med **begge kameraene** samtidig.
2. Bruk **samme fysiske mål** for begge kameraene.
3. Merk målbildene for **begge kameratypene** i filbrowseren.
4. Chloros vil bruke passende mål for kalibrering av hvert kamera.

### Kolonne for kameramodell

Kolonnen **Kameramodell** hjelper deg med å identifisere hvilke bilder som kommer fra hvilket kamera:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* osv.

Bruk denne kolonnen til å kontrollere at du har merket mål for hver kameratype i prosjektet ditt.

***

## Innstillinger for måldeteksjon

### Justere deteksjonsfølsomhet

Hvis Chloros ikke oppdager målene dine riktig, kan du justere disse innstillingene i [Prosjektinnstillinger](adjusting-project-settings.md):

**Minimum kalibreringsprøveområde:**

* **Standard**: 25 piksler
* **Øk** hvis du får falske deteksjoner på små gjenstander
* **Reduser** hvis målene ikke blir detektert

**Minimum målklynging:**

* **Standard**: 60
* **Øk** hvis målene blir delt opp i flere deteksjoner
* **Reduser** hvis mål med fargevariasjoner ikke blir fullstendig oppdaget

***

## Vanlige problemer med målbilder

### Problem: Ingen mål oppdaget

**Mulige årsaker:**

* Målbilder ikke merket i filbrowseren
* Målet er for lite i rammen (&lt; 30 % av bildet)
* Dårlig belysning (skygger, gjenskinn)
* Innstillingene for måldeteksjon er for strenge

**Løsninger:**

1. Kontroller at kolonnen Mål er merket for riktige bilder
2. Gjennomgå kvaliteten på målbildene i forhåndsvisningen
3. Ta opp målene på nytt hvis kvaliteten er dårlig
4. Juster innstillingene for måldeteksjon om nødvendig

### Problem: Falske måldeteksjoner

**Mulige årsaker:**

* Hvite bygninger, kjøretøy eller bakkeoverflater forveksles med mål
* Lyse flekker i vegetasjonen
* Deteksjonsfølsomheten er for lav

**Løsninger:**

1. Merk bare faktiske målbilder for å begrense deteksjonsområdet
2. Øk minimumsområdet for kalibreringsprøven
3. Øk minimumsverdien for målklynging
4. Sørg for at målbildene bare viser målet (minimalt med bakgrunnsstøy)

***

## Sjekkliste for verifisering

Før du starter behandlingen, må du verifisere valg av målbilder:

* [ ] Minst 1 målbilde merket per økt
* [ ] Avmerkingsboksene i målkolonnen er merket for alle målbilder
* [ ] Målbilder tatt innenfor samme tidsramme som undersøkelsen
* [ ] Målene er tydelig synlige i forhåndsvisningen når du klikker på dem
* [ ] Alle 4 kalibreringspanelene er synlige i hvert målbilde
* [ ] Ingen skygger eller hindringer på målene
* [ ] For dobbeltkamera: Mål merket for begge kameratyper

***

## Målfri behandling

### Behandling uten kalibreringsmål

Selv om det ikke anbefales for vitenskapelig arbeid, kan du behandle uten mål:

1. La alle avmerkingsboksene i målkolonnen være umerkede
2. **Deaktiver** «Refleksjonskalibrering» i prosjektinnstillingene
3. Vignettkorreksjon vil fortsatt bli brukt
4. Utdataene vil ikke bli kalibrert for absolutt refleksjonsevne

{% hint style=&quot;warning&quot; %}
**Anbefales ikke**: Uten refleksjonskalibrering representerer pikselverdiene bare relativ lysstyrke, ikke vitenskapelige refleksjonsmålinger. Bruk kalibreringsmål for nøyaktige, repeterbare resultater.
{% endhint %}

***

## Neste trinn

Når du har merket målbildene dine:

1. **Gjennomgå innstillingene dine** – Se [Justere prosjektinnstillinger](adjusting-project-settings.md)
2. **Start behandlingen** – Se [Starte behandlingen](starting-the-processing.md)
3. **Overvåk fremdriften** – Se [Overvåke behandlingen](monitoring-the-processing.md)

For mer informasjon om kalibreringsmålene, se [Kalibreringsmål](../calibration-targets.md).
