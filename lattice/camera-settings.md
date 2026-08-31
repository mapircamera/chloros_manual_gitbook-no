# Kamerainnstillinger

Fanen **Kameraer**er Chloros’ live-kontrollpanel for LATTICE-kameraer: et hovedområde som viser hvert tilkoblet kamera som en live-rute, og en sidepanel som kan veksle mellom tre sider —**kameralisten**, et**innstillingsvindu**(innstillinger per kamera, for kameraoppsett eller for opptak – én om gangen) og**indekskalkulatoren**. Denne siden dokumenterer alle kontrollelementene i kameralisten, innstillingsvinduet per kamera og innstillingsvinduet for kameraoppsett. Opptakmoduser, valg av eksporttype og «Capture All»-flyten finnes på den tilhørende siden [Opptaksinnstillinger og -moduser](capture.md).

Fanen «Kameraer» vises i sidepanelet når Chloros-backenden er klar. Alle kontrollelementene nedenfor kommuniserer med den lokale backenden via `127.0.0.1:5000`; endringer gjelder umiddelbart for live-kameraet, med mindre annet er angitt.

## Kameratyper som brukes på denne siden

Kontrollene vises eller skjules avhengig av hvilken type kamera som er valgt. Brukerhåndboken bruker følgende begreper gjennom hele teksten:

| Begrep | Betydning | Filterkanaler |
| --- | --- | --- |
| **RGB-kamera** | LATTICE M3C med FRGB-filter (modellen inneholder `-FRGB`) | Red / Green / Blue |
| **Bayer multispektralt** | LATTICE M3C med FRGN, FOCN eller FNGB | FRGN: Red / Green / NIR · FOCN: Orange / Cyan / NIR · FNGB: NIR / Green / Blue |
| **Mono (M3M)** | LATTICE M3M — ett smalbåndsfilter, ett kalibrert bånd | Enkeltbånd |
| **Array-medlem** | Et kamera koblet til som del av et synkronisert array (kombinert eller separat visning) | Per filter |

RGB-kameraer gjennomgår fotometrisk behandling (hvitbalanse, fargeprofiler, gamma); multispektrale og monokameraer får den radiometriske kjeden og hopper over de fotometriske kontrollene. Array-medlemmer overfører innstillinger på strømnivå (pikselformat, oppløsning, binning, utløser, bildefrekvens) til arrayet — disse radene blir skrivebeskyttet i ruten per kamera og flyttes i stedet til ruten for arrayinnstillinger.

## Hovedområdet

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

for strøm Når ingen kameraer er tilkoblet, viser strømområdet en velkomstmelding med teksten **«Koble til et kamera for å komme i gang»**og to knapper:**Koble til kamera**(grønn, åpner dialogboksen for tilkobling av enkeltkamera) og**Koble til array** (blå, åpner dialogboksen for tilkobling av array). Selve tilkoblingsdialogene er dokumentert i [Koble til kameraer](connecting.md); begreper knyttet til oppstillinger (synkronisering, nivåer, båndbredde) i [Oppstillinger med flere kameraer](arrays.md). Når du åpner et lagret prosjekt som inneholder kameraer, viser startskjermen i stedet en spinner med teksten «Åpner N lagrede kameraer på nytt…», mens Chloros gjenoppretter strømmene fra forrige økt.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Øverste linje

| Kontroll | Funksjon |
| --- | --- |
| **Veksle mellom visningsmodus**| Veksler mellom**rutenettvisning**(alle ruter som celler) og**listevisning** (oppsett i full bredde øverst, ÉT aktivt kamera under). Verktøytips: «Bytt til rutenettvisning» / «Bytt til listevisning». |
| **Rutenettlås**(hengelås) | Standard**låst** — ruter er låst på plass. Lås opp for å dra og omorganisere ruter til hvilken som helst plass (mellomrom beholdes). Rutenettet låses automatisk på nytt hver gang et nytt kamera kobles til. Verktøytips: «Lås opp rutenett (aktiver dra og slipp av ruter)» / «Lås rutenettet (frys ruter på plass)». |
| **Feed-zoom**-glidebryter | Rutestørrelse, fra 60 px opp til full bredde på beholderen. Cellene beholder et 4:3-sideforhold. Under 200 px cellebredde skjules navn og fps-overlegg for å holde ruten ren. |

### Feed-ruter

