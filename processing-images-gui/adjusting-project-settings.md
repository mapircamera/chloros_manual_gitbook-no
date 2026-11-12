# Justere prosjektinnstillinger

Før du behandler bildene dine, er det viktig å konfigurere prosjektinnstillingene slik at de samsvarer med kravene til arbeidsflyten din. Panelet «Prosjektinnstillinger» <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> gir deg omfattende kontroll over kalibrering, behandlingsalternativer, multispektrale indekser og eksportformater.

## Åpne prosjektinnstillingene

1. Åpne prosjektet ditt i Chloros
2. Klikk på **Prosjektinnstillinger**-ikonet <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> i venstre sidefelt
3. Panelet «Prosjektinnstillinger» viser alle konfigurasjonsalternativene

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>Panelet «Prosjektinnstillinger» — Visning, måldeteksjon og behandling</p></figcaption></figure>

{% hint style="info" %}
**Innstillingene lagres automatisk** sammen med prosjektet. Når du åpner et prosjekt på nytt, gjenopprettes alle innstillingene.
{% endhint %}

***

## Hurtigoppsett for vanlige arbeidsflyter

### Standardinnstillinger (anbefales for de fleste brukere)

Standardinnstillingene fungerer godt for typiske arbeidsflyter i Survey3 og LATTICE:

* ✅ **Vignettkorreksjon**: Aktivert
* ✅ **Refleksjonskalibrering / hvitbalanse**: Aktivert (bruker MAPIR-mål og/eller DAQ-lyssensordata)
* ✅ **Debayer-metode**: Standard (Rask, Middels kvalitet)
* ✅ **Eksportformat**: TIFF (16-bit) (TIFF (8-bit) for et prosjekt som utelukkende består av 8-bit)
* ✅ **Alle eksportprodukter**: Aktivert (LATTICE lagrer automatisk fan-out i debayered, forhåndsvisning, radiance og reflektans)

Bare importer bildene dine og start behandlingen med disse standardinnstillingene.

***

## Oversikt over prosjektinnstillinger

Panelet «Prosjektinnstillinger» er organisert i seksjonene nedenfor. To tilleggsseksjoner — **DAQ-lyssensor**og**Array-justering** — vises automatisk når prosjektet ditt inneholder de relevante filene. For fullstendig dokumentasjon, se [Prosjektinnstillinger](../project-settings/project-settings.md).

### Visning

* **Oppløsning på miniatyrbilder**: Oppløsningen på miniatyrbildene i bilde-rutenettet. Alternativer:**Standard (512 px)**,**1024 px**,**2048 px**,**Full oppløsning**. Gjelder kun visning – påvirker aldri behandlingen. Høyere verdier ser skarpere ut når du zoomer inn, men laster inn saktere.

### Målgjenkjenning

Styrer hvordan Chloros identifiserer kalibreringsmål i bildene dine.

**Viktige innstillinger:*** **Minimumsareal for kalibreringsprøve (px)**: Størrelsesterskel for målgjenkjenning (standard:**25**, område 0–10 000)
* **Minimum målgruppering (0–100)**: Likhetsgrense for gruppering av målområder (standard:**60**)**Når du bør justere:**

* Øk prøveområdet hvis du får falske deteksjoner
* Reduser hvis målene ikke blir oppdaget
* Juster grupperingen hvis målene blir delt opp i flere deteksjoner

{% hint style="info" %}
Disse innstillingene er nedtonet når **refleksjonskalibrering / hvitbalanse** er slått av — når denne funksjonen er deaktivert, kjører måldeteksjonen ikke i det hele tatt.
{% endhint %}

### Behandling

Hovedalternativer for bildebehandling og kalibrering.

