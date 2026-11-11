# Tilkobling av kameraer

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>Fanen «Kameraer» før noe er koblet til</p></figcaption></figure>

Chloros
oppdager LATTICE-kameraer på nettverket automatisk — fra fanen «Kameraer» i brukergrensesnittet, fra `chloros-cli lattice` eller fra Python
-SDK
-et. Kameraets modellstreng styrer alt videre: Chloros
henter sensorprofilen, båndoppsettet og fabrikkalibreringen fra kameraets `DeviceUserID` + `DeviceSerialNumber`, så **det er ingenting å konfigurere per kamera**.

Før du kobler til, må du sørge for at vertsnettverket er konfigurert — link-local-adressering, jumbo-rammer og, for matriser, innstillingene for mottaksbufferen på nettverkskortet. Dette er oppsett på maskinvaresiden og finnes i LATTICE-håndboken: [**Nettverksoppsett**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Koble til fra GUI

Åpne **Kameraer**-fanen i Chloros
-sidepanelet (maskinvarefanene vises når backenden er ferdig med å starte), eller bruk hovedmenyen → **Koble til kamera**. Begge åpner**Koble til kamera(er)**-dialogboksen.

### Dialogboksen **Koble til kamera(er)**Dialogboksen skanner nettverket så snart den åpnes («Skanner nettverk...»), og viser en liste over alle kameraene den finner. Hver rad viser kameraets**modell**(f.eks. `LATT-M3M-L41-F550`),**serienummer**og**IP-adresse**.

* **Klikk på en rad for å velge den**(grønn utheving). Du kan velge**flere kameraer** og koble dem til på én gang – Chloros
kobler dem til i rekkefølge.
* Rader med merket **«Koblet til»** er allerede koblet til og kan ikke velges på nytt.
* Rader med merket **«I gruppe»** tilhører en kameragruppe som for øyeblikket er koblet til. Koble fra gruppen først for å bruke det aktuelle kameraet alene.
* **Koble til** — kobler til det eller de valgte kameraene; knappen viser et antall, f.eks. «Koble til (3)», når mer enn ett er valgt.
* **Skann på nytt** — kjører søket på nytt.
* **Lukk** — lukker dialogboksen.
* Hvis søket avsluttes uten resultater, viser dialogboksen **«Ingen kameraer funnet på nettverket»** — se [Feilsøking](connecting.md#troubleshooting) nedenfor.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>Dialogboksen «Koble til kamera(er)» — vist her uten kameraer på nettverket</p></figcaption></figure>

### Første tilkobling: nedlasting av kalibreringspakke

Den **første gangen** et gitt kamera kobles til en maskin, henter Chloros
kameraets fabrikkalibreringspakke (\~3,8 MB) fra selve kameraet via GigE. Mens dette pågår, viser dialogboksen et grønt **«Laster ned kalibreringsdata fra kamera»**-panel med en fremdriftslinje for hvert serienummer — regn med omtrent**70 sekunder** per kamera. Med mer enn seks kameraer ruller fremdriftslinjene inne i panelet. Pakken lagres i hurtigbufferen på verten, slik at senere tilkoblinger av det samme kameraet hopper helt over nedlastingen (og viser aldri panelet).

### Analyser system

Dialogboksens **Analyser system**-knapp undersøker verten og nettverket (etiketten viser «Analyserer...» mens den kjører) og genererer en diagnostikkrapport:

* **Vert** — CPU-kjerner og RAM; GPU-navn og minne, eller «GPU: Ingen oppdaget».
* **Nettverksgrensesnitt** — navnet på hvert nettverkskort, koblingshastighet, MTU (med en «jumbo»-merking der dette er aktivt), opp-/ned-status, og om det sitter på en USB-buss.
* **Kameraer**— serienummer, modell, IP-adresse og**hvilket nettverkskort hvert kamera er koblet til**.
* **Ytelse** — gjeldende vs. ideell bildefrekvens (fps) per kamera for det aktuelle pikselformatet, med en grønn linje som sier «Potensial: N× forbedring mulig» når den ideelle verdien overstiger den gjeldende.
* **Advarsler og nummererte anbefalinger** — eller «Systemet ser bra ut for det nåværende antallet kameraer» når det ikke er noe å rette på.

Kjør verktøyet når oppdagelse eller strømming oppfører seg uventet — det identifiserer de fleste problemer på nettverkskort-siden (feil MTU, kamera på feil grensesnitt, begrensninger på USB-adapter) uten at du trenger å forlate dialogboksen.

### Koble til et array

For å koble til to eller flere kameraer som et **synkronisert array**, bruk i stedet veiviseren for array-tilkobling (**Koble til kamera-array**): den leder deg gjennom valg av master/slave (forhåndsutfylt av en synkroniseringskabel-sonde — ethvert kablet kamera kan være master, og RGB
-kameraet `-FRGB` er forhåndsvalgt når det er kablet), valg av visningsmodus (separate vs. kombinerte ruter) og en scene for oppstillingsinnstillinger med en live-projeksjon av oppnåelig fps og kabelbåndbredde før du bekrefter. Veiviseren og oppsettprosessene for oppsettet er beskrevet i avsnittet om flerkameraoppsett i denne håndboken; CLI
-tilsvaret er «LATTICE Camera First-Connect Workflow» i [CLI
-referansen](../reference/cli-reference.md).

## Tilkobling fra CLI
og SDK
Tilgang til

CLI
og SDK
krever et betalt Chloros
+-abonnement og at du er logget inn; dette håndheves på serversiden (`401 AUTH_REQUIRED` når du ikke er logget inn, `403 PLAN_UPGRADE_REQUIRED` på gratisabonnementet).

```bash
# List cameras on the network (vendor, model, serial, IP, MAC)
chloros-cli lattice info

# Single-camera smoke test: capture one frame (saves every applicable export type)
chloros-cli lattice capture -o output/

# Connect a synchronized array — same smart-prep flow as the GUI
chloros-cli lattice array-connect --serials 213800234,214000533
```

```python
import chloros_sdk

# Persistent live-camera session through the backend
with chloros_sdk.connect_camera("213800234") as cam:
    ...

# Array session (smart-prep: network probe, tier auto-pick, PTP, AE seeding, trigger config)
with chloros_sdk.connect_array(["213800234", "214000533"]) as array:
    ...
```

Fullstendige signaturer, alternativer og innsamlingsarbeidsflyter: [CLI
-referanse](../reference/cli-reference.md) § `chloros-cli lattice`, [SDK
-referanse](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Hvordan kalibrering håndteres ved tilkobling

Hvert LATTICE-kamera har sin fabrikkalibreringspakke **lagret på selve kameraet**, og Chloros
sjekker også MAPIR
’ sky når kameraet kobler seg til:

| Situasjon   | Hva Chloros
bruker                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Online**| Den**nyeste kalibreringen som er publisert for det serienummeret** — kopien i skyen har forrang fremfor kopien på kameraet. Et kamera som har blitt rekalibrert eller oppdatert av MAPIR
, oppdaterer seg derfor automatisk; ingen brukerhandling er nødvendig. |
| **Offline**|**Pakken på kameraet**, slik den er. Fullstendig offline arbeidsflyter fortsetter å fungere; de henter ganske enkelt ikke inn nyere kalibreringer før kameraet er online én gang (eller har fått fabrikkinnstillingene gjenopprettet).                                                  |

Ved opptakstidspunktet blir koeffisientene som faktisk brukes **låst fast i hvert bildes XMP-metadata**. En senere kalibreringsoppdatering endrer aldri bilder du allerede har tatt uten at du merker det — når du behandler et gammelt opptak på nytt, brukes koeffisientene som er lagret i XMP-metadataene, ikke de nyeste som finnes i dag.

## Feilsøking

* **«Ingen kameraer funnet på nettverket»**— kontroller den lokale nettverkskonfigurasjonen i [Nettverksoppsett](https://mapir.gitbook.io/lattice-camera/setup/network-setup): statisk nettverkskort for verten `169.254.x.x/16`, kameraer på samme nettverkslink, ingen DHCP/gateway forventet. Bruk deretter**Analyser system**i tilkoblingsdialogen for å sjekke hvilket nettverkskort hvert kamera er (eller ikke er) synlig på.**Skann på nytt** etter endringer i kabling eller nettverkskort.
* **Et anlegg som tidligere fungerte, nekter å koble seg til** (array-panelet viser feilmeldingene `FRAMES WILL DROP` / `Reduce ROI to enable`) — en oppdatering av nettverkskortdriveren har uten varsel tilbakestilt innstillingene for mottaksringen. Angi dem på nytt, eller kjør `chloros-cli lattice network --fix` fra en terminal med administratorrettigheter; se [Nettverksoppsett](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **Et kamera viser «In Array»** — det tilhører en tilkoblet array-økt. Koble fra arrayet for å bruke kameraet frittstående.
