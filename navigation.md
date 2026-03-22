# GUI: Navigasjon

Når du starter Chloros og Chloros (nettleser) for første gang, starter programmet opp sin backend. Så snart den er klar, vises hovedmenyikonet øverst til venstre <img src=".gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

Fra venstre til høyre inneholder toppmenyen:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Hovedmeny

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

Fra hovedmenyen kan du:

* **Nytt prosjekt** — opprett et nytt prosjekt
* **Åpne prosjekt** — åpne et eksisterende prosjekt
* **Åpne prosjektmappe** — åpne prosjektmappen i filutforskeren
* **Legg til filer** — legg til individuelle bildefiler til det aktuelle prosjektet _(synlig etter at et prosjekt er åpnet)_
* **Legg til mappe** — legg til en mappe med bilder i det aktuelle prosjektet _(synlig etter at et prosjekt er åpnet)_
* **Start behandling / Stopp behandling** — start eller stopp bildebehandlingsprosessen _(aktivert etter at filer er lagt til)_

{% hint style="info" %}
**Kun Windows**: Chloros-skrivebordsgrensesnittet er tilgjengelig på Windows. Linux-brukere bør se [CLI](CLI.md) og [Python SDK](api-python-sdk.md) dokumentasjonen for headless-behandling.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1).png" alt="" data-size="line"> Spill av/Start-knapp

Når denne er aktivert, starter startknappen bildebehandlingsprosessen.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> Fremdriftslinje <img src=".gitbook/assets/image (5).png" alt="" data-size="line">I den gratis Chloros-modusen, som behandler alle filer sekvensielt, viser fremdriftslinjen to trinn: Målregistrering og Behandling.

I den betalte Chloros+-lisensierte modusen, som behandler alle filer samtidig, viser fremdriftslinjen fire trinn: Registrering, Analyse, Kalibrering, Eksport. Hvis du holder musepekeren over Chloros+-fremdriftslinjen, vises et utvidet panel med fire fremdriftslinjer, slik at du kan følge med. Hvis du klikker på den øverste fremdriftslinjen, fryses nedtrekkspanelet, og hvis du klikker igjen, frigjøres det.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Sidemeny

Menyen i venstre sidefelt inneholder ulike ikoner du kan bruke:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Prosjektinnstillinger](project-settings/project-settings.md)

Fanen Prosjektinnstillinger lar deg justere globale prosjektinnstillinger og prosjektbehandlingsinnstillinger. Juster disse før du begynner å behandle filene dine.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Filbrowser

Legg til filer/mapper og fjern filer fra prosjektet. Dupliserte filer ignoreres. Merk av i målkolonnen for et hvilket som helst målbilde, og behandlingen vil kun se på merkede bilder for mål, noe som øker behandlingstiden betydelig. Bruk bryteren Bilde/Metadata for å veksle mellom visning av det valgte bildets miniatyrrutenett og en detaljert metadatatabell.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Bildeviser](image-viewer-gui/opening-an-image-full-screen.md)

Når du klikker på et bilde i hovedbildeviseren, åpnes det i fullskjerm i fanen Bildeviser.

#### <img src=".gitbook/assets/image (7).png" alt="" data-size="line"> [Kart](image-viewer-gui/map-markers.md)

Se bildene dine på et interaktivt 2D-kart basert på deres GPS-koordinater. Støtter Google Maps og ESRI-fliseleverandører, og velger automatisk den beste tjenesten for din posisjon. Hold musepekeren over markører for å se forhåndsvisninger av miniatyrbilder.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Feilsøkingslogg

Gå gjennom loggen for feilsøkingsutskrifter når det oppstår problemer. Kopier/last ned loggen og send den til [MAPIR Support](https://www.mapir.camera/community/contact) for å få hjelp.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Brukerinnlogging](chloros+-login.md)

Sidepanelet for brukerinnlogging lar deg logge inn på din Chloros+-konto for å låse opp avanserte funksjoner. Du kan også se den gjeldende versjonen av applikasjonen, samt justere språket på den viste teksten i Chloros GUI og CLI.
