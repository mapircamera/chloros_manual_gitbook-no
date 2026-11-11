# Bruk av Chloros sammen med AI-assistenter

Denne håndboken er skrevet for to målgrupper: mennesker og AI-assistentene som mennesker i stadig større grad bruker i arbeidet sitt. Hver side inneholder eksakte verdier, standardinnstillinger og kommandoer som kan kopieres og limes inn, slik at en assistent (Claude, ChatGPT, Copilot, en kodingsagent, …) kan skrive fungerende Chloros-automatisering ved første forsøk.

Chloros-versjon: **

1.3.0**. CLI-/SDK-plattformer: Windows 10/11 x64 og Linux (x86_64 / Jetson aarch64).

## Hva du skal gi assistenten din

| Ressurs | URL | Hva den er til |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Maskinlesbar indeks over alle sidene i denne håndboken. |
| **Referanse for CLI** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Den komplette kommandolisten for `chloros-cli`: alle kommandoer, flagg, standardverdier, avslutningskoder og regler for utdatamapper. Skrevet for bruk i LLM. |
| **SDK-referanse** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Det komplette `chloros_sdk` Python-API: klasser, signaturer, unntak og praktiske eksempler. Skrevet for bruk i LLM. |
| **Enhver side som rå Markdown** | legg til `.md` til sidens URL | f.eks. returnerer `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` siden som rå Markdown — ideelt for å lime inn i et kontekstvindu eller hente fra en agent. |

Koblinger i manualen: [CLI-referanse](reference/cli-reference.md) · [SDK-referanse](reference/sdk-reference.md).

{% hint style="info" %}
De to referansesidene er selvstendige: en assistent som har lest én av dem, trenger ikke resten av manualen for å skrive et korrekt skript.
{% endhint %}

## La assistenten din kjøre Chloros direkte: `chloros-mcp`

