# Flerkameraoppsett

Et LATTICE **oppsett**består av to eller flere LATTICE-kameraer som er koblet sammen til én synkronisert enhet. Ett kamera fungerer som**master**: det sender ut en GPIO-utløserpuls via en felles synkroniseringslinje (standard**Line2**), slik at hvert kamera eksponerer i samme øyeblikk. Chloros

legger til PTP-tidssynkronisering, en live-forhåndsvisning (ruter per kamera eller en enkelt justert flerbånds-kompositt) og synkronisert opptak — hver opptaksrunde produserer én **bildegruppe** der alle kameraene deler samme tidsstempel og bilde-ID (rapportert som `fid:N` på opptakutgangen).

Det er gjennom matriser at monokameraer (M3M) produserer vegetasjonsindekser – ett kamera bidrar med ett bånd, og matrisen justerer dem inn i en flerbåndsstabel. Se [Monokameraer og vegetasjonsindekser](mono-indices.md).

Det finnes tre likeverdige måter å koble til en matrise på, og alle kjører den samme «smart-prep»-prosessen:

| Grensesnitt | Inngangspunkt |
| --- | --- |
| GUI | Fanen «Kameraer» → **Koble til matrise** (blå knapp) |
| CLI

| `chloros-cli lattice array-connect --serials SN1,SN2,…` (første serienummer = master) |
| Python

-SDK

| `connect_array(serials=[…])` → `ArraySession` (første serienummer = master) |

