# Åpne et bilde i fullskjerm

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption><p>Et bilde åpnet i fullskjerm, med lagvelgeren øverst til høyre</p></figcaption></figure>

Chloros Image Viewer er grensesnittet i fullskjermmodus for visning, inspeksjon og måling av bildene dine. Det er her du leser **faktiske pikselverdier** — DN per kanal, refleksjonsprosent eller strålingsintensitet i W/m²/sr/nm — i stedet for den forstørrede forhåndsvisningen som vises på skjermen.

## Åpne bildeviseren

### Fra filbrowseren

1. Åpne fanen **Filbrowser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Klikk på et hvilket som helst **miniatyrbilde** i [bildegitteret](image-grid.md)
3. Bildet åpnes i fullskjerm i fanen **Bildeviser**

Bildet åpnes i det produktet som rutenettet viste. Hvis rutenettet er satt til `RAW (Reflectance)`, er det dette laget du havner på.

### Åpne sidepanelet i **Image Viewer**Klikk på**Image Viewer**-ikonet <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> i venstre sidepanel for å skyve ut analysepanelet. Det inneholder, fra topp til bunn:

* bildets navn og kameramodellen
* **Export/Save Image(s)**-knappen (kun når en indeks eller LUT er aktiv)
* avmerkingsboksene **Indeks**og**LUT** samt konfigurasjonspanelet for indeks — se [Indeks/LUT-sandkasse](index-lut-sandbox.md)
* **Markørverdier**-panelet: avlesning per kanal, laghistogram og GSD-kontrollen***

## Navigering og zooming

### Bla gjennom bilder

* **Neste bilde**: →-knappen eller**→**-tasten (høyre pil)
* **Forrige bilde**: ←-knappen eller**←**-tasten (venstre pil)
* **Gå til et bestemt bilde**: gå tilbake til rutenettet og klikk på miniatyrbildet

Zoom og panorering beholdes når du beveger deg mellom bildene, slik at du kan gå gjennom en serie mens du forblir på samme del av bildet.

### Zoome

Zoomen styres av **musehjulet**, i trinn på 15 %, og er forankret til markøren – punktet under markøren forblir under markøren. Området er begrenset av bilde- og vindusstørrelsen: du kan ikke zoome ut lenger enn «tilpass til vindu», og den øvre grensen bestemmes av bildets opprinnelige oppløsning.

Det finnes ingen dedikerte zoomtaster i fullskjermvisningen. (I rutenettet endrer **Ctrl + `+` / `−`** størrelsen på miniatyrbildene – en annen funksjon.)

### Panorering ved zooming

Klikk og hold nede venstre museknapp over bildet og dra. Panoreringen er begrenset, slik at bildet ikke kan dras utenfor skjermen.

### Inspeksjon per piksel ved høy forstørrelse

Når den effektive forstørrelsen overstiger **60×**, tegner Chloros en uthevet boks rundt den enkelte viste pikselen under markøren og viser en flytende verdi ved siden av den.

«Effektiv» forstørrelse tar hensyn til GSD-blokkstørrelsen: med en blokkstørrelse på 8 vises utheving ved 7,5× forstørrelse i stedet for 60×, fordi én vist piksel allerede består av 8 × 8 kildepiksler. Zoome ut igjen til under terskelen, og utheving forsvinner.

### Tastatursnarveier

