# Valg av målbilder

Å merke hvilke bilder som inneholder kalibreringsmål er et avgjørende trinn som gir en betydelig hastighetsøkning i Chloros-behandlingsprosessen. Ved å forhåndsvelge målbilder unngår du at Chloros må skanne hvert enkelt bilde i datasettet ditt for å finne kalibreringsmål.

## Hvorfor merke målbilder?

### Behandlingshastighet

Uten å merke målbilder må Chloros:

* Skanne hvert eneste bilde i prosjektet ditt
* Kjøre algoritmer for måldeteksjon på hvert bilde
* Sjekke hundrevis eller tusenvis av bilder unødvendig

**Resultat**: Behandlingen kan ta betydelig lengre tid, spesielt for store datasett.

### Med merkede målbilder

Når du merker av i Mål-kolonnen for bestemte bilder:

* Skanner Chloros kun de merkede bildene for mål
* Målgjenkjenningen fullføres mye raskere
* Den totale behandlingstiden reduseres betydelig

{% hint style="success" %}
**Hastighetsforbedring**: Å merke 2–3 målbilder i et datasett på 500 bilder kan redusere målgjenkjennings-tiden fra over 30 minutter til under 1 minutt.
{% endhint %}

***

## Hvordan merke målbilder

### Trinn 1: Identifiser målbildene dine

Gå gjennom de importerte bildene i filbrowseren og identifiser hvilke bilder som inneholder kalibreringsmål.

**Vanlige scenarier:*** **Mål før opptak**: Opptatt før du starter økten
* **Mål etter opptak**: Opptatt etter at økten er fullført
* **Mål i feltet**: Mål plassert innenfor opptaksområdet
* **Flere mål**: 2–3 målbilder per økt (anbefalt)

### Trinn 2: Sjekk målkolonnen

For hvert bilde som inneholder et kalibreringsmål:

1. Finn bildet i tabellen i filbrowseren
2. Finn **Mål**-kolonnen (kolonnen helt til høyre)
3. Klikk på avmerkingsboksen i Mål-kolonnen for det bildet
4. Gjenta for alle bilder som inneholder mål

### Trinn 3: Bekreft valget ditt

Før behandling, dobbeltsjekk:

* [ ] Alle bilder med kalibreringsmål er merket av
* [ ] Ingen bilder uten mål er merket av ved en feiltakelse
* [ ] Målene er tydelig synlige i de merkede bildene

***

## Beste praksis for målbilder

### Retningslinjer for opptak av målbilder

**Tidspunkt:**

* Ta målbilder umiddelbart før og gjennom hele opptakssesjonen
* Under de samme lysforholdene som DAQ-lyssensoren
* Ideelt sett bør du ta bilder av målene så ofte som mulig for å oppnå de beste resultatene. Ellers vil lyssensordataene brukes til å justere kalibreringen over tid.

**Kameraposisjon:**

* Hold kameraet over målet slik at det er sentrert og fyller rundt 40–60 % av bildets sentrum.
* Hold kameraet parallelt med/rett under målets overflate

**Belysning:**

* Samme omgivelsesbelysning som DAQ-lyssensoren
* Unngå skygger på måloverflatene
* Ikke blokker lyskilden med kroppen, kjøretøyet eller vegetasjon
* Overskyet vær gir de mest konsistente resultatene

**Målforhold:**

* Hold målpanelene rene og tørre
* Alle 4 panelene skal være tydelig synlige og uhindret
* Målene skal være vinkelrette/nadir i forhold til lyskilden hvis mulig

### Hvor mange målbilder?

**Minimum:**1 målbilde per økt.**Anbefalt:** 3–5 målbilder per økt.**Anbefalt fremgangsmåte:**

* Ta 3–5 bilder kort tid etter at lyssensoren har startet opptaket
* Roter kameraet mellom opptakene for å oppnå best mulig resultat
* Valgfritt: med jevne mellomrom midt i økten hvis lysforholdene endrer seg kontinuerlig

***

## Arbeid med flere kameraer

### Oppsett med to kameraer

Hvis du bruker to MAPIR-kameraer samtidig (f.eks. Survey3W RGN + Survey3N OCN):

