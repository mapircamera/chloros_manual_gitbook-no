# Flerkameraoppsett

Et LATTICE **oppsett**består av to eller flere LATTICE-kameraer som er koblet sammen som én synkronisert enhet. Ett av kameraene fungerer som**master**: det sender ut en GPIO-utløserpuls via en felles synkroniseringslinje (standard**Line2**), slik at alle kameraene tar bilde av det samme øyeblikket. Chloros legger til PTP-tidssynkronisering, en live-forhåndsvisning (fliser per kamera eller en enkelt justert flerbånds-kompositt) og synkronisert opptak — hver opptakssyklus produserer én**bildegruppe** der alle kameraene deler samme tidsstempel og bilde-ID (rapportert som `fid:N` på opptaksutdataene).

Det er gjennom matriser at monokameraer (M3M) produserer vegetasjonsindekser – ett kamera bidrar med ett bånd, og matrisen justerer dem inn i en flerbåndsstabel. Se [Monokameraer og vegetasjonsindekser](mono-indices.md).

Det finnes tre likeverdige måter å koble til en matrise på, og alle kjører den samme «smart-prep»-flyten:

| Grensesnitt | Inngangspunkt |
| --- | --- |
| GUI | Fanen «Kameraer» → **Koble til matrise** (blå knapp) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (første serienummer = master) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (første serienummer = master) |

Smart-prep utfører, i rekkefølge: en nettverksfunksjonstest (ICMP DF-ping + GVSP-test), valg av synkroniseringsnivå, automatisk reduksjon av rammestørrelsen for å passe til kabelen, aktivering av PTP, automatisk valg av pikselformat per kamera, automatisk innstilling av eksponering basert på hver kameras lagrede tilstand, og konfigurering av GPIO-utløser på Line2.

