# Bildegitter

Når du har importert bilder til et prosjekt, vil du se dem ordnet i et gitter i hovedområdet. Det er i dette gitteret du velger **hvilken versjon av hvert bilde du ønsker å se** – knappene over gitteret skifter alle miniatyrbildene samtidig mellom kildefilene og de bearbeidede versjonene.

## Miniatyrstørrelse

Bruk zoomglidebryteren øverst til høyre for å justere størrelsen på bildeminiatyrene. Glidebryteren går fra **64 px til 1200 px**.

* **Ctrl + mushjul** skalerer også miniatyrene.
* **Ctrl + `+`**/**Ctrl + `=`**og**Ctrl + `−`** endrer størrelsen med 4 px per trykk. Tastaturkommandoene stopper ved 64 px i den minste enden og, i den største enden, ved den størrelsen som passer nøyaktig to miniatyrbilder per rad i det aktuelle vinduet.
* Størrelsen du velger, lagres sammen med prosjektet (`UI → Grid thumbnail size` i `project.json`, standard `160`), slik at den gjenopprettes når du åpner prosjektet på nytt.

<figure><img src="../.gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>Miniatyrbildets *oppløsning* er en egen innstilling, atskilt fra miniatyrbildets *størrelse*: se **Visning → Oppløsning for miniatyrbilder** i [Prosjektinnstillinger](../project-settings/project-settings.md) (standard 512 px på den lange siden). Størrelse angir hvor stor ruten tegnes; oppløsning angir hvor mye detalj som hentes inn for å fylle den.

***

## Rutenettverktøylinjen

Knapperaden over rutenettet har opptil tre grupper, fra venstre til høyre:

1. **Per utløser / Per kamera** — grupperingsmodus. Vises kun for prosjekter som inneholder LATTICE-opptak.
2. **Kamerafilterknapper** — én per LATTICE-kamera. Vises kun i modus Per kamera.
3. **Knapper for eksport-/visningsmodus** — hvilket produkt hvert miniatyrbilde viser.

Når vinduet er for smalt til å romme alle, skjules gruppene fra høyre mot venstre i nedtrekksmenyer ved musepek: eksport-/visningsknappene skjules først, deretter kameraknappene. Den skjulte gruppen etterlater én enkelt knapp merket med det aktuelt aktive valget, og når du holder musepekeren over den, glir hele settet ned. **«Per utløser» / «Per kamera» kollapser aldri.

<!-- SCREENSHOT-NEEDED: Image grid toolbar of a LATTICE array project at full width, showing all three button groups inline: Per Trigger / Per Camera, three camera filter buttons labelled "LATT-M3M (serial)", and the export/view buttons including TIFF, RAW (Original), RAW (Radiance), RAW (Reflectance). -->

*****

## Knapper for eksportvisning

Disse knappene veksler mellom bildetyper i rutenettminiatyrene. **En knapp vises så snart produktet den refererer til eksisterer** — noe som for kildefilene betyr umiddelbart ved import, ikke etter behandling. Chloros skanner prosjektets produkter på nytt mens en kjøring pågår, så knappene vises under behandlingen etter hvert som hvert produkt begynner å lagres på disken.

### Grunnknappen

Eksportknappen helt til venstre er merket med **det du faktisk importerte**:

| Det du importerte | Knappetekst |
| --- | --- |
| Survey3 RAW+JPG | `JPG` |
| LATTICE-opptak med forhåndsvisning ved siden av RAW-bildet | `PNG` eller `TIFF`, avhengig av hvilke forhåndsvisninger som vises |
| LATTICE-opptak der basisfilen **er** RAW-bildet | *ingen knapp* — `RAW (Original)` viser allerede den filen |

I et blandet prosjekt følger etiketten den filtypen som flest bilder bruker.

### Produktknapper

| Knapp | Viser | Når den vises |
| --- | --- | --- |
| **Mål** | Bilder med et oppdaget kalibreringsmål | Etter en kjøring som oppdaget mål |
| **Refleksjonsgrad** | Kalibrerte refleksjonsgradsbilder | Kun Survey3-prosjekter — LATTICE-prosjekter bruker i stedet `RAW (Reflectance)`, så rutenettet viser aldri to refleksjonsgradsknapper |
| **Hvitbalansert** | Det hvitbalanserte produktet (RGB-kameraer) | Etter behandling |
| **Vignettkorrigert** | Den ukalibrerte, vignettkorrigerte reserveversjonen | Etter en kjøring der reflektanskalibrering ikke kunne brukes og *Vignettkorreksjon* var slått på |
| **Sensorrespons** | Den ukalibrerte reserveversjonen med sensorrespons | Det samme, men med *Vignettkorreksjon* slått av |
| **`RAW (<INDEX> Index)`** | Én knapp per beregnet indeks | Etter en kjøring med konfigurerte indekser |
| **`<INDEX> LUT`** | Én knapp per fargekartlagt indeks | Etter en kjøring med en konfigurert LUT |
| **`<Index> <Index\|LUT> <NNN>`** | Én knapp per [Indeks/LUT-sandkasse](index-lut-sandbox.md)-eksportkjøring | I det øyeblikket en sandkasseeksport er fullført |

### Knapper på LATTICE-nivå

Prosjekter som inneholder LATTICE-opptak, har disse knappene, merket med nivånavnet i stedet for et produktnavn:

| Knapp | Nivå |
| --- | --- |
| **RAW (Original)** | Kilde-RAW-bildet, slik det ble importert |
| **RAW (Radiance)** | Float32 spektral radianse, W/m²/sr/nm |
| **RAW (Reflectance)** | uint16 reflektans, 32768 = ρ 1,0 |

`RAW (Original)` er tilgjengelig fra importøyeblikket — det trenger ingen bearbeiding. Når en LATTICE-import ikke har noen basis-knapp i det hele tatt (hver opptakets basisfil er dens råramme), flytter rutenettet seg selv til den første tilgjengelige nivåknappen, slik at markeringen på verktøylinjen samsvarer med det du ser.

To-nivåers Chloros-eksportfiler får **ingen egen rutenettknapp**:

* **Debayered** — `RAW (Original)`-visningen gjengir allerede debayered, så en ekstra knapp på et visuelt identisk bilde ville være unødvendig. `RAW (Debayered)`-produktet skrives fortsatt til disken og kan fortsatt velges fra rullegardinmenyen for fullskjermlag.
* **Forhåndsvisning** — på RGB-kameraer er forhåndsvisningen registrert som `White Balanced`-laget, som har en knapp. På multispektrale kameraer er den registrert som `RAW (Preview)` og er tilgjengelig fra rullegardinmenyen for fullskjermlag.

{% hint style="info" %}
Disse nivåknappene vises kun for prosjekter som faktisk inneholder LATTICE-rammer. Survey3-prosjekter registrerer noen av de samme interne lagnavnene, og knappene blir filtrert bort for dem, slik at et Survey3-rutenett beholder sitt velkjente `JPG / Targets / Reflectance`-sett.
{% endhint %}

Når du klikker på et rutenettminiatyrbilde, åpnes [Bildeviser](opening-an-image-full-screen.md) i fullskjerm for **det samme produktet som rutenettet viser** — hvis rutenettet er satt til `Targets`, åpner miniatyrbildet det eksporterte målbildet.

<figure><img src="../.gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: This GIF predates the LATTICE level buttons and the toolbar group separators. Reshoot on a LATTICE project cycling base -> RAW (Original) -> RAW (Radiance) -> RAW (Reflectance) -> an index button, so the new button set and the level names are visible. -->

***

## Gruppering av et LATTICE-prosjekt: Per utløser vs. per kamera

Array-opptak produserer flere bilder av samme øyeblikk fra forskjellige kameramoduler. Grupperingen avgjør hvordan rutenettet stabler dem. Begge modusene viser sammenleggbare overskriftsfelt i full bredde; **hver gruppe starter utvidet**, og Chloros husker de du lukker. Sammenleggingsstatus spores separat per modus, så å lukke en gruppe i «Per kamera» lukker ikke noe i «Per utløser».

### Per kamera (standard)

Én gruppe per kameramodul. Overskriften viser kameramodellen og serienummeret (`LATT-M3M — <serial>`) samt antall bilder. Bildene i en gruppe er ordnet kronologisk etter opptakstidspunkt.

I denne modusen får verktøylinjen også én **kamerafilterknapp per kamera**, merket `MODEL (SERIAL)`. Alle kameraer er valgt som standard; når du klikker på en knapp, fjernes markeringen for det kameraet, og gruppen fjernes fra rutenettet. Dette er den raske måten å gjennomgå ett bånd over en hel flytur på.

### Per utløser

Én gruppe per opptakshendelse – settet med bilder som alle modulene tok ved samme utløser. Overskriften viser opptakstidspunktet, antall kameraer som bidro, og et ikon for hver kameramodell i gruppen. Rutene i en gruppe er sortert etter kameraets serienummer, slik at det samme båndet ligger i samme kolonne for hver utløser.

<!-- SCREENSHOT-NEEDED: Image grid in Per Trigger mode for a 3-camera LATTICE array, showing two consecutive trigger groups with their header bars (chevron, capture timestamp, "3 cameras", and the three model badges) and one group collapsed to show the closed state. -->
Bilder som ikke er fra LATTICE i et blandet prosjekt blir ikke gruppert – de vises som vanlige ruter etter gruppene.

***

## Miniatyrbildene i rutenettet følger GSD-blokkstørrelsen

Hvis du har angitt en **GSD (px)**-blokkstørrelse i sidepanelet på bildefanen, vises rutenettminiatyrene med den samme bakkeoppløsningen – ikke bare i fullskjermvisningen. En blokkstørrelse på 8 betyr at hver viste piksel er gjennomsnittet av en 8 × 8-blokk med kildepiksler, overalt i appen der bildet vises.

Siden en flis i utgangspunktet bare er et par hundre piksler bred, slutter grove blokkstørrelser å utgjøre en synlig forskjell i rutenettet lenge før de gjør det i fullskjermvisningen: en ramme på 4000 px tegnet inn i en flis på 160 px har allerede omtrent 25 kildepiksler per vist piksel. Se [Åpne et bilde i fullskjerm](opening-an-image-full-screen.md#gsd-block-size) for selve kontrollen.

***

## Relaterte sider

* [**Åpne et bilde i fullskjerm**](opening-an-image-full-screen.md) — fullskjermvisningen, markørverdier og histogram
* [**Bildelag**](image-layers.md) — lagmenyen i fullskjermvisningen
* [**Indeks/LUT-sandkasse**](index-lut-sandbox.md) — oppretting og eksport av indeksvisualiseringer
* [**Prosjektinnstillinger**](../project-settings/project-settings.md) — eksportbryterne som avgjør hvilke produkter som finnes i det hele tatt