1. Ta bilder av målet med **begge kameraene** samtidig
2. Bruk **samme fysiske mål** for begge kameraene
3. Merk målbildene for **begge kameratypene** i filbrowseren
4. Chloros vil bruke passende mål for kalibrering av hvert kamera

### Kolonnen Kameramodell

Kolonnen **Kameramodell** hjelper deg med å identifisere hvilke bilder som kommer fra hvilket kamera:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* osv.

Bruk denne kolonnen til å kontrollere at du har merket mål for hver kameratype i prosjektet ditt.

***

## Innstillinger for måldeteksjon

### Justere deteksjonsfølsomhet

Hvis Chloros ikke oppdager målene dine riktig, kan du justere disse innstillingene i [Prosjektinnstillinger](adjusting-project-settings.md):**Minimumsområde for kalibreringsprøve:*** **Standard**: 25 piksler
* **Øk** hvis du får falske deteksjoner på små artefakter
* **Reduser** hvis målene ikke blir oppdaget**Minimum målgruppering:*** **Standard**: 60
* **Øk** hvis målene blir delt opp i flere deteksjoner
* **Reduser** hvis mål med fargevariasjoner ikke blir fullstendig oppdaget***

## Vanlige problemer med målbilder

### Problem: Ingen mål oppdaget

**Mulige årsaker:**

* Målbilder ikke merket i filbrowseren
* Målet er for lite i bildet (&lt; 30 % av bildet)
* Dårlig belysning (skygger, gjenskinn)
* Innstillingene for måldeteksjon er for strenge

**Løsninger:**

1. Kontroller at kolonnen «Mål» er merket for riktige bilder
2. Gjennomgå kvaliteten på målbildene i forhåndsvisningen
3. Ta nye bilder av målene hvis kvaliteten er dårlig
4. Juster innstillingene for måldeteksjon om nødvendig

### Problem: Feilaktige måldeteksjoner

**Mulige årsaker:**

* Hvite bygninger, kjøretøy eller bakgrunnsdekke som forveksles med mål
* Lyse flekker i vegetasjonen
* Deteksjonsfølsomheten er for lav

**Løsninger:**

1. Merk kun faktiske målbilder for å begrense deteksjonsområdet
2. Øk minimumsarealet for kalibreringsprøven
3. Øk minimumsverdien for målgruppering
4. Sørg for at målbildene kun viser målet (minimalt med forstyrrende elementer i bakgrunnen)

***

## Sjekkliste for verifisering

Før du starter behandlingen, må du verifisere utvalget av målbilder:

* [ ] Minst 1 målbilde merket per økt
* [ ] Avkryssingsrutene i målkolonnen er merket av for alle målbilder
* [ ] Målbilder tatt innenfor samme tidsramme som undersøkelsen
* [ ] Målene er tydelig synlige i forhåndsvisningen når du klikker på dem
* [ ] Alle 4 kalibreringspaneler er synlige i hvert målbilde
* [ ] Ingen skygger eller hindringer på målene
* [ ] For dobbeltkamera: Mål merket for begge kameratypene

***

## Behandling uten mål

### Behandling uten kalibreringsmål

Selv om det ikke anbefales for vitenskapelig arbeid, kan du behandle uten mål:

1. La alle avmerkingsboksene i målkolonnen være umerkede
2. **Deaktiver** «Reflektanskalibrering» i prosjektinnstillingene
3. Vignettkorreksjon vil fortsatt bli brukt
4. Utdata vil ikke bli kalibrert for absolutt reflektans

{% hint style="warning" %}
**Ikke anbefalt**: Uten reflektanskalibrering representerer pikselverdiene kun relativ lysstyrke, ikke vitenskapelige reflektansmålinger. Bruk kalibreringsmål for nøyaktige, repeterbare resultater.
{% endhint %}

***

## Neste trinn

Når du har merket målbildene dine:

1. **Gjennomgå innstillingene dine** – Se [Justere prosjektinnstillinger](adjusting-project-settings.md)
2. **Start behandlingen** – Se [Starte behandlingen](starting-the-processing.md)
3. **Overvåk fremdriften** – Se [Overvåke behandlingen](monitoring-the-processing.md)

For mer informasjon om kalibreringsmålene, se [Kalibreringsmål](../calibration-targets.md).
