# Prosjektinnstillinger

Prosjektinnstillinger <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> i Chloros lar deg konfigurere alle aspekter av bildebehandling, kalibreringsmåldeteksjon, multispektrale indeksberegninger og eksportalternativer for prosjektet ditt. Disse innstillingene lagres sammen med prosjektet ditt og kan lagres som maler for gjenbruk i flere prosjekter.

## Tilgang til prosjektinnstillinger

Slik får du tilgang til prosjektinnstillinger:

1. Åpne et prosjekt i Chloros
2. Klikk på **Prosjektinnstillinger**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> i venstre sidefelt
3. Innstillingspanelet viser alle tilgjengelige konfigurasjonsalternativer organisert etter kategori

***

## Målregistrering

Disse innstillingene styrer hvordan Chloros registrerer og behandler kalibreringsmål i bildene dine.

### Minimum kalibreringsprøveområde (px)

* **Type**: Tall
* **Område**: 0 til 10 000 piksler
* **Standard**: 25 piksler
* **Beskrivelse**: Angir minimumsarealet (i piksler) som kreves for at et oppdaget område skal anses som et gyldig kalibreringsmålprøve. Mindre verdier vil oppdage mindre mål, men kan øke antall falske positive resultater. Større verdier krever større, tydeligere målområder for oppdagelse.
* **Når du skal justere**:
  * Øk hvis du får falske deteksjoner på små bildeartefakter.
  * Reduser hvis kalibreringsmålene dine vises små i bildene dine og ikke blir oppdaget.

### Minimum målklynging (0-100)

* **Type**: Tall
* **Område**: 0 til 100
* **Standard**: 60
* **Beskrivelse**: Kontrollerer grupperingsterskelen for å gruppere områder med lignende farger når kalibreringsmål oppdages. Høyere verdier krever at flere lignende farger grupperes sammen, noe som resulterer i mer konservativ målopdagelse. Lavere verdier tillater større fargevariasjon innenfor en målgruppe.
* **Når skal du justere**:
  * Øk hvis kalibreringsmål blir delt opp i flere oppdagelser.
  * Reduser hvis kalibreringsmål med fargevariasjon ikke oppdages fullt ut.

***

## Behandling

Disse innstillingene styrer hvordan Chloros behandler og kalibrerer bildene dine.

### Vignettkorreksjon

* **Type**: Avkryssingsboks
* **Standard**: Aktivert (avkrysset)
* **Beskrivelse**: Bruker vignettkorreksjon for å kompensere for mørkere kanter på bildene. Vignettering er et vanlig optisk fenomen der hjørnene og kantene på et bilde virker mørkere enn midten på grunn av linsens egenskaper.
* **Når du skal deaktivere**: Deaktiver bare hvis kamera-/linsekombinasjonen din allerede har brukt vignettkorreksjon, eller hvis du vil korrigere vignettering manuelt i etterbehandlingen.

### Refleksjonskalibrering / hvitbalanse

* **Type**: Avkryssingsboks
* **Standard**: Aktivert (avkrysset)
* **Beskrivelse**: Aktiverer automatisk refleksjonskalibrering ved hjelp av kalibreringsmål som er oppdaget i bildene dine. Dette normaliserer refleksjonsverdiene i datasettet ditt og sikrer konsistente målinger uavhengig av lysforhold.
* **Når du skal deaktivere**: Deaktiver bare hvis du ønsker å behandle rå, ukalibrerte bilder eller hvis du bruker en annen kalibreringsarbeidsflyt.

### Debayer-metode

* **Type**: Nedtrekksmeny
* **Alternativer**:
  * Høy kvalitet (raskere) – For øyeblikket det eneste tilgjengelige alternativet
* **Standard**: Høy kvalitet (raskere)
* **Beskrivelse**: Velger demosaicing-algoritmen som brukes til å konvertere rå Bayer-mønstersensordata til fullfargebilder. Metoden «Høy kvalitet (raskere)» gir en optimal balanse mellom behandlingshastighet og bildekvalitet.
* **Merk**: Ytterligere debayer-metoder kan bli lagt til i fremtidige versjoner av Chloros.

### Minimum intervall for rekalibrering

