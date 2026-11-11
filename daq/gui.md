# DAQ-fanen i Chloros

DAQ-fanen – merket **Lyssensorer** i Chloros-sidepanelet – er det direkte kontrollpanelet for [DAQ-U-, DAQ-M- og DAQ-E-lyssensorer](README.md): Koble til sensorer via hvilket som helst transportprotokoll, se kalibrerte spektra i sanntid, beregne refleksjonsgrad i sanntid fra et sensorpar og lagre `.daq`-filer direkte i prosjektet ditt.

Fanen blir tilgjengelig så snart Chloros-backenden er ferdig med oppstarten. Diagrammene på fanen får data fra Chloros’ DAQ-tjeneste via en live-tilkobling som kobler seg til igjen automatisk (2–10 sekunders ventetid) hvis den blir avbrutt; mens tjenesten er utilgjengelig, viser statusraden for en sensor **No Server**.

Oppsettet består av en **sensorsidebar**(én rad per tilkoblet sensor) pluss et**diagramområde** (én diagramrute per sensor eller gruppe).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Koble til en sensor

Klikk på **Koble til sensor** øverst i sidepanelet. Koblingsdialogen åpnes i hovedområdet (eller som et overlegg når du legger til en ny sensor – i så fall vises en Avbryt-knapp).

| Kontroll | Oppførsel |
| --- | --- |
| **Enhetstype** | `DAQ-U (USB)` (standard), `DAQ-M (Bluetooth)` eller `DAQ-E (Ethernet)`. Ved å bytte startes skanningen på nytt for den nyvalgte transporttypen. |
| **Port / BLE-enhet / Vertsnavn / IP** | Viser oppdagede enheter som `device - description`; den første oppføringen som gjenkjennes som en sensor, velges automatisk. Under skanningen vises `Scanning...` (USB), `Scanning (N)...` med en nedtelling på 8 sekunder (BLE) eller `Discovering ethernet sensors (N)...` med en nedtelling på 5 sekunder (Ethernet). Tomme resultater vises som `No ports` / `No BLE devices` / `No ethernet sensors found`. |
| **↻ Oppdater** | Skanner den valgte transporten på nytt umiddelbart (deaktivert midt i BLE-/Ethernet-skanning). |
| **Koble til sensor** | Aktiveres når en enhet er valgt; endrer betegnelsen til `Connecting...` mens tilkoblingen opprettes. |

Oppdagelsen kjører kun **mens tilkoblingsdialogen er på skjermen**, og gjentas hvert 15. sekund kun for den valgte transporten — det er ikke nok å åpne fanen for å utføre en skanning. Ved feil viser dialogen: *«Tilkoblingen mislyktes. Prøv å koble fra og koble til sensoren på nytt, og klikk deretter på Koble til igjen.»*

Sidepanelet åpnes automatisk når den første sensoren kobles til.

{% hint style="info" %}
**Vises ikke DAQ-E?**DAQ-E har ingen status-LED — sjekk PoE-/linkindikatoren på bryteren eller injektorporten den er koblet til, og vent noen sekunder etter oppstart slik at den får tid til å starte opp. Chloros-maskinen må være på samme kringkastingsdomene (mDNS går ikke gjennom rutere). Installasjonsprogrammets**brannmurregel for Chloros-backend (UDP-In)** slipper inn mDNS (UDP 5353), DAQ-E-data (UDP 5002) og PTP (UDP 319/320); en tredjeparts brannmur må ha de samme portene åpnet. To DAQ-E-enheter på ett LAN oppdages separat, hver under sitt eget `daq-e-<id>.local`-vertsnavn.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Enhetstype tilbyr DAQ-U (USB), DAQ-M (Bluetooth) og DAQ-E (Ethernet)</figcaption></figure>

***

## Sensorsidefeltet

Hver tilkoblet sensor får en rad (pluss én rad per «Ambient+Object»-gruppe). Radene kan omorganiseres ved å dra dem, og rekkefølgen deres endrer også rekkefølgen på diagramrutene. Klikk på en rad for å gjøre den sensoren/gruppen til det aktive diagrammet i listevisningen.

