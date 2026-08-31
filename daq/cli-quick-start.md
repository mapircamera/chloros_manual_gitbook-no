# CLI Hurtigstart (pool-*)

Den medfølgende `chloros-cli`-enheten styrer DAQ-sensorer via **`daq pool-*`** kommandofamilien — enkle HTTP-klienter som styrer sensoren via Chloros-backendens permanente sensorpool. Backenden eier transporten, så GUI-en, CLI- og SDK-skriptene deler alle ett live-håndtak i stedet for å kjempe om porten. Alt en kunde trenger, er tilgjengelig via `pool-*`: koble til, streame, ta opp kalibrerte `.daq`-filer og bytte cap-profiler.

`pool-*` er også den **eneste** DAQ-overflaten i utgitte versjoner. `chloros-cli daq --help` viser en liste over `pool-*`-underkommandoene, og hvis du utfører en direkte maskinvare-DAQ-underkommando på en utgitt versjon, avsluttes programmet med en eksplisitt feilmelding som angir den manglende pakken og henviser deg tilbake til `pool-*` — ingenting mislykkes i stillhet. (Kommandoene for direkte maskinvare kjører kun fra en MAPIR-kildekodeutlevering; `pip install chloros-sdk` tilbyr dem heller ikke.)

***

## Forutsetninger

* **Chloros-backenden må være i gang** – `pool-*`-kommandoene er HTTP-klienter, ikke maskinvaredrivere. På Windows starter du Chloros-skrivebordsappen (den starter backenden). På headless Linux/Jetson aktiverer du tjenesten: `sudo systemctl enable --now chloros-backend.service`.
* **En Chloros+ (betalt nivå) pålogging**: kjør `chloros-cli login` først. Håndhevelsen skjer på serversiden — uten pålogging mislykkes kommandoer med `401 AUTH_REQUIRED`; på gratisnivået (Iron) mislykkes de med `403 PLAN_UPGRADE_REQUIRED`.
* Kommandoene er som standard rettet mot `http://127.0.0.1:5000`; `daq pool-*`-familien respekterer miljøvariabelen `CHLOROS_BACKEND_URL` hvis backenden din kjører et annet sted.

***

## En fem-minutters økt

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)

# 2. List the pool — this shows the sensor_id used by every command below
chloros-cli daq pool-list

# 3. Read the most recent calibrated spectrum frame (add --json for scripting)
chloros-cli daq pool-latest --sensor-id daq-e-def330 --json

# 4. Record a calibrated .daq file for 60 seconds
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 60 \
  --device-name "field-A"

