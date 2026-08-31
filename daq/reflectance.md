# Arbeidsflyter for reflektans

En DAQ-lyssensor omregner radiometriske bilder til reflektans. Det finnes to forskjellige arbeidsflyter:

1. **Enkelsensor** — én DAQ måler nedadgående strålingsintensitet mens et kamera tar bilder, og Chloros deler kameraets strålingsintensitet med denne referansen.
2. **To sensorer** — to DAQ-sensorer, hvor den ene overvåker himmelen og den andre et objekt, produserer en spektral reflektanskurve i sanntid uten bruk av kamera.

## Én sensor + kamera (nedadgående referanse)

DAQ-enheten fungerer som den nedadgående lyssensoren (DLS): kameraet måler oppadgående stråling **L**(W/m²/sr/nm), DAQ-enheten måler nedadgående irradians**E** (W/m²/nm), og Chloros beregner reflektansen per bånd som:

> ρ = π · L / E

DAQ-avlesningen er alltid **tidsstemplet i samsvar med eksponeringen** — dette er grunnen til at DAQ-en og kameraene deler en PTP-synkronisert klokke (se [DAQ-E-nettverk og tidssynkronisering](ethernet-ptp.md)). Ta på deg Sunshine-kosinushetten for utendørsarbeid og oppgi den riktig; oppgivelsen av hetten skalerer E direkte (se [Cap Profiles &amp; Calibrated Range](caps-and-range.md)). For kvantitativt arbeid må du huske instrumentets egenskaper: kvantitativ irradians beregnes ut fra et gjennomsnitt av målinger over minst 15 sekunder.

### Live-opptak

Knytt DAQ-en til et kamera i fanen «Kameraer»: Innstillingspanelet for hvert kamera har en **Lyssensor**-rullegardinliste som viser alle tilkoblede DAQ-er (DAQ-U/M/E) fra fanen «Lyssensorer»; for et synkronisert array vil et valg av lyssensor for hele arrayet gjelde for hvert medlem (enkeltkameraer kan fortsatt overstyre dette). Når tilkoblingen er opprettet, mates sensorens spektra inn i kameraets DLS-spor, og eksporterte reflektansverdier deles med den tilordnede måleverdien.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->

To egenskaper det er verdt å kjenne til:

* **Ingen DAQ tilknyttet → reflektans avvises, ikke simuleres.** Chloros avviser reflektansproduktet og registrerer årsaken til utelatelsen, i stedet for å returnere et lavere produkt uten varsel.
* **Måleverdien som ble brukt, bevares.** For hvert reflektansbilde skrives den faktisk anvendte DAQ-målingen som en `.daq`-sidecar ved siden av bildet, slik at opptaket kan behandles på nytt senere ([Opptak og .daq-formatet](recording.md)).

### Behandling av innspilt bildemateriale

For behandling etter flyvningen skal du registrere en `.daq` under økten og oppbevare den sammen med bildematerialet – behandlingsrørledningen løser automatisk den tidsstempelmatchede nedadgående strålingen, og henter eventuelle manglende fabrikkalibreringer fra skyen til MAPIR ved første bruk. GUI-opptak legges automatisk til det åpne prosjektet når de avsluttes.

Refleksjonsreferansen kan velges ved behandlingstidspunktet — `--reflectance-source` på `chloros-cli process`, eller innstillingen for refleksjonskilde i GUI-ens prosjektinnstillinger:

| Verdi | Oppførsel |
| --- | --- |
| `auto` (standard) | Et QA-godkjent kalibreringsmål innenfor rammen er den absolutte referansen; DAQ-nedstråling (ρ = π·L/E) er reserveverdien |
| `daq` | DAQ-autoritativ |
| `target` | Strengt mål innenfor rammen; ingen DAQ-erstatning |

Se [Kalibreringsmål](../calibration-targets.md) for arbeidsflyter for mål og [LATTICE-kapitlet](../lattice/README.md) samt [CLI-referansen](../reference/cli-reference.md) for hele behandlingsrørledningen. Når du leser inn eksporterte reflektanspiksler, må du bruke den angitte skalaen (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3: 65535) – se [Utdataformater](../output-image-formats.md).

