# Chloros+ Innlogging

## Chloros og Chloros (nettleser) Innlogging

Brukerens <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> sidebarmenyen lar deg logge inn på din Chloros+-konto og låse opp flere funksjoner.

Når du er logget inn, vises kontoopplysningene dine:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## CLI-pålogging

Logg inn med dine Chloros+-påloggingsopplysninger for å aktivere CLI-behandling. På Linux (uten grafisk brukergrensesnitt) er dette den eneste måten å aktivere lisensen din på.

**Syntaks:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDK-brukere**: Python SDK tilbyr også en programmatisk `logout()`-metode for å slette påloggingsinformasjon i hurtigbufferen. Se [Python SDK dokumentasjonen](api-python-sdk.md#logout) for detaljer.
{% endhint %}

**Eksempel:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Spesialtegn**: Bruk enkle anførselstegn rundt passord som inneholder tegn som `$`, `!` eller mellomrom.
{% endhint %}

**Utdata:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### Lagring av påloggingsinformasjon

Cachelagret påloggingsinformasjon lagres på et plattformspesifikt sted:

| Plattform | Sti til cache for påloggingsinformasjon |
| --- | --- |
| **Windows** | `%APPDATA%\Chloros\cache\` |
| **Linux** | `~/.cache/chloros/` |

### Planens utløp

Planens utløp i brukergrensesnittet viser når lisensen din blir ugyldig. For gjentakende månedlige abonnementer er utløpet ved slutten av måneden. For årlige abonnementer er det ett år etter at du startet abonnementet. Lisenskontrollen krever en månedlig internettforbindelse for å verifisere, med en 30 dagers frist.

### Enhetsbegrensning

Hver Chloros+-plan tilbyr et forskjellig antall registrerte enheter. Hver enhet du logger inn på med en Chloros+-konto, teller med i antallet registrerte enheter. Du kan gi nytt navn til og fjerne en enhet på MAPIR Cloud-kontosiden din.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+-abonnement</th><th align="center">KOBBER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GULL</th></tr></thead><tbody><tr><td align="right">Enheter som støttes</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
