# Arbeidsflyter for reflektans

En DAQ-lyssensor omregner radiometriske bilder til reflektans. Det finnes to forskjellige arbeidsflyter:

1. **Enkelsensor** — én DAQ måler nedadgående strålingsintensitet mens et kamera tar bilder, og Chloros e deler kameraets strålingsintensitet med denne referanseverdien.
2. **To sensorer** — to DAQ-sensorer, hvor den ene overvåker himmelen og den andre overvåker et objekt, produserer en spektral reflektanskurve i sanntid uten bruk av kamera.

## Én sensor + kamera (nedadgående referanse)

DAQ-en fungerer som sensoren for nedadgående lys (DLS): kameraet måler oppadgående strålingsintensitet **L**(W/m²/sr/nm), DAQ-enheten måler nedadgående strålingsintensitet**E** (W/m²/nm), og Chloros beregner reflektansen per bånd som:

> ρ = π · L / E

DAQ-målingen er alltid **tidsstemplet i samsvar med eksponeringen** — dette er grunnen til at DAQ-en og kameraene deler en PTP-synkronisert klokke (se [DAQ-E-nettverk og tidssynkronisering](ethernet-ptp.md)). Monter Sunshine-kosinushetten for utendørsarbeid og deklarer den riktig; hetteerklæringen skalerer E direkte (se [Cap Profiles & Calibrated Range](caps-and-range.md)). For kvantitativt arbeid må du huske instrumentets egenskaper: kvantitativ irradians beregnes ut fra et gjennomsnitt av målinger over minst 15 sekunder.

### Live-opptak

Koble DAQ-en til et kamera i fanen «Kameraer»: Innstillingspanelet for hvert kamera har en **Lyssensor**-rullegardinmeny som viser alle tilkoblede DAQ-er (DAQ-U/M/E) fra fanen «Lyssensorer»; for et synkronisert array overføres et valg av lyssensor for hele arrayet til hvert medlem (enkeltkameraer kan fortsatt overstyre dette). Når tilknytningen er opprettet, mates sensorens spektra inn i kameraets DLS-spor, og eksportene av refleksjonsverdien deles med den tilpassede måleverdien. I et array bruker et medlem med egen sensor sin egen måling – for sine bånd i live-kompositten, sine eksporterte data og sin automatiske eksponering – mens de andre medlemmene bruker arrayets.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->


To oppførsler det er verdt å kjenne til:

* **Ingen DAQ-tilknytning → reflektans avvises, ikke forfalskes.** *Chloros
* avviser reflektansproduktet og registrerer årsaken til utelatelsen, i stedet for å stille levere tilbake et dårligere produkt.
* **Måleverdien som ble brukt, bevares.** For hvert reflektansbilde skrives den faktisk anvendte DAQ-målingen som en `.daq`-fil i opptakets `daq/`-mappe, slik at opptaket kan behandles på nytt senere ([Opptak og .daq-formatet](recording.md)). En `.daq.cameras.json` ved siden av angir hvilke kameraer den beskriver, og ved ny behandling blir den kun matchet mot bildene fra disse kameraene.

### Behandling av innspilte bilder

For behandling etter flyvningen bør du ta opp en `.daq` under økten og oppbevare den sammen med bildene — behandlingsrørledningen identifiserer automatisk de tidsstemplede nedadrettede strålene og henter eventuelle manglende fabrikkalibreringer fra MAPIRs sky ved første bruk. GUI-opptak legges automatisk til det åpne prosjektet når de avsluttes.

Reflektansreferansen kan velges ved behandlingstidspunktet – `--reflectance-source` på `chloros-cli process`, eller innstillingen for reflektanskilde i GUI-ens prosjektinnstillinger:

| Verdi | Oppførsel |
| --- | --- |
| `auto` (standard) | Et kalibreringsmål innenfor rammen som har bestått kvalitetssikring (QA), er den absolutte referansen; DAQ-nedstråling (ρ = π·L/E) er reserveverdien. Et mål som består kvalitetssikringen med en feil (et panel som har falt ut, eller et bånd som ikke fungerer), blir ikke automatisk foretrukket: kjeden med målet, målet+DAQ og kun DAQ blir prøvd i rekkefølge etter deres målte poengsum på utelatte paneler. |
| `daq` | DAQ-autoritativ |
| `target` | Strengt mål innenfor rammen; ingen DAQ-erstatning |