Hvert kamera gjengir en sammensatt live-rute; et kamera kan i tillegg vise tre gråtonede **kanaldelte** ruter (se [Kanaldeling](#display-overlays-drawn-over-the-live-feed)), og matriser viser en kombinert flis. Den aktive flisen har en markeringsring i kameraets (eller matrisens) farge.

Når du holder musepekeren over en flis, vises en **X**-lukkeknapp:

* Hvis du lukker en **sammensatt** flis mens kanaloppdelingen forblir synlig, skjules bare den sammensatte flisen.
* Hvis du lukker den **siste synlige flisen til et frittstående kamera**, kobles det kameraet fra.
* **Oppdelte fliser for medlemmer av en kombinert matrise kobler aldri fra** kameraet – de skjules bare.

Når rutenettet er ulåst, kan du dra en hvilken som helst flis til et hvilket som helst spor; oppsettet lagres sammen med prosjektet.

## Sidepanel — kameraliste

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

Den første siden i sidepanelet viser alle tilkoblede kameraer og oppsett:

* **Koble til kamera**(grønn) /**Koble til oppsett** (blå, viser «Oppdager...» mens den skanner). Begge er deaktivert mens en tilkoblingsdialogboks er åpen.
* **Ta opp alt** (rød) — tar opp alle kameraene på listen med eksporttypene valgt i Opptaksinnstillinger. Krever et åpent prosjekt. Fullstendig dokumentert i [Opptaksinnstillinger og -moduser](capture.md).
* **Tannhjulet for opptakinnstillinger** (ved siden av «Ta opp alt») — åpner [panelet for opptakinnstillinger](capture.md#the-capture-settings-pane). Deaktivert uten et prosjekt eller mens opptak pågår.

### Kamerarader

Hver kamerarad viser en fargekodet ramme (kameraets tilpassede farge), en «CAM»-etikett — med en blå **M**(master) eller grønn**S** (slave) som rollebetegnelse for medlemmer av kameragruppen — og visningsnavnet. Standardnavnet er `LATTICE-MODEL (serial)`; du kan gi det nytt navn fra innstillingspanelet for hvert kamera. Knapper i raden:

| Knapp | Effekt |
| --- | --- |
| **Øye**| Veksle mellom synlig og usynlig. Skjulte kameraer fjernes fra rutenettet og**ekskluderes fra «Capture All»**. |
| **Tannhjul** | Åpner innstillingspanelet for hvert kamera (neste avsnitt). |
| **Pause / Spill av**| Fryser live-forhåndsvisningen**kun på skjerm** — opptaket i bakgrunnen fortsetter. Kameraer i pause kan ikke ta opp. |
| **X** | Koble fra. Brukergrensesnittet oppdateres umiddelbart (i beste fall); selve frakoblingen i bakgrunnen kan ta 10–30 sekunder å fullføre. |

### Array-rader

En matriserad viser et «ARRAY»-merke i matrisens farge, matrisenavnet (kan endres i matriseinnstillingene) og en **DAQ · på/av**-knapp —**på** når lyssensoren på matrisenivå er aktivert *eller* et av medlemskameraene har en sensor per kamera; verktøytipset viser nøyaktig hvilken sensor som sender data til hva. Medlemskameraene vises innrykket under med egne rader. Knapper i matriseraden: **øye**(skjuler/viser ALLE medlemskameraene samlet),**tannhjul**(innstillingsruten for matrisen),**X**(kobler fra hele matrisen).

Statusen til lyssensoren (DLS) som brukes i matriseradene og innstillingsruten for matrisen, har fire tilstander:**av**,**venter**(ingen spektrumdata ennå),**aktiv**(et spektrum er mottatt i løpet av de siste 3 sekundene), og**foreldet** — ingen nytt spektrum på 3 sekunder, men den siste avlesningen *brukes fortsatt* (DAQ-avlesninger utløper aldri i innfangingsbanen).

Du kan dra enkeltstående kameraer og hele matrisegrupper forbi hverandre i sidepanelet for å endre rekkefølgen på listen; matrisemedlemmer kan ikke dras uavhengig av hverandre.

## Innstillingsruten per kamera

Åpnes med **tannhjulet** på en kamerarad. Ruten glir over kameralisten.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->

**Overskrift**: kameraets**fargeprøve**(klikk for å åpne et innebygd fargevalg — angir fargen på sidefeltets ramme og utvalgsringen for fliser),**navnet**med en blyantknapp for**Omdøpe**(hvis du lagrer et tomt navn, går det tilbake til standardnavnet `MODEL (serial)`), og**×** for å lukke.

### Live-histogram

Øverst i ruten vises et live-luma-histogram beregnet fra JPEG-forhåndsvisningen med en frekvens på ca. 8 Hz. Gjennomsnittet er Bayer-vektet — (R+2G+B)/4 — for å samsvare med kameraets egen AE-måling.

* **Orange stiplet linje**= AE-målet.**Dra den horisontalt for å endre målet** — én kommando sendes når du slipper, og ved å dra skifter AE-målmodusen til Manuell.
* **Green hel linje** = den faktiske gjennomsnittlige lumaen (det AE leverer for øyeblikket).
* **RGB-knapp** (øverst til høyre): veksler mellom båndvise overleggshistogrammer farget etter kameraets filter (f.eks. på FRGN: grå NIR, grønn, rød). På monokameraer (M3M) står det «MONO» på knappen, og den er deaktivert — mono viser alltid luma-histogrammet for ett bånd.
* X-akse-etikettene følger sensorens bitdybde for det gjeldende pikselformatet: 0..255, 0..1023, 0..4095 eller 0..65535.

### Rader

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

med kamerainformasjon | Rad | Funksjon |
| --- | --- |
| **Modell** | Skrivebeskyttet (f.eks. `LATT-M3C-L87-FRGN`). |
| **Radiometrisk kalibrering**| Green**«Aktiv»**-merke med en bildetekst som viser kalibreringsnivå, hash, kalibreringsdato og båndliste, lastet inn fra kameraets kalibreringspakke (se [Fabrikkradiometrisk kalibrering](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Skjult for RGB-kameraer** — disse har en fotometrisk hvitbalanskalibrering, ikke strålingsintensitet per bånd. |
| **Kalibreringsrapport**|**Last ned**-knapp — åpner kameraets NIST-kalibreringssertifikat (per serienummer) i PDF-format i visningsprogrammet på operativsystemet ditt. Hvis sertifikatet ikke er lagret i hurtigbufferen ennå, viser Chloros et tips i stedet. |
| **Serienummer** | Skrivebeskyttet. |
| **Firmware**| Viser gjeldende versjon, og henter deretter den tilgjengelige versjonen for denne modellen (lagret per modell — en gruppe på N kameraer sjekker serveren én gang). Status: «Sjekker…» →**«Oppdater til X»**-knapp → «Blinker…» → «Oppdatert fra A til B» / «Mislyktes: …» / «Hoppet over: …» / grønt**«Oppdatert»**. Verktøytips for oppdateringsknappen: «Tilbakestill til fabrikkinnstillinger + oppdatering + omprogrammering av UserSet1. Ca. 2–3 minutter; ikke koble fra.» |
| **IP** | Skrivebeskyttet. |
| **Temperatur** | Skrivebeskyttet, oppdateres hvert 3. sekund. Blir oransje ved ≥65 °C og rød med et ⚠ ved ≥75 °C. |
| **Kalibreringsmål**-avkryssingsrute | Aktiverer ArUco-refleksjonsmåldeteksjon med en valideringstabell per panel (NDVI) under live-strømmen (listevisning). Kun for den aktuelle økten — åpnes alltid av. |
| **Lyssensor**-rullegardin | Knytter en DAQ-lyssensor (DAQ-E/M/U, fra listen på fanen «Lyssensorer») til dette kameraet for belysningskorreksjon ved nedadrettet lys (DLS) og prediktiv automatisk eksponering. «Ingen» fjerner tilknytningen. Hvis ingen sensorer er tilkoblet, viser rullegardinmenyen «(ingen sensorer tilkoblet — åpne DAQ-fanen)». Tilknytningen lagres sammen med prosjektet. |

### Eksponering og forsterkning

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Alle numeriske inntastinger her bruker hold-for-å-accelerate-spinnere: trykk = ±1, hold &gt;1,5 s = ±10, hold &gt;3 s = ±100. Verdien sendes til kameraet når du slipper.

| Kontroll | Område / valg | Standard | Gjelder for | Hva den gjør |
| --- | --- | --- | --- | --- |
| **Eksponering (us)**| Kameraets live min./maks. | Auto | Alle | Eksponeringstid i mikrosekunder, med en**Auto/Manuell**-bryter. Auto = kontinuerlig automatisk eksponering på kamerasiden. |
| **Forsterkning (dB)**| Kameraets sanntids min./maks. (f.eks. opptil 48 dB) | Manuell (av) | Alle | Analog/digital forsterkning med egen**Auto/Manuell**-bryter. |
| **AE-mållysstyrke**| 0–255 | 80, modus**Auto**| Alle (kan redigeres når AE eller automatisk forsterkning er på) | Lysstyrken AE sikter mot. I**Auto**(standardinnstillingen) velger en histogrambasert bakgrunnskontroller selve målet, og holder eksponeringen på 60–75 % av sensorens maksimum. Hvis du skriver inn en verdi eller drar den oransje linjen i histogrammet, skifter den til**Manuell**. |
| **AE-utjevning** | 0,5–40, trinn 0,1 | 8,0 | Alle | AE-demping. Verktøytips: «Lavere = AE reagerer raskere (kan pulsere ved høy bildfrekvens). Høyere = jevnere / tregere.» Verdier langt under standardverdien kan føre til at AE pulserer og destabiliserer strømmingen ved høye bildefrekvenser; 8,0 er den stabile standardverdien. |
| **AE Region of Interest**| Aktiver avkryssingsrute +**Aim**-knapp | Av | Alle | Når denne er aktivert, måler AE kun det grønne, stiplede området i stedet for hele bildet.**Sikte** aktiverer «klikk-for-å-plassere» på live-strømmen: et klikk sentrerer et område på 30 % av bildet; klikk-og-dra tegner et tilpasset rektangel (minst 5 % × 5 %). «Aim» deaktiveres automatisk etter én plassering. Området tilpasses til kameraets opprinnelige koordinater i henhold til eventuelle rotasjons- eller speilingsinnstillinger du har angitt, og lagres sammen med prosjektet. |
| **AE-justeringshastighet** | 0,1–5, trinn 0,1 | 1,0 | Kun for array-medlemmer | Hvor raskt det automatiske AE-målet sporer endringer i scenens lysstyrke; 1,0× sjekker på nytt hvert 2,5. sekund. |
| **Highlight Protection** | Strict (1 %) / Normal (5 %) / Relaxed (15 %) | Strict | Kameraer som støtter denne innstillingen | Hvor mye av bildet som kan bli klippet til hvitt før AE gjør bildet mørkere. |

{% hint style="info" %}
**Lyskrav for multispektrale Bayer-kameraer (RGN / OCN / NGB):** scenen må ha tilstrekkelig lys i alle tre kanalene, ellers fungerer ikke kalibreringen riktig — én enkelt sensoreksponering dekker alle tre spektrene. Bruk en DAQ-lyssensor til å måle lyset, eller gå over til fullt mono (M3M) slik at hvert bånd får sin egen eksponering. Hvis en opptak bryter dette, oppdager Chloros det og gir deg en advarsel (varselet «unmix-clamp»).
{% endhint %}

### Pikselformat og

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

oppløsning**Array-medlemmer** viser skrivebeskyttede «Current»-rader (format + BxH) og «Binning»-rader med merknaden «Angitt i array-innstillinger» — en strøm-omstart på ett medlem ville ødelegge synkroniseringen, så disse administreres i [array-innstillingsruten](#array-settings-pane).**Frittstående kameraer** har:

| Kontroll | Valg | Hva den gjør |
| --- | --- | --- |
| **Pikselformat** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Sensorens pikselformat (bitdybde). |
| **Oppløsning** | Full / Halv / Kvart | I forhold til gjeldende binning: Full = 2048/N × 1536/N for N×N-binning. |
| **Binning** | 1x1 (ingen) / 2x2 / 4x4 | Maskinvarebasert N×N-binning — større verdier gir lavere oppløsning, men øker signal-støyforholdet (SNR) og bildefrekvensen. Endring av dette starter strømmen på nytt og tilbakestiller eventuelle ROI-er til det nye fulle synsfeltet. |
| **Gjeldende** | skrivebeskyttet | Den faktiske WxH og (x, y)-forskyvningen som er i kraft. |

### Live-forhåndsvisning

Alt i denne delen gjelder **kun visningssiden**— det endrer det du ser i live-strømmen, mens lagrede opptak forblir lineære og uendret — med ett unntak:**Vignette** er radiometrisk og påvirker også eksporter (beskrevet nedenfor).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->

| Kontroll | Område / valg | Standard | Gjelder for | Hva den gjør |
| --- | --- | --- | --- | --- |
| **Renderoppløsning** | 360p (raskest) / 480p / 720p / 1080p / Sensorens opprinnelige oppløsning (tregst) | 720p | Alle | Høyden som backenden kjører den radiometriske forhåndsvisningskjeden på. Lavere verdi gir høyere bildefrekvens uten å endre synsfeltet. |
| **Indeks**| Avkryssingsrute for aktivering + tannhjul | Av | Kun Bayer-multispektral,**ikke** medlemmer av kombinert matrise | Forhåndsvisning av vegetasjonsindeks i sanntid. Tannhjulet åpner den delte [Indekskalkulatoren](#index-calculator-pane) som er forhåndslastet med kameraets naturlige filterbånd (f.eks. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Det tilpassede uttrykket pluss LUT (på/av, standardnivå 3, standardmin 0,2, standardmaks 1) beregnes for hvert forhåndsvisningsbilde. Medlemmer av kombinert matrise skjuler denne raden — matrisen har én delt indeks. |
| **Hvitbalanse** | Av / En gang / Kontinuerlig + en knapp for ny registrering | Kontinuerlig | Kun RGB | Live hvitbalanse. Oppdateringsknappen registrerer hvitbalansen på nytt fra det gjeldende DLS-spektrumet (deaktivert når modusen er Av). |
| **Gamma** | På / Av | På | Kun RGB | Vis gamma (γ = 2,2 LUT) i liveforhåndsvisningen. Lagrede opptak forblir lineære. |
| **Støyreduksjon** | Avkryssingsrute + styrke 0–100 | Av / 50 | Alle (per kamera, også innenfor matriser) | Bilateralt filter i live-forhåndsvisningen. Høyere verdi gir jevnere, men mykere detaljer. |
| **Skarphet** | Avkryssingsrute + styrke 0–100 | Av / 30 | Alle | Uskarp maske i liveforhåndsvisningen, brukes til slutt. Kan forsterke støy. Kun i forhåndsvisningen. |
| **Vignett**| Avkryssingsrute + styrke 0–100 | Av / 0 | Alle | Manuell restfjerning av vignett (gjør hjørnene lysere), lagt på toppen av matrisens Smart Vignette-estimat.**Radiometrisk — påvirker livevisningen OG eksportene**, i motsetning til Støyfjerning/Skarphet. |
| **Fargeprofil** | Raw / Lineær / Naturlig / Forbedret / Egendefinert temperatur | Naturlig | Kun RGB | Se nedenfor. |
| **Fargetemperatur** | 2000–10000 K, trinn 100 | 5500 K | Kun RGB, profil for tilpasset temperatur | Låser hvitbalansen til en fast korrelert fargetemperatur (DLS-inngang ignoreres). Den sist valgte Kelvin-verdien lagres ved bytte mellom profiler. |
| **Metning** | 0–200 (100 = nøytral) | 100 | Kun RGB | HSV-metning i liveforhåndsvisningen. |
| **Kontrast** | 0–200 (100 = nøytral) | 100 | Kun RGB | Lineær kontrast rundt midtgrått i liveforhåndsvisningen. |
| **Speil H / Speil V** | Avmerkingsbokser | Av | Alle | Speilvend forhåndsvisningen horisontalt / vertikalt. |
| **Rotasjon**| 0° / 90° / 180° / 270° | 0° | Alle | Roter forhåndsvisningen. Retningen brukes på slutten av forhåndsvisningskjeden i backend —**lagrede opptak beholder kameraets opprinnelige retning**, og sammensatte visninger i matriser ignorerer den. |**Semantikk for fargeprofiler** (RGB-kameraer):

* **Raw** — omgå behandlingskjeden fullstendig.
* **Lineær** — mørkt signal + flatfelt + hvitbalanse; ingen fargematrise, ingen gamma.
* **Naturlig** *(standard)* — lineær pluss den målte fargekorrigeringsmatrisen og en scenetilpasset tonekurve.
* **Forbedret**— Naturlig pluss livlighet og CLAHE lokal kontrast. Tilleggskostnaden gjelder**kun live-forhåndsvisning** — lagrede bilder får alltid full etterbehandling uavhengig av profil.
* **Egendefinert temperatur** — Natural med hvitbalanse fastsatt til den valgte Kelvin-verdien.

{% hint style="warning" %}
For Naturlig, Forbedret og Egendefinert temperatur viser ruten en toneanmerkning: bildene blir lysnet opp i henhold til sin egen scene, så lagrede *skjerm*bilder kan ikke sammenlignes bilde for bilde. **Eksporter radianse eller reflektans for målinger.**
{% endhint %}

### Skjermoverlegg (tegnet over live-strømmen)

Disse finnes kun i frontend – de legges over videoen, uten å påvirke strømmen eller opptakene.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Overlegg | Kontroller | Standard | Hva det gjør |
| --- | --- | --- | --- |
| **Zebra** | Avkryssingsrute + terskel 200–255 | Av / 250 | Magenta diagonale striper på avklippede piksler. |
| **Kryss** | Avkryssingsrute | Av | Markering for bildesenter. |
| **Rutenett** | Av / 3 × 3 / 9 × 9 | Av | Komposisjonsrutenett. |
| **Histogram** | Avkryssingsrute + bredde 0,10–0,90 av bildet | Av / 0,25 | En histogramstripe på strømmen. |
| **Fokustopp** | Avkryssingsrute + terskel 20–200 + fargeprøve | Av / 80 / `#ff5722` | Sobel-kantmarkering for fokusering. |
| **Kanalinndeling** | «Vis delinger (Red / Green / NIR)» / «Skjul delinger»-knapp | Skjult | Legger til tre uavhengige gråtoner perkanal ved siden av sammensetningen (knappetiketten følger kameraets filterkanaler). Hver delingsrute kan dras og har samme kantfarge som kameraet. Ikke tilgjengelig på monokameraer. Lagres sammen med prosjektet. |

### Spotmåler

* **Klikk for å ta prøve**-avkrysningsrute: klikk på live-bildet for å ta en prøve av en enkelt piksel (et krysshårmerker den), eller klikk og dra over et område for å få et pikselgjennomsnitt.**Tøm**sletter prøven og krysshåret. Kan ikke brukes samtidig med AE-ROI**Sikte**-modus.
* **Vis**-rullegardin:**Raw (bitdybde)**— opprinnelige digitale tall med sensorens bitdybde (f.eks. 12-bit → 0..4095) — eller**Visning (8-bit)** (standard). Når en live-indeks er aktiv, viser Display i stedet den beregnede indeksverdien (f.eks. NDVI).
* Avlesningspanelet viser pikselkoordinater, bildestørrelse, pikselformat, bitdybde og en kanaltabell (Chan / Value / %) med båndbetegnelser og bølgelengder; Bayer-grønne par blir gjennomsnittsberegnet; regionprøver viser «N px avg».

Spotmålerens tilstand gjelder kun for den aktuelle økten.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Prediktiv automatisk eksponering (DLS-drevet)

Denne seksjonen vises kun når **minst én DAQ-lyssensor er tilkoblet** — løseren trenger et sanntids nedadrettet spektrum for å drive den.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->

| Kontroll | Område | Standard | Hva den gjør |
| --- | --- | --- | --- |
| **Aktiver** | Avkryssingsrute | På (frittstående kameraer) | En lukket løsningsalgoritme bruker DLS-spektrumet pluss skalærene fra kameraets kalibreringspakke for å få det lyseste båndet nær metning, samtidig som det mørkeste båndet holdes over SNR-grensen — én eksponeringsskriving per løsning, ingen stabiliseringssløyfe. Utviklet for solcelledrevet tidsforløp der hvert bilde må være riktig eksponert. Backenden faller stille tilbake til reaktiv AE når DLS-avlesningen er foreldet/mangler eller kalibreringspakken ikke er lastet inn. |
| **Utjevning (α)** | 0,05–1,0, trinn 0,05 | 0,3 | Utjevning av påfølgende prediktive løsninger (lavere verdi = jevnere). |
| **Scenerefleksjon**|**Rekalibrer ρ**-knapp | — | Beregner på nytt scenerefleksjonsfaktoren som løseren bruker. |

{% hint style="info" %}
**Array-tilkobling slår prediktiv AE av som standard** — for arrayer håndterer Chloross smarte AE pluss kameraets automatiske eksponering eksponeringen (med metningsbeskyttelse), og den prediktive AE-funksjonens enkeltestimat for scenerefleksjonsgrad er ikke pålitelig i blandede scener. Du kan aktivere den på nytt per kamera her hvis du spesifikt ønsker DLS-drevet radiometrisk eksponering.
{% endhint %}

**DAQ-styrt eksponeringstak og innfallsfestet AE.**Uavhengig av avmerkingsboksen ovenfor, når en DAQ-lyssensor er tilordnet et RGB-kamera, beregner Chloros — ut fra den målte absolutte nedadgående bestrålingsstyrken — den maksimale eksponeringen × forsterkningen der en overflate med 100 % refleksjonsevne holder seg under klipping, og bruker den som et**tak**på automatisk eksponering. Mens taket er aktivt, er kameraet**innfallsfestet**: det kjører i åpen sløyfe ved den innfallende-målt eksponering med forsterkning på 0 dB — eksponeringen følger det målte lyset, ikke motivets innhold. Siden taket kun kan forkorte eksponeringstiden, kan det ikke i seg selv forårsake klipping. Taket deaktiveres automatisk – og normal motiv-AE gjenopptas – når DAQ-avlesningen mangler, er foreldet (&gt;30 s) eller mørk, eller hvis ≥15 % av bildet klipper ved den faste eksponeringen (det vil si at sensoren og kameraet registrerer ulik belysning). Det finnes ingen bryter i brukergrensesnittet; dette er standardoppførsel når et RGB-kamera har en DAQ-tilkobling.

### Medlemmer av Acquisition &amp; Trigger

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

Array viser i tillegg skrivebeskyttede rader for **Rolle**(Master i blått / Slave i grønt),**Synkroniseringslinje**og**Peers**.

| Kontroll | Valg | Standard | Merknader |
| --- | --- | --- | --- |
| **Trigger Mode** | Off / On | On | Deaktivert for array-medlemmer (arrayen styrer utløsningen). |
| **Trigger Source** | Software / Line0 (M8) / Line1 / Line2 | Line0 | Skjult når Trigger Mode er Off; deaktivert for array-medlemmer. Linje 0 er den optoisolerte eksterne utløserinngangen på M8. |
| **Bildefrekvens**| Auto / Manuell + verdi | Auto |**Auto**: Kameraets begrensning på bildefrekvensen er deaktivert – eksponeringen bestemmer fps, og boksen viser den faktiske frekvensen i sanntid.**Manuell**: du begrenser antall bilder per sekund med en glidebryter (1 opp til det båndbreddebegrensede maksimumet), basert på den aktuelle faktiske frekvensen. Array-medlemmer ser en skrivebeskyttet «N fps (live)» med «Angitt i array-innstillinger». |

### Nettverk / Transport

| Rad | Oppførsel |
| --- | --- |
| **Pakke størrelse**| 1500 (Standard) / 9000 (Jumbo) — standard**Jumbo**. |
| **Gjennomstrømning** | Skrivebeskyttet grense for koblingsgjennomstrømning i MB/s. Backenden fordeler dette på nytt mellom alle tilkoblede kameraer ved hver tilkobling/frakobling. |
| **Bufferhåndtering** | Skrivebeskyttet modus for bufferhåndtering. |

### Opptak

Ruten avsluttes med en **«Åpne opptaksinnstillinger…»**-knapp som går til [Ruten for opptaksinnstillinger](capture.md#the-capture-settings-pane) (deaktivert inntil et prosjekt er åpent — «Opprett eller åpne et prosjekt for å lagre opptak»). Hvis kameraet er skjult eller satt på pause, minner et tips deg om å vise/gjenoppta før du tar opp.

## Innstillingsruten for matriser

Åpnes med **tannhjulet**på en ARRAY-rad. Overskrift: matrisenavn med en blyant for å endre navn, og**×** for å lukke. Seksjonene nedenfor merket *kun kombinert* vises bare for matriser som er koblet sammen i kombinert visningsmodus.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->

### Synkronisering

Skrivebeskyttede **Master**-,**Slaves**- og**Synkroniseringslinje**-rader.

### Omgivelseslyssensor

Vises for både kombinerte og separate matriser:

* **Kalibreringsmål**-avkrysningsrute — «Oppdag MAPIR ArUco-mål og valider NDVI mot panelrefleksjons-LUT»; styrer måloverlegget og valideringstabellen for den kombinerte flisen.
* **Lyssensor**-rullegardinmenyen — knytter én DAQ til hele matrisen. Valget trer i kraft umiddelbart, overføres til hvert enkelt kameras egen**Lyssensor**-rullegardinmeny (du kan fortsatt overstyre innstillingen per kamera), og begynner å videresende spektra til matrisen.
* Live **Status**-linje: Av · «Venter på første spektrum…» · «Aktiv — alle kameraer i matrisen er belysningskorrigert» · «Ingen nytt spektrum de siste 3 sekundene — bruker fortsatt den siste avlesningen (ingen tidsavbrudd for utdatert data)…».
* Merknad i ruten: «Radiometrisk korreksjon for hele oppstillingen. Innstillinger per kamera overstyrer dette.»

### Opptak — ensartede sensorinnstillinger *(kun kombinert)*

Disse innstillingene gjelder ensartet for hvert medlem (endringer per medlem ville ødelegge synkroniseringen). Endringer lagres midlertidig og brukes samlet.

<!-- SCREENSHOT-NEEDED: array settings Capture section — Pixel Format, Binning, Resolution preset, the ROI crop W/H/X/Y fields with the "max WxH" hint and Reset button, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons; ideally with the live orange crop-preview box visible on the array tile. -->

| Kontroll | Valg / område | Hva det gjør |
| --- | --- | --- |
| **Pikselformat** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Ensartet sensorformat for alle enheter. |
| **Binning** | 1x1 / 2x2 / 4x4 | Maskinvarebasert binning — bevarer hele synsfeltet samtidig som signal-støyforholdet (SNR) og bildefrekvensen økes. Endring av dette tilbakestiller ROI-feltene til det nye fulle synsfeltet. |
| **Oppløsning** forhåndsinnstilling | Full / Halv / Kvart | Relativt til binning; fyller ROI-feltene med et sentrert utsnitt. |
| **ROI-beskjæring (px)**| Numeriske felt for B / H / X / Y | Sensorbeskjæring. Bredde/høyde justeres til multipler av 16 (minimum 64); forskyvninger justeres til multipler av 4. Et «maks WxH»-tips viser øvre grense, og**Tilbakestill** går tilbake til fullt synsfelt. Under redigering tegnes en oransje forhåndsvisningsboks for beskjæring i sanntid på matriseflisen (inkludert et skjema over hele sensoren når beskjæringen utvides utover). |
| **Utløserfrekvens**| Veksle mellom**Auto**og**Manuell**+ fps 0,5–10, trinn 0,5 |**Auto**(standard): backend beregner utløserfrekvensen ut fra oppløsning og båndbredde — inndatafeltet er deaktivert og viser den beregnede verdien.**Manuell**: låser verdien din ved**Apply**. |

Merknad i ruten: «Endringer i format/oppløsning starter alle kameraer på nytt kortvarig. Utløserfrekvensen gjelder umiddelbart.» **Apply**/**Cancel**-knappene ligger nederst i ruten.

### Justering (samregistrering) *(kun kombinert)*

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* **Rest**-felt: «RMS x,xx px» — grønt under 1 px, gult under 3 px, rødt ellers eller hvis et kamera mislyktes; «ingen profil» før første løsning.
* Sammendragslinje: «✓ Alle kameraer justert (N)» / «⚠ p/N kameraer justert —  <serial (filter)="">mislykket» / «Beskjæring aktiv — Kalibrer på nytt for å justere (bruker hele sensoren)» / «Venter på at eksponeringen skal stabilisere seg…».
* Tabell per kamera: kamera (de siste 4 sifrene i serienummeret + filter), reprojeksjonsfeil i px med antall treff («ref» for masterkameraet), og den normaliserte krysskorrelasjonsscoren for overlappingen mot minimumsgrensen på 0,35.
* **«Rekalibrer justering»**-knapp (viser «Kalibrer justering» før den første profilen) — kjører samregistrering på nytt med ferske bilder.
* **«Automatisk eksponering av kameraer for justering»** (merket som standard) — gjør mørke eller flate kameraer midlertidig lysere (eksponering først, deretter forsterkning) slik at de får tekstur å matche, og gjenoppretter deretter automatisk eksponering.

Den kombinerte forhåndsvisningen justeres automatisk ved åpning; kalibrer på nytt hvis fokus eller scenedybde har endret seg. Justeringen er **utformet for å gjelde kun for den aktuelle økten** — den lagres aldri i en profil, fordi den avhenger av scenens avstand i det øyeblikket. Opptak kan fortsatt eksporteres med pikselregistrering (se [Justerte eksporter](capture.md#per-array-controls)).

### Smart vignett

* **Aktiver korreksjon**-avkryssingsrute — bruker vignettestimatet per kamera på den radiometriske kjeden (både live**og** eksporter).
* **Kalibrer fra gjeldende visning**— rett først kameraoppsettet mot et ensartet mål (flat skjerm, vegg eller himmel); hvert kamera blir utjevnet individuelt, og statusen rapporterer «n/N kameraer · −x,x %» utjevningsgevinst.**Tøm** fjerner estimatet.
* Finjuster per kamera med **Vignette**-glidebryteren for hvert kamera i [Live Preview](#live-preview).

### Live Preview *(kun kombinert)** **Indeks**: aktiver avkryssingsruten + tannhjulet — åpner den delte [Indeksberegneren](#index-calculator-pane) med bånd tegnet fra**alle** medlemskameraene. En forhåndsvisningslinje for uttrykk nedenfor viser det gjeldende uttrykket («Ingen uttrykk angitt — åpne beregneren for å opprette et»), som oppdateres hvert sekund.
* **Renderoppløsning**-rullegardin (samme forhåndsinnstillinger som per kamera, standard 720p): høyden på live-visningsstrømmen**og** den lagrede størrelsen på den sammensatte eksporten. Merknad i ruten: «Forhåndsvisning + lagret sammensatt størrelse. Bilder per kamera eksporteres alltid i full oppløsning.»

### Vis lag *(kun kombinert)** **Aktiver**-avkryssingsrute (standard av — hovedkameraet vises direkte; på = lagdelt sammensatt bilde).
* **Forgrunn**/**Bakgrunn**-rullegardinmenyer: hvert medlemskamera (etter navn) eller**Indeks**. Når Forgrunn er Indeks, viser piksler utenfor LUT Min/Maks bakgrunnslaget.

### Delt visning *(kun kombinert)*

**««Vis medlemskameraer»**— en**Dele / Skjul medlemskameraer**-knapp som legger til hvert medlems egen live-feed som separate rutenettfliser ved siden av sammensetningen. Flisene leser matrisens eksisterende rammebuffer (ingen ekstra kameratilkobling). Kun rutenettvisning; lagres per matrise sammen med prosjektet.

### Funksjoner

Et skrivebeskyttet panel som oppdateres hvert 5. sekund:

* **Nivåbetegnelse**: «Samtidig opptak» (grønt) · «Samtidig opptak (FTD-forsinket utsending)» (grønt) · «Forsinket opptak (100 ms avvik)» (gult) · «Konfigurasjonen er for stor» (rød).
* **Rammehelse**: «x,xx % ufullstendig» — grønn under 1 %, gul under 5 %, rød ved 5 % eller mer.
* **Linklinje**: «NIC {mbps} Mbps – vedvarende {MB/s} MB/s».

Dette er arrayets sanntids båndbreddebudsjett. For den underliggende fps- og nettverksmodellen – og hva som skal endres når nivået blir gult eller rødt – se [Multi-Camera Arrays](arrays.md) og [CLI-referansen](../reference/cli-reference.md).



<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->## Ruten «Index Calculator»

Den tredje siden i sidepanelet, som deles av indeksverktøyet for hvert enkelt kamera og indeksverktøyet for det kombinerte arrayet (ett om gangen – overskriften lyder «Indekskalkulator — <camera name="">» eller «Indekskalkulator —<array name="">

»). Den mottar båndlisten (kameraets filter-naturlige bånd, eller alle bånd på tvers av arrayets medlemmer), det gjeldende uttrykket og LUT-konfigurasjonen (på/av, nivå – standard 3, min – standard 0,2, maks – standard 1), samt et live-indekshistogram. **Apply** bekrefter uttrykket; LUT-endringer gjelder live i forhåndsvisningen.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Innstillinger per kamera vs. array-styrt

Hurtigreferanse for hva som finnes hvor når et kamera er medlem av en matrise:

| Matrisestyrt (skrivebeskyttet i kamerapanelet) | Fortsatt per kamera innenfor en matrise |
| --- | --- |
| Pikselformat, oppløsning, binning | Automatisk eksponering (eksponering, forsterkning, mål, utjevning, ROI) |
| Utløsermodus/kilde, bildefrekvens | Støyreduksjon, skarphet, vignettering |
| | Orientering (speiling/rotasjon), visningsoverlegg, punktmåler |
| | Indeks (matriser med separat visning), lyssensorbinding |

Annen tverrgående funksjonalitet:

* **Kombinert vs. separat visning** velges ved tilkobling til matrisen: kombinert = én justert sammensatt flis (medlemsstrømmer kun via Split View); separat = hvert medlem gjengir sin egen synkroniserte flis. Et kamera viser aldri både en frittstående strøm og en matriseflis samtidig.
* **Automatisk gjenkobling**: når du åpner et lagret prosjekt, gjenopprettes kameraene og matrisene, og alle lagrede innstillinger brukes på nytt på backend før strømmene gjenopptas.
* ****Opptakskontroll**: skjulte eller pausede kameraer ekskluderes fra «Capture All»; et array blokkeres fullstendig bare når ALLE medlemmene er skjult/pauset. Se [Opptaksinnstillinger og -moduser](capture.md).

## Hvordan innstillingene lagres

Tilstanden til kamerafanen lagres **sammen med prosjektet**, ikke i nettleseren:

* Hver reaktiv endring tar et øyeblikksbilde av kameraene og matrisene i prosjektets `cameras.json` (debounced 500 ms). Dette omfatter kameranavn og farger, innstillinger for eksponering/forsterkning/AE, pikselformat/oppløsning/binning, utløserfrekvens, forhåndsvisningsinnstillinger (renderoppløsning, støyreduksjon, skarphet, vignettering, fargeprofil, metning/kontrast), orientering, overlegg, kanaloppsplitting, indekskonfigurasjon, innstillinger for prediktiv AE, AE-ROI, matrisenavn, visningsmodus, innstillinger for matriseopptak (inkludert ROI-beskjæringsposisjon) og rutenettblokken (feed-zoom, visningsmodus, rutenettlås, manuell flisrekkefølge, skjulte kameraer, lukkede fliser, aktivt kamera).
* Lyssensorbindinger lagres i prosjektets `sensors.json`.
* Når prosjektet åpnes på nytt, kobles maskinvaren til igjen, og alle innstillingene blir brukt på nytt.
* **Ingen prosjekt åpent = kun sesjon**: uten prosjekt beholdes ingenting når Chloros lukkes.
* Kun sesjon uavhengig av prosjekt: pausetilstand, spotmålerprøver, avmerkingsboksen for kalibreringsmål per kamera (alltid avmerket ved åpning) og matrisejusteringsprofilen (beregnes på nytt per sesjon etter design).
* Ett unntak: **Valg for eksport av**opptaksinnstillinger** og opptaksmodus lagres per prosjekt i appens lokale lagringsplass i stedet for i `cameras.json` — se [Opptaksinnstillinger og -moduser](capture.md).</array></camera></serial>
