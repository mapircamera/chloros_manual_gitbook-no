# Monokromkameraer og vegetasjonsindekser

## Ett kamera = ett bånd

Et **M3M**-kamera er den monokrome versjonen av Bayer**M3C**: en monokrom IMX265-sensor bak et enkelt smalbåndsinterferensfilter. Modellstrengen angir båndet — `M3M-<lens>-F<wavelength>`, f.eks. `M3M-L87-F685` (vises i Chloros som `LATT-M3M-L87-F685`). Sensoren leverer et **enkelt gråtonebånd** uten Bayer-mosaikk: det er ingenting å demosaikere, ingen kryssforstyrrelser mellom kanalene å skille ut, og ingen hvitbalanse å stille inn.

Konsekvenser det er verdt å kjenne til før du planlegger et monosystem:

* **Strålingsintensitet og reflektans er fullt definert per bånd.**Dette er radiometriske kart per bånd, så ett M3M-kamera produserer kalibrert radianse i float32 (W/m²/sr/nm) og reflektans i uint16 (`32768` = ρ 1,0) nøyaktig slik et M3C-bånd gjør. Mono-bilder har en**identitets**-sensorresponsmatrise — ingen 3×3-unmixing er nødvendig eller blir brukt.
* **Et enkelt monokamera kan ikke produsere et vegetasjonsindeks.** NDVI, NDRE og lignende indekser krever minst to bånd. For å beregne indekser fra monoutstyr må du kombinere flere M3M-kameraer — se nedenfor.
* M3M-kameraer strømmer **Mono12** (12-bit: 2 byte/piksel over kabelen, eller 1,5 byte/piksel når et array bruker [pakket 12-bit-transport](arrays.md#packed-12-bit-transport)), noe som er viktig for [båndbreddeplanlegging for array](arrays.md#bandwidth-how-many-cameras-fit).

## Hva Chloros hopper over for mono — og hvordan det informerer deg om det

Trinnene i fargepipeline gjelder ganske enkelt ikke for en enkeltbåndssensor. Chloros **hopper over dem med en enlinjemelding** i stedet for å gi feilmelding, og kjører dem likevel normalt for ethvert M3C (Bayer)-kamera i samme økt:

| Trinn | Oppførsel i monokrom (M3M) | Oppførsel i M3C |
| --- | --- | --- |
| Demosaic / debayer | Hoppes over — eksportnivået `debayered` er et 1-kanals gråtonebilde. | 3-kanals demosaic. |
| Hvitbalanse (`lattice white-balance`) | Hoppes over med en enlinjemelding. | Kjører normalt. |
| Fargeprofil (`lattice color-profile`) | Hoppes over med en enlinjemelding. | Kjører normalt. |
| Metning/kontrast (`lattice color`) | Hoppes over med en enlinjemelding. | Kjører normalt. |
| Spektral kryssforstyrrelse – separering | Identitet (ingen 3×3-matrise). | 3×3-matrise per kamera anvendt. |
| Strålingsintensitet / refleksjonsgrad | **Kjører** — per bånd, fullt kalibrert. | Kjører per bånd. |

GUI-en bruker samme gating: for et monokamera skjuler innstillingspanelet per kamera radene som kun gjelder RGB (hvitbalanse, gamma, fargeprofil, metning, kontrast, kanaloppsplitting), og live-histogrammet er låst til en enkelt **MONO**-kurve. Diskriminatoren gjennom hele stakken er tokenet `M3M` i modellstrengen, som vises i GUI/SDK som `is_mono`.

## Indekser krever ≥ 2 bånd: juster → stakk → indeks

Arbeidsflyten for monoindekser består alltid av de samme tre trinnene:

1. **Juster** — rett flere M3M-kameraer mot forskjellige bølgelengder (f.eks. en F650 «Red» og en F850 «NIR»), koble dem sammen som et [multikamera-array](arrays.md), og la Chloros beregne ko-registreringsforvrengningen mellom kameraene.
2. **Stakk** — de justerte bildene blir til ett flerbåndsbilde (hvert kamera bidrar med ett navngitt bånd).
3. **Indeks** — beregne en indeksformel over stakkens bånd, eventuelt ved å gjengi den gjennom en LUT.

I brukergrensesnittet utgjør hele denne kjeden visningsmodusen **Combined Cameras**: den sammensatte live-visningen er allerede justert, og matrisens indeksberegner (nedenfor) definerer formelen den gjengir. Eksporterte opptak kan forvrenges til samme justering med opptaksalternativet**Aligned**.

## Indekskalkulatoren

Indekskalkulatoren utarbeider indeksuttrykket som brukes av livevisningen og indekseksportene per kamera. Det er ett felles vindu som åpnes fra to steder i sidepanelet på fanen «Cameras»:

* **Per kamera**— Live Preview →**Index**-tannhjulet (kun RGN-/OCN-/NGB-Bayer-kameraer; et enkelt monokamera har ingen indekskontroll, siden ett bånd ikke kan danne en indeks).
* **Per-array**— array-innstillinger → Live Preview →**Index**-tannhjul. Dette er mono-banen: båndlisten omfatter**alle medlemskameraer**, så et monopar bidrar med sine to bånd her.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Kontrollene, fra topp til bunn:

* **Båndbrikker** («Bånd — klikk for å legge til i uttrykket») — én knapp per tilgjengelig bånd, merket med fargenavn + bølgelengde i nm (dupliserte fargenavn skilles fra hverandre, f.eks. «Farge 850»). Ved å klikke settes båndtokenet inn ved markøren. Bånd fra kameraer som ikke kan produsere radianse per bånd (RGB/FRGB) blir filtrert bort.
* **Operator- og funksjonsknapper** — `+ - * / ( ) ^ ,` pluss `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Tekstfelt for uttrykk** — fritt innskrevet formel; plassholderen viser den klassiske NDVI-formen `(NIR - Red) / (NIR + Red)`. En skrivebeskyttet, tokenisert forhåndsvisning over den gjengir båndbrikker, tall og flagg som ukjente token.
* **Gyldighetsbanner**— grått «Tomt — ingen indeks vil bli brukt»; grønt «Gyldig uttrykk»; rødt med den spesifikke parsingsfeilen (ukjent bånd, tvetydig bånd fra flere kameraer, manglende parentes, …); eller gult når uttrykket er gyldig, men**konstanten** (f.eks. `X/X`, eller en NDVI-nevner skrevet som `−` i stedet for `+`) — en konstant tilordner hele bildet til én farge.
* En egen gul advarsel vises hvis det anvendte uttrykket er i orden, men **live-rammen er ensartet** (flat eller mettet scene) — histogramkollapsen oppdages automatisk for deg.
* **Bruk LUT**(standard på; av = gråtonestrekning),**Nivå**2/3/5/7-stop (standard 7-stop) og**Min / Maks**-innganger på hver side av gradientlinjen. Min er som standard satt til**0,2**— dette zoomer fargeskalaen inn i det vegetasjonsrelevante området, mens verdier under dette vises som gråtoner; sett Min til −1 for hele indeksområdet (**Tilbakestill**-knappen gjenoppretter −1…+1). Max er som standard satt til 1.
* **Live-histogram** over indeksfordelingen — kvadratrotskalerte søyler, ravfargede p2/p98-persentillinjer, en hvit medianlinje og avlesninger for verdier utenfor området («◀ N% < lo» / «hi < N% ▶") som blir ravfargede over 1 % som et signal om å utvide Min/Max-vinduet.
* **Apply** bruker uttrykket på live-strømmen; LUT-justeringer gjelder live uten å trykke på Apply. Når et prosjekt er åpent, lagres uttrykket og LUT-innstillingene sammen med prosjektet og gjenopprettes når du åpner det på nytt; uten prosjekt varer de bare så lenge sesjonen varer.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->



## CLI-banen

Den samme kjeden med justering → stabling → indeksering, som kan skriptes fra start til slutt:

```bash
chloros-cli lattice align-calibrate --serials SN_RED,SN_NIR --save align.json
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`align-calibrate` bruker en tilkoblet matrise som inneholder kameraene, eller åpner dem selv for den ene kjøringen. `index --live` åpner alltid kameraene selv (en [midlertidig matrise](../reference/cli-reference.md#multi-camera-transient-sessions)) og har ingen `--serials`: den åpner hvert kamera på nettverket. Så frigjør alle kameraene som Chloros holder før det — GUI-en, `cam-connect`, `array-connect` (`chloros-cli lattice array-disconnect --all`): å åpne et holdt kamera en gang til kan stoppe strømmen til den som holder det eller krasje backenden. Et kamera som ikke leverer noen ramme, fører til at kjøringen mislykkes, og ingen stabel eller indeks blir skrevet.

`--channel` tilordner symbolene i en forhåndsinnstilling til båndnavnene i stabelen. To regler kan redde deg fra en mislykket kjøring:

* **Symboler er store- og småbokstavssensitive** og må stemme nøyaktig overens med forhåndsinnstillingens kanalnavn — forhåndsinnstillinger bruker små bokstaver (for NDVI er det `red`,`nir`; se `--list-presets`). `--channel red=Red_660` fungerer; `--channel RED=660` mislykkes med en `channel_map missing entries`-feil.
* Båndsiden må angi navnet på et bånd i den justerte stakken (`lattice align-info --profile align.json` viser en liste over dem). Offline-modus godtar også 0-baserte båndindekser, f.eks. `--channel red=0 --channel nir=1`.

`lattice index` kjører også offline mot en lagret, justert flerbånds-TIFF-fil. Hvert skjema kjører på backend, så det må være i gang og logget inn på et betalt Chloros+-nivå; et relativt `--input` eller `--output` refererer til mappen du kjører kommandoen fra:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Indeksforhåndsinnstillinger

`lattice index --preset` (og [Index/LUT-sandkasse](../image-viewer-gui/index-lut-sandbox.md) på fanen «Image», som bruker samme motor) leveres med disse **22 forhåndsinnstillingene**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Kjør `chloros-cli lattice index --list-presets` for formelen og kanalsymbolene til hver forhåndsinnstilling, og `--list-gradients` for de tilgjengelige fargegradientene. Egendefinerte formler bruker `--formula EXPR` med samme syntaks som indekskalkulatoren. Merk at denne listen over forhåndsinnstillinger er spesifikk for LATTICE-indeksmotoren – rullegardinmenyen for prosessering under «Project Settings» for importerte bilder inneholder en annen liste (se [Multispektrale indeksformler](../project-settings/multispectral-index-formulas.md)).

Det komplette settet med flagg (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, justeringsknapper for `--live` og mer) er dokumentert i [CLI-referansen § Indeks-/vegetasjonsmatematikk](../reference/cli-reference.md#index--vegetation-maths); SDK-ekvivalenter finnes i [SDK-referansen](../reference/sdk-reference.md).

## Innhenting av indeksprodukter fra en mono-matrise

Når en matrise er tilkoblet og et indeksuttrykk er brukt, lagrer `array-capture` (eller GUI-funksjonen **Capture All**) lagrer eksportnivåene per kamera *og* indeksrenderingen — `--index`/`--no-index` slår dette på i CLI og lagrer som standard alle relevante nivåer. Et monokameras bidrag til hver opptaksgruppe er dets ene bånd på rå-/debayered- (gråtoner)/radianse-/refleksjonsnivåer, pluss den delte sammensatte indeksen når matrisen kjører i kombinert modus. Se [Flerkamera-matriser § Opptak](arrays.md#capturing-monitoring-vs-analysis).
