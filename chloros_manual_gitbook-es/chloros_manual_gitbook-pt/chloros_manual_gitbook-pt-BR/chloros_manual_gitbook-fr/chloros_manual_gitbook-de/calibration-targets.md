---
description: Lab-målte paneler som brukes til å kalibrere innfangede data i etterbehandling
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibreringsmål

MAPIR tilbyr ulike kalibreringsmål for å dekke en rekke bruksområder. Den kompakte T4-R50 som vises nedenfor inneholder 4 paneler som er målt for lysreflektans fra 250 - 2500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

T4 diffuse referansemål har følgende refleksjonskurver, [last ned data her](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 refleksjon :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 refleksjon :: 400-1000nm</p></figcaption></figure>

Når du ser på reflektansgrafen kan du se at verdiene er bølgelengde (x-aksen) kontra reflektansprosent (y-aksen). Når vi tar et bilde av kalibreringsmålet, skaper vi et forhold mellom pikselverdi og refleksjonsprosent, innenfor spekteret som hvert av kameraets sensorbånd er følsomme for.

Dette betyr at med hvert bilde du tar med kameraene våre, kan du bruke et bilde av refleksjonsmålene våre, for eksempel [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) eller [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) for å kalibrere bildene for refleksjon. Når de er kalibrert, er hver piksel i bildet lik prosent reflektans.

Hvis du sender ut de kalibrerte bildene i Chloros som typisk JPG eller TIFF, beregnes refleksjonsprosenten ved å dele pikselverdien med bitdybden til bildeformatet. Så for JPG del med 255, og for TIFF del med 65 535. Du kan også velge PROCENT-formatutdata i Chloros, og da vil hver piksel variere fra en prosentverdi på 0,0 til 1,0 (0% til 100% reflektans). Bare husk at noen bildeapplikasjoner ikke kan akseptere prosent (flytende komma) bilder, og de er store i størrelse lagringsmessig.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption>___1 <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