Smart-prep utfører: en test av nettverkskapasiteten (ICMP DF-ping + GVSP-test), valg av synkroniseringsnivå, en projeksjon av bildefrekvensen ved den forespurte bildestørrelsen, beslutningen om pakket 12-bits transport (se [Pakket 12-biters transport](#packed-12-bit-transport)), aktivering av PTP, automatisk valg av pikselformat per kamera, automatisk eksponeringsinnstilling basert på hvert kameras lagrede tilstand, samt GPIO-utløserkonfigurasjon på Line2.

**Full sensor som standard (nytt i 1.3.0).** Connect beholder den bildestørrelsen du ba om — hele 2048×1536-sensoren, uten binning og uten beskjæring, med mindre du velger noe annet — og lar bildefrekvensen falle til det verten og nettverket leverer. Den utfører aldri binning eller beskjæring på egen hånd (versjon 1.2.0 aktiverte automatisk binning under 1,5 fps). Om du vil bruke binning eller et mindre ROI, er opp til deg: I Connect-dialogboksen velger du ett av alternativene, og «Projected Outcome» viser bildefrekvensen det gir; `array-connect` viser de raskeste alternativene med deres forventede frekvenser.

{% hint style="info" %}
Kameraene må være tilgjengelige på forbindelsen før noe av dette fungerer — se [Koble til kameraer](connecting.md) for oppdagelse, adressering og nedlasting av kalibrering ved første tilkobling. For rigger med flere kameraer er innstillingene for mottaksringen på verts-NIC-en like viktige som koblingshastigheten; den komplette tabellen over symptomer og løsninger finnes i [CLI

-referanse § Oppsett og justering av verts-NIC](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Dialogboksen «Array Connect»

Fanen «Kameraer» → **Koble til array**åpner en veiviser i tre trinn:**Velg → Visningsmodus → Innstillinger**.

### Trinn 1 — Velg master og slaver

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered (an FRGB among them). Table showing Camera / Serial / IP / Master radio / Slave checkbox columns in camera order (M3C first, FRGB at the top), with the FRGB pre-selected as master and the green "Sync cable found on N cameras. Any of them can be master; the RGB camera (else the first) is pre-selected." probe banner visible above the table. -->



Dialogboksen skanner nettverket så snart den åpnes («Skanner nettverk...»), og sjekker deretter synkroniseringskablene («Sjekker synkroniseringskabler...») for å finne ut hvilke kameraer som er på M8-synkroniseringslinjen. Du trenger minst **2 kameraer** for å sette opp et array.

M8-synkroniseringskablene møtes i ett knutepunkt, så **ethvert kamera som er koblet til, kan være master**. Testen fyller inn rollene på forhånd — RGB

-kameraet (`-FRGB`) som master når det er tilkoblet, ellers det første tilkoblede kameraet, og de andre tilkoblede kameraene som slaver — og viser ett av tre banner:

| Banner | Betydning |
| --- | --- |
| «Synkroniseringskabel funnet på {n} kameraer. Hvilket som helst av dem kan være master; RGB

-kameraet (ellers det første) er forhåndsvalgt.» (grønt) | Proben fant kameraene på synkroniseringslinjen; avmerkingsboksene for master og slave er allerede merket av. |
| «Ingen master oppdaget – sjekk GPIO-kabelen» (oransje) | Ingen kameraer registrerte en utløserpuls; sjekk synkroniseringskablene. Rollene er fortsatt forhåndsutfylt (RGB

-kameraet, ellers det første kameraet i listen, som master), og du kan endre dem. |
| «Ingen synkroniseringskabel: {serialnumre}» (oransje) | De oppførte kameraene har ingen synkroniseringskabel tilkoblet. |

Kameratabellen har kolonnene **Kamera / Serienummer / IP / Master (radio) / Slave (avkryssingsrute)**. Radene er oppført i**kamerarekkefølge** — M3C (Bayer)-kameraer før M3M (mono); blant M3C, FRGB, FRGN, FOCN, deretter FNGB; M3M etter filterbølgelengde, laveste først (F450 før F808). Kameralisten i sidefeltet og rutenettets standardoppsett bruker samme rekkefølge.

* Velg nøyaktig **én master**og**én eller flere slaver**. Hvis du klikker på den gjeldende masterens radio igjen, fjernes markeringen. Det tar ett klikk å velge en annen master: den nye masteren forlater slavene, og den forrige masteren blir en slave (med mindre den ikke har synkroniseringskabel).
* Et kamera merket med **«Ingen synkroniseringskabel»** kan aldri velges som slave – en slave uten utløserkabling vil vente på synkroniseringslinjen i all evighet og levere et dødt signal. Koble i stedet dette kameraet til som et frittstående kamera.
* Kameraer som allerede er koblet til som frittstående, blir *ikke* deaktivert: array-tilkobling frigjør den frittstående sesjonen og åpner kameraet på nytt inne i arrayet.

**Neste: Visningsmodus →**aktiveres når en master og minst én slave er valgt.**Skann på nytt** kjører oppdagingsprosessen og ledningssøkningen på nytt.

{% hint style="warning" %}
**Avbryt** er deaktivert mens en skanning eller kabelsøk pågår — å avbryte midt i kabelsøket kan føre til at kamera-SDK

-et krasjer på LATTICE-kameraets fastvare. Vent til spinneren er ferdig.
{% endhint %}

### Trinn 2 — Visningsmodus

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->



| Modus | Hva du får |
| --- | --- |
| **Separate kameraer** | Én live-rute per kamera, alle utløses samtidig slik at bildene forblir synkroniserte. Hvert kamera beholder sin egen farge og innstillinger. |
| **Kombinerte kameraer** *(standard)* | En enkelt flis som gjengir den justerte flerbånds-NDVI

-/indekssammensetningen. Kameraene deler matrisens farge. |

Visningsmodusen endrer kun presentasjonen av live-forhåndsvisningen — opptaket fungerer på samme måte i begge modusene.

### Trinn 3 — Matriseinnstillinger og forventet resultat

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Trigger Rate / Wire Budget controls (1.3.0 has no "Pin resolution" checkbox), right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->



Når du går inn i denne scenen, ber Chloros

backend-systemet om en **anbefaling**og bruker den:**full sensor — ingen binning, ingen ROI-beskjæring — så lenge bildefrekvensen tillater det**, uansett hvilken hastighet verten og kabelen leverer. Først når ingen hastighet kan levere full sensor, reduserer anbefalingen bildet, og da foretrekker den binning fremfor ROI-beskjæring, fordi binning bevarer hele synsfeltet. Derfra bytter du oppløsning mot bildefrekvens. Hver endring du gjør, kjører analysen på nytt i sanntid og oppdaterer panelet**Forventet resultat** til høyre.

Venstre kolonne — innstillinger:

| Kontroll | Valg | Standard | Merknader |
| --- | --- | --- | --- |
| **ROI (synsfelt)** | Full (2048×1536) / Halv (1024×768) / Kvart (512×384) | Full | Sensorbeskjæring: Halv/kvart beskjæring til et mindre område med opprinnelig pikselavstand. |
| **Binning** | 1× / 2× (sum 2×2) / 4× (sum 4×4) | 1× | Maskinvarebinning: 2×2 = fullt synsfelt til en fjerdedel av ledningskostnaden; 4×4 = fullt synsfelt til 1/16. Skjult hvis kameraene ikke støtter binning. |
| **Bilde på ledningssiden** (avlesning) | — | — | Bredden × høyden etter binning som faktisk sendes over ledningen, avkortet til multipler av 16 (minimum 64). |
| **Utløserfrekvens** | 0,5–60 fps, trinn 0,1 | tom = auto | Masterens utløserfrekvens. La feltet stå tomt for å la Chloros

beregne den. |
| **Kabelkapasitet**| 20–2000 MB/s, trinn 10 | tomt = auto | Hvor mye verten faktisk kan håndtere, i MB/s —**det ene tallet som hele array-tildelingen avhenger av.** Registreres automatisk fra nettverkskortet. Senk det hvis arrayet rapporterer om ødelagte rammer: den registrerte verdien overvurderer USB-kort og delte svitsjer. Endring av dette kjører prosjekteringen på nytt i sanntid. Feltet åpnes med det budsjettet den siste matrisen var koblet til (lagret med prosjektet), så et budsjett du har korrigert én gang, beholdes. |

Avmerkingsboksen **Pin-oppløsning** fra versjon 1.2.0 er fjernet: «Koble til» beholder alltid den rammestørrelsen du velger og senker i stedet bildefrekvensen.

Høyre kolonne — **Projisert resultat**:

* **Synkroniseringsnivå** — «Samtidig opptak» (grønn), «Samtidig opptak (FTD-forsinket utsending)» (grønn), «Forsinket opptak (100 ms avvik)» (gul) eller «Konfigurasjonen er for stor» (rød). Chloros

velger nivå: samtidig opptak når ledningen kan deles med en hastighet som kameraene kan tilpasses til (bildefrekvensen synker etter hvert som hvert kameras andel reduseres), og forskjøvet opptak bare når dette ikke er mulig — slik at et array kobles til hvilket som helst nettverk, med den bildefrekvensen nettverket tillater.
* **fps-projeksjon** — bildefrekvensen som oppstillingen forventes å kjøre med. Ved samtidig opptak er det ett tall: frekvensen kommer først, og automatisk eksponering fyller den perioden den gir, deretter økes forsterkningen, slik at en mørk scene ikke senker den (se [Hvordan automatisk eksponering fordeler eksponering og forsterkning](camera-settings.md)). Ved forskjøvet opptak vises det som et intervall («svakt → sterkt»), fordi eksponeringstiden til hvert kamera forlenger forskjøvet i en svakt opplyst scene.
* **NIC-linje** — koblingshastighet og vedvarende gjennomstrømning («NIC {mbps} Mbps · vedvarende {N} MB/s»).
* **Sim-emit-burst-sjekk** — kan vertsmaskinens NIC-ring absorbere én samtidig burst fra alle kameraene («Sim-emit-burst: X MB · NIC-ring brukbar: Y MB ✓/✗»).
* **Kontroll av ledningsbudsjett** — samlet etterspørsel i stabil tilstand mot det kollisjonssikre taket for ledningen («Ledningsbudsjett: {etterspørsel} MB/s etterspurt av {n} kameraer · tak {tak} MB/s ✓/✗ overtegnet»).
* **«Maks antall kameraer på denne kabelen: {n} — fastsatt av båndbreddegrensen per kamera, så binning øker ikke denne.»** — vises når du nærmer deg (eller overskrider) taket for antall kameraer.
* **«BILLEDER VIL GÅ TAPT med disse innstillingene.&quot;**— rød advarsel med begrunnelsen fra backend, pluss en liste over hindringer og blå**forslag til løsninger** («For å få plass til dette oppsettet på nettverket» / «For å muliggjøre samtidig opptak»).**Bruk og koble til** er sperret inntil det foreligger en prognose, og etiketten forteller deg hvorfor den avviser:

| Knappetekst | Betydning | Hva som faktisk hjelper |
| --- | --- | --- |
| «Analyserer...» | Analysen pågår fortsatt. | Vent. |
| **«For mange kameraer for dette nettverket»**| Nettverket kan ikke håndtere oppsettet, selv ikke ett kamera om gangen (aggregatkontroll mislyktes): langt flere kameraer enn det kan håndtere, eller andre kameraer som strømmer på det. | Færre kameraer, jumbo-rammer fra ende til ende eller et raskere nettverkskort.**Et mindre ROI vil IKKE hjelpe** — se nedenfor. |
| **«Reduser ROI for å aktivere»** | Rammer vil gå tapt med disse innstillingene (burst-/ringkontroll mislyktes). | Reduser ROI, øk binning eller fikse nettverkskortets mottaksring. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->



Under tilkoblingen kan det vises et grønt **kalibreringsnedlastingspanel** med en fremdriftslinje for hver serienummer: første gang et kamera kobles til en maskin, henter Chloros

sin fabrikkkalibreringspakke på ca. 3,8 MB fra kameraet via GigE (ca. 70 sekunder per kamera). Kameraer som er lagret i hurtigbufferen, viser aldri dette panelet; ved mer enn seks kameraer ruller radene. Se [Koble til kameraer](connecting.md).

### Pakket 12-bits overføring

12-bits formater (`BayerRG12`, `Mono12`) overføres normalt som 16-bits ord. **Pakket**transport sender de samme 12 bitene i 1,5 byte per piksel og pakker dem ut på verten, slik at hver ramme tar 25 % kortere tid å sende – ved full sensor, omtrent en tredjedel flere rammer per sekund.**De lagrede dataene er identiske.** Array connect bestemmer dette automatisk (`auto`): den pakker kun når det er fordelaktig og verten har ledig CPU-kapasitet til å pakke ut. GUI-en har ingen innstilling for dette; fra CLI

-en bruker du `array-connect --packed {auto,on,off}` (standard `auto`). 8-biters og 16-biters formater pakkes aldri.

## Båndbredde: hvor mange kameraer som får plass

Hvor mye et array kan håndtere, avhenger av nettverksforbindelsen, ikke av Chloros

, så planleggingstallene finnes i maskinvarehåndboken: **[Planlegging av array-båndbredde](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Hva Chloros

gjør med dem: tilkoblingsdialogen kjører en nettverkssjekk, beregner den oppnåelige bildefrekvensen og velger et nivå som passer. Et array kobles til på hvilket som helst nettverk: på en treg forbindelse tilpasses hvert kamera til sin andel av båndbredden, og bildefrekvensen synker, i stedet for at arrayet blir avvist eller bildene reduseres. Bare et array som båndbredden ikke kan håndtere selv ett kamera av gangen, blir avvist (**For mange kameraer for dette nettverket**) i stedet for at pakker blir droppet uten varsel — se panelet for forventet utfall beskrevet ovenfor.

## Når bilder forsvinner

Et kamera kan være fraværende fra en publisert gruppe av to helt forskjellige årsaker,
og de krever motsatte løsninger. Chloros

teller dem separat i stedet for å rapportere ett
«ufullstendig» tall som ikke spesifiserer noen av dem:

| Hva skjedde | Hva det betyr | Hvor du skal lete |
| --- | --- | --- |
| **Korrupt**— rammen ankom, men var strukturelt feil | GVSP-pakketap på nettverksbanen |**Kabelbudsjettet**, NIC-mottaksringen, jumbo-rammer, svitsjen |
| **Kom aldri frem**— ingen ramme kom i det hele tatt | Kameraet utløste ikke, eller ingenting kom ut av det |**M8-synkroniseringskabelen**, synkroniseringslinjen, om alle enhetene er aktiverte |

Fordelingen vurderes på nytt hvert 10. sekund mens arrayet strømmer. Over 5 % blir det
loggført med begge tallene oppgitt, og hver ødelagt buffer rapporteres første gang det
skjer per kamera, deretter summeres det opp en gang i minuttet slik at en lang økt forblir lesbar.

**Korrupte bilder med null «aldri mottatt» betyr at utløsning og kabelsynkronisering er perfekt**og at hvert tapt bilde skyldes nettverksbanen. Løsningen er å senke**Wire Budget** og
koble til på nytt.

{% hint style="warning" %}
**Å senke utløserfrekvensen hjelper ikke mot korrupte bilder.** Kameraets pakkefrekvens
fastsettes én gang, ved tilkobling. Å senke utløserfrekvensen endrer hvor ofte en burst
skjer, ikke hvor raskt selve bursten sendes ut på kabelen. På et målt oppsett med 4 kameraer førte en
5× reduksjon i utløserfrekvensen ikke til noen endring, mens en reduksjon av wire budget fra 240 til
200 MB/s reduserte andelen ødelagte bilder på samme rigg fra 10,4 % til null.
{% endhint %}

Et array som er i drift, kan ikke omplanlegge seg selv – koble fra og koble til på nytt, slik at
tilkoblingstidvelgeren kan tilpasse seg det nye budsjettet.

### USB-nettverkskort har en øvre grense

Et USB-Ethernet-kort oppgir sin *Ethernet*-koblingshastighet, men hva det faktisk
kan opprettholde, er begrenset av USB-bussen og driveren. En USB 10GbE-dongle ble tidligere kreditert
med omtrent 1000 MB/s gjennomstrømning — et tall ingen noensinne hadde målt — og å kjøre
fire kameraer mot dette fantom-reservoaret førte til feil i 6–18 % av bildene, mens arrayet
fortsatt rapporterte en god målbildfrekvens. USB-tilkoblede adaptere er nå begrenset til
**200 MB/s** på Windows

og NVIDIA Jetson, og **400 MB/s** på Linux

x86_64. Grensen er et absolutt tall snarere enn en prosentandel, fordi begrensningen ligger i
bussen: en USB 1 GbE-adapter oppnår omtrent 80 MB/s og påvirkes ikke.

Hvis verten din faktisk er raskere enn grensen, må du øke **Wire Budget** for å angi dette.

## PTP-tidssynkronisering

*Synkronisering* av bilder skjer via maskinvaretriggeren; **PTP** (IEEE 1588 PTPv2) sørger for sammenlignbare *tidsstempler* på tvers av alle enheter. Dette er aktivert som standard ved tilkobling til arrayet:

* **Chloros

-vertsbackenden kjører PTP-grandmasteren**. LATTICE-kameraer og DAQ-E-lyssensorer fungerer som slaver til denne i domene 0, slik at bildets tidsstempler og DAQ-spektra havner på én klokke (~1 ms).
* `--no-ptp` (CLI

) deaktiverer det for benkarbeid — tidsstempler på tvers av kameraene er da **ikke** sammenlignbare.
* Kontroller synkroniseringsstatusen med CLI

:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity, health (OK, or why its packets are not reaching the network)
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Fanen «Kameraer» har i seg selv ingen PTP-indikator; synkroniseringsinformasjon per kamera som vises der, er de skrivebeskyttede feltene **Rolle**(Master/Slave),**Synkroniseringslinje** og matrisens funksjonsnivå. DAQ-E PTP-status vises i sensordetaljene på fanen «Lyssensorer».

## Live-visningen

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows in camera order (M3C before M3M, monos by wavelength), and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

av matrisen Hovedvisningsområdet tilbyr to oppsett (bytt mellom dem i topplinjen): **rutenettvisning**(hver flis er en celle; dra for å omorganisere når rutenettlåsen er ulåst) og**listevisning**(rader i full bredde: et**kombinert**array utgjør én rad – dets sammensatte rute – mens et**separat**array gir**hvert medlem sin egen rad**, med sin egen live-feed og sin egen kalibreringsmåltabell; under matrisene vises ett frittstående kamera – det aktive). Glidebryteren**Feed Zoom** justerer størrelsen på rutene; ved en cellebredde under 200 px skjules navn/fps-overleggene automatisk. Medlemskameraene vises i sidefeltet og i rutenettets standardoppsett i kamerarekkefølge (M3C før M3M, FRGB først, monokromkameraer etter bølgelengde); hvis du låser opp rutenettet og drar, overstyrer dette likevel rutenettets rekkefølge.

<!-- SCREENSHOT-NEEDED: list view of a connected SEPARATE array (1.3.0): one full-width row per member camera, each with its live feed and its own Calibration Target table below it. -->



**Separat modus** viser én flis per kamera. Hver flis viser:

* kameranavnet (øverst til venstre),
* en **fps-visning** (øverst til høyre) — dette er kameraets *faktiske opptakshastighet* som rapporteres av backend-systemet, ikke forhåndsvisningsfrekvensen (live-forhåndsvisning er begrenset til 30 fps uavhengig av opptakshastigheten),
* en statusprikk — grønn (strømming) / gul (laster) / rød (feil),
* en **spinner for utdatert bilde** når det ikke har kommet et nytt bilde på 2 sekunder — normalt i ca. 5 sekunder etter tilkobling/frakobling mens backenden omfordeler båndbredden mellom kameraene.**Kombinert modus**viser én sammensatt flis: backenden utfører debayering, skalering, justering, støyreduksjon, konverterer til strålingsintensitet per bånd (pluss DLS-refleksjonsgrad når en lyssensor er tilknyttet), evaluerer matrisens indeksuttrykk, bruker LUT-en og strømmer resultatet som MJPEG. Inntil det første justerte bildet vises, viser ruten sin status: «Forbereder matrise…», «Kalibrerer justering…», «Venter på første bilde…», eller – hvis flisens automatiske justeringsforsøk (ca. ett minutt) går ut – «Justering nødvendig» med en**Kalibrer justering**-knapp.

Matriseflisens fps-visning rapporterer også strømmens tilstand:**«Bruker innstillinger…»**mens en endring av innstillinger (format, binning, ROI) starter kameraene på nytt,**«Stanset»**når matrisen er aktiv, men har sluttet å levere bildegrupper, og**«Stoppet»**når den har sluttet å streame – koble til arrayet på nytt i så fall. Når den spektrale separasjonen måtte begrense et bånd vist i sammensetningen på en stor andel av pikslene (25 % eller mer), viser flisen**«Bånddata med lav pålitelighet · N %»**: indeksverdier basert på dette båndet er tvilsomme (vanligvis har ett bånd mottatt for lite lys).

Nyttige fakta om kombinert modus:

* Komposittbildet er registrert i forhold til **hovedkameraets**bilde. AE-ROI-fokusering og punktmåling på komposittbildet er nøyaktige for hovedkameraet og omtrentlige for slavekameraene; bruk**Split View** (matriseinnstillinger → «Vis medlemskameraer») for pikselnøyaktige fliser per kamera uten å åpne ekstra kameratilkoblinger.
* **Display Layers**(matriseinnstillinger; standard av) lar deg velge et forgrunns- og bakgrunnslag – hvilket som helst medlemskamera eller**Index**. Med forgrunn = Index viser piksler utenfor LUT Min/Max bakgrunnslaget.
* **Renderoppløsning** (standard 720p) angir høyden på direktestrømmen *og* størrelsen på den lagrede komposittfilene ved eksport. Bilder per kamera eksporteres alltid i full oppløsning.
* Justering beregnes per økt og lagres aldri. Chloros

justerer **hvert array automatisk etter tilkobling, i begge visningsmodusene**, så snart medlemmene har stabilisert sin automatiske eksponering (programmet fortsetter å prøve i opptil 5 minutter). I delen**Justering**i innstillingspanelet for matriser vises RMS-residualene og knappen**Juster på nytt** — også for separate matriser som ikke har noen sammensatt flis.

## Opptak: overvåking vs. analyse

Opptaksoverflatene for matriser deles tydelig inn i **overvåkingskvalitet**(ta opp det du ser) og**analysekvalitet** (ta opp rådata, kalibrer senere):

| Arbeidsflyt | Kvalitet | Hva lagres | GUI | CLI

|
| --- | --- | --- | --- | --- |
| **Opptak**(stillbilder) | Analyse | Én synkronisert bildegruppe per gjennomgang; filer per kamera på hvert valgt eksportnivå (rådata/debayered/radiance/reflektans/forhåndsvisning/indeks) + `.daq` sidecar |****Capture All**-knapp + innstillinger for opptak | `lattice array-capture` |
| **Ta opp indeksvideo** | Overvåking | Den kombinerte live-indekskomposisjonen slik den vises — 8-bit, forhåndsvisningsoppløsning, LUT innebygd. Fra CLI

-en uten åpen GUI-visning renderer backenden matrisens NDVI

(eller `--expr`) så snart matrisen er justert; en opptak som ikke skriver noen ramme mislykkes | ● Ta opp indeksvideo (kombinerte matriser) | `lattice array-record` |
| **Rå burst → lag video**| Analyse | Rå sensorbilder med full opptakshastighet + manifest + `.daq`, deretter offline rekonstruksjon til kalibrert strålings-/refleksjons-/indeksvideo, tidsjustert til DAQ-målinger | ⦿ Ta opp rå burst →**Lag video** | `lattice array-burst` → `lattice array-build-video` |

Tommelfingerregel: hvis pikslene skal levere *målinger*, bruk opptak eller burst (analyse-kvalitet); hvis du bare trenger å *se eller demonstrere* hva matrisen så, ta opp indeksvideoen (overvåkings-kvalitet).

### Innstillinger for opptak (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->



Tannhjulet ved siden av **Capture All** åpner ruten «Capture Settings» (krever et åpent prosjekt – opptakene lagres der):

* **Opptakmodus**:**Enkelt**(én gjennomgang) /**Kontinuerlig**(påfølgende; begrenset av antall opptak, standard 1, eller varighet, standard 10 s) /**Intervall** (tidsforløp: N opptak hvert X. intervall for totalt Y, standard 1 hvert 5. sekund i 1 minutt).
* **Eksporttyper per kamera**: Raw, Debayered, Radiance, Reflectance, Preview, Index – alt som er relevant er aktivert som standard. Radiance/Reflectance er skjult for kameraer med RGB

-filter; **Reflectance vises bare når kameraet har en DAQ-lyssensor** (sin egen eller arvet fra matrisen); Index krever et konfigurert indeksuttrykk.
* **Justert**(per matrise, standard**på**): tilpasser eksportene fra medlemmene til matrisens justeringsprofil, slik at eksportene blir pikselregistrert. Raw forblir alltid uforvrengt, men bærer transformasjonen i metadataene. Matriseelementfiler lagres i hvert kameras opprinnelige orientering (eventuelle innstillinger for speiling/rotasjon registreres i filen, men brukes ikke).
* **Raskeste opptak** (veksle): kun raw + den tildelte DAQ-avlesningen + den frie sammensatte indeksen, hopper over kalibreringsberegningene ved opptak for maksimal hastighet — gjenoppbygg stråling/refleksjon/indeks senere fra den lagrede `.daq`.
* Valgene beholdes i prosjektet. Skjulte eller pausede kameraer hoppes over.
* **Kontinuerlig** på matriser kjører i Chloros

-backenden (nytt i 1.3.0): når hvert kamera i listen tilhører en matrise, kjører backenden opptakssløyfen selv, og skriver én gruppe mens den velger den neste, uten forespørsel per opptak. En full harddisk stopper kjøringen («Stoppet: harddisk full. Lagret N, F mislyktes»). Se [Innstillinger og moduser for opptak](capture.md#capture-modes) for merknaden fra Microsoft Defender om høye opptaksfrekvenser.

Tilsvarende CLI

-kommando (samme backend-endepunkt, samme semantikk):

```bash
# One synced group, every applicable export level per camera (the default)
chloros-cli lattice array-capture -o output/

# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# 30-second monitoring clip of the combined index view, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# 5-second analysis-grade raw burst, then build the combined index video
chloros-cli lattice array-burst --duration 5 --build --products combined:index --fps 10 -o capture/
```

TIFF

-komprimering for opptak er `deflate` (tapsfri, standard) eller `none` — fullstendige flaggtabeller, opptaksmappestrukturen og regler for nybehandling finnes i [CLI

-referansen](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Koble til en DAQ-lyssensor

Forhåndsvisninger med refleksjons- og belysningskorrigering krever data om nedadrettet lys fra en DAQ-sensor (tilkoblet i fanen **Lyssensorer**):

* **Array-raden**i sidepanelet viser en**«DAQ · på/av»-knapp** — *på* når en lyssensor på array-nivå er angitt **eller** et hvilket som helst medlemskamera har sin egen; verktøytipset viser nøyaktig hvilken sensor som forsyner hvilket kamera.
* Tilordne for hele matrisen i matriseinnstillingene → **Omgivelseslyssensor**→ rullegardinmenyen**Lyssensor**. Valget beholdes i prosjektet, overføres til hvert enkelt kamera i matrisen, og de enkelte kameraene kan fortsatt overstyre det med sin egen sensor.
* Statuslinjen under viser den aktuelle tilstanden: **Av**→ «Venter på første spektrum…» →**«Aktiv — alle kameraer i oppstillingen er belysningskorrigert»** → eller, hvis det ikke har kommet noe nytt spektrum de siste 3 sekundene, en melding om utdatert data — den siste målingen fortsetter å bli brukt (målinger utløper aldri i opptakspolen).

Når en sensor er tilordnet: eksporttypen «Reflektans» blir tilgjengelig, live-forhåndsvisninger er belysningskorrigert, prediktiv automatisk eksponering kan bruke spektrumet, og hver reflektansopptak skriver DAQ-avlesningen som faktisk ble brukt, som en **`.daq`-fil** i opptakets `daq/`-mappe, slik at opptaket kan behandles på nytt senere. Ved siden av denne angir en `.daq.cameras.json`-fil navnene på kameraene hvis bilder denne målingen beskriver; ved ny behandling sammenstiller Chloros

er målingen kun med bilder fra disse kameraene.

## `array-connect`-CLI

-alternativer

| Flagg | Standard | Beskrivelse |
| --- | --- | --- |
| `--serials SN1,SN2,…` | automatisk oppdage alle LATTICE-kameraer (krever ≥2) | **Det første serienummeret er MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO-synkroniseringslinje. |
| `--target-fps F` | auto | Utløsningsfrekvens for MASTER. |
| `--binning {1,2,4}` | auto | Maskinvarebasert binning. «Auto» betyr full sensor (ingen binning) når en bildefrekvens støtter det; «connect» utfører aldri binning, bare for å opprettholde frekvensen. |
| `--packed {auto,on,off}` | `auto` | [Pakket 12-bits transport](#packed-12-bit-transport): `auto` pakker kun når det er nyttig og verten har kapasitet til å pakke ut. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Ekspertoverstyring av valg av synkroniseringsnivå. |
| `--wire-ceiling-mbps MB_PER_S` | auto-detected | Vertsnettverksbudsjett i MB/s — CLI

-formen av feltet **Wire Budget**. Senk det hvis arrayet rapporterer om korrupte rammer. Lagres sammen med prosjektet, slik at en senere tilkobling gjenoppretter det. |
| `--no-recommend` | av | Hopp over trinnet for nettverksanalyse. |
| `--no-ptp` | av | Deaktiver PTP (tidsstempler på tvers av kameraer kan da ikke sammenlignes). |

`lattice array-list`, `array-status` og `array-disconnect` styrer den vedvarende økten. Den fullstendige referansen til underkommandoer, inkludert justering (`align-calibrate` / `align-apply`) og nettverksverktøyene, finnes i [CLI

-referanse § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); de tilsvarende SDK

-kommandoene (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) finnes i [SDK

-referansen](../reference/sdk-reference.md). Fra Python

er ledningsbudsjettet `connect_array(..., wire_ceiling_mbps=120)`, og inndelingen mellom «live corrupt» og «never-arrived» finnes på [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
