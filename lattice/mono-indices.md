# Monokameraer og vegetasjonsindekser

## Ett kamera = ett bånd

Et **M3M**-kamera er den monokrome versjonen av Bayer**M3C**: en monokrom IMX265-sensor bak et enkelt smalbåndsinterferensfilter. Modellstrengen angir båndet — `M3M-<lens>-F<wavelength>`, f.eks. `M3M-L87-F685` (vises i Chloros som `LATT-M3M-L87-F685`). Sensoren leverer et**enkelt gråtonebånd** uten Bayer-mosaikk: det er ingenting å demosaikere, ingen kryssforstyrrelser mellom kanalene å skille ut, og ingen hvitbalanse å stille inn.

Konsekvenser det er verdt å kjenne til før du planlegger et monosystem:

* **Strålingsintensitet og refleksjonsgrad er fullstendig definert per bånd.**Dette er radiometriske kart per bånd, så ett M3M-kamera produserer kalibrert float32-stråling (W/m²/sr/nm) og uint16-refleksjonsgrad (`32768` = ρ 1,0) nøyaktig slik et M3C-bånd gjør. Monobilder har en**identisk** sensorresponsmatrise — ingen 3×3-unmixing er nødvendig eller blir brukt.
* **Et enkelt monokamera kan ikke produsere et vegetasjonsindeks.** NDVI, NDRE og lignende trenger minst to bånd. For å beregne indekser fra monohardware kombinerer du flere M3M-kameraer – se nedenfor.
* M3M-kameraer strømmer **Mono12** (12-bit, 2 byte/piksel over kabelen), noe som er viktig for [budsjettering av matrisebåndbredde](arrays.md#bandwidth-the-rules-of-thumb).

## Hva Chloros hopper over for mono — og hvordan det informerer deg om det

Trinnene i fargepipeline gjelder ganske enkelt ikke for en enkeltbåndssensor. Chloros **hopper over dem med en enlinjemelding** i stedet for å gi feilmelding, og kjører dem likevel normalt for ethvert M3C (Bayer)-kamera i samme økt:

| Trinn | Oppførsel for mono (M3M) | Oppførsel for M3C |
| --- | --- | --- |
| Demosaic / debayer | Hoppet over — eksportnivået for `debayered` er et 1-kanals gråtonebilde. | 3-kanals demosaic. |
| Hvitbalanse (`lattice white-balance`) | Hoppes over med en enlinjemelding. | Kjører normalt. |
| Fargeprofil (`lattice color-profile`) | Hoppes over med en enlinjemelding. | Kjører normalt. |
| Metning/kontrast (`lattice color`) | Hoppes over med en enlinjemelding. | Kjører normalt. |
| Spektral kryssforstyrrelse – separering | Identitet (ingen 3×3-matrise). | 3×3-matrise per kamera anvendt. |
| Strålingsintensitet / refleksjonsgrad | **Kjører** — per bånd, fullt kalibrert. | Kjører per bånd. |

GUI-en bruker samme gating: for et monokamera skjuler innstillingspanelet per kamera radene som kun gjelder RGB (hvitbalanse, gamma, fargeprofil, metning, Kontrast, kanaloppsplitting), og live-histogrammet er låst til én enkelt **MONO**-kurve. Diskriminatoren gjennom hele stakken er `M3M`-tokenet i modellstrengen, som vises i GUI/SDK som `is_mono`.

## Indekser krever ≥ 2 bånd: juster → stabl → indeks

Arbeidsflyten for monoindeksering består alltid av de samme tre trinnene:

1. **Juster** — rett flere M3M-kameraer mot forskjellige bølgelengder (f.eks. en F650 «Red» og en F850 «NIR»), koble dem sammen som et [flerkamera-array](arrays.md), og la Chloros beregne ko-registreringsforvrengningen mellom kameraene.
2. **Stabel** — de justerte bildene blir til ett multibåndsbilde (hvert kamera bidrar med ett navngitt bånd).
3. **Indeks** — beregne en indeksformel over stabelenes bånd, eventuelt ved å gjengi den gjennom en LUT.

I brukergrensesnittet utgjør hele denne kjeden visningsmodusen **Kombinerte kameraer**: det sammensatte livebildet er allerede justert, og matrisens indeksberegner (nedenfor) definerer formelen det gjengir. Eksporterte opptak kan forvrenges til samme justering med opptaksalternativet**Justert**.

## Indekskalkulatoren

Indekskalkulatoren utarbeider indeksuttrykket som brukes av livevisningen og indekseksportene per kamera. Det er ett felles vindu som åpnes fra to steder i sidepanelet på «Cameras»-fanen:

* **Per kamera**— Live Preview →**Index**-tannhjulet (kun RGN/OCN/NGB Bayer-kameraer; et enkelt monokamera har ingen indekskontroll, siden ett bånd ikke kan danne en indeks).
* **Per-array**— array-innstillinger → Live Preview →**Index**-tannhjulet. Dette er monoveien: båndlisten omfatter**alle medlemskameraene**, så et monopar bidrar med sine to bånd her.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Kontrollene, fra topp til bunn:

* **Båndbrikker** («Bånd — klikk for å legge til i uttrykket») — én knapp per tilgjengelig bånd, merket med fargenavn + bølgelengde i nm (dupliserte fargenavn skilles fra hverandre, f.eks. «Farge 850»). Ved å klikke settes båndtokenet inn ved markøren. Bånd fra kameraer som ikke kan produsere stråling per bånd (RGB/FRGB) filtreres bort.
* **Operator- og funksjonsknapper** — `+ - * / ( ) ^ ,` samt `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Tekstfelt for uttrykk** — fritt innskrevet formel; plassholderen viser den klassiske NDVI-formen `(NIR - Red) / (NIR + Red)`. En skrivebeskyttet, tokenisert forhåndsvisning over den gjengir båndbrikker, tall og flagg som ukjente token.
* **Gyldighetsbanner**— grå «Tom — ingen indeks vil bli brukt»; grønt «Gyldig uttrykk»; rødt med den spesifikke analysefeilen (ukjent bånd, tvetydig bånd fanget opp av flere kameraer, manglende parentes, …); eller gult når uttrykket er gyldig, men**konstant** (f.eks. `X/X`, eller en NDVI-nevner skrevet med `−` i stedet for `+`) — en konstant tilordner hele bildet til én farge.
* En egen gul advarsel vises hvis det anvendte uttrykket er i orden, men **live-bildet er ensartet** (flat eller mettet scene) — histogramkollapsen oppdages automatisk for deg.
* **Bruk LUT**(standard på; av = gråtonestrekning),**Nivå**2/3/5/7-stop (standard 7-stop) og**Min / Maks**-innganger på hver side av gradientlinjen. Min er som standard satt til**0,2**— dette zoomer fargeskalaen inn i det vegetasjonsrelevante området, mens verdier under dette vises som gråtoner; sett Min til −1 for hele indeksområdet (**Tilbakestill**-knappen gjenoppretter −1…+1). Max er som standard satt til 1.
* **Live-histogram** over indeksfordelingen — kvadratrotskalerte søyler, gule p2/p98-persentillinjer, en hvit medianlinje og avlesninger for verdier utenfor området («◀ N% &lt; lo» / «hi &lt; N% ▶&quot;) som blir ravfargede over 1 % som et signal om å utvide Min/Max-vinduet.
* **Apply**overfører uttrykket til live-strømmen; LUT-justeringer gjelder live uten å trykke på Apply. Uttrykkene er bevisst**kun for den aktuelle økten** — de lagres ikke mellom økter.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->

## CLI-banen

Den samme kjeden av justering → stabel → indeks, som kan skriptes fra start til slutt:

```bash
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` tilordner symbolene i en forhåndsinnstilling til båndnavnene i stabelen. To regler sparer deg for en mislykket kjøring:

* **Symboler er store og små bokstaver** og må stemme nøyaktig overens med forhåndsinnstillingens kanalnavn — forhåndsinnstillinger bruker små bokstaver (NDVI er `red`,`nir`; sjekk `--list-presets`). `--channel red=Red_660` fungerer; `--channel RED=660` mislykkes med en `channel_map missing entries`-feil.
* Båndsiden må angi navnet på et bånd i den justerte stakken (`lattice align-info --profile align.json` viser en liste over dem). Offline-modus godtar også 0-baserte båndindekser, f.eks. `--channel red=0 --channel nir=1`.

`lattice index` kjører også helt offline mot en lagret, justert flerbånds-TIFF:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Indeksforhåndsinnstillinger

`lattice index --preset` (og [Index/LUT sandbox](../image-viewer-gui/index-lut-sandbox.md) på fanen «Image», som bruker samme motor) leveres med disse **22 forhåndsinnstillingene**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Kjør `chloros-cli lattice index --list-presets` for formelen og kanalsymbolene til hver forhåndsinnstilling, og `--list-gradients` for de tilgjengelige fargegradientene. Egendefinerte formler bruker `--formula EXPR` med samme syntaks som indekskalkulatoren. Merk at denne listen over forhåndsinnstillinger er spesifikk for LATTICE-indeksmotoren — rullegardinmenyen for prosessering under «Prosjektinnstillinger» for importerte bilder inneholder en annen liste (se [Multispektrale indeksformler](../project-settings/multispectral-index-formulas.md)).

Det komplette flaggsettet (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, justeringsknapper for `--live`, og mer) er dokumentert i [CLI-referansen § Indeks / Vegetasjonsmatematikk](../reference/cli-reference.md#index--vegetation-maths); SDK-ekvivalenter finnes i [SDK-referansen](../reference/sdk-reference.md).

## Hente indeksprodukter fra en mono-matrise

Når en matrise er tilkoblet og et indeksuttrykk er brukt, lagrer `array-capture` (eller GUI-funksjonen **Capture All**) eksportnivåene per kamera *og* indeksrenderingen — `--index`/`--no-index` slår dette på og av på CLI, og standardinnstillingen er å inkludere alle relevante nivåer. Et monokameras bidrag til hver opptaksgruppe er dets ene bånd på rå-/debayered- (gråtoner)-/radiance-/reflektansnivåer, pluss den delte sammensatte indeksen når oppstillingen kjører i kombinert modus. Se [Flerkamera-matriser § Opptak](arrays.md#capturing-monitoring-vs-analysis).