* **Type**: Tall
* **Område**: 0 til 3600 sekunder
* **Standard**: 0 sekunder
* **Beskrivelse**: Angir minimum tidsintervall (i sekunder) mellom bruk av kalibreringsmål. Når verdien er satt til 0, vil Chloros bruke alle kalibreringsmål som oppdages. Når verdien er satt til en høyere verdi, vil Chloros bare bruke kalibreringsmål som er adskilt med minst dette antallet sekunder, noe som reduserer behandlingstiden for datasett med hyppige kalibreringsmålopptak.
* **Når skal du justere**:
  * Sett til 0 for maksimal kalibreringsnøyaktighet når lysforholdene varierer.
  * Øk (f.eks. til 60–300 sekunder) for raskere behandling når lysforholdene er jevne og du har hyppige kalibreringsmålbilder.

### Tidssoneforskyvning for lyssensor

* **Type**: Tall
* **Område**: -12 til +12 timer
* **Standard**: 0 timer
* **Beskrivelse**: Angir tidssoneforskyvningen (i timer fra UTC) for tidsstempler for lyssensordata. Dette brukes ved behandling av PPK-datafiler (Post-Processed Kinematic) for å sikre korrekt tidssynkronisering mellom bildeopptak og GPS-data.
* **Når skal du justere**: Sett dette til din lokale tidssoneforskyvning hvis PPK-dataene dine bruker lokal tid i stedet for UTC. For eksempel:
  * Pacific Time: -8 eller -7 (avhengig av sommertid)
  * Eastern Time: -5 eller -4 (avhengig av sommertid)
  * Central European Time: +1 eller +2 (avhengig av sommertid)

### Bruk PPK-korreksjoner

* **Type**: Avkryssingsboks
* **Standard**: Deaktivert (ikke avkrysset)
* **Beskrivelse**: Aktiverer bruk av etterbehandlede kinematiske (PPK) korreksjoner fra MAPIR DAQ-opptakere som inneholder GPS (GNSS). Når denne funksjonen er aktivert, vil Chloros bruke alle .daq-loggfiler som inneholder eksponeringspinndata i prosjektkatalogen din og bruke presise geolokasjonskorreksjoner på bildene dine.
* **Krav**: .daq-loggfil med eksponeringspinneoppføringer må være til stede i prosjektkatalogen din
* **Når skal du aktivere**: Det anbefales å alltid aktivere PPK-korreksjon hvis du har eksponeringsfeedbackoppføringer i .daq-loggfilen din.

### Eksponeringspinne 1

* **Type**: Nedtrekksmeny
* **Synlighet**: Kun synlig når «Bruk PPK-korreksjoner» er aktivert OG eksponeringsdata er tilgjengelig for pinne 1
* **Alternativer**:
  * Kameramodellnavn oppdaget i prosjektet
  * «Ikke bruk» – Ignorer denne eksponeringspinnen
* **Standard**: Valges automatisk basert på prosjektkonfigurasjonen
* **Beskrivelse**: Tilordner et bestemt kamera til eksponeringspinne 1 for PPK-tidssynkronisering. Eksponeringspinnen registrerer det nøyaktige tidspunktet når kameraets lukker utløses, noe som er avgjørende for nøyaktig PPK-geolokalisering.
* **Automatisk valg**:
  * Ett kamera + én pinne: Velger automatisk kameraet
  * Ett kamera + to pinner: Pinne 1 tilordnes automatisk til kameraet
  * Flere kameraer: Manuell valg kreves

### Eksponeringspinne 2

* **Type**: Valg fra rullegardinmeny
* **Synlighet**: Kun synlig når «Bruk PPK-korreksjoner» er aktivert OG eksponeringsdata er tilgjengelig for pinne 2
* **Alternativer**:
  * Kameramodellnavn oppdaget i prosjektet
  * «Ikke bruk» – Ignorer denne eksponeringspinnen