| Tast                             | Hvor       | Handling                              |
| ------------------------------- | ----------- | ----------------------------------- |
| **→**                           | Fullskjerm | Neste bilde                          |
| **←**                           | Fullskjerm | Forrige bilde                      |
| **Ctrl + R**                    | Fullskjerm | Tilbakestill indeks/LUT-sandkassen         |
| **Ctrl + `+`**/**Ctrl + `=`** | Rutenett        | Større miniatyrbilder (4 px per trykk)  |
| **Ctrl + `−`**                  | Rutenett        | Mindre miniatyrbilder (4 px per trykk) |

***

## Markørverdier

Flytt markøren over bildet, så viser **Markørverdier**-panelet verdien for hver kanal under det.

{% hint style="success" %}
**Dette er filens faktiske tall.** Lerretet på skjermen er en 8-bits strukket forhåndsvisning og kan ikke vise disse verdiene, så Chloros henter verdiene fra den faktiske produktfilen for visning. Det er derfor en 12-bits råramme viser verdier over 255, og hvorfor et float32-radianse-lag viser fysiske enheter.
{% endhint %}

### Hva kolonnene betyr

Panelet tilpasser seg laget du ser på:

| Laget du ser på              | Viste kolonner    | Merknader                                                                                           |
| ---------------------------------- | ---------------- | ----------------------------------------------------------------------------------------------- |
| Refleksjonsgrad                        | **DN**og**%** | Prosent beregnes ut fra filens egen skala — se nedenfor                                      |
| Strålingsintensitet                           | **W/m²/sr/nm**   | Fysiske flytende tall; ingen DN-kolonne, fordi en DN er meningsløs her                           |
| Rådata / Debayered / forhåndsvisning / JPG    | **DN**           | Hele digitale tall                                                                         |
| Eksport av 32-bits refleksjonsprosent | **%** kun       | Den lagrede flytende verdien er ikke en DN, så å avrunde den til et heltall ville gi en meningsløs utskrift som `0` eller `1` |

Hver rad er merket med kanalnavnet for kameraets filter — `Red / Green / NIR` for RGN, `Orange / Cyan / NIR` for OCN, `NIR / Green / Blue` for NGB, `Red / Green / Blue` for RGB, og navnet på det enkelte båndet for RE-, NIR- og mono-M3M-kameraer. Hver etikett har en farget prikk som samsvarer med kanalsirklene som brukes i indeksformelredigereren.

Lagrede **indeks- og LUT**-bilder er et spesielt tilfelle: de inneholder fargekartkomponenter i stedet for spektralbånd, så radene deres er merket `Red / Green / Blue` (eller `Index` for en enkeltkanals indeksfil) i stedet for med kameraets filternavn.

Når en indeks er aktiv i sandkassen, vises en ekstra rad under kanalene som viser **indeksverdien** ved markøren, sammen med indeksens navn og en hvit prikk som samsvarer med markøren på histogrammet.

### Refleksjonsprosent bruker hver fils egen skala

{% hint style="warning" %}
**Ikke anta at 65535 = 100 %.** Chloros lagrer reflektans på forskjellige skalaer avhengig av hvilket kamera som produserte den, og visningsprogrammet finner den riktige for hver fil.
{% endhint %}

| Kilde                  | DN som tilsvarer reflektans 1,0 | Hvordan den identifiseres                                                                                                                               |
| ----------------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **LATTICE**(M3C / M3M) |**32768**                      | XMP-taggen `Chloros:PixelScale=32768` skrives inn i hver LATTICE-refleksjonseksport. Den doble sikkerhetsmarginen gjør at filen kan inneholde ρ over 1,0 uten avskjæring |
| **Survey3**|**65535**                      | Ingen Chloros XMP-skaleringstag — Survey3-kalibrering skriver ρ × dtype-max og klipper ved 1,0                                                               |

Visningsprogrammet, indeks-/LUT-sandkassen og indekseksporten beregner alle skalaen gjennom den samme implementeringen, så en verdi du leser ved markøren er den samme verdien som indeksberegningen brukte.

To konsekvenser det er verdt å vite:

* En **32-biters prosent**TIFF lagrer DN/65535 som et flytende tall, og en**8-biters** PNG/JPG-eksport lagrer DN × 255/65535 — visningsprogrammet konverterer begge tilbake før det skriver ut en prosentverdi.
* Ett tilfelle kan ikke gjenopprettes: en **8-biters TIFF-eksport av et opptak med 8-biters kilde** blir beskåret til 0–255 i stedet for å bli omskalert, og har bevisst ingen skaleringsmerking. For disse filene viser panelet kun DN, uten prosentkolonne. Dette er det ærlige svaret, ikke en feil.

***

## Lagets histogram

Under markørradene vises et sanntids histogram for laget du ser på, i **256 intervaller**. Som standard tegnes det én samlet kurve, vektet `(R + 2G + B) / 4` — det samme målerommet som LATTICE-kameraets histogrammer bruker. Når du slår på**RGB**, erstattes den med kurver per kanal i kanalens farger, additivt blandet slik at overlappinger forblir lesbare. Monolag tegner alltid den ene kurven.

Den horisontale aksen er i lagets egen enhet:

| Lag       | Aksenhet  | Aksens maksimum                                               |
| ----------- | ---------- | ---------------------------------------------------------- |
| Refleksjonsgrad | prosent    | 125 % — produktets headroom tillater ρ over 1,0           |
| Strålingsintensitet    | W/m²/sr/nm | Rammens egen toppverdi, avrundet opp til to signifikante sifre |
| 8-bits data  | DN         | 255                                                        |
| 12-bits data | DN         | 4095                                                       |
| 16-bits data | DN         | 65535                                                      |

Når aksen er i DN og havner på en av disse tre øvre grensene, vet Chloros også bitdybden til det du ser på.

Over histogrammet finnes tre knapper:

| Knapp     | Standard | Effekt                                                                                                                                                                                                                                                                                   |
| ---------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **CURSOR** | På      | Tegner markørlinjer på histogrammet ved de nøyaktige verdiene som vises i radene ovenfor, slik at du kan se hvor pikselet under markøren befinner seg i bildets fordeling. I RGB-modus er det én markør per kanal i sin egen farge; ellers vises én hvit markør ved den kombinerte verdien |
| **INDEX**| På      | Vises kun mens en indeks er aktiv. Bytt histogrammet fra kildebåndene til**indeksverdifordelingen**, der de to klippeterskelverdiene er tegnet som oransje stiplede linjer og markørens indeksverdi som en hvit linje                                                          |
| **RGB**| Av     | Veksler fra den kombinerte kurven til kurver per kanal. På en monosensor viser denne knappen**MONO** og er deaktivert — det er bare én kanal å vise                                                                                                                                  |

Histogrammet beregnes ut fra **blokkene du ser**, ikke kildepikslene bak dem: endrer du GSD-blokkstørrelsen, og fordelingen beregnes på nytt, slik at histogrammet, markøren og det viste bildet alltid stemmer overens.

***

## GSD-blokkstørrelse

Nederst på panelet finner du **GSD (px)**-kontrollen: et tallfelt, en glidebryter fra**1 til 256**og en**RESET**-knapp.

Dette gjør det _viste_ bildet grovere ved å beregne gjennomsnittet av en N × N-blokk med kildepiksler til én vist piksel. `1` er opprinnelig oppløsning.

* Det påvirker **fullskjermvisningen, rutenettminiatyrbildene, markørvisningen og begge histogrammene** — alt som viser bildet, har samme grunnoppløsning.
* Det gjelder **kun visning**. Behandling og eksport påvirkes ikke. Det eneste unntaket er bevisst: en eksport fra [Index/LUT Sandbox](index-lut-sandbox.md) lagrer det du ser på, så den overfører den gjeldende blokkstørrelsen, og eksportpanelet advarer deg når blokkstørrelsen er over 1.
* Verdien lagres **per prosjekt** som `viewer_display.gsd_bin` i `project.json`, slik at den beholdes når programmet lukkes og åpnes på nytt.
* Markørvisningen viser blokken, ikke kildepikselen, når blokkstørrelsen er over 1 — verdien som vises er gjennomsnittet for blokken under markøren.

{% hint style="info" %}
**Hvorfor «blokkstørrelse» og ikke centimeter per piksel?** Et cm/px-tall krever en høyde over bakken. EXIF-dataene til et enkelt bilde inneholder GPS-høyde over gjennomsnittlig havnivå, ikke over terrenget det ble rettet mot, så Chloros vil ikke skrive ut en avstand til bakken som den ikke kan beregne på en pålitelig måte. Blokkstørrelsen i kildepiksler er den samme reserveverdien som MAPIR-skyverktøyene bruker når avstanden til bakken er ukjent.
{% endhint %}

***

## Bildetyper du kan vise

Lag-rullegardinmenyen øverst til høyre i visningsprogrammet viser alle versjoner av det aktuelle bildet. Hvilke oppføringer som vises, avhenger av kameraet og hva som er blitt behandlet – se [Bildelag](image-layers.md) for fullstendig liste og hvordan rullegardinmenyen fungerer.

### Survey3

* **JPG** — kameraets egen forhåndsvisningsfil
* **RAW (Original)** — kildedataene `.RAW`, debayert for visning, uten korreksjoner
* **RAW (Mål)** — et bilde som er identifisert som inneholdende et kalibreringsmål
* **RAW (reflektans)** — det kalibrerte reflektansproduktet (65535 = ρ 1,0)
* **Vignettkorrigert**/**Sensorrespons** — det ukalibrerte reserveproduktet
* **Hvitbalansert** — det hvitbalanserte produktet
* **RAW (`<INDEX>`-indeks)**og**`<INDEX>` LUT** — beregnede indeksbilder