**Viktige innstillinger:*** **Vignettkorreksjon**: Kompenserer for mørkere kanter på bildet ✅ Anbefalt
* **Refleksjonskalibrering / hvitbalanse**: Kalibrerer bilder ved hjelp av oppdagede mål (Survey3) og/eller data fra DAQ-lyssensoren (LATTICE) ✅ Anbefalt
* **Debayer-metode**: Algoritme for konvertering av RAW til 3-kanals multispektralt
* **Minimalt rekalibreringsintervall**: Minimumstid i sekunder mellom bruk av kalibreringsmål (standard:**0** = bruk alle, område 0–3600)**Ukalibrerte reserveprodukter:**Når et bilde ikke kan refleksjonskalibreres (ingen mål tilgjengelig, eller kalibrering deaktivert), eksporteres det som ett av to reserveprodukter —**nøyaktig ett av paret finnes per kjøring**, valgt av bryteren for vignettkorreksjon:

* **Eksporter sensorrespons**: skriver `Sensor_Response_Images` — brukes når vignettkorreksjon er**av*** **Eksporter vignettkorrigert**: skriver `Vignette_Corrected_Images` — brukes når vignettkorreksjon er**på**Avmerkingsboksen som ikke er i bruk, er nedtonet. Hvis du fjerner merket fra den aktive, vil ikke den filen bli skrevet i det hele tatt.**LATTICE-eksportprodukter** (vises for hvert prosjekt; gjelder LATTICE-opptak):

* **Eksporter debayert**: det lineære debayered-bildet (`Debayered_Images`): kalibreringspakkens mørkeforskyvning fjernes, deretter demosaicert, uten flatfelt eller unmix — den samme filen som en opptak lagrer på sitt debayered-nivå — i sensorens opprinnelige orientering. Gjelder for RGB- og multispektrale moduler.
* **Eksporter forhåndsvisning**: skjermforhåndsvisningen (`Preview_Images`). RGB = hvitbalanse (DAQ-lysstyrke når tilgjengelig, ellers gråverden) + gamma; multispektral = strekking av falske farger.
* **Eksporter radianse**: float32 spektral radianse (`Radiance_Images`, W/m²/sr/nm). Kun multispektrale moduler — gjelder ikke for RGB-masterfiler.
* **Eksporter reflektans**: uint16-reflektans (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0) når en `.daq`-nedstrålingsmåling eller et mål innenfor bildet dekker bildet. Gjelder kun multispektrale moduler.

Alle fire er **aktivert som standard**— én importert LATTICE-råramme fordeles til alle aktiverte og relevante produkter i én enkelt behandlingsomgang. Avkrysningsruten**Eksporter reflektans** er nedtonet når reflektanskalibrering er deaktivert. Innstillinger som er umulige å bruke på grunn av en overordnet innstilling, er alltid nedtonet med et verktøytips som angir hvilken innstilling som må endres.**Avanserte innstillinger:*** **Tidssoneforskyvning for lyssensor**: Antall timer fra UTC for tidsjustering av lyssensoren (standard: 0, område −12 til +12)
* **Bruk PPK-korreksjoner**: Bruker GPS-/eksponeringspin-data fra `.daq`-filer (standard: av)
* **Eksponeringspin 1/2**: Tilordner kameraer til eksponeringspinner for oppsett med to kameraer

{% hint style="info" %}
**LATTICE-inngangsnivået er automatisk.** LATTICE-opptak har sitt behandlingsnivå i XMP-metadata, og behandlingen starter alltid ved råbildet – det er ingenting å konfigurere i brukergrensesnittet. (CLI-flagget `--input-level` finnes som en overstyringsmulighet for avanserte brukere ved opptak med manglende metadata; se [CLI-referansen](../reference/cli-reference.md).)
{% endhint %}

### Debayer-metode

Vi tilbyr for øyeblikket to debayering-metoder i Chloros:

#### Standard (rask, middels kvalitet)

Standard-debayeren behandler raskt, men viser fargestøy fra debayeringen, noe som resulterer i mindre nøyaktige og mer støyende bilder.

#### Teksturbevisst (Langsom, høyeste kvalitet) \[Kun Chloros+]

