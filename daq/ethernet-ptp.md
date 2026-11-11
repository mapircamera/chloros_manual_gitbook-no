# DAQ-E nettverk og tidssynkronisering

> Det fysiske nettverksoppsettet for sensoren – kabling, PoE, IP-tildeling og enhetens egne nettverksinnstillinger – finnes i **[DAQ-brukerhåndboken](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Denne siden omhandler Chloros-siden: tilkobling, tidssynkronisering og hva du skal gjøre hvis oppdagelsen ikke gir noen resultater.

DAQ-E er Ethernet-medlemmet i DAQ-familien: den får strøm via PoE, oppdages via mDNS (tjeneste `_daq-e._tcp`) og kan adresseres med et vertsnavn avledet fra sensor-ID-en — `daq-e-<6 hex>.local`, f.eks. `daq-e-def330.local`. Denne siden omhandler hvordan den overfører data på nettverket og hvordan den deltar i PTP-tidssynkronisering.

## Transportmoduser

| Modus | Endepunkt | Brukere | Merknader |
| --- | --- | --- | --- |
| **Multicast** (standard) | UDP `239.10.10.10:5002` | Et hvilket som helst antall enheter på samme LAN mottar den samme strømmen | Hvert datagram er CRC-16/CCITT-valideret |
| **Raw** | TCP-port `5000` | Nøyaktig én klient (eksklusivt) | Direkte kompatibel med DAQ-U på byte-nivå |

Chloros bruker multicast som standard, noe som gjør at GUI, CLI og SDK alle kan overvåke én sensor samtidig.

## Nettverkskrav

* **Samme kringkastingsdomene.** Maskinen som kjører Chloros, må være på samme L2-nettverkssegment som sensoren — mDNS-oppdagelse går ikke gjennom rutere.
* **Brannmur.** På Windows legger installasjonsprogrammet til brannmurregelen **Chloros Backend (UDP-In)**for backenden i hver nettverksprofil, som dekker DAQ-E-data (UDP 5002), mDNS (UDP 5353) og PTP (UDP 319/320), samt en**«Chloros Backend (TCP-In)»**-regel, begrenset til det lokale subnettet, for tilkoblingen en DAQ-E oppretter tilbake til datamaskinen under en fastvareoppdatering. På Linux åpner `.deb` UDP-portene i `ufw` eller `firewalld` når den brannmuren er aktiv ved installasjon. En tredjeparts brannmur, eller en som aktiveres senere, må ha de samme portene åpne.
* **PoE-strøm, ingen status-LED.** DAQ-E har ingen egen LED — kontroller strømtilførselen via link/PoE-indikatoren på bryteren eller injektorporten, og vent noen sekunder etter oppstart slik at enheten kan starte opp og koble seg til nettverket.

## Tilkobling

**GUI:** Fanen «Lyssensorer» → «Koble til sensor» → Enhetstype «DAQ-E (Ethernet)». Oppdagelsen kjører kun mens tilkoblingsdialogen er på skjermen (mDNS-søk pluss en ARP-søk på Windows), og gjentas hvert 15. sekund; «Oppdater»-knappen utfører en ny skanning umiddelbart. Oppdagede sensorer vises i rullegardinmenyen; den første oppdagede sensoren velges automatisk.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (backend kjører):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

En DAQ-E som allerede finnes i puljen, gjenkjennes både på vertsnavnet og IP-adressen: hvis du kobler den til igjen under den andre adressen, returneres den eksisterende puljeoppføringen i stedet for at enheten åpnes en gang til.

### Verter med flere nettverkskort og den første tilkoblingen etter oppstart

På verter med mer enn ett aktivt nettverksgrensesnitt kan den **første** `pool-connect --eth` etter oppstart være tom selv om sensoren er i orden — oppdagingssøket kan overse grensesnittet sensoren befinner seg på mens ARP-cachen fortsatt er kald. Den pålitelige løsningen er å hoppe over oppdagelsen og angi adressen eksplisitt:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` aksepterer mDNS-vertsnavnet eller IP-adressen, retter seg alltid mot riktig sensor og er den anbefalte formen for skript og installasjoner uten grafisk grensesnitt. I GUI-en bruker du Oppdater-knappen i tilkoblingsdialogen og lar systemet gjennomføre en ny søkesyklus.

## Enhetsinnstillinger og fastvare

Sensoren lagrer selv sine nettverksinnstillinger: DHCP (fabrikkinnstillingen, som faller tilbake til en 169.254.x.x link-local-adresse når ingen DHCP-server svarer) eller en statisk adresse. Den lagrer også om den starter strømming ved oppstart. Endre disse i GUI-en (sensorens tannhjulsinnstillinger, **Nettverk**:**Endre…**,**Strøm ved oppstart**,**Start sensoren på nytt**; se [Fanen DAQ](gui.md)), eller fra CLI og SDK med `pool-set-network`, `pool-set-auto-stream` og `pool-reboot` (`DAQSensorSession.set_network()`, `set_auto_stream()`, `reboot()`; se [CLI-referansen](../reference/cli-reference.md#daq-e-device-settings)). Sensoren starter på nytt for å implementere en nettverksendring, og Chloros kobler den til på nytt. Det må finnes en statisk adresse på nettverket til datamaskinen som kjører Chloros: Chloros avviser en adresse den ikke kunne nå etter omstarten. Sensorens ID og vertsnavnet `daq-e-<id>.local` endres aldri. Nytt i 1.3.0.

**Firmwareoppdateringer er integrert i brukergrensesnittet.** Når en tilkoblet DAQ-E kjører firmware som er eldre enn den som følger med din Chloros-versjon, vises det en gul **Oppdatering tilgjengelig**-knapp i sensorraden, og innstillingsmodalen med tannhjulet tilbyr en «Oppdater til<version>»-knapp. Oppdateringen overføres via nettverket på omtrent 30 sekunder; sensoren starter på nytt og kobler seg til igjen automatisk, og en avbrutt overføring lar den gjeldende firmware være intakt. Fra CLI eller SDK, og på Linux – som ikke har grafisk brukergrensesnitt – kjører du `chloros-cli daq pool-update-firmware --sensor-id <id>` (`DAQSensorSession.update_firmware()`).

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## PTP-tidssynkronisering

DAQ-E-firmware v1.2.0+ deltar i IEEE 1588 PTPv2 som en vanlig (kun slave) klokke. **Chloros-vertens backend er PTP-grandmasteren** — hver DAQ-E og hvert LATTICE-kamera på LAN-nettverket er slaver til den i domene 0, og holder alle enhetens tidsstempler innenfor en toleranse på ca. 1 ms. Det er denne delte klokken som gjør at DAQ-målingene kan tidsstemples i samsvar med kameraets eksponeringer (se [Opptak og .daq-formatet](recording.md)).

Kontroller synkroniseringen fra CLI-en:

| Kommando | Viser |
| --- | --- |
| `chloros-cli time-sync status` | Verts-grandmaster-tilstand, BMCA-prioriteter, klokkeidentitet og `health`: `OK`, eller `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` med en `health_detail` som angir hvorfor pakkene ikke når nettverket |
| `chloros-cli time-sync peers` | Alle slaver som er oppdaget (DAQ-E-sensorer + LATTICE-kameraer) |
| `chloros-cli time-sync cameras` | PTP-tilstand per kamera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Start grandmaster-prosessen på nytt |

I brukergrensesnittet viser innstillingsvinduet for DAQ-E en **PTP Sync**-rad i sanntid med sensorens gjeldende PTP-tilstand.**En DAQ-E med PTP slått av forblir av.** Enheten lagrer sin PTP-innstilling (på/av) i sitt eget minne, så en enhet som hadde PTP slått av, forblir av etter en ny tilkobling eller strømavbrudd, og tidsstemplene blir ikke PTP-synkronisert. `chloros-cli daq pool-list` markerer en slik enhet (`!! PTP is switched OFF on this unit …`) og skriver ut kommandoen som løser problemet:

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Chloros slår aldri en enhet på igjen av seg selv, fordi «av» kan være bevisst valgt.

Detaljer for brukere som krever streng synkronisering:

* Hvert strømmet datagram har et flaggfelt; **bit 2 er satt på rammer der tidsstemplet er PTP-synkronisert**. Rørledninger som krever streng synkronisering mellom kamera og DAQ, bør basere sin datainnhenting på dette biten.
* Før en synkronisert opptak, må du bekrefte at sensoren vises i `chloros-cli time-sync peers` og at `chloros-cli daq pool-list` ikke markerer PTP-en som slått av. (MAPIRs interne verktøy for direkte maskinvarehåndtering kan også styre opptaket basert på PTP-lås ved hjelp av et `--wait-ptp`-flagg som venter i opptil 15 sekunder på at sensoren skal nå SLAVE-tilstand; dette verktøyet er ikke en del av den medfølgende CLI-en.)
* Mens PTP er aktivt i slave-modus, avviser sensoren manuelle klokkesignaler («PTP leverer klokkesignal»). Dette er etter design – stol på PTP.

## Merknader om Linux

* **PTP-porter uten root-rettigheter.** PTP binder seg til UDP 319/320. `.deb` skriver `/etc/sysctl.d/60-chloros-ptp.conf` (`net.ipv4.ip_unprivileged_port_start = 319`), slik at backenden binder dem og kjører dem som din bruker. Fra og med versjon 1.3.0 setter pakken ingen filkapasitet (`setcap`) på backenden og trenger ingen `libcap2-bin`; ikke legg til en kapasitet selv, fordi den utpakkede backenden da ikke starter. Hvis PTP ikke kan binde port 319, se [Linux-installasjon](../linux/linux-installation.md#ptp-will-not-start--cannot-bind-port-319).

* **Headless Jetson / Linux-vert:** pakken installerer backend-tjenestene, men aktiverer ingen av dem. For PTP som alltid er på (og DAQ-tilgjengelighet) uten at noen kjører en kommando, må du aktivere tjenesten som din bruker:

  ```bash
  sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
  ```

  Uten dette kjører PTP bare så lenge noe har startet backend-tjenesten – en `chloros-cli`-kommando eller et SDK-skript. Se [Linux-installasjon](../linux/linux-installation.md#always-on-ptp-for-headless-hosts) for root-brukeren `chloros-backend.service` og hvordan du bytter fra denne.

## Feilsøking: «Ingen DAQ-E-enheter funnet»

| Sjekk | Detalj |
| --- | --- |
| Strøm | Ingen LED på sensoren — sjekk PoE- og link-indikatorene på bryteren/injektorporten; vent noen sekunder etter oppstart |
| Broadcast-domene | Vert og sensor på samme L2-segment; mDNS ruter ikke |
| Brannmur | Windows: installasjonsprogrammets **Chloros Backend (UDP-In)**-regel slipper inn DAQ-E-strøm (UDP 5002), mDNS (5353) og PTP (319/320); en tredjeparts brannmur må ha de samme innstillingene. Linux: `.deb` åpner disse portene i `ufw` / `firewalld` bare hvis brannmuren var aktiv ved installasjon – tillat dem selv i en brannmur som aktiveres senere |
| Vert med flere nettverkskort | Oppdagingsfunksjonen søker i mDNS med ARP som reserve; `--eth-host <ip-or-hostname>` hopper over oppdagingen og kobler seg til direkte |
| Ny skanning via GUI | Oppdagingsfunksjonen kjører bare mens tilkoblingsdialogen er åpen; bruk Oppdater |</version>
