# Kamerainnstillinger

Fanen **Kameraer** er Chloros

’ kontrollpanel for LATTICE-kameraer i sanntid: et hovedområde som viser hvert tilkoblet kamera som en sanntidsrute, og en sidepanel som kan veksle mellom tre sider – **kameralisten**, et**innstillingsvindu**(innstillinger per kamera, for kameraoppstilling eller for opptak – én om gangen) og**indekskalkulatoren**. Denne siden dokumenterer alle kontrollelementene i kameralisten, innstillingsruten per kamera og innstillingsruten for kameraoppstilling. Opptaksmoduser, valg av eksporttype og «Capture All»-flyten finnes på den tilhørende siden [Opptakinnstillinger og -moduser](capture.md).

Fanen «Kameraer» vises i sidepanelet så snart Chloros

-backenden er klar. Alle kontrollene nedenfor kommuniserer med den lokale backenden via `127.0.0.1:5000`; endringer gjelder umiddelbart for live-kameraet, med mindre annet er angitt.

## Kameratyper brukt på denne siden

Kontrollene vises eller skjules avhengig av hvilken type kamera som er valgt. Håndboken bruker følgende begreper gjennomgående:

| Begrep | Betydning | Filterkanaler |
| --- | --- | --- |
| **RGB

-kamera** | LATTICE M3C med FRGB-filter (modellen inneholder `-FRGB`) | Red

/ Green

/ Blue

|
| **Bayer-multispektralt** | LATTICE M3C med FRGN, FOCN eller FNGB | FRGN: Red

t / Green

t / NIR

· FOCN: Orange

t / Cyan

t / NIR

· FNGB: NIR

/ Green

t / Blue

t |
| **Mono (M3M)** | LATTICE M3M — ett smalbåndsfilter, ett kalibrert bånd | Enkeltbånd |
| **Array-medlem** | Et kamera tilkoblet som del av et synkronisert array (kombinert eller separat visning) | I henhold til filteret |

 RGB

-kameraer får fotometrisk behandling (hvitbalanse, fargeprofiler, gamma); multispektrale og mono-kameraer får den radiometriske kjeden og hopper over de fotometriske kontrollene. Array-medlemmer overfører innstillinger på strømnivå (pikselformat, oppløsning, binning, utløser, bildefrekvens) til arrayet — disse radene blir skrivebeskyttet i ruten per kamera og flyttes i stedet til ruten for arrayinnstillinger.

## Hovedområdet for strøm

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->



Når ingen kameraer er tilkoblet, viser strømområdet en velkomstmelding med teksten **«Koble til et kamera for å komme i gang»**med to knapper:**Koble til kamera**(grønn, åpner tilkoblingsdialogen for enkeltkamera) og**Koble til array** (blå, åpner tilkoblingsdialogen for array). Selve tilkoblingsdialogene er dokumentert i [Koble til kameraer](connecting.md); array-begreper (synkronisering, nivåer, båndbredde) i [Kameraoppsett med flere kameraer](arrays.md). Når du åpner et lagret prosjekt som inneholder kameraer, viser startskjermen i stedet en spinner med «Åpner N lagrede kameraer på nytt…» mens Chloros

gjenoppretter strømmene fra forrige økt.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Øverste linje

| Kontroll | Hva den gjør |
| --- | --- |
| **Veksle mellom visningsmodus**| Veksler mellom**rutenettvisning**(alle ruter som celler) og**listevisning**(rader i full bredde: hvert kombinert oppsett som én sammensatt rad, hvert medlem av et**separat** oppsett som sin egen rad med egen strøm og kalibreringsmåltabell, deretter ÉT frittstående kamera – det aktive – under). Verktøytips: «Bytt til rutenettvisning» / «Bytt til listevisning». |
| **Rutenettlås**(hengelås) | Standard**låst** – ruter låst på plass. Lås opp for å dra og omorganisere ruter til hvilket som helst felt (mellomrom beholdes). Rutenettet låses automatisk på nytt hver gang et nytt kamera kobles til. Verktøytips: «Lås opp rutenett (aktiver dra og slipp av fliser)» / «Lås rutenett (frys fliser på plass)». |
| **Feed-zoom**-glidebryter | Flisestørrelse, fra 60 px opp til full bredde på beholderen. Cellene beholder et 4:3-bildeforhold. Når cellebredden er under 200 px, skjules navnet og fps-overleggene for å holde ruten ren. |

### Feed-ruter