### LATTICE

LATTICE-opptak bruker samme rullegardinmeny, med navnene på nivåene i behandlingskjeden:

| Lag                 | Hva det inneholder                                                        |
| --------------------- | -------------------------------------------------------------------- |
| **RAW (Original)**    | Kilde-RAW-bildet slik det ble tatt                                     |
| **RAW (Debayered)**   | Det lineære debayered-bildet                                           |
| **RAW (Preview)**     | Visningsforhåndsvisningen — falskfargestrekning for multispektrale kameraer |
| **White Balanced**    | Forhåndsvisningen for RGB-hovedkameraer (hvitbalanse + gamma)   |
| **RAW (Stråling)**    | Float32 spektral stråling i W/m²/sr/nm                              |
| **RAW (reflektans)** | uint16-reflektans, 32768 = ρ 1,0                                    |

Strålingsintensitet og reflektans er kun tilgjengelig for multispektrale bilder: et RGB-hovedkamera har ikkebåndradiometri, så disse lagene genereres ikke for det.

***

## Bruk av indeks og LUT

Bruk multispektrale indekser og farge-LUT-er fra sidepanelet:

1. Åpne **Image Viewer**-<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">-sidepanelet
2. Merk av for **Index**

3. Velg kameraets filter og en indeksformel, og dra deretter kanalsirklene inn i formelens felt
4. Legg til en LUT og velg en gradient, terskelverdier og en klippemodus
5. Les av verdiene ved markøren, og lagre resultatet med **Eksporter/Lagre bilde(r)**Se [Index/LUT Sandbox](index-lut-sandbox.md) for en fullstendig gjennomgang.

