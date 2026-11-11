# Kapselprofiler og kalibrert måleområde

> Selve dekslene — hvilke deksler som følger med hvilke sensorer, hvordan de monteres og deres optiske egenskaper — er dokumentert i **[DAQ-brukerhåndboken](https://mapir.gitbook.io/daq)**. Denne siden omhandler *angivelse* av det monterte dekselet til Chloros
, noe som er avgjørende for at korreksjonen skal bli riktig.

Den radiometriske kalibreringen fra fabrikken for hver DAQ-lyssensor beskriver den *nakne* sensoren. Det fysiske dekselet som er montert over diffusoren endrer hvilket lys sensoren fanger opp, så Chloros
bruker en **dekselkorreksjonsprofil** målt på fabrikken i tillegg til kalibreringspakken. Å oppgi riktig hette er en del av å få kalibrerte data – denne siden beskriver hvilke hetter som finnes for hver modell, hvordan du oppgir dem, og hva sensorens kalibrerte spektralområde faktisk er.

## Tilgjengelige deksler etter modell

| Dekselprofil (`cap_id`) | Fysisk deksel | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Deksel for kosinuskorreksjon ved sollys (**standard på alle modeller**) | Ja | Ja | Ja |
| `fov_15` / `fov_45` / `fov_90` | FOV-begrensende kjegler (15° / 45° / 90°) | Ja | — | Ja |
| `fov_30` / `fov_60` | Kjegler som begrenser synsfeltet (30° / 60°) | Ja | — | — |
| `none` | Ingen hette montert | — | — | Ja |

Modellspesifikke merknader:

* **DAQ-M har én hetteprofil: `sunshine_cosine`.** «Bare-plus-Sunshine-hette» er produktdefinisjonen, og en «bare» DAQ-M trenger ingen geometriprofil.
* **En «bare» DAQ-U er helt uten profil** — den trenger ingen geometriprofil i det hele tatt, og det er derfor det ikke finnes noen `none`-profil for den.
* **`none` på en DAQ-E er IKKE en «no-op».** DAQ-E-modellens innfelte, glassdekkede diffusor har en egen, reell geometrikorreksjon, så «uten hette» er i seg selv en målt profil på denne modellen.
* En **DAQ-E uten deksel kan ikke måle direkte sollys uansett høyde** — Sunshine-dekselet er standardkonfigurasjonen. Ikke planlegg utendørsarbeid med en DAQ-E uten deksel.

I GUI-ens innstillinger per sensor (tannhjulikonet i fanen «Lyssensorer») tilbyr rullegardinmenyen **Cap** også «None (bare sensor)» på DAQ-U og DAQ-M — på disse to modellene betyr «bare» ganske enkelt at det ikke brukes noen dekselkorreksjon, i henhold til merknadene ovenfor. Velg dette alternativet kun når hetten er fysisk fjernet.

## Angi hetten — og hvorfor det er viktig

**Den angitte koden `cap_id` må samsvare med hetten som fysisk sitter på sensoren.** Verken sensoren eller programvaren kan oppdage det monterte dekselet. Angivelsen styrer to ting:

1. Den **live-korreksjonen** som brukes på hvert spektrum.
2. **Dekselstempelet som skrives inn i hver `.daq`-opptak**, som den etterfølgende refleksjonsbehandlingen stoler på.

Sunshine-hetten demper lyset med omtrent **12× i henhold til designet**, så opptak med feil angitt hette gir feil skalering av spektrene med omtrent den faktoren. Angi endringer i hetten umiddelbart.

### Innstilling av hetten

GUI: Fanen «Lyssensorer» → tannhjulikonet på sensorrekken → **Hetten**-rullegardinmenyen. Standardverdien for alle modeller er `sunshine_cosine`, en antagelse (DAQ-U, DAQ-M og DAQ-E leveres uten kappe, og en opptak angir at kappen ble antatt inntil du velger en), og valget beholdes i prosjektet.



CLI<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->
(backend må være i gang):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

CLI
godtar syntaktisk hele `cap_id`-listen (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); hver profil valideres mot sensorens modell ved tilkobling, så en utilgjengelig cap-ID (for eksempel en E-only-ID på en DAQ-U) gir en tydelig feilmelding i stedet for feilkorrigering. Backend-standardverdien når ingenting angis, er `sunshine_cosine`.

Python
SDK
: Gi cap-en et navn ved tilkobling med `connect_daq_sensor(cap_id="sunshine_cosine")`, eller bytt den under kjøring med `DAQSensorSession.set_cap("fov_45")`; se [SDK
-referansen](../reference/sdk-reference.md#daq-sensor-sessions).

Avansert: profilene leveres som en del av Chloros
-installasjonen under `daq/cap_profiles/<u|m|e>/<cap_id>.json` og kan overstyres per bruker under `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Uavhengig av begrensninger får sensorer som aldri har blitt rekalibrert automatisk en liten, flåtebasert justering av mørkeforskyvningen — uten at brukeren trenger å gjøre noe.

## DAQ-E-S: fast diffusor og vinkel

**DAQ-E-S** er en DAQ-E med to forskjeller som er viktige for Chloros
. Alt annet (oppdagelse, tilkobling, PTP, opptak, enhetsinnstillinger) fungerer nøyaktig som for en DAQ-E.

* **Diffusoren er fastmontert, så den krever ingen kap-korreksjon.** Diffusoren var montert på enheten da fabrikkalibreringen ble målt, slik at kalibreringen allerede viser den faktiske nedadgående strålingsintensiteten. Chloros
bruker ingen cap-profil på den, uansett hva som blir bedt om: en cap oppkalt etter den blir ignorert med en melding, opptakene viser at capen stammer fra modellen, og advarselen om antatt cap utløses aldri for den. I Prosjektinnstillinger → DAQ-lyssensor viser raden for den ingen cap, og den prosjektomfattende cap-overstyringen påvirker den ikke. Ikke bruk en cap på dataene manuelt: Sunshine-korreksjonen på toppen vil da vise et verd som er ca. 11 ganger for høyt.
* **Den har et 3-akset akselerometer.** Diffusorens kosinusrespons er kun gyldig når den står vannrett, så en skråstilt sensor rapporterer feil innstråling uten andre symptomer. Hver DAQ-E-S-måling inneholder derfor informasjon om tilt, roll og pitch, og en `.daq`-fil som er registrert fra en slik sensor, lagrer disse opplysningene ved siden av hvert spektrum (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, …), slik at du kan skille mellom en sky og et skjevt stativ.

For hvert holdningsfelt og hvordan en feil i holdningsstrømmen rapporteres, se [DAQ-E-S i CLI
-referansen](../reference/cli-reference.md#daq-e-s--fixed-diffuser-and-tilt-sensor).

## Ytelse med solskjerm (utendørskonfigurasjonen)

Tall du kan basere prosedyrer på:

| Egenskap | Verdi |
| --- | --- |
| Synsfelt | 180° halvkuleformet |
| Kosinusresponsfeil | ≤ ±4 % opp til 60° innfallsvinkel; ≤ ±4,5 % opp til 70° |
| Grense for lav solhøyde | Anbefales ikke under ~15° solhøyde |
| Demping | ~12× (ifølge spesifikasjon) |
| Repeterbarhet ved montering av hetten | ≈ 1,5 % |
| Kvantitativ irradians | Gjennomsnitt av **≥ 15 s** med målinger (instrumentets egenskap, ikke en feil) |

For alle kvantitative irradiansverdier — inkludert refleksjonsreferanser — bruk et gjennomsnitt av minst 15 sekunder med målinger i stedet for et enkelt bilde.

## Kalibrert spektralområde

| Egenskap | Verdi |
| --- | --- |
| Spektral oppsamling | 340–1010 nm i trinn på 5 nm (135 punkter) |
| Radiometrisk kalibrert område | **~374–974 nm** (pålagt i programvaren) |

Sensoren rapporterer hele rutenettet 340–1010 nm, men den NIST-sporbare radiometriske forsterkningen strekker seg over ~374–974 nm. Chloros
**avviser inndelingen etter absolutt reflektans** for ethvert kamerabånd som har mindre enn halvparten av sin spektrale vekt innenfor dette området, og rapporterer hoppårsaken `dls-uncalibrated-band-<nm>` i stedet for å produsere et ukalibrert produkt. Blant de kamera-SKU-ene som er tilgjengelige, er det kun F988-filteret som faller utenfor dette området; det bruker i stedet arbeidsflyten for refleksjonspaneler — se [Arbeidsflyter for refleksjon](reflectance.md).

For sensormodeller, transporttyper og sensor-ID-er, se [DAQ-oversikten](README.md). For hvordan cap-stemplet brukes opp under behandlingen, se [Opptak og .daq-formatet](recording.md).