# 5. Release the sensor when done
chloros-cli daq pool-disconnect --sensor-id daq-e-def330
```

***

## `pool-connect` — åpne en sensor i puljen

| Variant | Betydning |
| --- | --- |
| `daq pool-connect` | Smart-detect: finn en hvilken som helst DAQ på denne maskinen. |
| `daq pool-connect --port PORT` | DAQ-U på en spesifikk seriell port (f.eks. `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | DAQ-M via BLE, MAC-adresse skannet automatisk. |
| `daq pool-connect --mac MAC` | DAQ-M på en kjent BLE-MAC (innebærer `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E med kjent vertsnavn eller IP — **den pålitelige banen**. |
| `daq pool-connect --eth` | DAQ-E med automatisk oppdagelse (mDNS, med ARP som reserve). Se forbeholdet nedenfor. |

Justeringsflagg, alle valgfrie:

| Flagg | Betydning |
| --- | --- |
| `--integration-time MS` / `-t MS` | Manuell integrasjonstid i millisekunder. |
| `--frame-avg N` / `-f N` | Gjennomsnittlig antall rammer per rapportert spektrum. |
| `--no-ae` | Deaktiver automatisk eksponering (AE er aktivert som standard). |
| `--no-stream` | Koble til uten å starte strømmen (fortsett senere med `pool-stream --start`). |
| `--cap-id CAP` | Cap-korreksjonsprofil; standard for backend er `sunshine_cosine`. Se [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

{% hint style="warning" %}
**`--eth` advarsel om automatisk oppdagelse.** På en vert med flere nettverksgrensesnitt (mer enn ett aktivt nettverksgrensesnitt) kan den *første* `pool-connect --eth` etter oppstart gi et tomt resultat selv om sensoren er i god stand — søket etter sensorer kan overse sensorens grensesnitt mens ARP-cachen er tom. Hvis `--eth` ikke finner noe, må du prøve på nytt, eller hoppe over oppdagelsen helt ved å bruke `--eth-host <ip-or-hostname>`, som er den pålitelige metoden på maskiner med flere nettverksgrensesnitt. DAQ-E-enhetens vertsnavn er `daq-e-<id>.local` (f.eks. `daq-e-def330.local`); den vanlige IP-adressen fungerer også.
{% endhint %}

## `pool-list` — se hva som er tilkoblet

Viser alle sensorer i backend-poolen, inkludert `sensor_id` som alle andre kommandoer trenger:

| Modell | `sensor_id`-format | Eksempel |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5-oktett med bindestrek | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

## `pool-latest` — les spektrumrammer

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Returnerer den siste rammen, eller de siste `--recent N`-rammene; `--json` sender maskinlesbar utdata for skripting. Rammene er radiometrisk kalibrert spektral irradians (W/m²/nm) på et rutenett med 135 punkter i området 340–1010 nm, der sensorens dekkprofil allerede er anvendt. For kvantitative strålingsstyrketall må du beregne gjennomsnittet av minst 15 sekunder med rammer — dette er en egenskap ved instrumentet, ikke en feil.

## `pool-stream` — pause eller gjenoppta strømming

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` — ta opp en `.daq`-fil

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| Flagg | Standard | Betydning |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | Opptakets lengde i sekunder; `0` betyr at programmet skal kjøre til du gir kommandoen `--stop`. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Utdatakatalog, oppløst **på maskinen som kjører backenden**. |
| `--device-name NAME` | — | Etikett lagret sammen med opptaket. |
| `--stop` | — | Stopp et pågående opptak. |

{% hint style="info" %}
Opptaket skjer i backend-en, så filen `.daq` havner på **backend-maskinens** filsystem — som standard i `~/Documents/DAQ Live View/` der, ikke nødvendigvis der du kjørte CLI. Filnavnene inneholder sensor-ID og en tidsstempel.
{% endhint %}

## `pool-set-cap` — angi hvilket deksel som er montert

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

Kappe-ID-en velger den fabrikkmålte korreksjonsprofilen som brukes på hvert spektrum, og den **må samsvare med kappen som er fysisk montert på sensoren** — verken sensoren eller programvaren kan oppdage kappen på egen hånd, og valget blir stemplet inn i hver `.daq`-fil. Standardinnstillingen overalt er `sunshine_cosine` (hver DAQ leveres med Sunshine-kosinuskorreksjonshetten installert, ~12× demping ifølge design — et udeklarert bytte av hetten fører til feilkorrigering av spektra med omtrent den faktoren).

| `--cap-id` | Tilgjengelig på |
| --- | --- |
| `sunshine_cosine` (standard) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | Kun DAQ-U |
| `none` | Kun DAQ-E — se merknad |

En deksel-ID utenfor sensorens sett avvises ved tilkobling med en tydelig feilmelding. `none` (DAQ-E) betyr at hetten er fysisk fjernet — den bruker fortsatt en fabrikkgeometriprofil for DAQ-E-ens innfelte glassdiffusor, så det er ikke en «no-op», og en «bar» DAQ-E er en benkkonfigurasjon, ikke en støttet feltkonfigurasjon. (En «bar» DAQ-U er helt uten deksel og trenger ingen korreksjonsprofil i det hele tatt; DAQ-M brukes med sitt Sunshine-deksel.)

## `pool-disconnect` — frigjør sensorer

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

***

## Kommandooversikt

| Kommando | Formål |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Åpne en sensor i backend-poolen. |
| `daq pool-list` | Vis alle sensorer i puljen sammen med deres `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | De N siste kalibrerte spektrumrammene. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Gjenoppta / pause strømming. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--stop]` | Start / stopp en `.daq`-opptak (backend-side). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Bytt cap-korreksjonsprofil under kjøring. |
| `daq pool-disconnect --sensor-id ID [--all]` | Frigjør én sensor eller alle. |

***

## Feilsøking ved første tilkobling av DAQ-E

1. DAQ-E har ingen status-LED — bekreft strømforsyningen via PoE-/link-indikatoren på bryteren eller injektorporten, og vent noen sekunder etter oppstart slik at enheten kan starte opp og koble seg til nettverket.
2. Backend-maskinen må være på **samme kringkastingsdomene** som sensoren – mDNS går ikke gjennom rutere.
3. På Windows må du godta Defender-brannmurmeldingen ved første oppstart (mDNS UDP 5353, DAQ-E-data UDP 5002, PTP UDP 319/320).
4. Fortsatt ingen respons fra `--eth`? Bruk `--eth-host` med enhetens vertsnavn (`daq-e-<id>.local`) eller IP-adresse — dette er den pålitelige metoden, spesielt på verter med flere nettverksadresser.

***{% hint style="info" %}**Tips til AI-assistenter.** Hver side i denne håndboken leveres som rå Markdown – legg til `.md` til sidens slug i små bokstaver (URL) (denne siden: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); den maskinlesbare indeksen er `https://mapir.gitbook.io/chloros/llms.txt`. For fullstendig dokumentasjon på flagnivå for `chloros-cli daq` og alle andre kommandofamilier, hent [CLI-referansen](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); banen til Python er `chloros_sdk.connect_daq_sensor()` i [SDK-referansen](../reference/sdk-reference.md).
{% endhint %}