* **Standard**: Velges automatisk basert på prosjektkonfigurasjonen
* **Beskrivelse**: Tilordner et bestemt kamera til eksponeringspinne 2 for PPK-tidssynkronisering når du bruker et oppsett med to kameraer.
* **Automatisk valg**:
  * Ett kamera + én pinne: Pinne 2 settes automatisk til «Ikke bruk»
  * Ett kamera + to pinner: Pinne 2 settes automatisk til «Ikke bruk»
  * Flere kameraer: Manuell valg kreves
* **Merk**: Det samme kameraet kan ikke tilordnes både pinne 1 og pinne 2 samtidig.

***

## Indeks

Disse innstillingene lar deg konfigurere multispektrale indekser for analyse og visualisering.

### Legg til indeks

* **Type**: Spesielt indekskonfigurasjonspanel
* **Beskrivelse**: Åpner et interaktivt panel hvor du kan velge og konfigurere multispektrale vegetasjonsindekser (NDVI, NDRE, EVI, etc.) som skal beregnes under bildebehandling. Du kan legge til flere indekser, hver med sine egne visualiseringsinnstillinger.
* **Tilgjengelige indekser**: Systemet inkluderer over 30 forhåndsdefinerte multispektrale indekser, blant annet:
  * NDVI (normaliserte forskjellsvegetasjonsindekser)
  * NDRE (Normaliserte forskjeller RedEdge)
  * EVI (Forbedret vegetasjonsindeks)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * Og mange flere (se [Multispektrale indeksformler](multispectral-index-formulas.md) for fullstendig liste)
* **Funksjoner**:
  * Velg mellom forhåndsdefinerte indeksformler
  * Konfigurer visualiseringsfargegraderinger (LUT – Look-Up Tables)
  * Angi terskelverdier for analyse
  * Opprett tilpassede indeksformler

### Tilpassede formler (Chloros+ funksjon)

* **Type**: Matrise med tilpassede formeldefinisjoner
* **Beskrivelse**: Lar deg opprette og lagre tilpassede multispektrale indeksformler ved hjelp av båndmatematikk. Tilpassede formler lagres med prosjektinnstillingene dine og kan brukes på samme måte som innebygde indekser.
* **Slik oppretter du**:
  1. I indekskonfigurasjonspanelet finner du alternativet for tilpassede formler
  2. Definer formelen ved hjelp av båndidentifikatorer (f.eks. NIR, Red, Green, Blue)
  3. Lagre formelen med et beskrivende navn
* **Formelsyntaks**: Standard matematiske operasjoner støttes, inkludert:
  * Aritmetikk: `+`, `-`, `*`, `/`
  * Parenteser for rekkefølgen av operasjoner
  * Båndreferanser: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Eksport

Disse innstillingene styrer formatet og kvaliteten på eksporterte behandlede bilder.

### Kalibrert bildeformat

* **Type**: Nedtrekksmeny
* **Alternativer**:
  * **TIFF (16-bit)** – Ukomprimert 16-bit TIFF-format
  * **TIFF (32-bit, prosent)** – 32-biters flytende komma TIFF med refleksjonsverdier i prosent
  * **PNG (8-bit)** - Komprimert 8-biters PNG-format
  * **JPG (8-biters)** - Komprimert 8-biters JPEG-format
* **Standard**: TIFF (16-biters)
* **Beskrivelse**: Velger filformat for lagring av behandlede og kalibrerte bilder.
* **Format anbefalinger**:
  * **TIFF (16-bit)**: Anbefales for vitenskapelig analyse og profesjonelle arbeidsflyter. Bevarer maksimal datakvalitet uten komprimeringsartefakter. Best for multispektral analyse og videre behandling i GIS-programvare.
  * **TIFF (32-bit, prosent)**: Best egnet for arbeidsflyter som krever refleksjonsverdier i prosent (0–100 %). Gir maksimal presisjon for radiometriske målinger.
  * **PNG (8-bit)**: Bra for visning på nettet og generell visualisering. Mindre filstørrelser med tapsfri komprimering, men redusert dynamisk område.
  * **JPG (8-bit)**: Minste filstørrelser, best for forhåndsvisning og visning på nettet. Bruker tapsrik komprimering som ikke er egnet for vitenskapelig analyse.

***

## Lagre prosjektmal

Denne funksjonen lar deg lagre gjeldende prosjektinnstillinger som en gjenbrukbar mal.

