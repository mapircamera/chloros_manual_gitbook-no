# Opptak og .daq-formatet

En `.daq`-fil er Chloross opptaksformat for lyssensorer: en **SQLite-database** med kalibrerte spektralbilder fra én DAQ-sensor. Ta opp en slik fil under en opptakssesjon, så kan reflektansbehandlingsrørledningen senere dividere hvert bilde med den nedadgående strålingsintensiteten som ble målt akkurat i det øyeblikket.

## Hva en .daq-fil inneholder

| Egenskap | Verdi |
| --- | --- |
| Beholder | SQLite-database, én fil per sensor per opptak |
| Filnavn | Inkluderer **sensor-ID**og en**tidsstempel**, f.eks. `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spektrum per ramme | 135 punkter, 340–1010 nm i trinn på 5 nm, pluss CIE XYZ-tristimulus |
| Enheter | Kalibrert spektral irradians, **W/m²/nm** (fabrikkalibreringspakke + kappe-profil anvendt) |
| Stemplede metadata | Sensor-ID (nøkkelen for å hente enhetens fabrikkalibrering) og den gjeldende kappe-profilen — se [Kappe-profiler og kalibrert område](caps-and-range.md) |

Formatet er identisk for DAQ-U, DAQ-M og DAQ-E, så nedstrømsbehandlingen bryr seg ikke om hvilken transport som har registrert det.

Kalibrert registrering krever sensorens fabrikkalibreringspakke. For DAQ-U og DAQ-M henter backenden pakken fra MAPIRs sky ved hjelp av sensor-ID (opptak avslås hvis dette ikke er mulig); DAQ-E-enheter er unntatt fordi de lagrer kalibreringen på selve enheten.

## Opptak fra GUI

Opptak i GUI krever et **åpent prosjekt** (ellers er Opptak-knappene deaktivert):

* **Ta opp alt / Stopp alt** — øverst i sidepanelet for lyssensorer; starter eller stopper et `.daq`-opptak på alle tilkoblede sensorer samtidig.
* **Ta opp / Stopp opptak** — per sensor, i innstillingsmodalen (tannhjul). En rød «REC»-indikator vises i sensorens live-informasjonsrader under opptak.

Filene lagres i `<project>/light_sensor/`, og når en opptak avsluttes — enten via «Stopp», «Stopp alt» eller ved å koble fra en opptakssensor — blir den ferdige `.daq`-filen **automatisk lagt til det åpne prosjektet**. Den vises i prosjektets filliste uten at du trenger å legge den til manuelt, og er allerede klar for reflektansbehandling.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->

## Opptak fra CLI

CLI tar opp via backendens sensorpool (backenden må være i gang – disse kommandoene er enkle HTTP-klienter):

```bash
# Connect the sensor into the backend pool
chloros-cli daq pool-connect --eth-host daq-e-def330.local

# Record for 150 seconds, with a human-friendly device label
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
    -o ./out --device-name "rooftop-A"

# Or run open-ended and stop explicitly
chloros-cli daq pool-record --sensor-id daq-e-def330            # --duration defaults to 0 = run until --stop
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

Hent verdien `--sensor-id` fra `chloros-cli daq pool-list`. To standardverdier det er verdt å kjenne til:

| Alternativ | Standard |
| --- | --- |
| `--duration` | `0` — ta opp til `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` på **backendens** filsystem, ikke på CLIs |

Forskjellen mellom utdatakatalogene er viktig når CLI retter seg mot en backend på en annen maskin: filen havner der backenden kjører.

## Opptak fra Python

`DAQSensorSession` (returneres av `chloros_sdk.connect_daq_sensor()`) viser det samme samlede opptaket: `record_start(output_dir=None, device_name=None)` returnerer filstien, mens `record_stop()` returnerer `{path, rows}`. Se [SDK-referansen](../reference/sdk-reference.md) for den fullstendige økten API. SDKs direkte maskinvareklasser (kun for installasjoner på stasjonære datamaskiner) skriver opptak til `~/Documents/DAQ/` som standard; for utgitte versjoner er den samlede banen ovenfor den støttede ruten.

## Bruk av en .daq-fil ved behandlingen

For å beregne reflektans fra bildemateriale trenger Chloros nedadrettet irradians som samsvarer med hver eksponering:

