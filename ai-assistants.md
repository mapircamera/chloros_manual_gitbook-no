# Bruk av Chloros med AI-assistenter

Denne håndboken er skrevet for to målgrupper: mennesker og AI-assistentene som mennesker i stadig større grad bruker i sitt arbeid. Hver side inneholder eksakte verdier, standardinnstillinger og kommandoer som kan kopieres og limes inn, slik at en assistent (Claude, ChatGPT, Copilot, en kodingsagent, …) kan skrive fungerende Chloros-automatisering ved første forsøk.

Chloros-versjon: **

1.2.0**. CLI/SDK-plattformer: Windows 10/11 x64 og Linux (x86_64 / Jetson aarch64).

## Hva du skal gi assistenten din

| Ressurs | URL | Hva den brukes til |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Maskinlesbar indeks over alle sidene i denne håndboken. |
| **CLI-referanse** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Den komplette `chloros-cli`-kommandoflaten: alle kommandoer, flagg, standardverdier, avslutningskoder og regler for utdatamapper. Skrevet for bruk av LLM. |
| **SDK-referanse** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Den komplette `chloros_sdk` Python API: klasser, signaturer, unntak og eksempler. Skrevet for bruk i LLM. |
| **Enhver side som rå Markdown** | legg til `.md` til siden URL | f.eks. `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` returnerer siden som ren Markdown — ideelt for å lime inn i et kontekstvindu eller hente fra en agent. |

Lenker i håndboken: [CLI Referanse](reference/cli-reference.md) · [SDK Referanse](reference/sdk-reference.md).

{% hint style="info" %}
De to referansesidene er selvstendige: en assistent som har lest en av dem, trenger ikke resten av manualen for å skrive et korrekt skript.
{% endhint %}

## Ferdige kommandoer

Kopier, fyll ut `<placeholders>` og lim inn i assistenten din.

### 1. Behandle en flymappe til NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Overvåk en katalog med opptak i batch-modus

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Koble til et LATTICE-array og ta opp

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Registrer spektra fra DAQ-lyssensorer

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
DAQ-skripting fra kommandolinjen går alltid gjennom `daq pool-*`-familien (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Andre `daq`-underkommandoer som assistenten din måtte finne på, er ikke tilgjengelige i de leverte versjonene og avsluttes med en feilmelding.
{% endhint %}

## Hvorfor AI-skrevne skript fungerer godt med Chloros

Hver av disse er en reell, verifisert funksjon i Chloros 1.2.0 — de eliminerer de klassiske feilmodusene ved maskinskrevet automatisering:

* **Ingen komplisert oppsett.**SDKs smart-connect-hjelpere (`connect_camera`, `connect_array`, `connect_daq_sensor`) og prosesseringsinngangspunktene (`ChlorosLocal`, `process_folder`)**starter den lokale backenden automatisk**. Et generert skript trenger ikke at GUI er åpent eller at en server startes manuelt — det trenger bare at desktop/CLI-pakken er installert.
* **Hele prosessen utføres med ett enkelt kall.** `chloros_sdk.process_folder("path", indices=["NDVI"])` kjører import → kalibrering → reflektans → indekseksport fra start til slutt. Mindre overflateareal, færre steder hvor et generert skript kan gå galt.
* **Kjøringer uten utdata utfører selvdiagnose.** Etter `process()` legges sammendraget av kjøringen ved resultatet, og hvert behandlingshint (f.eks. *hvorfor* en kjøring ikke ga noe utdata) sendes også ut på nytt som en Python `UserWarning` — slik at selv et skript som aldri sjekker resultatdiktet, viser diagnosen.
* **CLI mislykkes med et høyt varsel.**En `chloros-cli process`-kjøring som ba om produkter, men ikke skrev ut noen, skriver ut `Processing finished but wrote no image products.` og**avsluttes med en verdi ulik null**, slik at shell-skript og CI oppdager det ved en enkel sjekk av avslutningskoden. Vellykkede kjøringer rapporterer `Image products written: N`.

En asymmetri en assistent bør kjenne til: `process()` i SDK utløser bevisst **ikke** en feil ved en kjøring uten produkter — den rapporterer i stedet via sammendraget/tipsene. Hvis en Python-pipeline må stoppe ved en tom kjøring, sjekk sammendraget (oppskrift 2 gjør det).

## Forbehold

* **Chloros+ innlogging kreves.**CLI og SDK krever et**betalt** Chloros+-nivå, håndheves på serversiden: forespørsler mislykkes med `401 AUTH_REQUIRED` når du ikke er logget inn og med `403 PLAN_UPGRADE_REQUIRED` på gratisnivået. Kjør `chloros-cli login` én gang per maskin før du kjører genererte skript. Se [Chloros+ Innlogging](chloros+-login.md).
* **Capture-kommandoer styrer ekte maskinvare.** Kommandoene `lattice` / `daq` / `project` og sesjonsobjektene SDK kobler til, streamer og utløser fysiske kameraer og sensorer. Gjennomgå et generert skript før det kjøres for første gang, og kjør det mens du overvåker maskinvaren.
* **Gjør stikkprøver av utdataene.** Kontroller produktmappene og noen få pikselverdier før du publiserer resultatene. Spesielt skal TIFF-filer med reflektans skaleres per kilde — les `Chloros:PixelScale` XMP-taggen (LATTICE: 32768 = 1,0 reflektans; Survey3: 65535) i stedet for å anta en divisor. Begge referansesidene dokumenterer dette under «Lesing av reflektanspiksler».
* **Små fallgruver som kan forårsake feil i generert kode:**`pool-record` skriver til**backend-vertens** filsystem (standard `~/Documents/DAQ Live View/`); på maskiner med flere nettverksgrensesnitt bør du foretrekke `daq pool-connect --eth-host <ip-or-hostname>` fremfor automatisk oppdagelse; og bruk `http://127.0.0.1:5000` (aldri `localhost`) overalt der en backend URL dukker opp.
