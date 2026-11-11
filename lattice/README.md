# LATTICE-kameraer

LATTICE er MAPIRs modulære multispektrale kamerasystem for bildetakning innen landbruk og vitenskap. Alle LATTICE-kameraer er basert på Sony IMX265 global-shutter-sensor (**3,1 MP, 3,45 µm piksler**) og kobles til via Ethernet som en**GigE Vision**-enhet.

Chloros 1.3.0 styrer LATTICE-kameraene i sanntid – oppdagelse, forhåndsvisning, opptak og synkroniserte flerkamerasystemer – fra tre grensesnitt:

| Grensesnitt    | Hvor                                                          | Plattformer                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| GUI        | Fanen **Kameraer** i Chloros-sidepanelet                         | Windows 10/11 x64                                        |
| CLI        | `chloros-cli lattice`-kommandofamilien                           | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |

> **Leter du etter maskinvaren?**Kameramoduler, objektiver, filtre og bånd, rammer og monteringsutstyr, kabler, PoE og utløserkabler er dokumentert i [**LATTICE-brukerhåndboken**](https://mapir.gitbook.io/lattice-camera). Dette kapitlet omhandler styring av kameraene fra Chloros.

LATTICE-opptak er standard `.tif`/`.tiff`-filer, og Chloros behandler dem alltid med utgangspunkt i råopptaket. Se [CLI-referansen](../reference/cli-reference.md) og [SDK-referansen](../reference/sdk-reference.md) for den komplette kommando- og API-grensesnittet.

## To sensorkonfigurasjoner

| Konfigurasjon | Sensor       | Filter                                | Hva ett kamera leverer                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Bayer-farge | trippel båndpassfilter                |**Tre kalibrerte bånd fra én enkelt eksponering**                 |
| **M3M**| Monokrom   | enkelt smalbåndsinterferensfilter |**Ett kalibrert bånd**; kombiner flere M3M-kameraer for indekser |

Siden et M3M-kamera er monokromt bak et enkelt filter, får hvert bånd sin egen eksponering. Et M3C-kamera dekker alle tre båndene med én sensoreksponering.

## Modellstrenger og navngiving

Hvert kamera lagrer sin identitet i GenICam `DeviceUserID` som en modellstreng:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros viser den med prefikset `LATT-` (for eksempel `LATT-M3M-L87-F450`). Den samme strengen `LATT-…` skrives inn i EXIF-taggen `Model` ved hver eksport og brukes som navn på kameraets utdatamappe i bearbeidede prosjekter.

| Komponent | Verdier                                                   | Betydning                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Sensor    | `M3C` / `M3M`                                            | Bayer-farge / monokrom                                                                          |
| Objektiv      | `L41` / `L87`                                            | Tallet angir **det horisontale synsfeltet i grader**: L41 = smalt (41°), L87 = bredt (87°)    |
| Filter    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) eller `F<nm>` (M3M) | Se [Filtre og spektralbånd](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

Modellstrengen styrer alt videre nedstrøms: Chloroshenter sensorprofilen, båndoppsettet og fabrikkalibreringen fra `DeviceUserID` + `DeviceSerialNumber`. Det er ingenting å konfigurere per kamera — se [Tilkobling av kameraer](connecting.md).

## Filtre og bånd

Båndsentre, FWHM-kanter og hele M3M-katalogen med 23 SKU-er er produktspesifikasjoner, så de finnes i maskinvarehåndboken: [**Filtre og spektralbånd**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Det som er viktig på programvaresiden: filterkoden i modellstrengen bestemmer hvilke produkter Chloros kan bygge. RGB-filterkameraer (`FRGB`) leverer kun debayered- og forhåndsvisningsprodukter — strålingsintensitet og reflektans per bånd er ikke meningsfullt for en bredbåndssensor, så Chloros hopper over dem og opplyser om dette. Alle andre filtre gir den fullstendige kjeden strålingsintensitet → reflektans → indeks.

## Radiometrisk kalibrering i korthet

Hvert LATTICE-kamera kalibreres individuelt på fabrikken mot en NIST-sporbart kjede og leveres med et sertifikat for hvert enkelt kamera. Hva dette omfatter, hvordan det måles og hvilken nøyaktighet du kan oppgi, finnes i maskinvarehåndboken: [**Radiometrisk kalibrering fra fabrikken**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

På programvaresiden er det viktig at Chloros finner riktig kalibrering når et kamera kobles til, og låser de anvendte koeffisientene i hver eksport — se [Koble til kameraer](connecting.md).

## I dette kapittelet

* [Koble til kameraer](connecting.md) — automatisk gjenkjenning, tilkoblingsdialogen i brukergrensesnittet, tilsvarende CLI- og SDK-kommandoer, og hvordan fabrikkalibrering fastsettes (pakke på kameraet vs. skyen) når et kamera kobles til.
* [Kamerainnstillinger](camera-settings.md) — alle kontrollene i fanen «Kameraer»: kameralisten, innstillingsruten for hvert enkelt kamera og innstillingsruten for matrisen (inkludert ROI-redigeringsverktøyet og justering).
* [Opptakinnstillinger og -moduser](capture.md) — «Capture All», enkelt-/kontinuerlig-/intervallopptak, eksporttyper og mappenes struktur for opptak.
* [Flerkameraoppsett](arrays.md) — veiviseren for tilkobling av oppsett, synkroniseringskabel, båndbredde, PTP og livevisningen av oppsettet.
* [Monokameraer og vegetasjonsindekser](mono-indices.md) — M3M-kameraer og arbeidsflyten juster → stabl → indeks.

Den komplette kommandolisten finnes i [CLI-referansen](../reference/cli-reference.md) og [SDK-referansen](../reference/sdk-reference.md).