***

## Feilsøking

### Bildet åpnes ikke

**Mulige årsaker**: filen ble flyttet eller slettet etter import; produktet ble aldri skrevet; ikke nok minne for et svært stort bilde.**Hva du bør gjøre**:

1. Sjekk at lagets fil fremdeles finnes i prosjektets utdatatre
2. Åpne filen i en ekstern visningsprogram for å bekrefte at den er intakt
3. Lukk andre programmer for å frigjøre minne

### Bildet er svart, hvitt eller har helt urimelige farger

**Mulige årsaker**: skjermutvidelsen har ingenting å jobbe med (en nesten konstant ramme); et float32-lag med uvanlige verdier; en indeks som ikke produserte gyldige data.**Hva du bør gjøre**:

1. Les av markørverdiene – hvis alle kanalene er på eller nær null, ligger problemet i dataene, ikke i visningen
2. Sjekk histogrammet: en enkelt topp i den ene enden indikerer at rammen er avkuttet eller tom
3. Sjekk behandlingsloggen for kjøringen som produserte laget

### Verdiene ser feil ut

**Mulige årsaker**: du er på et annet lag enn du tror; du sammenligner en prosentandel med en rå DN; du sammenligner en LATTICE-fil med en Survey3-fil ved å bruke samme divisor.**Hva du bør gjøre**:

1. Bekreft det valgte laget i rullegardinmenyen – enhetene i panelet følger laget
2. For reflektans, bruk **%**-kolonnen i stedet for å dividere DN-verdien selv; hvis du må dividere, bruk den filens `Chloros:PixelScale` (32768 for LATTICE, hvis den mangler betyr det 65535 for Survey3)
3. Sett GSD-blokkstørrelsen tilbake til 1 – over 1 leser du et blokkgjennomsnitt, ikke en piksel
4. Kontroller at reflektanskaliseringen faktisk ble kjørt for det bildet; et ukalibrert reserveprodukt (Sensor Response / Vignette Corrected) er ikke reflektans

***

## Neste trinn

* [**Bildelag**](image-layers.md) — hvert lagnavn, når det finnes, og hva verdiene betyr
* [**Indeks/LUT-sandkasse**](index-lut-sandbox.md) — bygg, finjuster og eksporter indeksvisualiseringer
* [**Kartmarkører**](map-markers.md) — det samme bildesettet på et kart
* [**Multispektrale indeksformler**](../project-settings/multispectral-index-formulas.md) — indeksreferansen

For arbeidsflyten for behandling, se [Behandling av bilder (GUI)](../processing-images-gui/adding-files-to-a-project.md).
