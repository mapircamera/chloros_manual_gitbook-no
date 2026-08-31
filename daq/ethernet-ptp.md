# DAQ-E nettverk og tidssynkronisering

> Det fysiske nettverksoppsettet for sensoren – kabling, PoE, IP-tildeling og enhetens egne nettverksinnstillinger – finnes i **[DAQ-brukerhåndboken](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Denne siden omhandler Chloros-siden: tilkobling, tidssynkronisering og hva du skal gjøre hvis oppdagelsen ikke gir noen resultater.

DAQ-E er Ethernet-medlemmet i DAQ-familien: strømforsynt via PoE, oppdages via mDNS (tjeneste `_daq-e._tcp`) og kan adresseres med et vertsnavn avledet fra sensor-ID-en — `daq-e-<6 hex>.local`, f.eks. `daq-e-def330.local`. Denne siden omhandler hvordan den overfører data på nettverket og hvordan den deltar i PTP-tidssynkronisering.

## Transportmoduser

| Modus | Endepunkt | Mottakere | Merknader |
| --- | --- | --- | --- |
| **Multicast** (standard) | UDP `239.10.10.10:5002` | Et hvilket som helst antall enheter på samme LAN mottar den samme strømmen | Hvert datagram er CRC-16/CCITT-valideret |
| **Raw** | TCP-port `5000` | Nøyaktig én klient (eksklusivt) | Direkte kompatibel med DAQ-U på byte-nivå |

Chloros bruker multicast som standard, noe som gjør at GUI, CLI og SDK alle kan overvåke én sensor samtidig.

## Nettverkskrav

* **Samme kringkastingsdomene.** Maskinen som kjører Chloros må være på samme L2-nettverkssegment som sensoren — mDNS-oppdagelse går ikke gjennom rutere.
* **Brannmurmelding for Windows: godta den.** Første gang Chloros binder multicast-soklene, spør Windows Defender én gang. Hvis du tillater det, omfatter dette DAQ-E-data (UDP 5002), mDNS (UDP 5353) og PTP (UDP 319/320). På Linux skjer dette uten melding.
* **PoE-strøm, ingen status-LED.** DAQ-E har ingen egen LED — sjekk strømmen via link/PoE-indikatoren på bryteren eller injektorporten, og vent noen sekunder etter oppstart slik at den kan starte opp og koble seg til nettverket.

## Tilkobling

**GUI:** Fanen «Lyssensorer» → «Koble til sensor» → Enhetstype «DAQ-E (Ethernet)». Oppdagelsen kjører bare mens tilkoblingsdialogen er på skjermen (mDNS-søk pluss en ARP-søk på Windows), og gjentas hvert 15. sekund; «Oppdater»-knappen utfører en ny søk umiddelbart. Oppdagede sensorer vises i rullegardinmenyen; den første oppdagede sensoren velges automatisk.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (backend kjører):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

### Verter med flere nettverkskort og den første tilkoblingen etter oppstart

På verter med mer enn ett aktivt nettverksgrensesnitt kan den **første** `pool-connect --eth` etter oppstart være tom selv om sensoren er i orden — oppdagingssøket kan overse grensesnittet sensoren befinner seg på mens ARP-cachen fortsatt er kald. Den pålitelige løsningen er å hoppe over oppdagelsen og angi adressen eksplisitt:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` aksepterer mDNS-vertsnavnet eller IP-adressen, retter seg alltid mot riktig sensor og er den anbefalte formen for skript og installasjoner uten grafisk grensesnitt. I GUI-en bruker du «Oppdater»-knappen i tilkoblingsdialogen og lar systemet gjennomføre en ny skanningssyklus.

## Enhetsinnstillinger og fastvare

Sensoren selv inneholder nettverksinnstillinger – statisk IP kontra DHCP + link-local-adressering, enhetsnavn, automatisk strømming ved oppstart, OTA-passord. Disse innstillingene på enhetssiden er ikke tilgjengelige som kommandoer i den leverte CLI; de administreres via Chloros-brukergrensesnittet der de vises, eller med MAPIR-støtte.

**Firmwareoppdateringer er integrert i brukergrensesnittet.**Når en tilkoblet DAQ-E kjører en firmwareversjon som er eldre enn den som følger med Chloros-versjonen din, vises det en gul**Oppdatering tilgjengelig**-pille i sensorraden, og innstillingsvinduet med tannhjulikonet tilbyr en «Oppdater til<version>

»-knapp. Oppdateringen overføres via nettverket på omtrent 30 sekunder; sensoren starter på nytt og kobler seg til igjen automatisk, og en avbrutt overføring lar den gjeldende firmware være intakt.

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## PTP-tidssynkronisering

DAQ-E-firmware v1.2.0+ deltar i IEEE 1588 PTPv2 som en vanlig (kun slave) klokke. **Backenden til Chloros-verten er PTP-grandmasteren** — hver DAQ-E og hvert LATTICE-kamera på LAN-nettverket fungerer som slaver til den i domene 0, og holder alle enheters tidsstempler innenfor en toleranse på ca. 1 ms. Det er denne delte klokken som gjør at DAQ-målingene kan tidsstemples i samsvar med kameraets eksponeringer (se [Opptak og .daq-formatet](recording.md)).

Kontroller synkroniseringen fra CLI:

| Kommando | Viser |
| --- | --- |
| `chloros-cli time-sync status` | Verts-grandmaster-tilstand, BMCA-prioriteter, klokkeidentitet |
| `chloros-cli time-sync peers` | Alle registrerte slaver (DAQ-E-sensorer + LATTICE-kameraer) |
| `chloros-cli time-sync cameras` | PTP-tilstand per kamera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Start grandmaster-prosessen på nytt |

I brukergrensesnittet viser modalvinduet for DAQ-E-innstillinger en live **PTP Sync**-rad med sensorens gjeldende PTP-tilstand.

Detaljer for brukere som krever streng synkronisering:

* Hvert strømmet datagram har et flaggfelt; **bit 2 er satt på rammer der tidsstemplet er PTP-synkronisert**. Rørledninger som krever streng synkronisering mellom kamera og DAQ, bør basere seg på dette bettet.
* Før en synkronisert opptak, må du bekrefte at sensoren vises i `chloros-cli time-sync peers`. (MAPIRs interne verktøy for direkte maskinvarehåndtering kan også styre opptaket basert på PTP-lås med et `--wait-ptp`-flagg som venter i opptil 15 sekunder på at sensoren skal nå SLAVE-tilstand; denne funksjonaliteten er ikke en del av den leverte CLI.)
* Mens PTP er aktivt i slave-modus, avviser sensoren manuelle klokkesignaler («PTP leverer klokke»). Dette er tilsiktet — stol på PTP.

## Merknader om Linux

* **PTP trenger `libcap2-bin` ved installasjon.** `.deb` postinst gir `cap_net_bind_service=+ep` tilgang til `/usr/lib/chloros/chloros-backend`, slik at den kan binde PTP-portene 319/320 uten root-rettigheter. Hvis `libcap2-bin` mangler, hoppes dette trinnet over, og PTP vil ikke kunne startes. Løsning:

  ```bash
  sudo apt install libcap2-bin
  sudo apt reinstall chloros
  ```

* **Headless Jetson / Raspberry Pi:** Ved første installasjon genereres systemd-enheten `chloros-backend.service`, men den er ikke aktivert. For PTP (og DAQ-tilgjengelighet) som alltid er aktivt uten grafisk brukergrensesnitt:

  ```bash
  sudo systemctl enable --now chloros-backend.service
  ```

  Uten denne kjører PTP bare så lenge Chloros-GUI-en er åpen.

## Feilsøking: «Ingen DAQ-E-enheter funnet»

| Kontroll | Detalj |
| --- | --- |
| Strøm | Ingen LED på sensoren — sjekk PoE- og link-indikatorene på bryteren/injektorporten; vent noen sekunder etter oppstart |
| Broadcast-domene | Vert og sensor på samme L2-segment; mDNS ruter ikke |
| Windows-brannmur | Godta Defender-spørsmålet ved første oppstart (UDP 5002, 5353, 319/320) |
| Vert med flere nettverkskort | Første oppdagelse etter oppstart kan overse sensoren — koble til med `--eth-host <ip-or-hostname>` |
| GUI-gjenkanning | Oppdagelsen kjører bare mens tilkoblingsdialogen er åpen; bruk Oppdater |</version>