Teksturbevisst bruker en kantbevisst debayer av høy kvalitet kombinert med en AI/ML-støyreduksjonsmodell som fjerner nesten all støy fra debayeringen. Modellen krever GPU-minne (VRAM) for å kjøre: med **7 GB eller mer VRAM** kan den behandle flere bilder samtidig; under 7 GB behandler den ett bilde om gangen (merkbart tregere). Se [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**LATTICE-opptak bruker alltid standard demosaik.** Det finnes ingen LATTICE-trent «Texture Aware»-modell, så alternativet tilbys ikke for LATTICE-bilder — Survey3-bilder i samme prosjekt kan fortsatt bruke den.
{% endhint %}

### Indeks (multispektrale indekser)

Konfigurer hvilke vegetasjonsindekser som skal beregnes og eksporteres. Nedtrekksmenyen i brukergrensesnittet tilbyr **27 forhåndsdefinerte indeksformler**.**Slik legger du til indekser:**

1. Klikk på**«Legg til indeks»-knappen»**

2. Velg en indeks fra rullegardinmenyen (NDVI, NDRE, GNDVI osv.)
3. Konfigurer visualiseringsinnstillinger (LUT-farger, verdiområder)
4. Legg til flere indekser etter behov

**Populære indekser:*** **NDVI**: Generell vegetasjonshelse (mest vanlig)
* **NDRE**: Tidlig påvisning av stress med RedEdge
* **GNDVI**: Følsom for klorofyllkonsentrasjon
* **OSAVI**: Fungerer godt med synlig jord
* **EVI**: Områder med høy bladarealindeks (LAI)

**Egendefinerte formler:**

* Lag egendefinerte multispektrale indeksformler med båndmatematikk over alle bildekanaler
* Lagre egendefinerte formler for gjenbruk
* Egendefinerte formler er en funksjon i Chloros+; tilgjengeligheten avhenger av hvilket abonnement du har

For alle tilgjengelige indekser og formler – inkludert hvilke navn som kun finnes i GUI og hvilke som også fungerer i CLI/SDK – se [Multispektrale indeksformler](../project-settings/multispectral-index-formulas.md).

### Eksport

Styrer utdatafilformatet.

**Tilgjengelige formater**(innstilling:**Kalibrert bildeformat**, standard**TIFF (16-bit)**):

* **TIFF (16-bit)**: Anbefales for GIS og vitenskapelig analyse
* **TIFF (8-bit)**: 8-bit TIFF
* **TIFF (32-bit, prosent)**: Flytende desimalverdier
* **PNG (8-bit)**: Tapsfri komprimering for visualisering
* **JPG (8-bit)**: Minst mulig filstørrelse, komprimering med tap

Listen viser hva prosjektets bilder kan inneholde: Chloros oppskalerer ikke bitdybden. Alle 8-bit-bilder tilbyr kun TIFF (8-bit), PNG (8-bit) og JPG (8-bit), og standardinnstillingen er TIFF (8-bit). I et blandet prosjekt eksporteres hvert opptak med sin egen dybde eller lavere. Radiance forblir 32-bit flytende.

Utdataene lagres i prosjektmappen, gruppert etter kamera og format: `<project>/<camera>/<format>/<Product>_Images/`. Radiance lagres **alltid** som float32 i mappen `tiff32`, uavhengig av denne innstillingen. Eksporterte filer beholder kildfilnavnet – mappen identifiserer produktet. Multispektrale LATTICE-eksportfiler kan også skrive klampemasker per piksel til en `clamp_masks/`-mappe ved siden av produktmappene. Se [Fullføre behandlingen](finishing-the-processing.md) for den fullstendige utdatastrukturen.

{% hint style="warning" %}
**Avlesning av refleksjonsverdier**: DN-verdien som tilsvarer ρ = 1,0 avhenger av kildekameraet – LATTICE bruker 32768 (merket som XMP `Chloros:PixelScale`), mens Survey3 bruker 65535. Les taggen i stedet for å anta en konstant. Se [Utgangsbildeformater](../output-image-formats.md).
{% endhint %}

### DAQ-lyssensor

Denne delen viser alle DAQ-nedstrålingsfiler (`.daq` / `.csv`) i prosjektet ditt, én rad per fil, med sensormodell, filnavn og diffusorens **cap** korreksjon som gjelder for den aktuelle filen.

* **Overskrivning av kappe (alle filer)**: en enkelt nedtrekksmeny som gjelder for hele prosjektet.**Auto** (standard) bruker den registrerte cap-verdien for hver fil — det antas å være solskinn der ingenting er registrert, men DAQ-U, DAQ-M og DAQ-E leveres uten cap, og Chloros kan ikke se hva som er tilpasset, så behandlingen gir en advarsel om en antatt cap: velg den cap-verdien som ble tilpasset. Valg av grense overstyrer alle filer unntatt DAQ-E-S-filer, der diffusoren er en del av kalibreringen: råopptak korrigeres med den, og opptak som allerede har en grense blir referert på nytt (den registrerte korreksjonen blir tilbakestilt, og den valgte grensen blir brukt).
* Rader gir en advarsel når en registrert kappe var hubens antatte standardinnstilling i stedet for å være bekreftet av operatøren, og når den valgte kappen ikke har noen profil for den aktuelle enhetsmodellen (overstyringen avslås for den filen).

DAQ-opptak laget i fanen «Lyssensorer» legges automatisk til i det åpne prosjektet, og importerte `.daq` / `.csv`-filer vises her så snart de er lagt til.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Nedre prosjektinnstillinger — Indeks, eksportformat, DAQ-delen for lyssensorer og kontrollene for prosjektmal/mappe</p></figcaption></figure>### Matrisejustering

Denne seksjonen vises **kun** når minst ett bilde i prosjektet har den modul-til-modul-justeringstransformasjonen som LATTICE-matriser merker ved opptak (`Chloros:Alignment*` XMP). Den viser hvor mange bilder som har slike merker og hvilket kamera som er referansen, med følgende kontroller:

* **Bruk matrisejustering** (standard: på): forvrenger hvert bearbeidet produkt (debayered / forhåndsvisning / radiance / reflektans / indeks) til matrisens felles referansegeometri. Av = eksporter i sensorens opprinnelige geometri.
* **Beskjær til felles overlapping** (standard: på): beskjær justerte eksporter til området alle modulene deler, slik at hvert bånd har samme fotavtrykk. Av beholder hele sensorområdet (svart utfylling utenfor kilden).
* **Resampling**:**Bilineær (jevn, standard)**,**Nærmeste (bevarer eksakte verdier)**— ingen blanding mellom piksler, for streng radiometrisk analyse — eller**Kubisk (skarpest)**.

***

## Lagre og laste inn innstillinger

### Lagre prosjektmal

Lag maler som kan gjenbrukes for konsistente arbeidsflyter:

1. Konfigurer alle ønskede innstillinger i panelet «Prosjektinnstillinger»
2. Bla ned til seksjonen **«Lagre prosjektmal»** nederst
3. Skriv inn et beskrivende navn på malen (f.eks. «Survey3N\_RGN\_Agriculture»)
4. Klikk på lagringsikonet

**Fordeler:**

* Bruk identiske innstillinger på tvers av flere prosjekter
* Del konfigurasjoner med teammedlemmer
* Oppretthold konsistens ved gjentatte undersøkelser

### Last inn mal i nytt prosjekt

Når du oppretter et nytt prosjekt:

1. Velg **«Nytt prosjekt»** fra hovedmenyen
2. Velg en prosjektmal i det valgfrie malvalgvinduet
3. Alle innstillingene fra malen blir automatisk brukt

### Arbeidskatalog

Innstillingen **«Arbeidskatalog»** angir hvor nye prosjekter opprettes som standard:

* **Standardplassering**: `Documents\MAPIR\Chloros Projects` i Windows-mappen «Dokumenter» (vanligvis `C:\Users\[Username]\Documents\MAPIR\Chloros Projects`; den følger «Dokumenter»-mappen hvis OneDrive eller en roamingprofil flytter den)
* **Endre plassering**: Klikk på redigeringsikonet og velg en ny mappe
* **Delt med CLI**: `chloros-cli` bruker den samme standardinnstillingen for prosjektmappen
* **Hold det kort i Windows**: med mindre lange stier i Windows er aktivert, avvises en prosjektmappe med et navn lengre enn ca. 130 tegn før behandlingen starter — se [Starte behandlingen](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer)
* **Når du bør endre**:
  * Nettverksstasjon for teamsamarbeid
  * Annen stasjon med mer lagringsplass
  * Organisert mappestruktur etter år/kunde

***

## PPK-oppsett (Post-Processed Kinematic)

Hvis du bruker MAPIR DAQ-opptakere med GPS for presis geolokalisering:

### Forutsetninger

* MAPIR DAQ med GPS (GNSS)-modul
* .daq-loggfil med oppføringer for eksponeringspinner
* Kamera koblet til DAQ-eksponeringspinner under opptakssesjonen

### Konfigurasjonstrinn

1. Plasser .daq-loggfilen i prosjektmappen din
2. I Prosjektinnstillinger aktiverer du avmerkingsboksen **«Bruk PPK-korreksjoner»**

3. Angi**«Tidssoneforskjell for lyssensor»** om nødvendig (standard: 0 for UTC)
4. Tilordne kameraer til eksponeringspinnene:
   * **Ett kamera**: Tilordnes automatisk til pinne 1
   * **To kameraer**: Tilordne hvert kamera manuelt til riktig pinne**Tildeling av eksponeringspinner:*** **Eksponeringspinne 1**: Velg kameramodell fra rullegardinmenyen
* **Eksponeringspinne 2**: Velg det andre kameraet eller «Ikke bruk»
* Samme kamera kan ikke tilordnes begge pinnene

{% hint style="warning" %}
**Viktig**: Eksponeringspinnene må tilordnes riktig til de respektive kameraene. Feil tilordning vil føre til feil geolokaliseringsdata.
{% endhint %}

***

## Avanserte scenarier

### Prosjekter med flere kameraer

Når du behandler bilder fra flere MAPIR-kameraer i ett prosjekt:

1. Chloros gjenkjenner automatisk hver kameramodell (både Survey3 og LATTICE)
2. Hvert kamera får passende behandlingsprofiler, og hvert kamera får sin egen utdatamappestruktur
3. PPK: Tilordne hvert Survey3-kamera manuelt til riktig eksponeringspinne
4. Alle kameraer bruker samme eksportformat og indekser

**Eksempler**: Survey3W RGN + Survey3N OCN-rigg med to kameraer, eller et LATTICE-array som kombinerer en RGB-master med smalbåndsmoduler

### Tidsforløpsopptak eller undersøkelser over flere datoer

Ved gjentatte undersøkelser av samme område over tid:

1. Opprett en mal med standardinnstillingene dine
2. Bruk et konsistent oppsett av kalibreringsmål ved hver økt
3. Behandle hver dato som et eget prosjekt
4. Bruk identiske innstillinger for sammenlignbare resultater
5. Eksporter i samme format for tidsanalyse

### Store datasett

For prosjekter med mange bilder (500+):

* Vurder å dele opp i mindre prosjekter etter dato eller område
* Bruk parallellbehandling i Chloros+ for raskere resultater
* Vurder å bruke CLI eller API for automatisering av batchbehandling
* Juster minimumsintervallet for rekalibrering for å redusere tiden det tar å oppdage målene

***

## Kontrollere innstillingene dine

Før du begynner å behandle, bør du gå gjennom disse viktige innstillingene:

* [ ] Kameramodellen er riktig gjenkjent i filbrowseren
* [ ] Vignettkorreksjon er aktivert
* [ ] Refleksjonskalibrering er aktivert
* [ ] For Survey3: minst ett kalibreringsmålbilde er importert og sjekket; for «LATTICE»: et mål og/eller en `.daq`-nedstrålingsregistrering er til stede
* [ ] Ønskede multispektrale indekser er lagt til
* [ ] Eksportformat som passer til arbeidsflyten din
* [ ] PPK-innstillinger er konfigurert (hvis du bruker .daq med eksponeringshendelser)

***

## Neste trinn

Når innstillingene er konfigurert:

1. **Merk kalibreringsmålbilder** – Se [Velge målbilder](choosing-target-images.md)
2. **Start behandlingen** – Se [Starte behandlingen](starting-the-processing.md)
3. **Overvåk fremdriften** – Se [Overvåke behandlingen](monitoring-the-processing.md)

For fullstendige detaljer om alle tilgjengelige innstillinger, se referansedokumentasjonen [Prosjektinnstillinger](../project-settings/project-settings.md).
