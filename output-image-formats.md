---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Utdataformater for bilder

Chloros eksporterer bearbeidede produkter i fire filformater. Velg formatet i prosjektinnstillingene (GUI), med `--format` (CLI) eller med `export_format` (SDK). CLI og SDK godtar nøyaktig de strengene som er angitt nedenfor.

| Formatstreng | Filtype | Pikseltype | Pikselområde | Merknader |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(standard)* | `.tif` | uint16 digitalt tall | 0 – 65535 | Anbefales for fotogrammetri / GIS. |
| `TIFF (8-bit)` | `.tif` | uint8 digitalt tall | 0 – 255 | For opptak som utelukkende er på 8-bit, som kun tilbyr 8-bit-formater: Chloros oppskalerer ikke bitdybden. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100 % reflektans. Noen programmer kan ikke lese TIFF-filer med flytende desimaler; filene er større. |
| `PNG (8-bit)` | `.png` | uint8 digitalt tall | 0 – 255 | Tapsfri komprimering, egnet for visning på nettet og visualisering. |
| `JPG (8-bit)` | `.jpg` | uint8-tall | 0 – 255 | Komprimering med tap, minste filstørrelse. |

## Hvor utdatafilene lagres

Produktene lagres i prosjektmappen, gruppert etter kamera og deretter etter filformat:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera (model+lens+filter)
    ├── tiff16/                          # follows --format: tiff16, tiff8, png8, jpg8, or tiff32
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one <INDEX>_Index_Images/ folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Kameramappen er `LATT-<sensor>-<lens>-F<filter>` for LATTICE og `<model>_<filter>` (f.eks. `Survey3N_RGN`) for Survey3. **Hvert eksportert produkt beholder kildfilens navn — det er mappen som identifiserer produktet, ikke et filnavnsuffiks.** Se [Hvor utdataene lagres](reference/cli-reference.md) i CLI-referansen for de fullstendige reglene.

## LATTICE-produkter (innsamlings- og eksportnivåer)

Én LATTICE-råramme deles opp i alle forespurte produkter i én enkelt gjennomgang. Hver produkttype har sin egen bryter (avkryssingsruter i GUI, eller CLI-kommandoene `--debayered` / `--preview` / `--radiance` / `--reflectance`, alle er slått på som standard):

| Nivå | Innhold | Datatype |
| --- | --- | --- |
| `raw` | Bayer-data direkte fra sensoren (monokameraer: det ene båndet). Behandlingen starter alltid fra rådata. | Som tatt opp |
| `debayered` | Lineær demosaik — 3-kanals for M3C, 1-kanals gråtoner for M3M: kalibreringspakkens mørkeforskyvning fjernes, deretter utføres en kantbevisst demosaik uten linearitetskorreksjon, flatfelt eller unmix. En `process`-eksport skriver den samme filen som en opptak lagrer på dette nivået, i alle behandlingsmoduser. | Lineær DN |
| `radiance` | Absolutt spektral strålingsintensitet fra hele den radiometriske kjeden, i **W/m²/sr/nm**. Skrives alltid som 32-bits TIFF (`tiff32/Radiance_Images/`), uavhengig av valgt eksportformat. | float32 |
| `reflectance` | Refleksjonsgrad ρ, der **DN 32768 = ρ 1,0 (100 %)** med spillerom opp til ρ 2,0. Pix4D-klar. | uint16 |
| `preview` | Skjermklar rendering: RGB-farge = hvitbalanse + gamma; multispektral = strekking av falske farger. | 8-bit skjerm |

### Orientering

`raw`- og `debayered`-filer beholder alltid sensorens **opprinnelige** orientering; kameraets innstilling for speilvendning/rotasjon brukes kun på de andre produktene (forhåndsvisning, radianse, reflektans, indeks). Array-opptak beholder hvert medlems filer i opprinnelig orientering, med mindre du ber om orientering (`--orient` i CLI, `orient=True` i SDK). Hver fil registrerer orienteringen som tilhører dens piksler i XMP — `ChlorosRotation`, `ChlorosFlipHorizontal`, `ChlorosFlipVertical` — og `ChlorosOrientationApplied` angir om dens egne piksler inneholder denne informasjonen (`false` på råfiler og debayerte filer). Chloros-bildeviseren konverterer RAW- og debayered-filer i henhold til denne oppføringen kun for visning; selve filene blir aldri overskrevet. En fil uten oppføring (skrevet før versjon 1.3.0, eller av et annet verktøy) vises slik den er lagret.

## Klemmasker

Strålings- og refleksjonsproduktene fra et multispektralt M3C-kamera gjennomgår en 3×3-bånds separasjon. Der separasjonen gir en negativ verdi i et bånd, blir denne verdien kuttet til null (live-visninger kutter på nøyaktig samme måte). Nytt i versjon 1.3.0: Et produkt der minst én piksel er klippet, får en tilhørende **klippemaske** som angir nøyaktig hvilke piksler og bånd det gjelder:

* **Hvor:**i en `clamp_masks/`-mappe**ved siden av** produktmappene, aldri inne i en: `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`, og `tiff32/Radiance_Images/x.tif` → `tiff32/clamp_masks/Radiance_Images/x_clampmask.tif`.
* **Hva:** en enkeltbånds `uint8`-TIFF-fil på produktets eget pikselrutenett (inkludert orientering, ROI, binning og matrisejustering). Bit 0 (`mask & 1`) angir hvor båndet i produktets R-kanal ble klippet, bit 1 (`& 2`) G-kanalen, bit 2 (`& 4`) B-kanalen. Andelen avklippede piksler per bånd er også merket på produktet som `Chloros:BandRUnmixClampFraction`, `Chloros:BandGUnmixClampFraction` og `Chloros:BandBUnmixClampFraction`.
* **Slik angis det i produktet:** XMP-taggen `Chloros:UnmixClampMask` inneholder maskens sti i forhold til produktets mappe (`../clamp_masks/<product folder>/<name>_clampmask.tif`, alltid atskilt med `/`), `none` når ingen piksler ble klippet (ingen maske er skrevet), eller `unavailable` når en maske ikke kunne skrives. Et produkt uten tag har aldri kjørt unmix (kameraer med RGB- og monokanaler, rådata og debayerte nivåer).

Siden ingen produktmappe inneholder en maske, kan du overføre en produktmappe som for eksempel `Reflectance_Calibrated_Images/` til Pix4D, Metashape eller annen fotogrammetriprogramvare slik den er, inkludert undermapper — overfør produktmappen, ikke formatmappen over den. Chloros’ egen import hopper over klemmasker. CLI-en teller dem separat: `Image products written: N (plus M clamp masks)`.

## Indeksprodukter

Hver indeks havner i sin egen `<INDEX>_Index_Images/`-mappe (f.eks. `NDVI_Index_Images/`).

* **Heltallsformater**(`TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`) lagrer en**lineær strekking per bilde**, ikke et −1…1-kart: DN 0 er den laveste indeksverdien for det aktuelle bildet, og full skala er den høyeste. Filen inneholder `Chloros:IndexScaling=linear_minmax`, `Chloros:IndexMin` og `Chloros:IndexMax`, altså også `value = IndexMin + DN / full_scale × (IndexMax − IndexMin)`. Siden hvert bilde har sine egne endepunkter, må bildene sammenlignes som verdier, ikke som DN.
* **`TIFF (32-bit, Percent)`** lagrer selve indeksverdiene som float32, uten omskalering — det beste valget for kvantitativt arbeid på tvers av en flyvning.

Nytt i versjon 1.3.0: Et indeksprodukt beskriver **seg selv** i stedet for å gjenta taggene fra kildedataene. GPS- og kamerataggene kopieres fra produktet det ble beregnet fra (reflektansproduktet, hvis det finnes), og deretter:

| Tag | Verdi |
| --- | --- |
| `Chloros:ProcessingLevel` | `index` (slik at filen aldri importeres på nytt som et opptak) |
| `Chloros:IndexApplied` / `Chloros:IndexExpression` | `True` / formelen som ble brukt |
| `Chloros:IndexLevel` | `reflectance` eller `radiance` — hva indeksen ble beregnet ut fra (mangler når ingen av delene er til stede) |
| `Chloros:IlluminationInvariant` | `True` for en indeks beregnet ut fra reflektans, `False` ut fra strålingsintensitet eller sensorverdier |
| `Chloros:ChlorosPipelineVersion` | den Chloros-versjonen som skrev indeksen |

`Chloros:PixelScale` er fjernet fra indeksprodukter — bruk `IndexMin` / `IndexMax` ovenfor. Den fullstendige oppskriften for tilbakeavlesning finnes under «Et heltallsindeksraster er en kontrastutvidelse per bilde» i [CLI-referansen](reference/cli-reference.md).

## Avlesning av reflektansverdier for piksler

Refleksjonsverdien lagres som et heltall, og **DN-verdien som tilsvarer ρ = 1,0 (100 % refleksjon) avhenger av kildekameraet**:

| Kildekamera | ρ = 1,0 er DN | Hvordan finne ut av det |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (margin opp til ρ 2,0) | XMP-taggen `Chloros:PixelScale=32768` er stemplet på filen. |
| Survey3 | `65535` (avskåret ved ρ 1,0) | Ingen `Chloros:PixelScale`-tag – fraværet er selve signalet. (Produkter fra Survey3 har andre `Chloros:*`-behandlingsmerker, for eksempel `Chloros:CalibrationApplied`, i alle behandlingsmoduser fra og med versjon 1.3.0.) |

**Les `Chloros:PixelScale` XMP-taggen og del med den** i stedet for å anta en konstant. Taggen er definert i uint16-domenet, så den forblir `32768` på tvers av utdataformater som skalerer om — normaliser først den lagrede datatypen tilbake til uint16 (×257 fra 8-bit, ×65535 fra float32).

{% hint style="warning" %}
**8-bits opptak.** Eksportdataene fra et 8-bits opptak er 8-bits. Reflektansen skaleres om og merkes med `Chloros:PixelScale=32768`, slik at skalaen opprettholdes; 8-biters DN-produkter og reflektansfiler skrevet før versjon 1.3.0 blir beskåret til 0–255 og har ingen skalamerking. Hvis merkingen mangler på en LATTICE-reflektansfil, må du ikke anta at det foreligger en skala; eksporter den i stedet på nytt i 16-bit eller 32-bit.
{% endhint %}

For de fullstendige reglene (inkludert MicaSense-kompatible merker), se **«Lesing av reflektanspiksler»** i [CLI-referansen](reference/cli-reference.md).
