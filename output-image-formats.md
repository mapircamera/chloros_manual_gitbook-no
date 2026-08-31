---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Utdataformater

Chloros eksporterer bearbeidede produkter i fire filformater. Velg formatet i prosjektinnstillingene (GUI), med `--format` (CLI) eller med `export_format` (SDK). CLI og SDK godtar nøyaktig de strengene som er angitt nedenfor.

| Formatstreng | Utvidelse | Pikseltype | Pikselområde | Merknader |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(standard)* | `.tif` | uint16 digitalt tall | 0 – 65535 | Anbefales for fotogrammetri / GIS. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100 % refleksjonsgrad. Noen applikasjoner kan ikke lese TIFF-filer med flytende desimaler; filene blir større. |
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

Kameramappen er `LATT-<sensor>-<lens>-F<filter>` for LATTICE og `<model>_<filter>` (f.eks. `Survey3N_RGN`) for Survey3. **Hvert eksportert produkt beholder kildfilens navn – mappen identifiserer produktet, ikke et filnavnsuffiks.** Se [Hvor utdataene havner](reference/cli-reference.md) i CLI-referansen for de fullstendige reglene.

## LATTICE-produkter (opptaks- og eksportnivåer)

Én LATTICE-råramme fordeles til hvert forespurt produkt i én omgang. Hver produkttype har sin egen bryter (GUI-avmerkingsbokser eller CLI `--debayered` / `--preview` / `--radiance` / `--reflectance`, alle er slått på som standard):

| Nivå | Innhold | Datatype |
| --- | --- | --- |
| `raw` | Bayer-data direkte fra sensoren (monokameraer: det enkelte båndet). Behandlingen starter alltid fra rådata. | Som tatt opp |
| `debayered` | Lineær demosaik — 3-kanals for M3C, 1-kanals gråskala for M3M. | Lineær DN |
| `radiance` | Absolutt spektral stråling fra hele den radiometriske kjeden, i **W/m²/sr/nm**. Skrives alltid som 32-biters TIFF (`tiff32/Radiance_Images/`), uavhengig av valgt eksportformat. | float32 |
| `reflectance` | Refleksjonsgrad ρ, der **DN 32768 = ρ 1,0 (100 %)** med spillerom opp til ρ 2,0. Pix4D-klar. | uint16 |
| `preview` | Visningsklar gjengivelse: RGB = hvitbalanse + gamma; multispektral = falskfargestrekning. | 8-bits visning |

## Lesing av refleksjonspikselverdier

Refleksjonsgraden lagres som et heltall, og **DN-verdien som tilsvarer ρ = 1,0 (100 % refleksjonsgrad) avhenger av kildekameraet**:

| Kildekamera | ρ = 1,0 er DN | Hvordan vite det |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (margin opp til ρ 2,0) | XMP-taggen `Chloros:PixelScale=32768` er stemplet på filen. |
| Survey3 | `65535` (klippet ved ρ 1,0) | Ingen `Chloros:*` XMP-tagger — fraværet er selve signalet. |

**Les XMP-taggen `Chloros:PixelScale` og del med den** i stedet for å anta en konstant. Taggen er definert i uint16-domene, så den forblir `32768` på tvers av utdataformater som skalerer om — normaliser først den lagrede datatypen tilbake til uint16 (×257 fra 8-bit, ×65535 fra float32).

{% hint style="warning" %}
**Ett tilfelle har ingen skalering, etter design.** Når en 8-bits kildeopptak (BayerRG8) skrives som 8-bits TIFF, klipper rørledningen ned til 0–255 i stedet for å skalere om, så ingen skala beskriver filen — Chloros utelater bevisst `Chloros:PixelScale` der. Hvis taggen mangler i en LATTICE-refleksjonsfil, må du ikke anta at det finnes en skalafaktor; eksporter i stedet på nytt i 16-bit eller 32-bit.
{% endhint %}

For de fullstendige reglene (inkludert MicaSense-kompatible tagger), se **«Lesing av reflektanspiksler»** i [CLI-referansen](reference/cli-reference.md).