| Element | Betydning |
| --- | --- |
| Farget venstre kant | Sensorens grafikkfarge. |
| Transportmerke | `DAQ-U` / `DAQ-M` / `DAQ-E`, eller et grønt `REF`-merke for en Ambient+Object-refleksjonsgruppe. |
| Enhetsnavn | Standardinnstillingen er sensorens serienummer (dens faste identitet for kalibrering, `.daq`-filnavn og importmatching); egendefinerte navn beholdes per prosjekt. |
| **Kalibrert**-pille (grønn) | Vises når sensorens fabrikkalibreringspakke er lastet inn, dvs. at spektrene er ekte W/m²/nm. |
| **Oppdatering tilgjengelig**-ikon (gult, kun DAQ-E) | Den kjørende fastvaren er eldre enn bildet som følger med denne Chloros-versjonen. Under en oppdatering viser den fremdriften i sanntid (`Flashing… N%`, `Restarting sensor…`, deretter `Updated X → Y` eller `Failed`). |
| Øye | Slår synligheten til denne sensoren på diagrammet av og på. |
| Tannhjul | Åpner modalen for innstillinger per sensor (nedenfor). |
| ✕ (rød) | Kobler fra sensoren, eller fjerner en «Ambient+Object»-gruppe. |

Over radene finnes to knapper:

* **Koble til sensor** — åpner tilkoblingsdialogen (endrer navn til `Connecting...` mens den er opptatt).
* **Ta opp alt / Stopp alt**— starter eller stopper en `.daq`-opptak på**alle**tilkoblede sensorer. Krever minst én sensor**og et åpent prosjekt** (verktøytips: «Åpne et prosjekt for å ta opp»); den blir rød mens en opptak kjører.

I tom tilstand står det «Ingen sensorer tilkoblet».

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Innstillinger per sensor (tannhjulsmodalen)

Åpnes med tannhjulikonet på en sensorrekke. Innhold i rekkefølge:

* **Informasjonsrader** — Enhetstype (DAQ-U/M/E), Tilkobling (`Serial (USB)` / `Bluetooth` / `Ethernet`), port (COM-port, BLE-adresse eller vert) og seriell.
* **Kalibreringsrapport: Last ned** — henter enhetens NIST-sporbare kalibreringssertifikat (PDF) og åpner det i PDF-viseren din. Tilgjengelig så snart serienummeret er kjent; sertifikatet lagres i hurtigbufferen ved første tilkobling.
* **Enhetsnavn** — klikk på blyanten for å endre navn; gjelder per prosjekt.
* **Graflinjefarge** — fargeprøve; gjelder per prosjekt.
* **Integrasjonstid (ms)**— glidebryter + tall,**1–500 ms**, standard**32 ms**. Deaktivert når AE er PÅ.
* **Ramme-gjennomsnitt**— glidebryter + tall,**1–50 rammer**, standard**20**.
* **AE: PÅ/AV**— bryter for automatisk eksponering;**standard PÅ** ved tilkobling. Slå den av for å stille inn integrasjonstiden manuelt.
* **Stopp strømming / Start strømming** — sett live-strømmen på pause eller gjenoppta den.
* **Record / Stop Recording** — opptak per sensor `.daq` (krever et åpent prosjekt).
* **Cap** — profilen for eksponeringskorreksjon (neste avsnitt).
* **Rader med live-informasjon** — Integrasjonstid (ms), FPS, Prøver, Opptak (rødt `REC` eller `Off`) og Status (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Kun DAQ-E: rader for nettverk, fastvare og PTP

* **Vertsnavn / IP** — enhetens gjeldende adresse.
* **Firmware**— gjeldende firmwareversjon, pluss en handlingscelle: en**Oppdater til \<version\>

**-knapp vises når denne Chloros-versjonen inneholder et nyere DAQ-E-firmwarebilde. Oppdateringen overføres via nettverket på omtrent 30 sekunder; sensoren starter på nytt og kobler seg til igjen automatisk, og en avbrutt overføring lar den gjeldende firmwaren forbli intakt. Fremdriften vises i sanntid (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`), og cellen viser `Up to date` når den er oppdatert.
* **PTP-synkronisering** — den aktuelle PTP-tilstanden (faller tilbake til `unknown`). DAQ-E-firmware v1.2.0+ deltar i IEEE 1588 PTPv2 som en ren slaveklokke; Chloros-vertens backend er PTP-grandmaster, og alle DAQ-E- og LATTICE-kameraer på LAN-nettverket er slaver til denne i domene 0, noe som holder tidsstemplene innenfor omtrent 1 ms.
* **Nettverk**— adressen enheten kjører på og hvordan den fikk den (DHCP, link-local eller statisk), samt en lagret innstilling som gjelder etter en omstart.**Endre…**åpner et skjema:**Automatisk (DHCP)**eller**Statisk IP**med en IP-adresse, en nettverksmaske og en valgfri gateway.**Bruk og start på nytt** lagrer innstillingene, starter sensoren på nytt og kobler den til igjen. En statisk adresse utenfor denne datamaskinens nettverk avvises, fordi Chloros ikke kunne nå sensoren etter omstarten; det vises da en boks for å tillate det likevel, for en datamaskin som skal omadresseres eller rutes til det nettverket. **Strøm ved oppstart**angir om sensoren skal begynne å strømme så snart den slås på, før noen programvare kobler seg til.**Restart sensor** starter sensoren på nytt og kobler den til igjen med de gjeldende innstillingene. En omstart avslås mens sensoren registrerer data. Nytt i 1.3.0.

For en Ambient+Object-gruppe viser tannhjul-modalen kun gruppens kildesensorer, enhetsnavn og graflinjefarge.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->



### Valg av deksel

Nedtrekksmenyen **Deksel** forteller Chloros hvilket fysisk deksel som er montert over sensorens diffusor, og bruker dekselets fabrikkmålte korreksjonsprofil på hvert spektrum. Valgene avhenger av modellen:

| Modell | Dekselvalg |
| --- | --- |
| DAQ-U | Ingen (bar sensor), FOV 15°, FOV 30°, FOV 45°, FOV 60°, FOV 90°, Sunshine (kosinuskorrektor) |
| DAQ-M | Ingen (bar sensor), Sunshine (kosinuskorrektor) |
| DAQ-E | Ingen (bar sensor), FOV 15°, FOV 45°, FOV 90°, Sunshine (kosinuskorrektor) |

**Standardinnstillingen for alle modeller er «Sunshine» (kosinuskorrektor)** — MAPIR leverer alle DAQ-enheter med «Sunshine»-hetten montert, og dette er standardkonfigurasjonen for utendørs bruk: et 180° halvkuleformet synsfelt med kosinusfeil ≤ ±4 % ut til 60° og ≤ ±4,5 % ut til 70° (anbefales ikke under ~15° solhøyde), med innebygd demping (~12×). Valget ditt lagres i prosjektet.

{% hint style="warning" %}
**Valget av hette må samsvare med den fysiske hetten.**Verken sensoren eller programvaren kan oppdage hvilken hette som er montert. Valget styrer både den løpende korreksjonen og stempelet som skrives inn i hver `.daq`-fil — med Sunshine-hettenes ~12× demping, vil et udeklarert dekselbytte føre til feilkorrigering av spektra med omtrent den samme faktoren. (Å fjerne og montere det samme dekselet på nytt gir en avvik på ca. 1,5 %.) Velg**Ingen (bar sensor)** kun når dekselet er fysisk fjernet; på en DAQ-E bruker «None» fortsatt en fabrikkgeometriprofil for den innfelte glassdiffusoren – det er ikke en «no-op» – og en DAQ-E uten deksel er en benkkonfigurasjon, ikke en støttet feltkonfigurasjon.
{% endhint %}

{% hint style="info" %}
Oppgradering fra en tidligere bruksanvisning: bryteren «Sunshine Diffuser Installed» på nettlesersiden fra versjon 1.1.0 er fjernet. Håndtering av dekselet skjer nå via denne dekselprofilen per sensor, som brukes på serversiden.
{% endhint %}

***

## Diagramområdet

En fast topplinje inneholder en **bryter for visning av liste ⇄ rutenett**og en**zoomglidebryter for diagram** (rutenettstørrelse 200–2000 px). Visningen skifter automatisk til rutenett når det finnes mer enn én diagramgruppe, og tilbake til liste når det er én eller færre. Visningsmodus og diagramstørrelse lagres per prosjekt.**Spektrumdiagrammet** for hver sensor viser:

* **X-aksen** — Bølgelengde (nm). Sensorrutenettet er 340–1010 nm med 5 nm mellomrom (135 punkter), interpolert til 1 nm for visning.
* **Y-aksen**— for en kalibrert sensor (grønn**Kalibrert**-knapp),**Effekt (W/m²/nm)**— radiometrisk kalibrert spektral irradians på alle tre transportformene — med et automatisk SI-prefiks (m/µ/n) valgt fra toppen. En sensor hvis kalibreringspakke ikke er lastet inn, plotter sine råmålinger som**Intensitet (tall)**, uskalert. Når et diagram viser begge typer, vises effekt på venstre akse og tellinger på høyre akse.
* En regnbuefarget spektralfylling under en enkelt kurve; flere sensorer på ett diagram vises som fargede linjer med nedtonet fylling.
* **Hold musepekeren over**— en vertikal markør med bølgelengde og verdi per sensor (W/m²/nm for en kalibrert sensor, ellers hele tellinger);**dra** for å zoome (en zoom-ut-knapp vises når du zoomer ut).
* En **+**-knapp (kun i rutenettvisning) for å legge til en sensor i dette diagrammet eller opprette en gruppe (se nedenfor).
* Enhetsnavnet er sentrert øverst, og det vises en spinner inntil det første bildet kommer inn.

**Metthet** er ikke markert på selve diagrammet: en mettet sensor viser rød `SATURATED`-statustekst og en rød `Saturated: Yes`-rad i tabellen med sanntidsdata. Senk integrasjonstiden eller aktiver AE på nytt for å fjerne den.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->



***

## Tabell med sanntidsdata (listevisning)

Under diagrammet i listevisning, oppdateres hvert 500. millisekund:

* **Alle modeller**: Lysfargeprøve (sRGB fra CIE XYZ), Mettet (Ja/Nei), CIE 1931 X/Y/Z, Kromatisitet x/y, CIE u′/v′, CCT (K), CRI (Ra), dominerende bølgelengde (nm), toppbølgelengde (nm), eksitasjonsrenhet, Duv, CIE L\*/a\*/b\* og Munsell H/V/C.
* **Kun kalibrerte sensorer**(hvilken som helst av DAQ-U / DAQ-M / DAQ-E så snart fabrikkalibreringspakken er lastet inn – det grønne**Kalibrert**-merket på sensorrekken er kjennetegnet): Total effekt (W/m²), fotopisk lux (lx), skotopisk lux (lx), S/P-forhold, PPFD samt PPFD Red /Green /Blue (µmol/m²/s) og de opiske strålingsintensitetene – S-kone, melanopisk, rhodopisk, M-kone, L-kone (alle W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Refleksjonsgrupper (Omgivelser + Objekt)

To tilkoblede sensorer kan kombineres til en sanntidsvisning av reflektans — uten bruk av kamera:

1. I rutenettvisningen klikker du på **+**på en diagramrute og velger**Kombiner omgivelser + objekt**.
2. Velg en **Omgivelseslyskilde**-sensor og en**Objektskanner**-sensor (to forskjellige sensorer), og klikk deretter**Opprett**.

Chloros beregner R(λ) = objekt(λ) / omgivelse(λ) per bølgelengde fra de to sanntidsstrømmene (0 der omgivelse ≤ 0). Gruppens etikett følger sensorenes kalibreringsklasse:

* Begge sensorene er kalibrert (pakke lastet inn) → **«Tilsynelatende reflektans»**.
* En av sensorene er ikke kalibrert → **«Relativ reflektans»**.

Gruppen vises som en grønn `REF`-rad i sidefeltet og i sitt eget diagram (regnbuefylling, vis verdier med 4 desimaler ved å holde musepekeren over, dra for å zoome).

**+**-menyen tilbyr også**Legg til ny sensor** med tre plasseringsalternativer: *Kombiner ny sensor* (legg til i dette diagrammet), *Flytt eksisterende sensor hit* eller *Vis ny sensor* (eget diagram).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->



### Tabell over vegetasjonsindeks

I listevisningen ligger en tabell over vegetasjonsindeks under diagrammet til en reflektansgruppe, beregnet ut fra sanntidsreflektansen ved båndsentrene **blå 450 / grønn 550 / rød 670 / NIR 800 nm** (verdier med 4 desimaler, `---` når ikke beregning er mulig; hold musepekeren over et indeksnavn for å se det fulle navnet):

* **Vises alltid** (skalainvariant, uansett sensorkombinasjon): NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR.
* **Kun når begge sensorene er effektkalibrert** (begge pakkene lastet inn): EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI.

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

***

## Opptak av `.daq`-filer

* Opptak krever et **åpent prosjekt** — ellers er både «Record All» (sidefeltet) og «Record»-knappen for hver enkelt sensor deaktivert.
* Filene lagres i **`<project folder>/light_sensor/`**; filnavnene inneholder sensor-ID og et tidsstempel, og enhetsnavnet lagres sammen med opptaket.
* Når en opptak stopper (Stopp, Stopp alt eller en frakobling midt i opptaket), blir den ferdige `.daq`-filen **automatisk lagt til det åpne prosjektet** — den vises i prosjektets filliste uten manuell tillegg, klar til å brukes som nedadrettede data for [refleksjonsbehandling](README.md).
* En rød `REC`-indikator vises i de aktive radene i innstillingsmodalen under opptak.

For kvantitative strålingsstyrketall må du beregne gjennomsnittet av minst 15 sekunders data — dette er en egenskap ved instrumentet, ikke en feil.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->



***

## Oppsett med flere sensorer og prosjektlagring

* Kombiner flere sensorer på ett diagram (delte akser), behold separate diagrammer (automatisk rutenettoppsett), flytt sensorer mellom diagrammer, dra og omorganiser rader/ruter, og skjul enkeltstående sensorer med øye-knappen.
* Chloros lagrer følgende per prosjekt: enhetsnavn, grafikkfarger, diagramstørrelse, visningsmodus og hver sensors innstillinger (integrasjonstid, rammeutjevning, AE-tilstand, kapselvalg).
* **Når du åpner et prosjekt på nytt, kobles sensorene automatisk til igjen** via adresse – COM-port for DAQ-U, BLE-enhet for DAQ-M, mDNS-vertsnavn for DAQ-E (løses selv om enhetens IP-adresse har endret seg) – og bruker på nytt hver sensors lagrede cap-profil, rammeutjevning, AE-tilstand og manuell integrasjonstid.***

## Kameraparring (DLS)

Det er ingenting å parre. I motsetning til DLS-arbeidsflyter for droner som knytter en lyssensor til et kamera på forhånd, samsvarer Chloros DAQ-data med bildene i etterkant: ved import/behandling blir `.daq`-målingene interpolert til eksponeringstidsstemplet for hvert bilde. Ta opp med en hvilken som helst tilkoblet sensor (`.daq` havner automatisk i prosjektet), og refleksjonsbehandlingen finner de riktige målingene etter tid — se [DAQ-lyssensorer](README.md) for hvordan dataene om nedadgående lysstråling brukes. For **live**LATTICE-opptak velger du i stedet sensoren i rullegardinmenyen**Lyssensor** på fanen Kameraer; hver opptak lagrer da måleverdien den brukte, kun tilordnet kameraene til det aktuelle opptaket (se [Refleksjonsarbeidsflyter](reflectance.md)).</version\>
