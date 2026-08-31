# GUI: Prosjekter

Chloros lar deg opprette prosjekter som kan åpnes på nytt senere. Et prosjekt er en vanlig mappe (i prosjektmappen din) som inneholder:

* `project.json` — prosjektinnstillinger, filliste og visningsinnstillinger
* `cameras.json` — kameraer og matriser som var tilkoblet mens prosjektet var åpent, med tilhørende innstillinger
* `sensors.json` — DAQ-lyssensorer som var tilkoblet mens prosjektet var åpent, samt kamera↔sensor-tilknytninger
* dine opptak, `.daq`-opptak og mapper med bearbeidet utdata

Det finnes ikke noe proprietært prosjektfilformat — mappen og dens JSON-filer utgjør prosjektet, noe som også gjør det enkelt å kopiere, arkivere og kjøre prosjekter fra [CLI](CLI.md) eller [Python SDK](api-python-sdk.md).

## Nytt prosjekt

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>Velg «Nytt prosjekt» fra hovedmenyen og skriv inn et unikt navn på prosjektet ditt.

Hvis du har lagret prosjektmaler, vises en rullegardinmeny med **Velg mal** under navnefeltet — når du velger en, starter det nye prosjektet med innstillingene fra den malen. Maler lagres fra [Prosjektinnstillinger](project-settings/project-settings.md): skriv inn et navn i feltet «Lagre prosjektmalnavn» og klikk på lagringsikonet.

## Åpne prosjekt

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>«Åpne prosjekt» viser en liste over alle prosjektene i prosjektmappen din, med <strong>«Åpne prosjektmappe»</strong> nederst</p></figcaption></figure>Velg «Åpne prosjekt» for å se en liste over eksisterende prosjekter i prosjektmappen. Hvis det ikke finnes noen prosjekter, åpnes ikke den sekundære sidemenyen. Du kan se noen prosjekter opprettet via brukergrensesnittet (t1, t2, t3) oppført på bildet ovenfor. Prosjektene DATE\_TIME ble opprettet av CLI ved hjelp av standardnavngivningsskjemaet for prosjekter. Hvis du klikker på et hvilket som helst prosjektnavn, åpnes det.

Hvis du klikker på «Åpne prosjektmappe»-knappen, åpnes filutforskeren på datamaskinen din med prosjektbanen. Du kan justere prosjektbanen i [Prosjektinnstillinger](project-settings/project-settings.md).

Hvis noen av prosjektets kildebildefiler har blitt flyttet eller slettet siden det sist ble åpnet, viser Chloros en dialogboks som lister opp nøyaktig hvilke filer som mangler, i stedet for å åpne et tomt rutenett.

## Dupliser prosjekt

Tilgjengelig når et prosjekt er åpent. Velg «Dupliser prosjekt» for å kopiere det aktuelle prosjektet under et nytt navn — Chloros foreslår det neste ledige navnet (f.eks. «Mitt prosjekt (2)») — og duplikatet åpnes umiddelbart.

## Legg til filer

Når et prosjekt er åpnet, velger du «Legg til filer» fra hovedmenyen for å legge til enkeltstående bildefiler til det aktuelle prosjektet. Dette speiler filutforskerens funksjon for å legge til filer, men er for enkelhets skyld tilgjengelig direkte fra hovedmenyen.

## Legg til mappe

Når et prosjekt er åpnet, velger du «Legg til mappe» fra hovedmenyen for å legge til mapper med bilder til det aktuelle prosjektet. Du kan velge flere mapper på én gang. Dupliserte filer ignoreres.

## Start / Stopp behandling

Når filer er lagt til i et prosjekt, blir «Start behandling» tilgjengelig i hovedmenyen. Dette er den samme handlingen som å klikke på Avspill/Start-knappen øverst i toppmenyen. Under behandlingen endres menypunktet til «Stopp behandling», slik at du kan stoppe behandlingsprosessen.

## Koble til kamera / Koble til lyssensor

Nederst i hovedmenyen finnes to maskinvaresnarveier, tilgjengelige uansett om et prosjekt er åpent eller ikke:

* **Koble til kamera** — åpner [fanen Kameraer](lattice/) for å koble til et LATTICE-kamera eller -array.
* **Koble til lyssensor** — åpner [fanen Lyssensorer](daq/) for å koble til en DAQ-lyssensor.

Hvis du kobler til maskinvare mens et prosjekt er åpent, lagres tilkoblingen i prosjektet (se nedenfor). Uten et åpent prosjekt gjelder tilkoblingene kun for den aktuelle økten.

{% hint style="info" %}
Menyelementene **Legg til filer**,**Legg til mappe**og**Start/Stopp behandling**er kun synlige eller aktiverte når et prosjekt er åpent og filer er lagt til. De gir rask tilgang til handlinger som også er tilgjengelige via sidepanelet**Filbrowser** og knappene i toppmenyen.
{% endhint %}

## Prosjekter husker maskinvaren din

Nytt i 1.2.0: Et prosjekt beholder maskinvaren du kobler til så lenge det er åpent. Kameraer og matriser (med innstillinger per kamera, navn, farger og rutenettoppsett) lagres som øyeblikksbilder i `cameras.json`, og lyssensorer (med navn, farger og kameratilknytninger) i `sensors.json` — automatisk, mens du arbeider.

Når du **åpner** et prosjekt på nytt, kobler Chloros seg ikke umiddelbart til noen maskinvare. Hver halvdel kobler seg til på nytt første gang du går inn på fanen den tilhører:

* Når du åpner **Kameraer**-fanen, kobles de lagrede kameraene og matrisene til på nytt, og de lagrede innstillingene deres blir brukt på nytt.
* Når du åpner fanen **Lyssensorer**, kobles de lagrede DAQ-sensorene til på nytt.

På denne måten vil det å åpne et prosjekt bare for å bla gjennom eller eksportere bilder aldri sette kameraene i strømmingsmodus. Hvis en lagret enhet ikke kan finnes når fanen åpnes, vil en dialogboks vise hvilke enheter som ikke er tilgjengelige, slik at du kan koble dem til på nytt eller fjerne dem.

## DAQ-opptak og .daq-filer i et prosjekt

* `.daq`-opptak som er laget mens prosjektet er åpent (fra fanen **Lyssensorer**eller under opptak),**legges automatisk til i prosjektet**.
* Importerte `.daq`-filer og alle prosjektopptak vises i delen **DAQ-lyssensor** under [Prosjektinnstillinger](project-settings/project-settings.md), hver med sin egen profil for lysstyrkekorreksjon.
* Under behandlingen leverer prosjektets `.daq`-filer nedadrettet belysning for reflektansprodukter — se [Utgangsbildeformater](output-image-formats.md).

## Kjøre et lagret prosjekt uten grafisk brukergrensesnitt

Et lagret prosjekt kan kjøres uten grafisk brukergrensesnitt:

* **CLI**: `chloros-cli project open / connect / capture / sensor / align / run` opererer på en prosjektmappebane — se [CLI-referansen](reference/cli-reference.md).
* **SDK**: `chloros_sdk.open_project(path)` returnerer et prosjekt-handle; `connect_all()` kobler alle lagrede kameraer og sensorer til nettet med de lagrede innstillingene – se [SDK-referanse](reference/sdk-reference.md).
