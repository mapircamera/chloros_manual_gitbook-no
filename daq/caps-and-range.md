# Kapselprofiler og kalibrert måleområde

> Selve kapslene – hvilke kapsler som følger med hvilke sensorer, hvordan de monteres og deres optiske egenskaper – er dokumentert i **[DAQ-brukerhåndboken](https://mapir.gitbook.io/daq)**. Denne siden omhandler *angivelse* av det monterte dekselet til Chloros, noe som er avgjørende for at korreksjonen skal bli riktig.

Den radiometriske kalibreringen fra fabrikken for hver DAQ-lyssensor beskriver den *nakne* sensoren. Det fysiske dekselet som er montert over diffusoren endrer hvilket lys sensoren fanger opp, så Chloros bruker en **dekselkorreksjonsprofil** målt på fabrikken i tillegg til kalibreringspakken. Å angi riktig hette er en del av å oppnå kalibrerte data – denne siden beskriver hvilke hetter som finnes for hver modell, hvordan man angir dem, og hva sensorens kalibrerte spektralområde faktisk er.

## Tilgjengelige deksler etter modell

| Dekselprofil (`cap_id`) | Fysisk deksel | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Deksel for kosinuskorreksjon ved sollys (**standard på alle modeller**) | Ja | Ja | Ja |
| `fov_15` / `fov_45` / `fov_90` | FOV-begrensende kjegler (15° / 45° / 90°) | Ja | — | Ja |
| `fov_30` / `fov_60` | FOV-begrensende kjegler (30° / 60°) | Ja | — | — |
| `none` | Ingen hette montert | — | — | Ja |

Modellspesifikke merknader:

* **DAQ-M har én hetteprofil: `sunshine_cosine`.** «Bare-plus-Sunshine-hette» er produktdefinisjonen, og en «bare» DAQ-M trenger ingen geometriprofil.
* **En «bare» DAQ-U er helt uten deksel** — den trenger ingen geometriprofil i det hele tatt, og det er derfor det ikke finnes noen `none`-profil for den.
* **`none` på en DAQ-E er IKKE en «no-op».** DAQ-E-modellens innfelte, glassdekkede diffusor har en egen, reell geometrisk korreksjon, så «uten hette» er i seg selv en målt profil på denne modellen.
* En **DAQ-E uten deksel kan ikke måle direkte sollys uansett høyde** — Sunshine-dekselet er standardkonfigurasjonen. Ikke planlegg utendørsarbeid med en DAQ-E uten deksel.

I GUI-ens innstillinger per sensor (tannhjulikonet i fanen «Light Sensors») tilbyr rullegardinmenyen **Cap** også «None (bare sensor)» på DAQ-U og DAQ-M — på disse to modellene betyr «bare» ganske enkelt at det ikke brukes noen dekselkorreksjon, i henhold til merknadene ovenfor. Velg dette alternativet kun når dekselet er fysisk fjernet.

## Angi dekselet — og hvorfor det er viktig

**Det angitte `cap_id` må samsvare med dekselet som fysisk sitter på sensoren.** Verken sensoren eller programvaren kan oppdage det monterte dekselet. Deklarasjonen styrer to ting:

1. **Live-korreksjonen** som brukes på hvert spektrum.
2. **Dekselstemplet som skrives inn i hver `.daq`-opptak**, som den påfølgende refleksjonsbehandlingen stoler på.

Sunshine-hetten demper lyset med omtrent **12 ganger i henhold til designet**, så opptak med feil angitt hette gir feil skalering av spektrene med omtrent den faktoren. Angi endringer i hetten umiddelbart.

### Innstilling av hetten

GUI: Fanen «Lyssensorer» → tannhjulikonet på sensorrekken → **Hetten**-rullegardinmenyen. Standardinnstillingen for alle modeller er `sunshine_cosine` (alle DAQ-sensorer leveres med kosinuskorrektor installert), og valget beholdes i prosjektet.

<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->

CLI (backend må være i gang):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

CLI aksepterer syntaktisk hele `cap_id`-listen (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); hver profil valideres mot sensorens modell ved tilkobling, så en utilgjengelig kap-ID (for eksempel en E-only-ID på en DAQ-U) mislykkes med en tydelig feilmelding i stedet for å korrigere feil. Standardverdien for backend når ingenting sendes inn, er `sunshine_cosine`.

Python SDK Merk: `cap_id` er **ikke** en SDK-knott — `connect_daq_sensor()` / `DAQSensorSession` har ingen cap-parameter. Velg cap via CLI-kommandoene ovenfor eller nedtrekksmenyen i GUI; se [SDK-referansen](../reference/sdk-reference.md).

Avansert: Profiler leveres som en del av Chloros-installasjonen i `daq/cap_profiles/<u|m|e>/<cap_id>.json` og kan overstyres per bruker i `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Uavhengig av begrensningene får sensorer som aldri har blitt rekalibrert automatisk en liten finjustering av mørkeforskyvningen basert på flåtedata – uten at brukeren trenger å gjøre noe.

## Ytelse for sollysbegrensning (utendørskonfigurasjonen)

Tall du kan basere prosedyrer på:

| Egenskap | Verdi |
| --- | --- |
| Synsfelt | 180° halvkuleformet |
| Kosinusresponsfeil | ≤ ±4 % ut til 60° innfallsvinkel; ≤ ±4,5 % ut til 70° |
| Grense for lav solhøyde | Anbefales ikke under ~15° solhøyde |
| Demping | ~12× (etter design) |
| Repeterbarhet ved montering av solskjerm | ≈ 1,5 % |
| Kvantitativ innstråling | Gjennomsnitt av **≥ 15 s** med målinger (instrumentets egenskap, ikke en feil) |

For alle kvantitative innstrålingstall — inkludert refleksjonsreferanser — bruk et gjennomsnitt av minst 15 sekunder med målinger i stedet for et enkelt bilde.

## Kalibrert spektralområde

| Egenskap | Verdi |
| --- | --- |
| Spektral oppsamling | 340–1010 nm i trinn på 5 nm (135 punkter) |
| Radiometrisk kalibrert område | **~374–974 nm** (pålagt i programvaren) |

Sensoren rapporterer hele rutenettet 340–1010 nm, men den NIST-sporbare radiometriske forsterkningen strekker seg over ~374–974 nm. Chloros **avviser delingen etter absolutt refleksjonsgrad** for ethvert kamerabånd hvor mindre enn halvparten av dets spektrale vekt ligger innenfor dette området, og rapporterer hoppårsaken `dls-uncalibrated-band-<nm>` i stedet for å produsere et ukalibrert produkt. Blant de kamera-SKU-ene som er tilgjengelige, er det kun F988-filteret som faller utenfor dette området; det bruker i stedet arbeidsflyten for reflektanspaneler — se [Arbeidsflyter for reflektans](reflectance.md).

For sensormodeller, transporttyper og sensor-ID-er, se [DAQ-oversikten](README.md). For hvordan cap-stemplet brukes under behandlingen, se [Opptak og .daq-formatet](recording.md).