### Bånd utenfor DAQ-enhetens kalibrerte område

DAQ-enhetens radiometrisk kalibrerte område er ~374–974 nm. Chloros avviser DAQ-basert reflektans for ethvert kameraband hvor mindre enn halvparten av dets spektrale vekt ligger innenfor dette området, og rapporterer utelatelsesårsaken `dls-uncalibrated-band-<nm>`. Blant de leverte SKU-ene påvirker dette kun F988: Reflektansen til F988 kalibreres ved hjelp av et reflektanspanel i scenen: båndet ligger utenfor det kalibrerte området til DAQ-lyssensoren, så Chloros bruker den siste panelopptakelsen og beholder den mellom panelobservasjonene. Hvis et F988-kamera kun kjører i DAQ-modus, avviser Chloros DAQ-basert reflektans for det aktuelle båndet med hoppårsaken `dls-uncalibrated-band-988` – panelarbeidsflyten er den støttede fremgangsmåten.

## Dobbel sensor (omgivelseslys + objekt)

To DAQ-sensorer — hvilket som helst par, uavhengig av transport — gir et live refleksjonsspektrum uten kamera: én sensor vender mot himmelen (**Omgivelseslyskilde**), én vender mot motivet (**Objektskanner**), og Chloros beregner per bølgelengde:

> R(λ) = objekt(λ) / omgivelser(λ)

(null der omgivelser ≤ 0).

### I brukergrensesnittet

Når begge sensorene er tilkoblet i fanen «Lyssensorer», åpner du overlegget for å legge til sensor (knappen «+» på en diagramflis i rutenettvisningen) og velger **Kombiner omgivelseslys + objekt**. Velg de to sensorene i rullegardinmenyene for omgivelseslyskilde og objekt-skanner, og klikk på Opprett. Gruppen vises som et eget diagram og som en rad i sidepanelet med et grønt**REF**-merke.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->

Under reflektansdiagrammet (listevisning) beregner en sanntids **vegetasjonsindekstabell** indekser fra kurven ved hjelp av båndsentre på blå 450 / grønn 550 / rød 670 / NIR 800 nm. Forholdsbaserte indekser som utligner den absolutte skalaen (NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR) vises alltid; indekser som krever absolutt reflektans (EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) vises bare når begge sensorene er strømkalibrerte modeller.

### Tilsynelatende vs. relativ — merkingsregelen

Chloros merker utdataene fra dobbeltsensoren etter hva sensorparet faktisk kan oppgi:

| Sensorpar | Merking |
| --- | --- |
| Begge sensorene kalibrert — fabrikkpakke lastet inn | **Tilsynelatende refleksjonsgrad** |
| En av sensorene er ikke kalibrert | **Relativ reflektans** |

Alle tre modellene er radiometriske: når en sensors fabrikkalibreringspakke er lastet inn, er spektrene absolutte W/m²/nm, så et par kalibrerte sensorer gir forholdet til en absolutt tilsynelatende reflektans — transporten bestemmer ikke dette. En sensor som fortsatt sender rå telleverdier (ingen pakke tilgjengelig), nedgraderer resultatet til en relativ kurve (spektralformen er fortsatt gyldig). Begge sensorene bør ha korrekt angitte takverdier ([Takprofiler og kalibrert område](caps-and-range.md)).

### Fra Python

Det finnes ingen dedikert funksjon for to sensorer i den samlede SDK-overflaten: åpne to økter med `chloros_sdk.connect_daq_sensor()` og sammenlign deres `latest()`-spektra selv, ved å bruke samme merkingskonvensjon. (Det finnes også et opptaksverktøy for to sensorer på MAPIRs interne, direkte maskinvarebaserte grensesnitt, oppført i [CLI-referansen](../reference/cli-reference.md) for fullstendighetens skyld – det er ikke en del av den leverte CLI; GUI-arbeidsflyten ovenfor er den støttede live-banen.)
