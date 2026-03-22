---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibreringsmål

MAPIR tilbyr ulike kalibreringsmål for en rekke bruksområder. Den kompakte T4-R50 som vises nedenfor, inneholder 4 paneler som er målt for lysrefleksjon i området 250–2 500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>T4-referansemålene for diffus refleksjon har følgende refleksjonskurver, [last ned data her](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Refleksjon :: 250–2 500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4-refleksjon :: 400–1000 nm</p></figcaption></figure>T4P-diffusreferansemålene har følgende reflektanskurver, [data nedlasting her](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P-refleksjon :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P-refleksjonsgrad :: 400–1000 nm</p></figcaption></figure>Når du ser på reflektansgrafen, kan du se at verdiene er bølgelengde (x-aksen) mot reflektansprosent (y-aksen). Når vi tar et bilde av kalibreringsmålet, oppretter vi deretter en sammenheng mellom pikselverdi og reflektansprosent, innenfor det spektrumet som hvert av kameraets sensorband er følsomme for.

Dette betyr at for hvert bilde du tar med våre kameraer, kan du bruke et bilde av våre reflektansmål, for eksempel [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) eller [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), til å kalibrere bildene for reflektans. Når kalibreringen er fullført, tilsvarer hver piksel i bildet en prosentandel av reflektansen.

Hvis du eksporterer de kalibrerte bildene i Chloros som vanlig JPG eller TIFF, beregnes refleksjonsprosenten ved å dele pikselverdien med bitdybden til bildeformatet. For JPG deler du altså med 255, og for TIFF deler du med 65 535. Du kan også velge utdataformatet PERCENT i Chloros, og da vil hver piksel variere fra en prosentverdi på 0,0 til 1,0 (0 % til 100 % reflektans). Bare husk at noen bildeprogrammer ikke kan håndtere bilder i prosent (flytende komma), og at de tar mye lagringsplass.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
