# GUI: Navigering

Når du starter Chloros for første gang, starter programmet sin prosesseringsbackend. Så snart backenden er klar, vises hovedmenyikonet øverst til venstre <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">, og fanene «Kameraer» og «Lyssensorer» blir tilgjengelige i venstre sidefelt (de er nedtonet inntil da).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Fra venstre til høyre inneholder toppmenyen:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Hovedmeny

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>Fra hovedmenyen kan du:

* **Nytt prosjekt**— opprette et nytt prosjekt. Hvis du har lagret prosjektmaler, vises en rullegardinmeny med**Velg mal**, slik at det nye prosjektet starter med innstillingene fra en mal.
* **Åpne prosjekt**— åpne et eksisterende prosjekt. Listen inneholder en**Åpne prosjektmappe**-knapp som åpner prosjektmappen i filutforskeren din.
* **Dupliser prosjekt** — kopier det nåværende åpne prosjektet under et nytt navn (et ledig navn som «MittProsjekt (2)» foreslås) og åpne kopien. _(synlig etter at et prosjekt er åpnet)_
* **Legg til filer** — legg til enkeltstående bildefiler i det aktuelle prosjektet _(synlig etter at et prosjekt er åpnet)_
* **Legg til mappe** — legg til én eller flere mapper med bilder i det aktuelle prosjektet _(synlig etter at et prosjekt er åpnet)_
* **Start behandling / Stopp behandling** — start eller stopp bildebehandlingsprosessen _(aktiveres etter at filer er lagt til)_
* **Koble til kamera** — gå til [fanen Kameraer](lattice/) for å koble til et LATTICE-kamera eller -array. Fungerer uten at et prosjekt er åpent.
* **Koble til lyssensor** — gå til [fanen Lyssensorer](daq/) for å koble til en DAQ-lyssensor. Fungerer uten at et prosjekt er åpent.

{% hint style="info" %}
**Kun Windows**: Chloros

Desktop GUI er tilgjengelig på Windows

. Linux

-brukere bør se dokumentasjonen for [CLI

](CLI.md) og [Python

SDK

](api-python-sdk.md) for headless-behandling.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">

Spill av/Start-knapp

Når den er aktivert, starter «Start behandling»-knappen bildebehandlingsprosessen.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line">

Fremdriftslinje <img src=".gitbook/assets/image (5).png" alt="" data-size="line">



I den gratis Chloros

-modusen, som behandler alle filer sekvensielt, viser fremdriftslinjen to trinn: Målgjenkjenning og Behandling.

I den lisensierte Chloros

+-modusen, som behandler alle filer samtidig, viser fremdriftslinjen fire trinn: Gjenkjenning, Analyse, Kalibrering, Eksport. Hvis du holder musepekeren over fremdriftslinjen i Chloros

+, vises et utvidet panel med de fire trinnene, slik at du kan følge med. Hvis du klikker på den øverste fremdriftslinjen, fryses utvidelsespanelet; klikker du igjen, frigjøres det.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Sidemeny

Menyen i venstre sidefelt inneholder ulike ikoner du kan bruke, i denne rekkefølgen fra topp til bunn:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Prosjektinnstillinger](project-settings/project-settings.md)

Fanen «Prosjektinnstillinger» lar deg justere globale prosjektinnstillinger og innstillinger for prosjektbehandling. Juster disse før du begynner å behandle filene dine.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Filbrowser

Legg til filer/mapper og fjern filer fra prosjektet. Dupliserte filer ignoreres. Merk av i målkolonnen for et hvilket som helst målbilde, så vil behandlingen kun se på de merkede bildene som mål, noe som vil øke behandlingstiden betydelig. Bruk vekslerknappen «Bilde/Metadata» for å veksle mellom visning av miniatyrrutenettet for det valgte bildet og en detaljert metadatatabell.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Bildeviser](image-viewer-gui/opening-an-image-full-screen.md)

Når du klikker på et bilde i hovedbildeviseren, åpnes det i fullskjerm i fanen «Bildeviser».

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [Kartviser](image-viewer-gui/map-markers.md)

Se bildene dine på et interaktivt 2D-kart basert på GPS-koordinatene deres. Støtter Google Maps og ESRI-fliseleverandører, og velger automatisk den beste tjenesten for din posisjon. Hold musepekeren over markører for å se forhåndsvisninger av miniatyrbilder.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [Kameraer](lattice/)

Koble til og styr LATTICE-kameraer i sanntid — ett om gangen eller som synkroniserte flerkamerasystemer. Fanen viser live forhåndsvisningsruter med overlegg og histogrammer, innstillinger per kamera og per oppsett, samt opptaksinnstillinger som bestemmer hvilke kameraer og eksporttyper «Capture All» skal produsere. Tilgjengelig så snart backenden er klar; se [LATTICE-seksjonen](lattice/) for fullstendig veiledning.

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [Lyssensorer](daq/)

Koble til DAQ-lyssensorer — DAQ-U (USB), DAQ-M (Bluetooth) og DAQ-E (Ethernet) — og se deres kalibrerte spektrumdiagrammer i sanntid i W/m²/nm. Herfra kan du registrere `.daq`-filer i det åpne prosjektet, gi sensorene nye navn, velge profiler for kapasitanskorreksjon og oppdatere DAQ-E-firmware. Tilgjengelig når backenden er klar; se [DAQ-delen](daq/) for fullstendig veiledning.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Feilsøkingslogg

Gå gjennom loggen for feilsøkingsutskrifter når det oppstår problemer. Kopier/last ned loggen og send den til [MAPIR-support](https://www.mapir.camera/community/contact) for å få hjelp.

Nytt i 1.3.0: Feilikonet øverst på loggfanen (**Rapporter en feil**) sender en rapport til MAPIR med din beskrivelse, opptil 10 bilder, Chloros-loggene og systemdetaljer. Du må være logget inn for å kunne sende en rapport. Se [Rapportere en feil fra loggfanen](processing-images-gui/monitoring-the-processing.md#reporting-a-bug-from-the-log-tab).

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Brukerinnlogging](chloros+-login.md)

Sidepanelet for brukerinnlogging lar deg logge inn på Chloros+-kontoen din for å få tilgang til avanserte funksjoner. Du kan også se den gjeldende versjonen av applikasjonen, samt endre språket på teksten som vises i Chloros GUI og CLI.