Se [Kalibreringsmål](../calibration-targets.md) for arbeidsflyter for mål og [LATTICE-kapittelet](../lattice/README.md) samt [CLI-referansen](../reference/cli-reference.md) for den fullstendige behandlingsprosessen. Når du leser inn eksporterte refleksjonspiksler, bruk den stemplede skalaen (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3: 65535) — se [Utdata-bildeformater](../output-image-formats.md).

### Bånd utenfor DAQ-enhetens kalibrerte område

DAQ-enhetens radiometrisk kalibrerte område er ~374–974 nm. Chloros-programmet avviser DAQ-basert reflektans for ethvert kameraband hvor mindre enn halvparten av dets spektrale vekt ligger innenfor dette området, og rapporterer utelatelsesårsaken `dls-uncalibrated-band-<nm>`. Blant de leverbare SKU-ene påvirker dette kun F988: Reflektansen til F988 kalibreres ved hjelp av et reflektanspanel i scenen: båndet ligger utenfor det kalibrerte området til DAQ-lyssensoren, så Chloros bruker den siste panelopptakelsen og beholder den mellom panelmålingene. Hvis et F988-kamera kun kjører i DAQ-modus, avviser Chloros DAQ-basert reflektans for det båndet med hoppårsaken `dls-uncalibrated-band-988` – panelarbeidsflyten er den støttede metoden.

## Dobbel sensor (omgivelseslys + objekt)

To DAQ-sensorer — hvilket som helst par, uavhengig av transportform — gir et live refleksjonsspektrum uten kamera: én sensor vender mot himmelen (**Omgivelseslyskilde**), én vender mot motivet (**Objektskanner**), og Chloros beregner per bølgelengde:

> R(λ) = objekt(λ) / omgivelser(λ)

(null der omgivelser ≤ 0).

### I brukergrensesnittet

Når begge sensorene er tilkoblet i fanen «Lyssensorer», åpner du overlegget for å legge til sensor (**+**-knappen på en diagramrute i rutenettvisningen) og velger**Kombiner omgivelseslys + objekt**. Velg de to sensorene i rullegardinmenyene for omgivelseslyskilde og objektskanner, og klikk på Opprett. Gruppen vises som et eget diagram og som en rad i sidefeltet med et grønt**REF**-merke.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->


Under refleksjonsdiagrammet (listevisning) beregner en sanntids **vegetasjonsindekstabell** indekser fra kurven ved hjelp av båndsentre på blå 450 / grønn 550 / rød 670 / NIR 800 nm. Forholdsbaserte indekser som utligner absolutt skala (NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR) vises alltid; indekser som krever absolutt reflektans (EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) vises kun når begge sensorene er strømkalibrerte modeller.

### Tilsynelatende vs. relativ — merkingsregelen

Chloros merker utdataene fra dobbeltsensoren etter hva sensorparet faktisk kan oppgi:

| Sensorpar | Merking |
| --- | --- |
| Begge sensorer kalibrert — fabrikkpakke lastet inn | **Tilsynelatende reflektans** |
| En av sensorene er ikke kalibrert | **Relativ reflektans** |

Alle tre modellene er radiometriske: så snart en sensors fabrikkalibreringspakke er lastet inn, er spektrene absolutte W/m²/nm, så et par kalibrerte sensorer gir forholdet til en absolutt tilsynelatende reflektans — transporten avgjør ikke dette. En sensor som fortsatt sender rå tellinger (ingen pakke tilgjengelig) nedgraderer resultatet til en relativ kurve (spektralformen er fortsatt gyldig). Begge sensorene bør ha korrekt deklarerte grenseverdier ([Grenseverdiprofiler og kalibrert område](caps-and-range.md)).

### Fra Python


Det finnes ingen dedikert funksjon for to sensorer i det samlede SDK-grensesnittet: åpne to økter med `chloros_sdk.connect_daq_sensor()` og sammenlign spektrene deres med `latest()` selv, ved å bruke samme merkingskonvensjon. (Det finnes også et opptaksverktøy for to sensorer på MAPIRs interne direkte-hardware-grensesnitt, oppført i [CLI-referansen](../reference/cli-reference.md) for fullstendighetens skyld – det er ikke en del av den leverte CLI-en; GUI-arbeidsflyten ovenfor er den støttede live-veien.)
