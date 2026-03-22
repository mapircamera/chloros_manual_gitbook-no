---
metaLinks: {}
---

# Kom i gang

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros er et program fra [MAPIR](https://www.mapir.camera) for behandling av bilder og andre sensordata.

***{% hint style="success" %}**Hva er nytt i Chloros 1.1.0**: Innebygd støtte for Linux (amd64 og arm64), NVIDIA Jetson edge computing, Dynamic Compute Adaptation, 4-tråds behandlingspipeline, nye CLI-kommandoer og -alternativer. Se [Last ned](download.md) for fullstendig endringslogg.
{% endhint %}

Chloros er tilgjengelig i 3 applikasjonsmoduser:

## Chloros: Desktop GUI-applikasjon

Frittstående, separat vindu med alle funksjoner. _Kun Windows._

## [Chloros CLI: Kommandolinjegrensesnitt](CLI.md)

Batchbehandling via kommandolinjen. Perfekt for automatisering, skripting og headless-drift. Tilgjengelig på **Windows, Linux amd64 og Linux arm64 (NVIDIA Jetson)**. _CLI krever en Chloros+-lisens for tilgang._

## [Chloros API: Python SDK](api-python-sdk.md)

Programmatisk Python-grensesnitt for automatisering og tilpassede arbeidsflyter. Perfekt for forskningspipelines, integrasjon med eksisterende Python-applikasjoner og utvikling av tilpassede verktøy. Tilgjengelig på **alle plattformer** via `pip install chloros-sdk`. _API krever en Chloros+-lisens for tilgang._***

## Støttede plattformer

| Plattform | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11** | Ja | Ja | Ja |
| **Linux amd64 (x86_64)** | Nei | Ja | Ja |
| **Linux arm64 (NVIDIA Jetson)** | Nei | Ja | Ja |

For installasjonsinstruksjoner for Linux, se avsnittet [Linux &amp; Edge Computing](linux/linux-overview.md).

***

## Chloros+

Selv om Chloros er gratis å bruke til de fleste oppgaver, kan det hende du ønsker mer. Da kan en betalt lisens for Chloros+ være til nytte for deg. Med en Chloros+-lisens kan du låse opp nye funksjoner som:

* **Flertrådet behandling**: øker hastigheten på bildebehandlingen betydelig for større prosjekter ved å behandle bilder samtidig gjennom rørledningen.
* **GPU (CUDA)-akselerasjon**: dra nytte av dagens høyere GPU-minnealternativer for å øke hastigheten på bildebehandlingsrørledningen ytterligere. Vi anbefaler 4 GB eller mer VRAM for best resultat.
* **Chloros+**[**CLI**](CLI.md)**Tilgang**: kjør Chloros+ fra kommandolinjen for å automatisere og integrere i din egen programvare.
* **Chloros+**[**API**](api-python-sdk.md)**Tilgang:** kjør Chloros+ fra Python for programmatisk kontroll, noe som muliggjør sømløs integrasjon med dine forskningspipelines, arbeidsflyter for dataanalyse og tilpassede applikasjoner.
* **Bruk på flere enheter**: hver Chloros+-lisens tillater registrering av 2+ enheter. Bruk din MAPIR Cloud-konto til å administrere registrerte enheter. Legg til støtte for flere enheter ved å oppgradere din Chloros+-lisens.
* **Avansert teksturbevisst debayer-metode:** en høykvalitets kantbevisst debayer kombinert med en AI/ML-støyreduksjonsmodell som fjerner nesten all debayering-støy. 
* **Tilpassede multispektrale indeksformler:** legg inn tilpassede multispektrale indekser i Chloros-rasterkalkulatorene, både for behandling og bildesandkassen.
* **Linux &amp; Edge Computing:** kjør Chloros på Linux x86\_64- og ARM64-plattformer, inkludert NVIDIA Jetson, for felt- og kantbehandling. Se [Linux Oversikt](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Priser og registrering</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
