---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/supported-cameras
---

# Støttede kameraer

Chloros behandler bilder fra to MAPIR-kamerafamilier på **alle plattformer** (Windows, Linux amd64 og Linux arm64/Jetson):

* **Survey3** — Survey3W (bred) og Survey3N (smal) kameraer. Inndata: `RAW+JPG`.
* **LATTICE**— M3C- og M3M-multispektrale kameramoduler. Inndata: Opptak fra `.tif`/`.tiff`. LATTICE-kameraer kan også**styres i sanntid** fra Chloros — via GUI-fanen «Kameraer» (Windows) eller `chloros-cli lattice` / Python SDK (Windows og Linux) — inkludert synkroniserte flerkameraoppsett. Se [LATTICE-veiledningen](lattice/).

Behandlingsrørledningen godtar også `.dng`-inndatafiler.

## Survey3

<table data-header-hidden><thead><tr><th width="156">Produsent</th><th width="250">Kameramodell</th><th width="138">Filtermodell</th><th width="187">Bildetype</th></tr></thead><tbody><tr><td><strong>Produsent</strong></td><td><strong>Kameramodell</strong></td><td><strong>Filtermodell</strong></td><td><strong>Bildetype</strong></td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>OCN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RE</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NIR</td><td>RAW+JPG, JPG</td></tr></tbody></table>## LATTICE

LATTICE-serien er et modulært multispektralt kamerasystem basert på Sony IMX265 global-shutter-sensor (3,1 MP, 3,45 µm piksler). Hvert kamera lagrer sin identitet som en modellstreng:

```
<sensor>-<lens>-F<filter>        e.g.  M3C-L41-FRGN,  M3M-L87-F550
```

Chloros viser den med prefikset `LATT-` (for eksempel `LATT-M3M-L41-F550`), og modellstrengen styrer alt nedstrøms — sensorprofil, båndoppsett og kalibrering løses automatisk; det er ingenting å konfigurere for hvert enkelt kamera. Objektivnummeret angir **det horisontale synsfeltet i grader**: `L41` = smalt 41°, `L87` = bredt 87°.

Det finnes to sensorkonfigurasjoner:

| Konfigurasjon | Sensor      | Filtertype                           | Bånd per kamera                                                        |
| ------------- | ----------- | ------------------------------------- | ----------------------------------------------------------------------- |
| **M3C**       | Bayer-farge | Trippel båndpass                       | 3 spektralbånd fra én enkelt eksponering                                 |
| **M3M**       | Monokrom  | Enkelt smalbåndsinterferensfilter | 1 kalibrert bånd — kombiner flere M3M-kameraer for vegetasjonsindekser |

### M3C (Bayer) filteralternativer

| Filter | Bånd (navn ved sentrum nm / FWHM nm)       |
| ------ | ---------------------------------------- |
| `FRGB` | Blue 475/30 · Green 550/30 · Red 625/30  |
| `FRGN` | Red 660/21 · Green 550/30 · NIR 850/30   |
| `FOCN` | Orange 615/21 · Cyan 490/38 · NIR 808/14 |
| `FNGB` | Blue 475/30 · Green 550/30 · NIR 850/30  |

### M3M (mono) filterkatalog — 23 varenumre

F-tallet er SKU-betegnelsen; det målte båndet (stemplet inn i hver kalibrert eksport) er filterskanningen per parti:

| SKU    | Sentrum (nm, målt) | FWHM-kanter (nm) | Bredde (nm) |
| ------ | --------------------- | --------------- | ---------- |
| F385   | 379,4                 | 367–392         | 25         |
| F405   | 403,9                 | 390–417         | 27         |
| F450   | 443,7                 | 430–458         | 28         |
| F485   | 489,7                 | 478–502         | 24         |
| F520   | 519,9                 | 504–536         | 32         |
| F550   | 548,4                 | 531–566         | 35         |
| F590   | 589,0                 | 570–608         | 38         |
| F615   | 623,8                 | 614–634         | 20         |
| F632   | 633,4                 | 616–651         | 35         |
| F650   | 651,1                 | 636–666         | 30         |
| F685   | 686,2                 | 675–698         | 23         |
| F715   | — (nominell)           | 706–724         | 18         |
| F725   | 725.2                 | 712–738         | 26         |
| F750   | 746.0                 | 729–763         | 34         |
| F780   | 775.1                 | 754–796         | 42         |
| F808   | 810.3                 | 789–832         | 43         |
| F832   | 826,1                 | 810–843         | 33         |
| F850   | 846,5                 | 828–865         | 37         |
| F880   | — (nominelt)           | 867–893         | 26         |
| F905   | — (nominelt)           | 892–920         | 28         |
| F940   | 940,6                 | 923–958         | 35         |
| F950   | 945,1                 | 929–961         | 32         |
| F988 † | 985,3                 | 968–1003        | 35         |

_«Båndkanter måles som verdier for full bredde ved halv maksimal intensitet fra MAPIRs filterskanninger per parti — de samme verdiene som Chloros stempler inn i hver kalibrert eksport.»_ «— (nominell)» = ingen skanning per parti ennå; for disse SKU-ene er det angitte sentrumet SKU-nummeret, og bredden er produsentens oppgitte verdi.

† «F988-refleksjonen kalibreres ved hjelp av et refleksjonspanel i scenen: båndet ligger utenfor det kalibrerte området til DAQ-lyssensoren, så Chloros bruker din siste panelopptak og beholder den mellom panelobservasjonene.» Se [Kalibreringsmål](calibration-targets.md).

For live kamerastyring, matriser, nettverksoppsett og den radiometriske behandlingskjeden, se [LATTICE-veiledningen](lattice/).