Hvert kamera gjengir en sammensatt live-rute; et kamera kan i tillegg vise tre gråtonede **kanaloppsplittede** ruter (se [Kanaloppsplitting](#display-overlays-drawn-over-the-live-feed)), og matriser viser en kombinert flis. Den aktive flisen har en markeringsring i kameraets (eller matrisens) farge.

Når du holder musepekeren over en flis, vises en **X**-lukkeknapp:

* Hvis du lukker en **sammensatt** flis mens kanaloppdelingene fortsatt er synlige, skjules bare den sammensatte flisen.
* Hvis du lukker den **siste synlige flisen til et frittstående kamera**, kobles det kameraet fra.
* **Oppdelte fliser for medlemmer av en kombinert matrise kobler aldri fra** kameraet – de skjules bare.

Når rutenettet er ulåst, kan du dra en hvilken som helst flis til et hvilket som helst spor; oppsettet lagres sammen med prosjektet.

## Sidepanel – kameraliste

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

Den første siden i sidepanelet viser en liste over alle tilkoblede kameraer og matriser:

* **Koble til kamera**(grønn) /**Koble til oppstilling** (blå, viser «Oppdager...» mens den skanner). Begge er deaktivert mens en tilkoblingsdialog er åpen.
* **Ta opp alt** (rød) — tar opp alle kameraene på listen med eksporttypene valgt i Opptakinnstillinger. Krever et åpent prosjekt. Fullstendig dokumentert i [Innstillinger og moduser for opptak](capture.md).
* **Tannhjul for opptaksinnstillinger** (ved siden av «Ta opp alt») — åpner [panelet for opptaksinnstillinger](capture.md#the-capture-settings-pane). Deaktivert uten et prosjekt eller mens opptak pågår.

### Kamerarader

Hver kamerarad viser en fargekodet ramme (kameraets tilpassede farge), en «CAM»-etikett — med en blå **M**(master) eller grønn**S** (slave) som angir rolle for medlemmer i oppstillingen — samt visningsnavnet. Standardnavnet er `LATTICE-MODEL (serial)`; du kan gi det et nytt navn fra innstillingsruten for hvert kamera. Knapper i raden:

| Knapp | Effekt |
| --- | --- |
| **Øye**| Veksle mellom synlighet. Skjulte kameraer fjernes fra rutenettet og**ekskluderes fra «Capture All»**. |
| **Tannhjul** | Åpner innstillingspanelet for hvert kamera (neste avsnitt). |
| **Pause / Spill av**| Fryser live-forhåndsvisningen**kun på skjerm** — opptaket på serversiden fortsetter. Kameraer i pause kan ikke ta opp. |
| **X** | Koble fra. Brukergrensesnittet oppdateres umiddelbart (i beste fall); selve frakoblingen i bakgrunnen kan ta 10–30 sekunder å fullføre. |

### Array-rader

En array-rad viser et «ARRAY»-merke i arrayets farge, arrayets navn (kan omdøpes i array-innstillingene) og en **DAQ · på/av**-knapp —**på** når lyssensoren på array-nivå er aktivert *eller* et hvilket som helst medlem har en sensor per kamera; verktøytipset viser nøyaktig hvilken sensor som sender data til hva. Medlemskameraene er oppført innrykket under med egne rader. Knapper i matriserader: **øye**(skjuler/viser ALLE medlemmer samtidig),**tannhjul**(innstillingsruten for matrisen),**X**(kobler fra hele matrisen).

Statusen til lyssensoren (DLS) som brukes i matriserader og i innstillingsruten for matrisen, har fire tilstander:**av**,**venter**(ingen spektrumdata ennå),**aktiv**(et spektrum kom inn i løpet av de siste 3 sekundene) og**foreldet** — ingen nye spektrumdata på 3 sekunder, men den siste avlesningen *brukes fortsatt* (DAQ-målinger utløper aldri på innfangingsbanen).

Inntil du endrer rekkefølgen, viser listen først matriser, deretter frittstående kameraer, med kameraer (og hvert matrisemedlems kameraer) i **kamerarekkefølge**: M3C (Bayer) før M3M (mono); blant M3C, FRGB, FRGN, FOCN, deretter FNGB; M3M etter filterbølgelengde, laveste først. Rutenettets standardoppsett følger samme rekkefølge. Du kan dra frittstående kameraer og hele matrisegrupper forbi hverandre i sidepanelet for å endre rekkefølgen på listen; matrisemedlemmer kan ikke dras uavhengig av hverandre.

## Innstillingsruten for hvert kamera

Åpnes med **tannhjulet** på en kamerarad. Ruten glir over kameralisten.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->



**Overskrift**: kameraets**fargeprøve**(klikk for å åpne en innebygd fargevelger — angir fargen på sidefeltets ramme og utvelgelsesringen for fliser),**navnet**med en blyantknapp for**Omdøpe**(hvis du lagrer et tomt navn, går det tilbake til standardnavnet `MODEL (serial)`) og**×** for å lukke.

### Live-histogram

Øverst i ruten vises et live-luma-histogram beregnet fra JPEG

-forhåndsvisningen med en frekvens på ca. 8 Hz. Gjennomsnittet er Bayer-vektet — (R+2G+B)/4 — for å samsvare med kameraets egen AE-måling.

* **Den Orange

n stiplede linjen**= AE-målet.**Dra den horisontalt for å endre målet** — én kommando sendes når du slipper, og når du drar, skifter AE-målmodusen til Manuell.
* **Den Green

n heldekkende linjen** = den faktiske gjennomsnittlige lumaen (det AE leverer for øyeblikket).
* **RGB

-knapp** (øverst til høyre): veksler mellom båndvise overleggshistogrammer farget etter kameraets filter (f.eks. på FRGN: grå NIR

, grønn, rød). På monokameraer (M3M) står det «MONO» på knappen, og den er deaktivert — mono viser alltid luma-histogrammet for ett enkelt bånd.
* Etikettene på X-aksen følger sensorenes bitdybde for det gjeldende pikselformatet: 0..255, 0..1023, 0..4095 eller 0..65535.

### Rader

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

med kamerainformasjon

| Rad | Oppførsel |
| --- | --- |
| **Modell** | Skrivebeskyttet (f.eks. `LATT-M3C-L87-FRGN`). |
| **Radiometrisk kalibrering** | Green

t **«Aktiv»**-merke med en bildetekst som viser kalibreringsnivå, hash, kalibreringsdato og båndliste, lastet inn fra kameraets kalibreringspakke (se [Fabrikkradiometrisk kalibrering](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Skjult for RGB

-kameraer** — disse har en fotometrisk hvitbalansekalibrering, ikke strålingsintensitet per bånd. |
| **Kalibreringsrapport**|**Last ned**-knapp — åpner kameraets NIST-kalibreringssertifikat (per serienummer) i PDF-format i operativsystemets visningsprogram. Hvis sertifikatet ikke er lagret i hurtigbufferen ennå, viser Chloros

et tips i stedet. |
| **Serienummer** | Skrivebeskyttet. |
| **Firmware**| Viser gjeldende versjon, og finner deretter den tilgjengelige versjonen for denne modellen (lagret per modell — en gruppe på N kameraer sjekker serveren én gang). Status: «Sjekker…» →**«Oppdater til X»**-knapp → «Blinker…» → «Oppdatert fra A til B» / «Mislyktes: …» / «Hoppet over: …» / grønt**«Oppdatert»**. Verktøytips for oppdateringsknappen: «Tilbakestill til fabrikkinnstillinger + oppdatering + omprogrammering av UserSet1. ~2–3 minutter; ikke koble fra.» |
| **IP** | Skrivebeskyttet. |
| **Temperatur** | Skrivebeskyttet, oppdateres hvert 3. sekund. Blir oransje ved ≥65 °C og rød med et ⚠ ved ≥75 °C. |
| **Kalibreringsmål** avkryssingsrute | Aktiverer ArUco-refleksjonsmåldeteksjon med en NDVI

-valideringstabell per panel under live-strømmen (listevisning) — se [Kalibreringsmåltabellen](#the-calibration-target-table). Kun for den aktuelle økten — åpnes alltid i av-posisjon. |
| **Lyssensor**-rullegardin | Knytter en DAQ-lyssensor (DAQ-E/M/U, fra listen på fanen «Lyssensorer») til dette kameraet for belysningskorreksjon av nedadrettet lys (DLS) og prediktiv automatisk eksponering. ««Ingen» fjerner tilknytningen. Hvis ingen sensorer er tilkoblet, viser rullegardinmenyen «(ingen sensorer tilkoblet — åpne DAQ-fanen)». Tilknytningen lagres sammen med prosjektet. |

#### Tabellen «Kalibreringsmål»

Tabellen er **kun til valideringsformål** («kun validering — endrer ikke kalibreringen»). Overskriften angir kameraets serienummer, filter, målnummer og markørstørrelse i piksler, og den har én rad per panel:

* **Panel**, deretter én kolonne per bånd med panelets verdi — et Bayer-kamera viser sine tre bånd etter navn og bølgelengde (f.eks. FRGN: Red

660 nm / Green

550 nm / NIR

850 nm); et **mono (M3M)-kamera viser sitt ene bånd** (f.eks. `F750`) og har ingen NDVI

-kolonner.
* **Sat** — andelen av panelets piksler som er mettede (gult fra 1 %, rødt fra 5 %).
* **NDVI

Meas**,**NDVI

Theo** (panelets referanse-reflektans-NDVI

) og **Δ**— kun for Bayer-multispektralkameraer. Et lite tegn etter «NDVI

Meas» angir hva den målte NDVI

-verdien er beregnet ut fra:**ρ**(reflektans — en lyssensormåling er tilgjengelig),**L**(strålingsintensitet — ingen lyssensormåling) eller**DN**(rå kameraverdier). Kun en**ρ**-NDVI

er sammenlignbar med NDVI

Theo: for L og DN er Δ-kolonnen nedtonet, og en merknad angir dette («Ingen lyssensormåling: NDVI

er basert på strålingsintensitet, så Δ er ikke en kalibreringskontroll» / «NDVI

er basert på rå kameraverdier, så Δ er ikke en kalibreringskontroll»). Med ρ er Δ grønn under 0,02, gul under 0,05 og rød ellers.

<!-- SCREENSHOT-NEEDED: a 1.3.0 Calibration Target table under a live feed in list view — a Bayer camera's table showing the band columns, Sat, and "NDVI Meas ρ" / NDVI Theo / Δ; ideally next to a mono camera's single-band table with no NDVI columns. -->

### Eksponering og forsterkning

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Alle numeriske inndata her bruker spinnere som akselererer når du holder inne: trykk = ±1, hold inne &gt;1,5 s = ±10, hold inne &gt;3 s = ±100. Verdien sendes til kameraet når du slipper.

| Kontroll | Område / valg | Standard | Gjelder for | Hva den gjør |
| --- | --- | --- | --- | --- |
| **Eksponering (us)**| Kameraets live min./maks. | Auto | Alle | Eksponeringstid i mikrosekunder, med en**Auto/Manuell**bryter. Auto = kontinuerlig automatisk eksponering på kamerasiden. Hvis du stiller inn manuell eksponering, forblir**Gain** på Auto (hvis den var det), slik at du fritt kan bruke hele kameraets forsterkningsområde, inntil du redigerer forsterkningen eller stiller den til Manuell. |
| **Forsterkning (dB)**| Kameraets sanntids min./maks. (f.eks. opptil 48 dB) | Manuell (av) | Alle | Analog/digital forsterkning med egen**Auto/Manuell**-bryter. |
| **AE-mållysstyrke**| 0–255 | 80, modus**Auto**| Alle (kan redigeres når AE eller automatisk forsterkning er på) | Lysstyrken AE sikter mot. I**Auto**(standardinnstillingen) velger en histogrambasert bakgrunnskontroller målet selv, slik at eksponeringen holdes på 60–75 % av sensorens maksimum. Hvis du skriver inn en verdi eller drar den oransje linjen i histogrammet, skifter den til**Manuell**. |
| **AE-utjevning** | 0,5–40, trinn 0,1 | 8,0 | Alle | AE-demping. Verktøytips: «Lavere = AE reagerer raskere (kan pulsere ved høy bildefrekvens). Høyere = jevnere / tregere.» Verdier langt under standardinnstillingen kan føre til at AE pulserer og destabiliserer strømmingen ved høye bildefrekvenser; 8,0 er den stabile standardinnstillingen. |
| **AE Region of Interest**| Aktiver avkryssingsrute +**Sikt**-knapp | Av | Alle | Når denne er aktivert, måler AE kun det grønnstrekkede området i stedet for hele bildet.**Sikte** aktiverer «klikk for å plassere» på live-strømmen: et klikk sentrerer et område på 30 % av bildet; klikk og dra tegner et tilpasset rektangel (minst 5 % × 5 %). «Aim» deaktiveres automatisk etter én plassering. Området tilpasses til kameraets egne koordinater i henhold til eventuelle rotasjons- eller speilingsinnstillinger du har angitt, og lagres sammen med prosjektet. |
| **AE Tune Speed** | 0,1–5, trinn 0,1 | 1,0 | Kun for Array-medlemmer | Hvor raskt det automatiske AE-målet sporer endringer i scenens lysstyrke; 1,0× sjekker på nytt hvert 2,5. s. |
| **Høylysbeskyttelse** | Streng (1 %) / Normal (5 %) / Løs (15 %) | Streng | Kameraer som eksponerer innstillingen | Hvor mye av bildet som kan klippes til hvitt før AE gjør bildet mørkere. |

{% hint style="info" %}
**Lyskrav for Bayer-multispektralkameraer (RGN

/ OCN

/ NGB

):** scenen må ha nok lys i alle tre kanalene, ellers fungerer ikke kalibreringen riktig — én enkelt sensoreksponering dekker alle tre spektrene. Bruk en DAQ-lyssensor til å måle lyset, eller gå over til fullt mono (M3M) slik at hvert bånd får sin egen eksponering. Hvis en opptak bryter dette, oppdager Chloros

det og varsler deg (varselet om «unmix-clamp»). I sanntid viser en kanaloppdelt rute **«Bånddata med lav pålitelighet · N %»** når unmix har klemt det aktuelle båndet på 25 % eller mer av dets piksler.
{% endhint %}

{% hint style="info" %}
**Hvordan automatisk eksponering fordeler eksponeringstid og forsterkning (1.3.0).** Når forsterkningen er satt til «Auto», holder Chloros

forsterkningen på 0 dB inntil eksponeringen har nådd den lengste tillatte verdien (den øvre grensen for automatisk eksponering, utløsningsperioden for et array eller eksponeringsgrensen i DAQ); først da øker forsterkningen. I et array kommer utløsningsfrekvensen først: automatisk eksponering fyller perioden som frekvensen angir, deretter øker forsterkningen, slik at en mørkere scene ikke lenger senker en automatisk utløserfrekvens (i versjon 1.2.0 gjorde den det). En **manuell**eksponering på et array-medlem som varer lenger enn utløserperioden, senker arrayets utløserfrekvens for å tilpasse seg den; en automatisk frekvens gjenopprettes så snart hvert element som bremset den, er tilbake på automatisk eksponering. Når både**Eksponering**og**AE-mållysstyrke** er satt til Auto og en DAQ-lyssensor gir en måling, holdes eksponeringen for hvert kamera fast så snart den automatiske eksponeringen har stabilisert seg, så lenge det målte lyset holder seg innenfor ca. ±25 %; en større, vedvarende lysendring, en omstart av innstillingene (for eksempel en endring av ROI) eller din egen endring av eksponeringen utløser den på nytt.
{% endhint %}

{% hint style="info" %}
**Hvordan automatisk eksponering fungerer (1.3.0).**

* Ved manuell eksponering forblir forsterkningen på «Auto» inntil du endrer forsterkningen.
* Små høylys gjør ikke scenen mørkere: et speilende høylys på opptil 1 % av bildet ignoreres, og i panelene til et kjent kalibreringsmål, opptil 0,5 % av et panel.
* DAQ-eksponeringstaket gjelder kun mens et kalibreringsmål er i synsfeltet; når ingen paneler er synlige, eksponerer kameraet for scenen.
* Eksponeringen holder seg stabil ved konstant lys og stabiliserer seg i løpet av få sekunder etter en stor endring. En klipping begrenser ikke lenger eksponeringen permanent, og en endring i ROI finner eksponeringen på nytt.
* Lysstyrkemålet går ikke utenfor kameraetsrekkevidde, slik at et lys som kommer tilbake ikke forårsaker lysreflekser i bildet.
* Et kamera hvis automatiske eksponering slutter å reagere i sterkt sollys, holdes på nivået for lyset det befinner seg i, målt av lyssensoren eller, hvis ingen er tilkoblet, av kameraets egne bilder, og slippes når lyset endres. En mørk eller tildekket lyssensor forstyrrer ikke dette.
{% endhint %}

### Pikselformat og oppløsning

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

**Array-medlemmer** viser skrivebeskyttede «Current» (format + BxH) og «Binning»-rader med merknaden «Angis i array-innstillinger» — en strøm-omstart på ett medlem ville ødelegge synkroniseringen, så disse administreres i [array-innstillingsruten](#array-settings-pane).**Frittstående kameraer** har:

| Kontroll | Valg | Hva den gjør |
| --- | --- | --- |
| **Pikselformat** | 8-bit / 10-bit / 12-bit / 16-bit | Sensorens bitdybde; formatfamilien er sensorens. Standardverdien er 12-bit, og et prosjekt lagrer det du angir, inkludert 8-bit, og kobler seg til igjen med denne innstillingen. |
| **Oppløsning** | Full / Halv / Kvart | Relativt til gjeldende binning: Full = 2048/N × 1536/N for N×N-binning. |
| **Binning** | 1x1 (ingen) / 2x2 / 4x4 | Maskinvarebasert N×N-binning — større verdier gir lavere oppløsning, men øker SNR og bildefrekvens. Endring av dette starter strømmen på nytt og tilbakestiller eventuelle ROI-er til det nye fulle synsfeltet. |
| **Gjeldende** | skrivebeskyttet | Den faktiske WxH og (x, y)-forskyvningen som gjelder. |

### Live-forhåndsvisning

Alt i denne delen gjelder **kun visningssiden**— det endrer det du ser i live-strømmen, mens lagrede opptak forblir lineære og uendret — med ett unntak:**Vignette** er radiometrisk og påvirker også eksporter (som nevnt nedenfor).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->



| Kontroll | Område / valg | Standard | Gjelder for | Hva den gjør |
| --- | --- | --- | --- | --- |
| **Renderoppløsning** | 360p (raskest) / 480p / 720p / 1080p / Sensorens opprinnelige oppløsning (tregst) | 720p | Alle | Høyden der backenden kjører den radiometriske forhåndsvisningskjeden. Lavere innstilling gir høyere bildefrekvens uten å endre synsfeltet. |
| **Indeks**| Avkryssingsrute for aktivering + tannhjul | Av | Kun Bayer-multispektral,**ikke** kombinerte matriser | Forhåndsvisning av live vegetasjonsindeks. Tannhjulet åpner det delte [Indekskalkulator](#index-calculator-pane)-vinduet, som er forhåndslastet med kameraets naturlige filterbånd (f.eks. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Det tilpassede uttrykket pluss LUT (på/av, standardnivå 7-stop, standard min. 0,2, standard maks. 1) beregnes for hvert forhåndsvisningsbilde. Medlemmer av kombinert matrise skjuler denne raden — matrisen har én delt indeks. |
| **Hvitbalanse** | Av / En gang / Kontinuerlig + en knapp for ny opptak | Kontinuerlig | Kun RGB

| Live hvitbalanse. Oppdateringsknappen tar et nytt bilde av hvitbalansen fra det gjeldende DLS-spektrumet (deaktivert når modusen er Av). |
| **Gamma** | På / Av | På | Kun RGB

| Vis gamma (γ = 2,2 LUT) i liveforhåndsvisningen. Lagrede opptak forblir lineære. |
| **Støyreduksjon** | Avkryssingsrute + styrke 0–100 | Av / 50 | Alle (per kamera, også innenfor matriser) | Bilateralt filter i live-forhåndsvisningen. Høyere verdi = jevnere, men mykere detaljer. |
| **Skarphet** | Avkryssingsrute + styrke 0–100 | Av / 30 | Alle | Uskarp maske i liveforhåndsvisningen, brukes til slutt. Kan forsterke støy. Kun i forhåndsvisning. |
| **Vignett**| Avkryssingsrute + styrke 0–100 | Av / 0 | Alle | Manuell fjerning av gjenværende vignett (lyser opp hjørnene), lagt på toppen av matrisens Smart Vignette-estimat.**Radiometrisk — påvirker livevisningen OG eksportene**, i motsetning til Støyfjerning/Skarphet. |
| **Fargeprofil** | Raw / Lineær / Naturlig / Forbedret / Egendefinert temperatur | Naturlig | Kun RGB

| Se nedenfor. |
| **Fargetemperatur** | 2000–10 000 K, trinn 100 | 5500 K | Kun RGB

, tilpasset temperaturprofil | Fester hvitbalansen til en fast korrelert fargetemperatur (DLS-inngang ignoreres). Den sist valgte Kelvin-verdien huskes ved bytte av profil. |
| **Høylys** | −100 til 0, trinn 1 | 0 (av) | Kun RGB

| Nytt i 1.3.0. Mørkner de lyseste, hviteste områdene i bildet (0 = uendret, −100 = sterkest). Gjelder for live-forhåndsvisning **og for opptak på skjermnivå (Preview)**, som lagrer innstillingen (`Chloros:DisplayHighlights`); eksport av strålingsintensitet og reflektans påvirkes aldri. Det kan ikke gjenopprette sensordata som er klippet – senk eksponeringen for å unngå dette. Zebra-overlegget følger det mørkere bildet. Deaktivert med Raw-fargeprofilen («Brukes ikke med Raw-fargeprofilen.»). |
| **Metning** | 0–200 (100 = nøytral) | 100 | Kun RGB

| HSV-metning i live-forhåndsvisningen. |
| **Kontrast** | 0–200 (100 = nøytral) | 100 | Kun RGB

| Lineær kontrast rundt midtgrått i liveforhåndsvisningen. |
| **Speil H / Speil V** | Avmerkingsbokser | Av | Alle | Speilvend forhåndsvisningen horisontalt / vertikalt. |
| **Rotasjon**| 0° / 90° / 180° / 270° | 0° | Alle | Roter forhåndsvisningen. Retningen brukes på slutten av forhåndsvisningskjeden i backend.**Lagrede opptak (1.3.0):**Forhåndsvisning, Radiance, Reflectance og Index-produkter fra et frittstående kamera roteres ved hjelp av Speil/Rotasjon, mens**Raw og Debayered beholder kameraets opprinnelige orientering**; hver opptaksfil lagrer innstillingen i sin XMP, slik at bearbeiding av en raw-fil senere anvender den på samme måte.**Filer i et array beholder sin opprinnelige orientering** (innstillingen lagres, men brukes ikke), og sammensatte visninger av arrayet ignorerer den. |**Semantikk for fargeprofiler** (RGB

-kameraer):

* **Raw** — omgår behandlingskjeden fullstendig.
* **Lineær** — mørksignal + flatfelt + hvitbalanse; ingen fargematrise, ingen gamma.
* **Naturlig** *(standard)* — lineær pluss den målte fargekorreksjonsmatrisen og en scenetilpasset tonekurve.
* **Forbedret**— Naturlig pluss vibrans og CLAHE lokal kontrast. Den ekstra kostnaden gjelder**kun live-forhåndsvisning** — lagrede opptak får alltid full etterbehandling uavhengig av profil.
* **Egendefinert temperatur** — Naturlig med hvitbalanse festet til den valgte Kelvin-verdien.

{% hint style="warning" %}
For «Naturlig», «Forbedret» og «Egendefinert temperatur» viser ruten en toneanmerkning: bildene lyses opp i henhold til sin egen scene, slik at lagrede *visnings*bilder ikke er sammenlignbare fra bilde til bilde. **Eksporter radianse eller reflektans for målinger.**
{% endhint %}

### Visningsoverlegg (tegnet over live-strømmen)

Disse finnes kun i brukergrensesnittet — de legges over videoen, uten å påvirke strømmen eller opptakene.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Overlegg | Kontroller | Standard | Hva det gjør |
| --- | --- | --- | --- |
| **Zebra** | Avkryssingsrute + terskel 200–255 | Av / 250 | Magenta diagonale striper på avskårne piksler. |
| **Kryss** | Avkryssingsrute | Av | Markering for rammesenter. |
| **Rutenett** | Av / 3 × 3 / 9 × 9 | Av | Komposisjonsrutenett. |
| **Histogram** | Avkryssingsrute + bredde 0,10–0,90 av bildet | Av / 0,25 | En histogramstripe på strømmen. |
| **Fokusmarkering** | Avkryssingsrute + terskel 20–200 + fargeprøve | Av / 80 / `#ff5722` | Sobel-kantmarkering for fokusering. |
| **Kanaloppsplitting** | «Vis oppsplitting (Red

/ Green

/ NIR

)» / «Skjul delinger»-knapp | Skjult | Legger til tre uavhengige gråtoner per kanal ved siden av sammensetningen (knappetiketten følger kameraets filterkanaler). Hver delingsrute kan dras og har samme kantfarge som kameraet. Ikke tilgjengelig på monokameraer. Lagres med prosjektet. |

### Spotmåler

* **Klikk for å ta prøve**-avkryssingsrute: klikk på livebildet for å ta en prøve av en enkelt piksel (et kryssmerke markerer den), eller klikk og dra over et område for å beregne et pikselgjennomsnitt.**Tøm**fjerner prøven og kryssmerket. Kan ikke brukes samtidig med AE-ROI**Sikt**-modus.
* ****Vis**-rullegardin:**Raw (bitdybde)**— opprinnelige digitale tall med sensorens bitdybde (f.eks. 12-bit → 0..4095) — eller**Visning (8-bit)**(standard). Når en live-indeks er aktiv, viser**Display** i stedet den beregnede indeksverdien (f.eks. NDVI

).
* Avlesningspanelet viser pikselkoordinater, bildestørrelse, pikselformat, bitdybde og en kanaltabell (Chan / Value / %) med båndbetegnelser og bølgelengder; Bayer-grønne par blir gjennomsnittsberegnet; regionprøver viser «N px avg».

Spotmeter-tilstanden gjelder kun for den aktuelle økten.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Prediktiv automatisk eksponering (DLS-drevet)

Denne delen vises kun når **minst én DAQ-lyssensor er tilkoblet** — løseren trenger et live nedadrettet spektrum for å drive den.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->



| Kontroll | Område | Standard | Hva den gjør |
| --- | --- | --- | --- |
| **Aktiver** | Avkryssingsrute | På (frittstående kameraer) | En lukket løsningsalgoritme bruker DLS-spektrumet samt skalærene fra kameraets kalibreringspakke for å plassere det lyseste båndet nær metning, samtidig som det mørkeste båndet holdes over SNR-grensen — én eksponeringsskriving per løsning, ingen stabiliseringssløyfe. Utviklet for solcelledrevet tidsforløp der hvert bilde må være riktig eksponert. Backenden faller automatisk tilbake til reaktiv AE når DLS-avlesningen er utdatert/mangler eller kalibreringspakken ikke er lastet inn. |
| **Utjevning (α)** | 0,05–1,0, trinn 0,05 | 0,3 | Utjevning av påfølgende prediktive løsninger (lavere = jevnere). |
| **Scenerefleksjon**|**Rekalibrer ρ**-knapp | — | Beregner på nytt scenerefleksjonsfaktoren som løseren bruker. |

{% hint style="info" %}
**Array-tilkobling deaktiverer prediktiv AE som standard** — for arrayer håndterer Chloros

’ smarte AE samt kameraets automatiske eksponering eksponeringen (med metningsbeskyttelse), og den prediktive AE-ens estimat for enkelt scene-refleksjonsestimat er ikke sikkert på tvers av blandede scener. Du kan slå den på igjen per kamera her hvis du spesifikt ønsker DLS-drevet radiometrisk eksponering.
{% endhint %}

**DAQ-drevet eksponeringstak og innfallsfestet AE.** Uavhengig av avmerkingsboksen ovenfor, når en DAQ-lyssensor er tilordnet et RGB

-kamera, beregner Chloros

— ut fra den målte absolutte nedadgående irradians — den maksimale eksponeringen × forsterkningen der en overflate med 100 % reflektans forblir under klipping, og bruker den som et **tak**for automatisk eksponering. Siden versjon 1.3.0 gjelder taket kun så lenge et kalibreringsmål er innenfor kameraets synsfelt (sett i bildet i løpet av de siste 10 sekundene); når det ikke er noe mål i synsfeltet, kjører i stedet kameraets vanlige automatiske eksponering, med sin egen høylysbeskyttelse, slik at resten av scenen ikke holdes mørkere enn nødvendig. Mens taket er aktivt, er kameraet**festet til innfallende lys**: det kjører i åpen sløyfe med eksponering målt etter innfallende lys og forsterkning på 0 dB — eksponeringen følger det målte lyset, ikke innholdet i scenen. Siden takverdien kun kan forkorte eksponeringstiden, kan den ikke i seg selv forårsake klipping. Siden versjon 1.3.0 beregnes takverdien med kameraets egen eksponeringskonstant fra kalibreringspakken (en flåtegjennomsnittsverdi bare når pakken ikke inneholder en). Takverdien deaktiveres automatisk – og normal scene-AE gjenopptas – når det ikke er noe kalibreringsmål i synsfeltet, DAQ-avlesningen mangler, er foreldet (&gt;30 s), mørk, eller hvis ≥15 % av bildet klipper ved den faste eksponeringen (noe som betyr at sensoren og kameraet registrerer ulik belysning). Det finnes ingen bryter i brukergrensesnittet; dette er standardoppførsel når et RGB

-kamera er koblet til en DAQ.

### Innsamling og utløser

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

Array-medlemmer viser i tillegg skrivebeskyttede rader for **Rolle**(Master i blått / Slave i grønt),**Synkroniseringslinje**og**Peers**.

| Kontroll | Valg | Standard | Merknader |
| --- | --- | --- | --- |
| **Utløsermodus** | Av / På | På | Deaktivert for array-medlemmer (arrayen styrer utløsningen). |
| **Utløserkilde** | Programvare / Linje 0 (M8) / Linje 1 / Linje 2 | Linje 0 | Skjult når utløsermodus er av; deaktivert for array-medlemmer. Linje 0 er den optoisolerte eksterne utløserinngangen på M8. |
| **Bildefrekvens**| Auto / Manuell + verdi | Auto |**Auto**: kameraets begrensning på bildefrekvensen er deaktivert — eksponeringen bestemmer fps, og boksen viser den faktiske frekvensen i sanntid.**Manuell**: du setter en øvre grense for fps med en glidebryter (1 opp til det båndbreddebegrensede maksimumet), med utgangspunkt i den aktuelle faktiske frekvensen. Array-medlemmer ser en skrivebeskyttet «N fps (live)» med «Angitt i array-innstillinger». |

### Nettverk / Transport

| Rad | Oppførsel |
| --- | --- |
| **Pakke størrelse**| 1500 (Standard) / 9000 (Jumbo) — standard**Jumbo**. |
| **Gjennomstrømning** | Skrivebeskyttet grense for koblingsgjennomstrømning i MB/s. Backenden fordeler dette på nytt mellom alle tilkoblede kameraer ved hver tilkobling/frakobling. |
| **Bufferhåndtering** | Skrivebeskyttet modus for bufferhåndtering. |

### Opptak

Ruten avsluttes med en **«Åpne opptaksinnstillinger…»**-knapp som går til [Opptaksinnstillinger-ruten](capture.md#the-capture-settings-pane) (deaktivert inntil et prosjekt er åpent — «Opprett eller åpne et prosjekt for å lagre opptak»). Hvis kameraet er skjult eller satt på pause, minner et tips deg om å vise det igjen/gjenoppta før du tar opptak.

## Ruten for matriseinnstillinger

Åpnes med **tannhjulet**på en ARRAY-rad. Overskrift: array-navn med en blyant for å endre navn, og**×** for å lukke. Seksjonene nedenfor merket *kun kombinert* vises bare for arrayer som er koblet sammen i kombinert visningsmodus.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->



### Synkronisering

Skrivebeskyttede **Master**-,**Slaves**- og**Synkroniseringslinje**-rader.

### Omgivelseslyssensor

Vises for både kombinerte og separate matriser:

* **Lyssensor**-rullegardin — knytter én DAQ til hele matrisen. Valget trer i kraft umiddelbart, spredes til hvert enkelt kameras egen rullegardinmeny for lyssensor (du kan fortsatt overstyre per kamera), og begynner å videresende spektra til matrisen.
* Live **Status**-linje: Av · «Venter på første spektrum…» · «Aktiv — alle kameraene i oppstillingen er belysningskorrigert» · «Ingen nytt spektrum de siste 3 sekundene — bruker fortsatt den siste avlesningen (ingen tidsavbrudd for utdatert data)…».
* Merk i ruten: «Radiometrisk korreksjon for hele matrisen. Innstillinger per kamera overstyrer dette.»

### Kalibreringsmål

Har sin egen gruppe rett etter «Omgivelseslyssensor», både for kombinerte og separate matriser:

* **Aktiver**-avkryssingsrute — «Oppdag MAPIR

ArUco-mål og valider NDVI

mot panelrefleksjons-LUT»; styrer overlegg og valideringstabeller for det kombinerte rutenettet. På et **separat** rutenett aktiverer den hvert medlems egen [kalibreringsmåltabell](#the-calibration-target-table) (én per kamerarad i listevisningen).

### Opptak — ensartede sensorinnstillinger

Vises for både kombinerte og separate matriser (gruppeoverskriften lyder «Opptak (ensartet på tvers av kameraer)»). Disse innstillingene gjelder ensartet for hvert medlem (endringer per medlem ville ødelegge synkroniseringen). Endringer lagres midlertidig og brukes sammen med **Apply**— bortsett fra veksleren**ROI on / ROI off**, som gjelder umiddelbart.

<!-- SCREENSHOT-NEEDED: array settings Capture section (1.3.0) — Pixel Format, Binning, Resolution preset, the ROI crop row with the "ROI on" toggle, the "Select" button, the "max WxH" hint and Reset button, the W/H/X/Y fields, the full-sensor ROI map below them with the orange crop box and its "W×H @ X,Y" label above the box, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons. -->

| Kontroll | Valg / område | Hva den gjør |
| --- | --- | --- |
| **Pikselformat** | 8-bit / 10-bit / 12-bit / 16-bit | Ensartet bitdybde for alle medlemmer; hvert kamera bruker formatet med den dybden for sin egen sensor. |
| **Binning** | 1x1 / 2x2 / 4x4 | Maskinvarebasert binning — bevarer hele synsfeltet samtidig som signal-støyforholdet (SNR) og bildefrekvensen økes. Endring av dette tilbakestiller ROI-feltene til det nye fulle synsfeltet. |
| **Oppløsning** forhåndsinnstilling | Full / Halv / Kvart | Relativt til binning; fyller ROI-feltene med et sentrert utsnitt. |
| **ROI-beskjæring (px)**|**ROI på / ROI av**-bryter,**Velg**-knapp, B / H / X / Y-felt, kart over hele sensoren | Sensorbeskjæring — se [ROI-redigereren](#the-roi-editor). Bredde/høyde justeres til multipler av 16 (minimum 64); forskyvninger justeres til multipler av 4. Et «maks. BxH»-hint viser øvre grense, og**Tilbakestill** gjenoppretter feltene til fullt synsfelt. Under redigering sporer den oransje beskjæringsboksen alle feltendringer på kartet og på matriseflisen. |
| **Utløserfrekvens**| Veksle mellom Auto / Manuell + fps 0,5–10, trinn 0,5 |**Auto**(standard): backenden beregner utløserfrekvensen ut fra oppløsning og båndbredde — inndatafeltet er deaktivert og viser den beregnede verdien.**Manuell**: låser verdien din når du trykker på «Apply». En låst frekvens som en ny bildestørrelse ikke kan håndtere, holdes på den raskeste frekvensen som størrelsen tillater, og gjenopprettes når bildestørrelsen tillater det igjen. |

Merknad i ruten: «Endringer i format/oppløsning starter alle kameraer på nytt kortvarig. Utløserfrekvensen gjelder i sanntid.» **Bruk / Avbryt**-knappene ligger nederst i ruten.

#### ROI-redigeringsverktøyet

Nytt i 1.3.0. ROI-beskjæringsraden og kartet under den fungerer sammen:

* **ROI på / ROI av**— slår sensorbeskjæringen på eller av**umiddelbart**(kameraene starter på nytt kortvarig; matriseflisen viser «Bruker innstillinger…»).**Av**leser av hele sensoren og**husker beskjæringen din**— feltene viser den fortsatt;**På** bruker den på nytt. Hvis du slår på ROI uten at det er angitt noen beskjæring, brukes en sentrert beskjæring i halv størrelse («Ingen beskjæring ble angitt, så en sentrert beskjæring i halv størrelse ble brukt. Juster den med feltene.»).
* **Velg**— tegner beskjæringen på kartet: dra over den, eller klikk på to motsatte hjørner (Esc avbryter). Rektangelet legges inn i W/H/X/Y-feltene, justert etter trinnene ovenfor; ingenting blir brukt før du trykker på**Bruk**, så du kan finjustere det med feltpilene først. Ett rektangel per trykk på Velg.
* **Kartet**viser beskjæringen på hele sensoren, slik at du kan utvide den igjen: «Hele sensoren, live» mens kameraene leser av hele sensoren, eller «Fullformat-øyeblikksbilde fra {tid}. Kameraene leser kun utsnittet.» mens de er beskåret. Hvis matrisen kun har strømmet beskåret materiale, viser kartet «Ingen fullformatvisning ennå: denne matrisen har kun strømmet beskåret materiale.» med en**Vis fullformat**-knapp, som slår av ROI-området. Utsnittets `W×H @ X,Y`-etikett ligger utenfor boksen (over den, eller under når boksen berører toppen), både på kartet og på live-flisen.
* **Apply**bekrefter feltene: et beskjæringsbilde slår ROI på; fullformatet (etter**Reset**) lar det være av, mens det forrige beskjæringsbildet fortsatt er lagret; et ROI som er av og hvis felter fortsatt viser det lagrede beskjæringsbildet, forblir av når du bruker en annen endring (format, binning, frekvens).
* **På et justert array plasseres hvert medlems beskjæringsvindu i henhold til justeringen**, slik at hvert kamera rammer inn den samme delen av scenen. Et array som er beskåret før det noen gang ble justert, justeres først på hele sensoren (beskjæringen fjernes, justeringen løses, og beskjæringen gjenopprettes med vinduer per kamera).
* Etter en beskjæringsendring tilpasses den automatiske eksponeringen på nytt til den nye innrammingen. Under en beskjæring kan eksponeringen for et medlem fortsatt løpe frem til utløsningsperioden.
* ROI-tilstanden (på/av) og den lagrede beskjæringen lagres sammen med prosjektet.

### Justering (samregistrering)

Vises for **både kombinerte og separate matriser** (versjon 1.2.0 viste det kun for kombinerte matriser); gruppeoverskriften lyder «Justering».

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* **Residual**-felt: «RMS x,xx px» — grønt under 1 px, gult under 3 px, rødt ellers eller hvis et kamera mislyktes; «ingen profil» før den første løsningen.
* Sammendragslinje: «✓ Alle kameraer justert (N)» / «⚠ p/N kameraer justert —  <serial (filter)="">feil» / «Beskjæring aktiv — Kalibrer på nytt for å justere (bruker hele sensoren)» / «Venter på at eksponeringen skal stabilisere seg…» / «Bruker innstillinger…» (kameraene starter på nytt) / «Arrayet har sluttet å levere bilder — koble til arrayet på nytt».
* Tabell per kamera: kamera (de siste 4 i serienummeret + filter), reprojeksjonsfeil i px med antall treff («ref» for masteren), og normalisert krysskorrelasjonsscore for overlappingen mot minimumsgrensen på 0,35.
* **Rekalibrer justering**-knapp (viser «Kalibrer justering» før den første profilen) — kjører samregistrering på nytt med nye bilder.
* **«Auto-eksponer kameraer for justering»** avkryssingsrute (merket som standard) — gjør mørke eller flate kameraer midlertidig lysere (eksponering først, deretter forsterkning) slik at de får tekstur å matche, og gjenoppretter deretter AE.

Chloros justerer hvert array automatisk etter tilkobling, i begge visningsmodusene, så snart medlemmene har stabilisert sin automatiske eksponering (programmet fortsetter å prøve i opptil 5 minutter); den kombinerte forhåndsvisningen justeres også ved åpning. Et separat array har ingen sammensatt flis, så denne delen sjekker justeringsstatusen og kjører «Recalibrate» selv. Kalibrer på nytt hvis fokus eller scenedybde endres. Justering er **utformet for å gjelde kun for den aktuelle økten** — den lagres aldri i en profil, fordi den avhenger av scenens avstand i det øyeblikket. Opptak kan fortsatt eksporteres med pikselregistrering (se [Justerte eksporter](capture.md#per-array-controls)).

### Smart vignett

* **Aktiver korreksjon**-avkryssingsrute — bruker vignettestimatet per kamera på den radiometriske kjeden (live**og** eksporter).
* **Kalibrer fra gjeldende visning**— rett først matrisen mot et ensartet mål (flat skjerm, vegg eller himmel); hvert kamera blir utjevnet individuelt, og statusen rapporterer «n/N kameraer · −x,x %» utjevningsgevinst.**Tøm** fjerner estimatet.
* Status: «Kalibrert», «Ikke kalibrert» eller «Delvis kalibrert — kjør på nytt» når bare estimatene til noen av medlemmene fortsatt gjelder i deres nåværende bildeutsnitt (for eksempel etter at en «Rekalibrer»-handling har flyttet et medlems beskjæringsvindu) — kjør «Kalibrer» på nytt.
* Finjuster per kamera med **Vignette**-glidebryteren for hvert kamera i [Live Preview](#live-preview).

### Live Preview

* **Histogram**-avkryssingsrute + breddeskyver (0,10–0,90 av bildet) — histogramstripen vises på feedet på alle medlemmers ruter samtidig (inkludert ruter med delte kanaler), mens [Histogramoverlegg](#display-overlays-drawn-over-the-live-feed) som er angitt for hele matrisen. Det vises som aktivert når alle medlemmer allerede har det. På en**separat** matrise er dette den eneste raden i gruppen.
* *Kun kombinert:* **Indeks**: aktiver avkrysningsrute + tannhjul — åpner den delte [indeksberegneren](#index-calculator-pane) med bånd tegnet fra**alle** medlemskameraer. En forhåndsvisningslinje for uttrykk under viser det gjeldende uttrykket («Ingen uttrykk angitt — åpne beregneren for å opprette et»), som oppdateres hvert sekund.
* *Kun kombinert:* **Renderoppløsning**-rullegardin (samme forhåndsinnstillinger som per kamera, standard 720p): høyden på live-visningsstrømmen**og** den lagrede størrelsen på den sammensatte eksporten. Merk i ruten: «Forhåndsvisning + lagret sammensatt størrelse. Bilder per kamera eksporteres alltid i full oppløsning.»

### Visningslag *(kun kombinert)** **Aktiver**-avkryssingsrute (standard av – hovedkameraet vises direkte; på = lagdelt sammensatt bilde).
* **Nedtrekksmenyer for**Forgrunn**/**Bakgrunn**: hvert medlemskamera (etter navn) eller**Indeks**. Når Forgrunn er Indeks, viser piksler utenfor LUT Min/Maks bakgrunnslaget.

### Delt visning *(kun kombinert)*

**«Vis medlemskameraer»**— en**Del / Skjul medlemskameraer**-knapp som legger til hvert medlems egen live-strøm som separate rutenettfliser ved siden av sammensetningen. Flisene leser arrayets eksisterende rammebuffer (ingen ekstra kameratilkobling). Kun rutenettvisning; lagres per array sammen med prosjektet.

### Funksjoner

Et skrivebeskyttet panel som oppdateres hvert 5. sekund:

* **Nivåbetegnelse**: «Samtidig opptak» (grønn) · «Samtidig opptak (FTD-forsinket utsending)» (grønn) · «Forsinket opptak (100 ms avvik)» (gul) · «Konfigurasjonen er for stor» (rød).
* **Rammestatus**: «x,xx % ufullstendig» — grønn under 1 %, gul under 5 %, rød ved 5 % eller mer.
* ****Linklinje**: «NIC {mbps} Mbps – vedvarende {MB/s} MB/s».

Dette er arrayets sanntids båndbreddebudsjett. For den underliggende fps- og nettverksmodellen — og hva du bør endre når nivået blir gult eller rødt — se [Flerkamera-arrayer](arrays.md) og [CLI-referansen](../reference/cli-reference.md).

<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->

## Ruten «Index Calculator»

Den tredje siden i sidepanelet, som deles av «Index»-tannhjulet per kamera og «Index»-tannhjulet for det kombinerte arrayet (ett om gangen – overskriften lyder «Index Calculator — <camera name="">» eller «Index Calculator — <array name="">»). Den mottar båndlisten (kameraets naturlige filterbåndnaturlige bånd, eller alle bånd på tvers av arrayets medlemmer), det gjeldende uttrykket og LUT-konfigurasjonen (på/av, nivå – standard 7-stop, min – standard 0,2, maks – standard 1), pluss et live indeks-histogram. **Apply** bekrefter uttrykket; LUT-endringer gjelder direkte i forhåndsvisningen.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Innstillinger per kamera vs. array-administrerte innstillinger

Hurtigreferanse for hva som finnes hvor når et kamera er medlem av et array:

| Array-administrert (skrivebeskyttet i kamerapanelet) | Fortsatt per kamera innenfor et array |
| --- | --- |
| Pikselformat, oppløsning, binning | Automatisk eksponering (eksponering, forsterkning, mål, utjevning, ROI) |
| Utløsermodus/kilde, bildefrekvens | Støyreduksjon, skarphet, vignett |
| | Orientering (speiling/rotasjon), visningsoverlegg (matriseinnstillinger → Live Preview angir histogrammet på alle medlemmer samtidig), punktmåler |
| | Indeks (matriser med separat visning), tilknytning til lyssensor |

Annen tverrgående funksjonalitet:

* **Kombinert vs. separat visning** velges ved tilkobling av matrise: kombinert = én justert sammensatt flis (medlemmer sender kun via delt visning); separat = hvert medlem viser sin egen synkroniserte flis. Et kamera viser aldri både en frittstående strøm og en matriseflis samtidig.
* ****Automatisk tilkobling**: når du åpner et lagret prosjekt, gjenopprettes kameraene og matrisene, og alle lagrede innstillinger brukes på nytt på backend før strømmene gjenopptas.
* **Opptakskontroll**: skjulte eller pausede kameraer ekskluderes fra «Capture All»; en matrise blokkeres fullstendig bare når ALLE medlemmene er skjulte/pausede. Se [Innstillinger og moduser for opptak](capture.md).

## Hvordan innstillingene lagres

Tilstanden til kamerafanen lagres **sammen med prosjektet**, ikke i nettleseren:

* Hver reaktiv endring tar et øyeblikksbilde av kameraene og oppsettene i prosjektets `cameras.json` (debounced 500 ms). Dette omfatter kameranavn og farger, innstillinger for eksponering/forsterkning/AE, pikselformat/oppløsning/binning, utløserfrekvens, forhåndsvisningsinnstillinger (renderoppløsning, støyreduksjon, skarphet, vignettering, fargeprofil, høylys, metning/kontrast), orientering, overlegg, kanaldeling, indekskonfigurasjon, innstillinger for prediktiv AE, AE-ROI, matrisenavn, visningsmodus, innstillinger for matriseopptak (inkludert ROI-beskjæring, om ROI er aktivert, og den lagrede beskjæringen når den er deaktivert), matrisens «Wire Budget» og rutenettblokken (feed-zoom, visningsmodus, rutenettlås, manuell flisrekkefølge, skjulte kameraer, lukkede fliser, aktivt kamera).
* Lyssensorbindinger lagres i prosjektets `sensors.json`.
* Når prosjektet åpnes på nytt, kobles maskinvaren til igjen, og alle innstillingene gjelder på nytt.
* **Ingen prosjekt åpent = kun sesjon**: uten prosjekt beholdes ingenting når Chloros lukkes.
* Kun sesjon uavhengig av prosjekt: pausetilstand, spotmålerprøver, avmerkingsboksen for kalibreringsmål per kamera (åpnes alltid av) og profilen for matrisejustering (beregnes på nytt per sesjon etter design).
* Ett unntak: Valg for eksport av **opptaksinnstillinger** og opptaksmodus beholdes per prosjekt i appens lokale lagring i stedet for `cameras.json` — se [Opptaksinnstillinger og -moduser](capture.md).</array></camera></serial>