* **Type**: Tekstinngang + Lagre-knapp
* **Beskrivelse**: Skriv inn et beskrivende navn for innstillingsmalen og klikk på lagre-ikonet. Malen lagrer alle gjeldende prosjektinnstillinger (måldeteksjon, behandlingsalternativer, indekser og eksportformat) for enkel gjenbruk i fremtidige prosjekter.
* **Bruksområder**:
  * Opprett maler for forskjellige kamerasystemer (RGB, multispektral, NIR)
  * Lagre standardkonfigurasjoner for bestemte avlingstyper eller analysearbeidsflyter
  * Del konsistente innstillinger på tvers av et team
* **Slik bruker du**:
  1. Konfigurer alle ønskede prosjektinnstillinger
  2. Skriv inn et malnavn (f.eks. «RedEdge Survey3 NDVI Standard»)
  3. Klikk på lagringsikonet
  4. Malen kan nå lastes inn når du oppretter nye prosjekter

***

## Lagre prosjektmappe

Denne innstillingen angir hvor nye prosjekter lagres som standard.

* **Type**: Visning av katalogbane + Rediger-knapp
* **Standard**: `C:\Users\[Username]\Chloros Projects`
* **Beskrivelse**: Viser den gjeldende standardkatalogen der nye Chloros-prosjekter opprettes. Klikk på redigeringsikonet for å velge en annen katalog.
* **Når du bør endre**:
  * Sett til en nettverksstasjon for teamsamarbeid.
  * Endre til en stasjon med mer lagringsplass for store datasett.
  * Organiser prosjekter etter år, klient eller prosjekttype i forskjellige mapper.
* **Merk**: Endring av denne innstillingen påvirker bare NYE prosjekter. Eksisterende prosjekter forblir på sine opprinnelige plasseringer.

***

## Innstillingspersistens

Alle prosjektinnstillinger lagres automatisk med prosjektfilen din (`.mapir`-prosjektformat). Når du åpner et prosjekt på nytt, gjenopprettes alle innstillinger nøyaktig slik du forlot dem.

### Innstillingshierarki

Innstillinger brukes i følgende rekkefølge:

1. **Systemstandarder** – Innebygde standarder definert av Chloros
2. **Malinnstillinger** – Hvis du laster inn en mal når du oppretter et prosjekt
3. **Lagrede prosjektinnstillinger** – Innstillinger som er lagret med prosjektfilen
4. **Manuelle justeringer** – Eventuelle endringer du gjør under den aktuelle økten

### Innstillinger og bildebehandling

De fleste endringer i innstillingene (spesielt i kategoriene Behandling og Eksport) vil utløse ny behandling av bildene for å gjenspeile de nye innstillingene. Noen innstillinger er imidlertid «kun for eksport» og krever ikke umiddelbar ny behandling:

* Lagre prosjektmal
* Arbeidskatalog
* Kalibrert bildeformat (gjelder ved eksport)

***

## Beste praksis

1. **Start med standardinnstillingene**: Standardinnstillingene fungerer godt for de fleste MAPIR-kamerasystemer og typiske arbeidsflyter.
2. **Opprett maler**: Når du har optimalisert innstillingene for en bestemt arbeidsflyt eller et bestemt kamera, lagrer du dem som en mal for å sikre konsistens på tvers av prosjekter.
3. **Test før full behandling**: Når du eksperimenterer med nye innstillinger, bør du teste på et lite utvalg av bilder før du behandler hele datasettet.
4. **Dokumenter innstillingene dine**: Bruk beskrivende malnavn som angir kamerasystemet, behandlingstypen og tiltenkt bruk (f.eks. «Survey3\_RGB\_NDVI\_Agriculture»).
5. **Valg av eksportformat**: Velg eksportformat basert på sluttbruken:
   * Vitenskapelig analyse → TIFF (16-bit eller 32-bit)
   * GIS-behandling → TIFF (16-bit)
   * Rask visualisering → PNG (8-bit)
   * Deling på nettet → JPG (8-bit)

***

For mer informasjon om multispektrale indekser i Chloros, se siden [Multispektrale indeksformler](multispectral-index-formulas.md).