Alt ovenfor innebærer at assistenten din *skriver* et skript eller en kommando. Med `chloros-mcp` styrer en assistent som bruker [Model Context Protocol](https://modelcontextprotocol.io) (MCP), for eksempel Claude Desktop, Claude Code eller Cursor, Chloros-selve programmet. Du spør på vanlig språk, og den påkaller serverens verktøy:

* **Behandle bilder:** opprett et prosjekt, importer en mappe med opptak, velg innstillinger, behandle og rapporter hva kjøringen eksporterte.
* **Kjør maskinvare:** koble til LATTICE-kameraer, matriser og DAQ-lyssensorer, forhåndsvis, still inn eksponering og farge, ta opp, ta opp og oppdater fastvare.

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

Serverens verktøy dekker det samme som [Python SDK](api-python-sdk.md) gjør. Det installeres sammen med SDK-et, så det er vanligvis ikke noe annet å installere: på Windows plasserer Chloros-installasjonsprogrammet `chloros-mcp.exe` i Pythons `Scripts`-mappe, og i Linux-versjonen av `.deb` plasseres `chloros-mcp` i `/usr/bin`.

**Claude-kode:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop:**legg til dette i `claude_desktop_config.json` (**Innstillinger > Utvikler > Rediger konfigurasjon**åpner den), og start deretter Claude Desktop på nytt.**Cursor** bruker den samme oppføringen i `~/.cursor/mcp.json`.

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

Hvis klienten melder at serveren ikke kunne startes, er `chloros-mcp` sannsynligvis ikke på den `PATH` den ser: angi den fullstendige banen i stedet (`where chloros-mcp` på Windows, `which chloros-mcp` på Linux).

Du må ha Chloros installert på samme datamaskin og være logget inn der, enten i appen eller via `chloros-cli login`, på et hvilket som helst betalt Chloros+-abonnement (Copper eller høyere). Serveren kan ikke logge deg inn for deg.

{% hint style="info" %}
Klientkonfigurasjon for hver app, alternativer, hvert verktøy og hvordan en kjøring fungerer: [MCP-server (AI-assistenter)](reference/sdk-reference.md#mcp-server-ai-assistants) i SDK-referansen.
{% endhint %}

## Kommandooppskrifter

Kopier, fyll ut `<placeholders>` og lim inn i assistenten din.

### 1. Behandle en flygemappe til NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Overvåk en katalog med opptak i batch

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Koble til et LATTICE-array og ta opptak

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Registrere spektra fra DAQ-lyssensorer

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

Hver av disse er en reell, verifisert funksjon i Chloros 1.3.0 — de eliminerer de klassiske feilmodusene ved maskinskrevet automatisering:

* **Ingen komplisert oppsett.**SDK-ets smart-connect-hjelpefunksjoner (`connect_camera`, `connect_array`, `connect_daq_sensor`) og behandlingsinngangspunktene (`ChlorosLocal`, `process_folder`)**starter den lokale backenden automatisk**. Et generert skript trenger ikke at GUI-en er åpen eller at en server startes manuelt — det trenger bare at desktop-/CLI-pakken er installert.
* **Hele pipelinen utføres med ett enkelt kall.** `chloros_sdk.process_folder("path", indices=["NDVI"])` kjører import → kalibrering → reflektans → indekseksport fra start til slutt. Mindre overflateareal, færre steder hvor et generert skript kan gå galt.
* **Kjøringer uten utdata utfører selvdiagnose.** Etter `process()` legges sammendraget av kjøringen ved resultatet, og hvert behandlingshint (f.eks. *hvorfor* en kjøring ikke ga noe utdata) sendes også ut på nytt som en Python-`UserWarning` — slik at selv et skript som aldri sjekker resultat-dicten, viser diagnosen.
* **CLI-en mislykkes tydelig.**En `chloros-cli process`-kjøring som ba om produkter, men ikke skrev noen, skriver ut `Processing finished but wrote no image products.` og**avsluttes med en utgangskode ulik null**, slik at shell-skript og CI oppdager det ved en enkel sjekk av utgangskoden. Vellykkede kjøringer rapporterer `Image products written: N`, pluss `(plus M clamp masks)` når masker ble skrevet. Capture-kommandoer oppgir på samme måte navnene på alle filene de ikke klarte å skrive til, og avsluttes med `1`.
* **Relative baner betyr det et skript forventer.** Med en lokal backend blir en relativ utdatabane (`-o out/` i CLI, `output_dir="out/"` i SDK) løst i forhold til mappen kommandoen eller skriptet kjører fra.

En asymmetri en assistent bør kjenne til: SDKs `process()` utløser bevisst **ikke** en feil ved en kjøring uten resultater — den rapporterer i stedet gjennom sammendraget/tipsene. Hvis en Python-pipeline må stoppe ved en tom kjøring, sjekk sammendraget (oppskrift 2 gjør det).

## Forbehold

* **Chloros+-pålogging kreves.**CLI-en og SDK krever et**betalt** Chloros+-abonnement, noe som håndheves på serversiden: forespørsler mislykkes med feilkoden `401 AUTH_REQUIRED` når du ikke er logget inn, og med feilkoden `403 PLAN_UPGRADE_REQUIRED` på gratisabonnementet. Kjør `chloros-cli login` én gang per maskin før du kjører genererte skript. Se [Chloros+-pålogging](chloros+-login.md).
* **Capture-kommandoer styrer fysisk maskinvare.** `lattice` / `daq` / `project`-kommandoer, SDK-sesjonsobjektene og verktøyene til `chloros-mcp` kobler til, streamer og utløser fysiske kameraer og sensorer. Gjennomgå et generert skript før det kjøres for første gang, og kjør det mens du har tilgang til maskinvaren.
* **Gjør stikkprøver av utdataene.** Kontroller produktmappene og noen få pikselverdier før du publiserer resultatene. Spesielt skal refleksjons-TIFF-filer skaleres per kilde — les `Chloros:PixelScale` XMP-taggen (LATTICE-filer har den: 32768 = 1,0 refleksjon; Survey3-filer har den ikke, og bruker 65535 = 1,0) i stedet for å anta en divisor. Begge referansesidene dokumenterer dette under «Lesing av reflektanspiksler».
* **Små fallgruver som kan forstyrre generert kode:**`pool-record` skriver til**backend-vertens** filsystem (standard `~/Documents/DAQ Live View/`); på maskiner med flere nettverksgrensesnitt bør du foretrekke `daq pool-connect --eth-host <ip-or-hostname>` fremfor automatisk oppdagelse; bruk `http://127.0.0.1:5000` (aldri `localhost`) overalt der en backend-URL forekommer; et skript som går gjennom et prosjekts utdatatre bør hoppe over `clamp_masks/`-mapper (deres `*_clampmask.tif`-filer er masker, ikke bilder — se [Utdataformater](output-image-formats.md#clamp-masks)); og på Windows, der lange baner er deaktivert, bør prosjektmappene holdes korte — en prosjektmappe som er dypere enn ca. 130 tegn, avvises før behandlingen starter.
