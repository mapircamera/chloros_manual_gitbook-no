# CLI-hurtigstart (pool-*)

De medfølgende `chloros-cli`-enhetene styrer DAQ-sensorer via **`daq pool-*`**-kommandofamilien — enkle HTTP-klienter som styrer sensoren gjennom Chloros-backendens permanente sensorpool. Backenden styrer transporten, slik at GUI, CLI og SDK-skriptene alle deler ett aktivt håndtak i stedet for å konkurrere om porten. Alt en kunde trenger, er tilgjengelig via `pool-*`: koble til, streame, ta opp kalibrerte `.daq`-filer og bytte cap-profiler.

`pool-*` er den støttede DAQ-grensesnittet i utgitte versjoner. `chloros-cli daq --help` viser en liste over `pool-*`-underkommandoene, og `daq discover` / `daq list` kjører den samme skanningen som `pool-discover`. Der en DAQ-underkommando for direkte maskinvare ikke er tilgjengelig, avsluttes den med en eksplisitt feilmelding som angir den manglende pakken og henviser deg tilbake til `pool-*` — ingenting mislykkes i stillhet. (`pip install chloros-sdk` tilbyr heller ikke kommandoer for direkte maskinvare.)

***

## Forutsetninger

* **Chloros-backenden må være i gang** — kommandoene `pool-*` er HTTP-klienter, ikke maskinvaredrivere. På Windows starter du Chloros-skrivebordsappen (den starter backenden). På headless Linux/Jetson må du aktivere tjenesten som din bruker: `sudo systemctl enable --now chloros-backend@$USER.service` (se [Linux-installasjon](../linux/linux-installation.md#always-on-ptp-for-headless-hosts)).
* **Chloros+ (betalt nivå) pålogging**: kjør `chloros-cli login` først. Håndhevelsen skjer på serversiden — uten pålogging mislykkes kommandoer med `401 AUTH_REQUIRED`; på gratisnivået (Iron) mislykkes de med `403 PLAN_UPGRADE_REQUIRED`.
* Kommandoene retter seg mot `http://127.0.0.1:5000` som standard; `daq pool-*`-familien respekterer miljøvariabelen `CHLOROS_BACKEND_URL` hvis backenden din kjører et annet sted.

***

## En fem minutters økt

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)
#    (a DAQ-M's BLE MAC is not printed anywhere: find it with `chloros-cli daq pool-discover --only ble`)

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
| `daq pool-connect` | Smart-detect: finn alle DAQ-enheter på denne maskinen. |
| `daq pool-connect --port PORT` | DAQ-U på en bestemt seriell port (f.eks. `COM3`, `/dev/ttyUSB0`). |
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

Når du kobler til en DAQ-E som allerede finnes i puljen under en annen av dens adresser — vertsnavnet `daq-e-<id>.local` eller IP-adressen — returneres den eksisterende puljeoppføringen i stedet for å åpne enheten en gang til.

{% hint style="warning" %}
**`--eth` advarsel om automatisk oppdagelse.** På en vert med flere nettverksgrensesnitt (mer enn ett aktivt nettverksgrensesnitt) kan den *første* `pool-connect --eth` etter oppstart komme opp tom selv om sensoren er i orden — oppdagingssøket kan overse sensorens grensesnitt mens ARP-cachen er kald. Hvis `--eth` ikke finner noe, prøv på nytt, eller hopp helt over oppdagelsen med `--eth-host <ip-or-hostname>`, som er den pålitelige metoden på maskiner med flere nettverksgrensesnitt. DAQ-E-enhetens vertsnavn er `daq-e-<id>.local` (f.eks. `daq-e-def330.local`); den vanlige IP-adressen fungerer også.
{% endhint %}

## `pool-discover` — skann uten å koble til

```bash
chloros-cli daq pool-discover                  # USB, BLE and Ethernet
chloros-cli daq pool-discover --only ble       # BLE only: this is how you find a DAQ-M's MAC
chloros-cli daq pool-discover --json           # machine-readable
```

Skanner etter sensorer du kan koble deg til og viser adressen du skal koble deg til, uten å koble til. `--only` tar en kommaseparert delmengde av `usb,ble,eth`; `--timeout SEC` angir skannetiden per transport (standard 5 s, begrenset til 1–20 s). Sensorer som allerede er åpne i puljen, vises ikke (en tilkoblet DAQ-M slutter å annonsere) — bruk `pool-list` for disse. I en utgitt versjon kjører `chloros-cli daq discover` og `daq list` denne samme skanningen.

## `pool-list` — se hva som er tilkoblet

Viser alle sensorer i backend-poolen, inkludert `sensor_id` som alle andre kommandoer trenger:

| Modell | `sensor_id`-format | Eksempel |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5-oktett med bindestrek | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

En DAQ-E der PTP er slått av (innstillingen er lagret på enheten), merkes i `pool-list` med `!! PTP is switched OFF on this unit (saved in its flash): its timestamps are not PTP-synced.` og kommandoen som slår den på igjen — se [`pool-enable-ptp`](#pool-enable-ptp-switch-a-daq-es-ptp-back-on).

## `pool-latest` — les spektrumrammer

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Returnerer den siste rammen, eller de siste `--recent N`-rammene; `--json` genererer maskinlesbar utdata for skripting. Rammene er radiometrisk kalibrert spektral irradians (W/m²/nm) på et rutenett med 135 punkter i området 340–1010 nm, med sensorens dekkprofil allerede anvendt. For kvantitative strålingsstyrketall må du beregne gjennomsnittet av minst 15 sekunder med rammer — dette er en egenskap ved instrumentet, ikke en feil.

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

| Flag | Standard | Betydning |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | Opptakslengde i sekunder; `0` betyr at opptaket skal kjøre til du gir kommandoen `--stop`. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Utdatakatalog. Med backenden på denne datamaskinen blir en relativ sti løst i forhold til mappen der du kjørte kommandoen; med en ekstern backend blir stien løst **på maskinen som kjører backenden**. |
| `--device-name NAME` | — | Etikett som lagres sammen med opptaket. |
| `--stop` | — | Stopp et pågående opptak. |
| `--confirm-after SEC` | `20` | Etter oppstart, vent opptil denne tiden på at den første raden skal skrives, og gi et tydelig varsel (`!! WARNING: no rows written after …s. This recording is producing an EMPTY file.`) hvis ingen kommer. `0` deaktiverer kontrollen. |
| `--heartbeat SEC` | `30` | Med `--duration` skal antall rader skrives ut hvert SEC sekund, og det skal gis en advarsel hvis antallet slutter å øke. `0` deaktiverer den. |

{% hint style="info" %}
Opptaket skjer i backend, så filen `.daq` havner på **backend-maskinens** filsystem — som standard i `~/Documents/DAQ Live View/` der, ikke nødvendigvis der du kjørte CLI-en. Filnavnene inneholder sensor-ID og en tidsstempel.
{% endhint %}

## `pool-set-cap` — angi den monterte hetten

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

Kappe-ID-en velger den fabrikkmålte korreksjonsprofilen som brukes på hvert spektrum, og den **må samsvare med kappen som er fysisk montert på sensoren** — verken sensoren eller programvaren kan oppdage kappen på egen hånd, og valget blir stemplet inn i hver `.daq`-fil. Standardverdien overalt er `sunshine_cosine` (hver DAQ leveres med Sunshine-kosinuskorreksjonshetten installert, ~12× demping etter design — et udeklarert bytte av hette fører til feilkorrigering av spektra med omtrent den faktoren).

| `--cap-id` | Tilgjengelig på |
| --- | --- |
| `sunshine_cosine` (standard) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | Kun DAQ-U |
| `none` | Kun DAQ-E — se merknad |

En deksel-ID utenfor sensorens sett avvises ved tilkobling med en tydelig feilmelding. `none` (DAQ-E) betyr at hetten er fysisk fjernet — den bruker fortsatt en fabrikkgeometriprofil for DAQ-E-enhetens innfelte glassdiffusor, så det er ikke en «no-op», og en «bar» DAQ-E er en benkkonfigurasjon, ikke en støttet feltkonfigurasjon. (En DAQ-U uten deksel er helt uten deksel og trenger ingen korreksjonsprofil i det hele tatt; DAQ-M brukes med sitt Sunshine-deksel.)

## `pool-disconnect` — frigjør sensorer

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

`pool-disconnect` frigjør en sensor uansett hvem som åpnet den, så den frigjør også en sensor som et krasjet skript har etterlatt i puljen (en opptak på den må først være fullført). En sensor som ikke er i puljen har ingenting å frigjøre: kommandoen skriver ut `<sensor-id>: already released (not in the pool)` og avsluttes med `0`. En frigjøring som backenden nektet eller ikke bekreftet, avsluttes med `1`.

## `pool-enable-ptp` — slå PTP på en DAQ-E på igjen

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

En DAQ-E lagrer sin PTP-innstilling (på/av) i sitt eget minne, så en enhet hvis PTP ble slått av, forblir av — og tidsstemplene er ikke PTP-synkronisert — inntil den slås på igjen. `pool-list` markerer en slik enhet og skriver ut denne kommandoen for den. Chloros slår aldri en enhet på igjen av seg selv, fordi «av» kan være bevisst. Nytt i 1.3.0.

## DAQ-E-enhetsinnstillinger

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto
chloros-cli daq pool-set-auto-stream --sensor-id daq-e-def330 --on
chloros-cli daq pool-reboot --sensor-id daq-e-def330
chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

`pool-info` viser sensorens fastvare (og om denne Chloros-utgaven inneholder en nyere versjon), adressen og hvordan den fikk den, den lagrede nettverksinnstillingen dersom den avviker fra den aktive, strømming ved oppstart og PTP. `pool-set-network` starter sensoren på nytt for å gjennomføre endringen og kobler den til igjen med de innstillingene den hadde; `--no-reboot` lagrer den bare. En statisk adresse må finnes på denne datamaskinens nettverk, ellers avviser Chloros den; `--force` overstyrer dette, men kun for en datamaskin som skal omadresseres eller rutes til den. Omstart avviser mens sensoren registrerer data. `pool-update-firmware` er GUI-ens **Oppdater til \<version\>** for kommandolinjen, og måten å oppdatere en DAQ-E på Linux, som ikke har noe GUI. Nytt i 1.3.0.

***

## Kommandooversikt

| Kommando | Formål |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Åpne en sensor i backend-poolen. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Søker etter sensorer uten å koble til (finner BLE-MAC-adressen til en DAQ-M). |
| `daq pool-list` | Viser alle sensorer i puljen med tilhørende `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | De N siste kalibrerte spektrumrammene. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Gjenoppta / pause strømming. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--confirm-after SEC] [--heartbeat SEC] [--stop]` | Start / stopp en `.daq`-opptak (backend-side). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Bytt cap-korreksjonsprofil under kjøring. |
| `daq pool-disconnect --sensor-id ID [--all]` | Frigjør én sensor eller alle. |
| `daq pool-enable-ptp --sensor-id ID` | Slå PTP på igjen for en DAQ-E der PTP er slått av. |
| `daq pool-info --sensor-id ID [--json]` | En DAQ-Es fastvare, adresse, lagrede nettverksinnstillinger, strømming ved oppstart og PTP. |
| `daq pool-set-network --sensor-id ID (--auto \| --static IP [--mask M] [--gateway G]) [--no-reboot] [--force]` | DHCP eller en statisk adresse; starter sensoren på nytt og kobler den til på nytt. |
| `daq pool-set-auto-stream --sensor-id ID (--on \| --off)` | Strøm ved oppstart. |
| `daq pool-reboot --sensor-id ID` | Start en DAQ-E på nytt og koble den til på nytt. |
| `daq pool-update-firmware --sensor-id ID` | Oppdater en eldre sensor med denne versjonens DAQ-E-firmware. |

***

## Feilsøking ved første tilkobling av en DAQ-E

1. DAQ-E har ingen status-LED — bekreft strømforsyningen via PoE-/link-indikatoren på bryteren eller injektorporten, og vent noen sekunder etter oppstart slik at den kan starte opp og koble seg til nettverket.
2. Backend-maskinen må være på **samme kringkastingsdomene** som sensoren – mDNS går ikke gjennom rutere.
3. En brannmur må slippe inn mDNS (UDP 5353), DAQ-E-data (UDP 5002) og PTP (UDP 319/320). Windows-installasjonsprogrammet legger til en regel for dette, og Linux-versjonen `.deb` åpner dem i en `ufw` / `firewalld` som var aktiv ved installasjonstidspunktet; en tredjeparts brannmur, eller en som aktiveres senere, må tillate disse.
4. Fortsatt ingenting fra `--eth`? Bruk `--eth-host` med enhetens vertsnavn (`daq-e-<id>.local`) eller IP-adresse, noe som hopper over oppdagingsprosessen.

***
{% hint style="info" %}
**Tips for AI-assistenter.** Hver side i denne håndboken leveres som rå Markdown — legg til `.md` til sidens URL-slug i små bokstaver (denne siden: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); den maskinlesbare indeksen er `https://mapir.gitbook.io/chloros/llms.txt`. For fullstendig dokumentasjon på flagnivå for `chloros-cli daq` og alle andre kommandofamilier, hent [CLI-referansen](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); Python-banen er `chloros_sdk.connect_daq_sensor()` i [SDK-referansen](../reference/sdk-reference.md).
{% endhint %}
