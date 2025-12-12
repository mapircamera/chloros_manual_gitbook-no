# Chloros+ Logg inn

## Chloros og Chloros (nettleser) Logg inn

Brukeren <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> sidefeltmenyen lar deg logge på Chloros+-kontoen din og låse opp tilleggsfunksjoner.

Når du er logget på vil kontoopplysningene dine vises:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>

## CLI-pålogging

Logg på med Chloros+-legitimasjonen din for å aktivere CLI-behandling.

**Syntaks:**

```bash
chloros-cli login <email> <password>
```

**Eksempel:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Spesialtegn**: Bruk enkle anførselstegn rundt passord som inneholder tegn som `$`, `!` eller mellomrom.
{% endhint %}

**Produksjon:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>

### Planens utløp

Planens utløp i GUI viser når lisensen din blir ugyldig. For tilbakevendende månedlige abonnementer er utløpet ved slutten av måneden. For årsabonnement er det ett år etter at du startet abonnementet. Lisenssjekken krever en månedlig internettforbindelse for å bekrefte, med en 30-dagers utsettelsesperiode.

### Enhetsgrense

Hver Chloros+-plan tilbyr et annet antall registrerte enheter. Hver enhet du logger på med en Chloros+-konto vil telle mot antallet registrerte enheter. Du kan gi nytt navn til og fjerne en enhet på MAPIR Cloud-kontosiden din.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+ Plan</th><th align="center">KOPER</th><th align="center">BRONSE</th><th align="center">SØLV</th><th align="center">GULL</th></tr></thead><tbody><tr><td align="right">Enheter Støttes</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