* **Oppbevar `.daq` sammen med bildematerialet.**Ved behandling beregner prosesskjeden automatisk**nedadrettet innstråling med tilpasset tidsstempel** fra en innspilt `.daq` (hvilken som helst DAQ-modell) — eller fra en DAQ-M-spesifikk `.csv` — som ligger sammen med bildene. GUI-opptak oppfyller dette automatisk, siden de legges til prosjektet så snart de avsluttes.
* **Kalibrering hentes på forespørsel.** Hvis en fabrikkalibreringspakke per kamera eller per DAQ ikke allerede er lagret lokalt, henter Chloros den automatisk fra MAPIRs sky ved første bruk (krever internettilgang én gang; lagres i hurtigbufferen under `~/.chloros/`).
* **Liveopptak skriver sin egen sidecar-fil.** For alle reflektansbilder som er tatt live, lagres den faktisk brukte DAQ-målingen som en `.daq`-sidecar-fil ved siden av bildene, slik at opptaket kan behandles på nytt senere uten den opprinnelige opptaket.

## Hente ut strålingsintensiteten igjen

Når et prosjekt behandles, eksporteres også alle lyssensoropptakene det inneholder, til en
`Light Sensor/`-mappe ved siden av bildeproduktene. Dette krever **ikke** bilder: en
lyssensor som er fløyet alene utgjør en fullstendig opptak, og en mappe som kun inneholder `.daq`-
filer er et gyldig inndata. Kjøringen rapporterer hvor mange lyssensorprodukter den har skrevet.

| Produkt | Hva det er |
| --- | --- |
| `<name>_calibrated.daq` | Et arkiv som kan bearbeides på nytt, med samme oppbygning som et liveopptak, men som nå angir kalibreringspakken som produserte det. Å importere det på nytt fører **ikke** til at det kalibreres en gang til. |
| `<name>_calibrated.csv` | Spektral irradians i W/m²/nm på sensorens eget bølgelengdegitter, én rad per måling, pluss fotometriske kolonner: total effekt, fotopisk og skotopisk lux, PPFD med oppdeling i blått/grønt/rødt, og toppbølgelengde. |

En DAQ-U eller DAQ-M hvis kalibreringspakke ikke kan hentes – du er frakoblet, eller
sensoren har ingen kalibrering i filen – **hoppes over med en begrunnelse**, og skrives aldri ut
som en «kalibrert» fil som inneholder rå måleverdier. Koble til internett og kjør på nytt. En DAQ-E
har sin egen kalibrering, så den trenger dette bare når enheten ikke er tilkoblet og
ingenting er lagret lokalt.

### DAQ-A: råverdier, og hvorfor det er det riktige svaret

**DAQ-A** er eldre enn systemet med kalibreringspakker per seriell port og har ingen pakke å
hente. Dette er ikke en forglemmelse: en DAQ-A kalibreres i felten mot et
refleksjonsmål, og målbasert kalibrering trenger bare sensorens *relative*
respons – som er nøyaktig det råtellingsverdiene utgjør. Chloros kalibrerer med dem i dag.

En DAQ-A-opptak eksporteres altså, men under et annet navn:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, ikke `_calibrated` – et annet filnavn i stedet for en markør inne i filen,
fordi opplysningen må overleve at filen sendes videre via e-post som et rent filnavn. `.csv`-
overskriften sier `raw spectral sensor counts (NOT irradiance)` og advarer om at verdiene er
sammenlignbare **innenfor** filen og ikke på tvers av sensorer. Kolonnene som kun har
betydning for reell irradians — total effekt, lux, PPFD — er tomme i stedet for å
beregnes ut fra tellinger.

Eldre DAQ-A-SD-opptak (skjema v1.01 / v1.02) registrerer bare filens skrivetid, ikke et
tidsstempel per avlesning. Chloros vil ikke sammenstille bilder med disse – å knytte et bilde til et
skrivetidspunkt ville være feil uten at det noen gang ser feil ut – men eksporten leser dem uten problemer, og
CSV angir hvilken klokke den er på.

For den fullstendige beskrivelsen av reflektans – enkeltsensor med kamera og dobbeltsensor for omgivelser/objekt – se [Reflektans-arbeidsflyter](reflectance.md).
