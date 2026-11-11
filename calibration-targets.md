---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibreringsmål

MAPIR tilbyr ulike kalibreringsmål for en rekke bruksområder. Den kompakte T4-R50-modellen som vises nedenfor, består av 4 paneler som er målt for lysrefleksjon i området 250–2 500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>De diffuse referansemålene T4 har følgende refleksjonskurver, [last ned data her](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Refleksjon :: 250–2 500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4-refleksjonsgrad :: 400–1 000 nm</p></figcaption></figure>De diffuse referansemålene T4P har følgende refleksjonskurver, [last ned data her](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P-refleksjonsgrad :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P-refleksjonsgrad :: 400–1000 nm</p></figcaption></figure>Når du ser på reflektansgrafen, kan du se at verdiene viser bølgelengde (x-aksen) mot reflektans i prosent (y-aksen). Når vi tar et bilde av kalibreringsmålet, oppretter vi deretter en sammenheng mellom pikselverdi og reflektans i prosent, innenfor det spektrumet som hvert av kameraets sensorband er følsomme for.

Dette betyr at for hvert bilde du tar med våre kameraer, kan du bruke et bilde av våre reflektansmål, for eksempel [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) eller [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), til å kalibrere bildene for reflektans. Når kalibreringen er fullført, tilsvarer hver piksel i bildet en prosentandel av reflektansen.

For **Survey3**-utdata: Hvis du eksporterer de kalibrerte bildene i Chloros

som vanlige JPG- eller TIFF

-filer, beregnes refleksjonsprosenten ved å dele pikselverdien med bildformatets bitdybde. For JPG deler du altså med 255, og for TIFF

deler du med 65 535. Du kan også velge utdataformatet PERCENT i Chloros

, og da vil hver piksel ligge i området fra 0,0 til 1,0 prosent (0 % til 100 % reflektans). Bare husk at noen bildeprogrammer ikke støtter bilder i prosentformat (flytende desimaler), og at disse bildene tar mye lagringsplass.

{% hint style="info" %}
**LATTICE-refleksjonsgrad bruker en annen pikselskala.** LATTICE-refleksjonsgrad lagres med DN 32768 = 100 % refleksjonsgrad (ikke 65 535), og hver fil har en XMP-tag (`Chloros:PixelScale`) som angir skalaen. Les taggen og del med den i stedet for å anta en konstant — se [Utgangsbildeformater](output-image-formats.md).
{% endhint %}

## Kalibreringsmål med LATTICE-kameraer

Med LATTICE-kameraer er et kalibreringsmål **valgfritt** for reflektans: Chloros

kan i stedet referere reflektansen til den nedadgående irradians som måles av en DAQ-lyssensor (ρ = π·L/E). Referansen velges med prosjektets innstilling for reflektanskilde, som angis med `chloros-cli process --reflectance-source` (brukergrensesnittet har ingen kontroll over dette og bruker prosjektets lagrede verdi, som standard `auto` — se [Prosjektinnstillinger](project-settings/project-settings.md)):

| Verdi | Oppførsel |
| --- | --- |
| `auto` *(standard)* | Et mål innenfor rammen som har bestått kvalitetssikring (QA), er den **absolutte referansen**; når det ikke finnes noe mål eller QA mislykkes, faller Chloros

tilbake til DAQ-nedstrømningsdelingen. Et mål som består QA med en feilmelding (et panel som har falt ut, eller et bånd som er ute av drift) blir ikke automatisk foretrukket: kjeden med målet, målet+DAQ og kun DAQ blir prøvd i rekkefølge etter deres målte poengsum på utelatte paneler. |
| `target` | Strengt mål-kun — ingen DAQ-erstatning. |
| `daq` | DAQ-autoritativ — nedstrålingsmålingen er alltid referansen. |

Ytterligere måladferd for LATTICE:

* **Målgeometrier** — ArUco-merkede paneler, paneler med fast ROI og stripemål støttes alle; geometrien hentes fra prosjektets målkonfigurasjon.
* **Mål-refleksjonsdata målt per enhet** — `--target-reflectance-dir DIR` peker på en katalog med refleksjonsskanninger målt per enhet (`<serial>.csv`, slått opp etter målenhetens serienummer/QR-kode). Ved en bom faller Chloros

tilbake til de nominelle T3/T4P-spektrene.
* **Tidsforankring** — et oppdaget mål kalibrerer bildene rundt seg og holdes fast mellom måloppsøkninger.

Fullstendig flaggsemantikk og eksempler finnes i [CLI

-referansen](reference/cli-reference.md) (se «Eksportbrytere per produkt»).

### F988

«F988-reflektansen kalibreres ved hjelp av et reflektanspanel i scenen: båndet ligger utenfor det kalibrerte området til DAQ-lyssensoren, så Chloros

bruker din siste panelopptak og beholder den mellom panelobservasjonene.»

Hvis F988 kjøres med kun DAQ-kalibrering, avviser Chloros

den DAQ-baserte reflektansen for det aktuelle båndet og oppgir årsaken (hopp over årsak `dls-uncalibrated-band-988`); arbeidsflyten med panelet er den støttede metoden.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