{% hint style="info" %}
Kameraene må være tilgjengelige på koblingen før noe av dette fungerer — se [Koble til kameraer](connecting.md) for oppdagelse, adressering og nedlasting av kalibrering ved første tilkobling. For oppsett med flere kameraer er innstillingene for mottaksringen på verts-NIC-en like viktige som koblingshastigheten; den fullstendige tabellen over symptomer og løsninger finnes i [CLI Referanse § Oppsett og justering av verts-NIC](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Dialogboksen «Koble til array»

Fanen «Kameraer» → **Koble til array**åpner en veiviser i tre trinn:**Velg → Visningsmodus → Innstillinger**.

### Trinn 1 — Velg master og slaver

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered. Table showing Camera / Serial / IP / Master radio / Slave checkbox columns, with the green "GPIO master detected — selections pre-populated" probe banner visible above the table. -->

Dialogboksen skanner nettverket så snart den åpnes («Skanner nettverk...»), og sjekker deretter GPIO-utløserkablene («Sjekker GPIO-kabling...»). Du trenger minst **2 kameraer** for å bygge en matrise.

Koblingssjekken fyller ut rolevalget på forhånd når det er mulig, og viser ett av tre banner:

| Banner | Betydning |
| --- | --- |
| «GPIO-master oppdaget — valg forhåndsutfylt» (grønn) | Testen fant utløsertopologien; avmerkingsboksene for master og slave er allerede merket av. |
| «Ingen master oppdaget – sjekk GPIO-kabelen» (oransje) | Ingen kameraer oppfattet en utløserpuls; sjekk synkroniseringskablene. Du kan fortsatt velge roller manuelt. |
| «Ingen synkroniseringskabel: {serienumre}» (oransje) | De oppførte kameraene har ingen synkroniseringskabel tilkoblet. |

Kameratabellen har kolonnene **Kamera / Serienummer / IP / Master (radio) / Slave (avkryssingsrute)**:

* Velg nøyaktig **én master**og**én eller flere slaver**. Hvis du klikker på den nåværende masterens radio igjen, fjernes markeringen.
* Et kamera merket med **«Ingen synkroniseringskabel»** kan aldri velges som slave — en slave uten utløserkabling vil vente på synkroniseringslinjen i all evighet og levere et dødt bilde. Koble i stedet dette kameraet til som et frittstående kamera.
* Kameraer som allerede er koblet til som frittstående, blir *ikke* deaktivert: array-tilkobling frigjør den frittstående sesjonen og åpner kameraet på nytt i arrayet.

**Neste: Visningsmodus →**aktiveres når en master og minst én slave er valgt.**Skann på nytt** kjører oppdagingsprosessen og koblingssjekken på nytt.

{% hint style="warning" %}
**Avbryt** er deaktivert mens en skanning eller kabelsøk pågår — å avbryte midt i kabelsøket kan føre til at kameraet SDK krasjer på LATTICE-kameraets fastvare. Vent til spinneren er ferdig.
{% endhint %}

### Trinn 2 — Visningsmodus

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

| Modus | Hva du får |
| --- | --- |
| **Separate kameraer** | Én live-rute per kamera, alle utløses samtidig slik at bildene forblir synkroniserte. Hvert kamera beholder sin egen farge og innstillinger. |
| **Kombinerte kameraer** *(standard)* | En enkelt flis som gjengir den justerte flerbånds NDVI/index-kompositten. Kameraene deler matrisefargen. |

Visningsmodusen endrer kun presentasjonen av live-forhåndsvisningen — opptaket fungerer på samme måte i begge tilfeller.

### Trinn 3 — Array-innstillinger og forventet resultat

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Pin resolution / Trigger Rate controls, right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->

Når du går inn i denne scenen, ber Chloros backenden om en **anbefaling**og bruker automatisk en kombinasjon av ROI og binning som passer til mottakerringen på nettverkskortet ditt (programmet foretrekker binning fremfor ROI-beskjæring, fordi binning bevarer hele synsfeltet). Hver endring du gjør, utfører analysen på nytt i sanntid og oppdaterer panelet**Forventet resultat** til høyre.

Venstre kolonne – innstillinger:

| Kontroll | Valg | Standard | Merknader |
| --- | --- | --- | --- |
| **ROI (synsfelt)** | Full (2048×1536) / Halv (1024×768) / Kvart (512×384) | Full | Sensorbeskjæring: Halv/kvart beskjæring til et mindre område med opprinnelig pikselavstand. |
| **Binning** | 1× / 2× (sum 2×2) / 4× (sum 4×4) | 1× | Maskinvarebinning: 2×2 = fullt synsfelt til en fjerdedel av ledningskostnaden; 4×4 = fullt synsfelt til 1/16. Skjult hvis kameraene ikke støtter binning. |
| **Bilde på ledningssiden** (utlesning) | — | — | Bredden og høyden etter binning som faktisk sendes over ledningen, avrundet ned til multipler av 16 (minimum 64). |
| **Pin-oppløsning**| avkryssingsrute | av | Chloros øker normalt binning automatisk ved tilkobling når den forventede hastigheten faller under**1,5 fps**. Å holde fast ved pin-innstillingen bevarer den valgte bildestørrelsen og aksepterer den lavere frekvensen — og gjør at en konfigurasjon med for høy belastning resulterer i en hard tilkoblingsavvisning i stedet for en automatisk nedjustering. |
| **Utløserhastighet** | 0,5–60 fps, trinn 0,1 | tom = auto | Masterens utløserhastighet. La feltet stå tomt for å la Chloros beregne den. |
| **Wire Budget**| 20–2000 MB/s, trinn 10 | tom = auto | Hvor mye verten faktisk kan absorbere, i MB/s —**det ene tallet som hele matrisetildelingen avhenger av.** Registreres automatisk fra nettverkskortet. Senk verdien hvis matrisen rapporterer ødelagte rammer: den oppdagede verdien overvurderer USB-kort og delte svitsjer. Endring av verdien fører til at prognosen kjører på nytt i sanntid. |

Høyre kolonne — **Prognosert resultat**:

* **Synkroniseringsnivå** — «Samtidig opptak» (grønn), «Samtidig opptak (FTD-forsinket utsending)» (grønn), «Forsinket opptak (100 ms avvik)» (gul) eller «Konfigurasjonen er for stor» (rød).
* **fps-projeksjon** — vises som et intervall («svakt → sterkt»), fordi hastigheten til et synkronisert array begrenses av eksponeringstiden til det tregeste kameraet.
* **NIC-linje** — koblingshastighet og vedvarende gjennomstrømning («NIC {mbps} Mbps · vedvarende {N} MB/s»).
* **Sim-emit-burst-sjekk** — kan vertsmaskinens NIC-ring absorbere én samtidig burst fra alle kameraene («Sim-emit-burst: X MB · NIC-ring tilgjengelig: Y MB ✓/✗»).
* ****Kontroll av ledningsbudsjett** — samlet etterspørsel i stabil tilstand mot det kollisjonssikre taket for ledningen («Ledningsbudsjett: {etterspørsel} MB/s etterspurt av {n} kameraer · tak {tak} MB/s ✓/✗ overtegnet»).
* **«Maks antall kameraer på denne ledningen: {n} — fastsatt av båndbreddegrensen per kamera, så binning øker ikke dette tallet.»** — vises når du nærmer deg (eller over) taket for antall kameraer.
* **«BILDER VIL GÅ TAPT med disse innstillingene.»**— rød advarsel med begrunnelsen fra backenden, pluss en liste over hindringer og blå**forslag til løsninger** («For å få plass til dette oppsettet på nettverket» / «For å muliggjøre samtidig opptak»).**Bruk og koble til** er sperret inntil det foreligger en prognose, og etiketten forteller deg hvorfor den avviser:

| Knappetikett | Betydning | Hva som faktisk hjelper |
| --- | --- | --- |
| «Analyserer...» | Analysen pågår fortsatt. | Vent. |
| **«For mange kameraer for dette nettverket»**| Arrayet overbelaster nettverket (aggregatkontroll mislyktes). | Færre kameraer, jumbo-rammer fra ende til ende eller et raskere nettverkskort.**En mindre ROI vil IKKE hjelpe** — se nedenfor. |
| **«Reduser ROI for å aktivere»** | Rammer vil gå tapt med disse innstillingene (burst-/ringkontroll mislyktes). | Reduser ROI, øk binning eller reparer nettverkskortets mottaksring. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->

Under tilkoblingen kan et grønt **kalibreringsnedlastingspanel** vises med en fremdriftslinje per seriell port: første gang et kamera kobles til en maskin, henter Chloros sin fabrikkkalibreringspakke på ca. 3,8 MB fra kameraet via GigE (ca. 70 sekunder per kamera). Kameraer som er lagret i hurtigbufferen, viser aldri dette panelet. Se [Koble til kameraer](connecting.md).

## Båndbredde: hvor mange kameraer som får plass

Hvor mange kameraer et array kan håndtere, avhenger av kabelen, ikke av Chloros, så planleggingstallene finnes i maskinvarehåndboken: **[Planlegging av array-båndbredde](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Hva Chloros gjør med dem: tilkoblingsdialogen kjører en nettverkssondering, beregner den oppnåelige bildefrekvensen og velger et nivå som passer. Hvis arrayet overbelaster kabelen, nekter det å koble til i stedet for å droppe pakker uten varsel — se panelet for forventet utfall beskrevet ovenfor.

## Når rammer forsvinner

Et kamera kan være fraværende fra en publisert gruppe av to helt forskjellige årsaker,
og de krever motsatte løsninger. Chloros teller dem separat i stedet for å rapportere ett
«ufullstendig» tall som ikke spesifiserer noen av dem:

| Hva skjedde | Hva det betyr | Hvor du skal lete |
| --- | --- | --- |
| **Korrupt**— rammen ankom og var strukturelt feil | GVSP-pakketap på nettverksbanen |**Kabelbudsjettet**, NIC-mottaksringen, jumbo-rammer, svitsjen |
| **Kom aldri frem**— ingen ramme kom i det hele tatt | Kameraet utløste ikke, eller ingenting kom ut av det |**M8-synkroniseringskabelen**, synkroniseringslinjen, om alle enhetene er aktivert |

Fordelingen vurderes på nytt hvert 10. sekund mens arrayet strømmer. Over 5 % blir det
loggført med begge tallene oppgitt, og hver ødelagt buffer rapporteres første gang det
skjer per kamera, deretter summeres det opp en gang i minuttet slik at en lang økt forblir lesbar.

**Korrupte bilder med null «aldri mottatt» betyr at utløsning og kabelsynkronisering er perfekt**og at hvert tapt bilde skyldes nettverksbanen. Løsningen er å redusere**Wire Budget** og
koble til på nytt.

{% hint style="warning" %}
**Å senke utløserfrekvensen hjelper ikke mot korrupte bilder.** Kameraets pakkefrekvens
fastsettes én gang, ved tilkobling. Å senke utløserfrekvensen endrer hvor ofte en burst
skjer, ikke hvor raskt selve bursten sendes over kabelen. På et målt oppsett med 4 kameraer førte en
5× reduksjon i utløserfrekvensen ikke til noen endring, mens å senke wire budget fra 240 til
200 MB/s reduserte andelen ødelagte rammer i det samme oppsettet fra 10,4 % til null.
{% endhint %}

Et array i drift kan ikke omplanlegge seg selv – koble fra og koble til på nytt, slik at
tidsvelgeren for tilkobling kan arbeide ut fra det nye budsjettet.

### USB-nettverkskort har en øvre grense på 200 MB/s

Et USB-Ethernet-kort oppgir sin *Ethernet*-koblingshastighet, men hva det faktisk
kan opprettholde, er begrenset av USB-bussen og driveren. En USB 10GbE-dongle ble tidligere kreditert
med en gjennomstrømning på omtrent 1000 MB/s — et tall som ingen noensinne hadde målt — og å kjøre
fire kameraer mot dette fantom-reserverommet førte til feil i 6–18 % av bildene, mens matrisen
fortsatt rapporterte en god målbildfrekvens. USB-tilkoblede adaptere er nå begrenset til
**200 MB/s**. Begrensningen er et absolutt tall snarere enn en prosentandel, fordi grensen ligger i
bussen: en USB 1 GbE-adapter oppnår ca. 80 MB/s og påvirkes ikke.

Hvis verten din faktisk er raskere enn grensen, øk **Wire Budget** for å angi dette.

## PTP-tidssynkronisering

*Synkronisering* av rammer skjer via maskinvaretriggeren; **PTP** (IEEE 1588 PTPv2) sørger for sammenlignbare *tidsstempler* på tvers av alle enheter. Det er aktivert som standard når arrayet kobles til:

* **Chloros-vertsbackenden kjører PTP-grandmasteren**. LATTICE-kameraer og DAQ-E-lyssensorer fungerer som slaver til denne i domene 0, slik at bildetidsstempler og DAQ-spektra havner på én klokke (~1 ms).
* `--no-ptp` (CLI) deaktiverer den for laboratoriearbeid — tidsstempler på tvers av kameraene er da **ikke** sammenlignbare.
* Kontroller synkroniseringsstatusen med CLI:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Fanen «Kameraer» har i seg selv ingen PTP-indikator; synkroniseringsinformasjonen per kamera som vises der, er de skrivebeskyttede feltene **Rolle**(Master/Slave),**Synkroniseringslinje** og matrisens funksjonsnivå. DAQ-E PTP-status vises i sensordetaljene på fanen «Lyssensorer».

## Live-visningen

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows, and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

av matrisen Hovedvisningsområdet tilbyr to oppsett (bytt mellom dem i topplinjen): **rutenettvisning**(hver rute er en celle; dra for å endre rekkefølgen når rutenettlåsen er ulåst) og**listevisning**(matriser i full bredde øverst, ett aktivt kamera under). Glidebryteren**Feed Zoom** justerer størrelsen på rutene; ved en cellebredde under 200 px skjules navn/fps-overleggene automatisk.**Separat modus** viser én rute per kamera. Hver rute viser følgende overlegg:

* kameranavnet (øverst til venstre),
* en **fps-visning** (øverst til høyre) — dette er kameraets *faktiske opptakshastighet* rapportert av backend, ikke forhåndsvisningsfrekvensen (live-forhåndsvisning er begrenset til 30 fps uavhengig av opptakshastigheten),
* en statusprikk — grønn (strømmer) / gul (laster) / rød (feil),
* en **spinner for utdatert bilde** når det ikke har kommet et nytt bilde på 2 sekunder — normalt i ca. 5 sekunder etter tilkobling/frakobling mens backenden omfordeler båndbredden mellom kameraene.**Kombinert modus**viser én sammensatt rute: backenden utfører debayering, skalering, justering, støyreduksjon, konverterer til strålingsintensitet per bånd (pluss DLS-reflektans når en lyssensor er tilknyttet), evaluerer matrisens indeksuttrykk, bruker LUT-en og strømmer resultatet som MJPEG. Inntil det første justerte bildet vises, viser ruten sin status: «Forbereder matrise…», «Kalibrerer justering…», «Venter på første bilde…», eller – hvis budsjettet for automatisk justeringsforsøk (~30 s) er brukt opp – «Justering påkrevd» med en**Kalibrer justering**-knapp.

Nyttige fakta om kombinert modus:

* Komposittbildet er registrert i forhold til **hovedkameraets**bilde. AE-ROI-fokusering og punktmåling på sammensetningen er nøyaktige for masterkameraet og omtrentlige for slavekameraene; bruk**Split View** (matriseinnstillinger → «Vis medlemskameraer») for pikselnøyaktige fliser per kamera uten å åpne ekstra kameratilkoblinger.
* ****Display Layers**(matriseinnstillinger; standard av) lar deg velge et forgrunns- og bakgrunnslag – hvilket som helst medlemskamera eller**Index**. Når forgrunn = Index, viser piksler utenfor LUT Min/Max bakgrunnslaget.
* **Renderoppløsning** (standard 720p) angir høyden på direktestrømmen *og* størrelsen på den lagrede komposittfilene ved eksport. Bilder per kamera eksporteres alltid i full oppløsning.
* Justering beregnes per økt og lagres aldri – se avsnittet om justering i panelet for arrayinnstillinger for RMS-residualer og knappen «Rekalibrer».

## Opptak: overvåking vs. analyse

Arrayets opptaksoverflater deles tydelig inn i **overvåkingskvalitet**(ta opp det du ser) og**analysekvalitet** (ta opp rådata, kalibrer senere):

| Arbeidsflyt | Kvalitet | Hva som lagres | GUI | CLI |
| --- | --- | --- | --- | --- |
| **Opptak**(stillbilder) | Analyse | Én synkronisert bildegruppe per gjennomgang; filer per kamera på hvert valgt eksportnivå (rådata/debayered/radiance/refleksjonsgrad/forhåndsvisning/indeks) + `.daq` sidecar |**Ta opp alt**-knapp + Opptaksinnstillinger | `lattice array-capture` |
| **Ta opp indeksvideo** | Overvåking | Den kombinerte live-indekskomposisjonen slik den vises — 8-bit, forhåndsvisningsoppløsning, LUT innebygd; krever at livestreamen er åpen | ● Ta opp indeksvideo (kombinerte matriser) | `lattice array-record` |
| **Rå bildeserie → lag video**| Analyse | Rå sensorbilder med full opptakshastighet + manifest + `.daq`, deretter offline rekonstruksjon til kalibrert strålings-/refleksjons-/indeksvideo, tidssynkronisert med DAQ-målinger | ⦿ Ta opp rå burst →**Lag video** | `lattice array-burst` → `lattice array-build-video` |

Tommelfingerregel: hvis pikslene skal levere *målinger*, bruk opptak eller burst (analyse-kvalitet); hvis du bare trenger å *se eller demonstrere* hva matrisen så, ta opp indeksvideoen (overvåkings-kvalitet).

### Innstillinger for opptak (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->

Tannhjulet ved siden av **Capture All** åpner ruten for opptaksinnstillinger (krever et åpent prosjekt – opptakene lagres i dette):

* **Opptaksmodus**:**Single**(én gjennomgang) /**Continuous**(kontinuerlig; begrenset av antall opptak, standard 1, eller varighet, standard 10 s) /**Intervall** (tidsforløp: N opptak hvert X-intervall for Y totalt, standard 1 hvert 5. sekund i 1 minutt).
* **Eksporttyper per kamera**: Raw, Debayered, Radiance, Reflectance, Preview, Index — alt som er relevant er aktivert som standard. Radiance/Reflectance er skjult for RGB-filterkameraer;**Reflectance vises bare når kameraet har en DAQ-lyssensor** (sin egen eller arvet fra matrisen); Index krever et konfigurert indeksuttrykk.
* **Justert**(per matrise, standard**på**): tilpasser eksportene av elementene til matrisens justeringsprofil, slik at eksportene blir pikselregistrerte. Raw forblir alltid uforvrengt, men bærer transformasjonen i metadataene.
* **Raskeste opptak** (veksle): kun rådata + den tildelte DAQ-avlesningen + den frie sammensatte indeksen, hvor kalibreringsberegningene hoppes over ved opptak for maksimal hastighet — gjenoppbygg stråling/refleksjonsgrad/indeks senere fra den lagrede `.daq`.
* Valgene beholdes i prosjektet. Skjulte eller pausede kameraer hoppes over.

Den tilsvarende CLI (samme backend-endepunkt, samme semantikk):

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

TIFF-komprimering for opptak er `deflate` (tapsfri, standard) eller `none` – fullstendige flaggtabeller, opptaksmappestrukturen og regler for nybehandling finnes i [CLI-referansen](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Koble til en DAQ-lyssensor

Forhåndsvisninger med refleksjons- og belysningskorrigering krever data om nedadrettet lys fra en DAQ-sensor (tilkoblet i fanen **Lyssensorer**):

* **Array-raden**i sidepanelet viser en**«DAQ · på/av»-knapp** — *på* når en lyssensor på array-nivå er angitt **eller** et hvilket som helst medlemskamera har sin egen; verktøytipset viser nøyaktig hvilken sensor som forsyner hvilket kamera.
* Tilordne for hele matrisen i matriseinnstillingene → **Omgivelseslyssensor**→ rullegardinmenyen**Lyssensor**. Valget beholdes i prosjektet, overføres til hvert enkelt kamera i matrisen, og de enkelte kameraene kan fortsatt overstyre det med sin egen sensor.
* Statuslinjen under viser den aktuelle tilstanden: **Av**→ «Venter på første spektrum…» →**«Aktiv — alle kameraene i matrisen er belysningskorrigert»** → eller, hvis det ikke har kommet noe nytt spektrum de siste 3 sekundene, en melding om foreldet data — den siste målingen fortsetter å bli brukt (avlesningene utløper aldri i opptaksbanen).

Når en sensor er tilordnet: eksporttypen «Reflektans» blir tilgjengelig, live-forhåndsvisninger er belysningskorrigert, prediktiv automatisk eksponering kan bruke spektrumet, og hver reflektansopptak skriver DAQ-avlesningen som faktisk ble brukt som en **`.daq` sidecar** ved siden av bildet, slik at opptaket kan behandles på nytt senere.

## `array-connect` CLI-alternativer

| Flagg | Standard | Beskrivelse |
| --- | --- | --- |
| `--serials SN1,SN2,…` | automatisk oppdage alle LATTICE-kameraer (krever ≥2) | **Det første serienummeret er MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO-synkroniseringslinje. |
| `--target-fps F` | auto | Utløsningsfrekvens for master. |
| `--binning {1,2,4}` | auto | Maskinvarebinning. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Ekspertoverstyring av synkroniseringsnivåvelgeren. |
| `--wire-ceiling-mbps MB_PER_S` | automatisk oppdaget | Vertsledningsbudsjett i MB/s — CLI-formen av feltet **Wire Budget**. Senk det hvis arrayet rapporterer ødelagte rammer. Lagres med prosjektet, slik at en senere tilkobling gjenoppretter det. |
| `--no-recommend` | av | Hopp over trinnet med nettverksanalyse. |
| `--no-ptp` | av | Deaktiver PTP (tidsstempler på tvers av kameraer kan da ikke sammenlignes). |

`lattice array-list`, `array-status` og `array-disconnect` styrer den vedvarende økten. Den fullstendige referansen for underkommandoer, inkludert justering (`align-calibrate` / `align-apply`) og nettverksverktøyene, finnes i [CLI-referanse § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); SDK-ekvivalenter (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) finnes i [SDK-referansen](../reference/sdk-reference.md). Fra Python er ledningsbudsjettet `connect_array(..., wire_ceiling_mbps=120)`, og inndelingen mellom «live», «korrupt» og «aldri ankommet» finnes i [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
