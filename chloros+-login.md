# Chloros+ Innlogging

## Chloros og Chloros (nettleser) Innlogging

Bruker <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> sidebar-menyen lar deg logge inn på din Chloros+-konto og låse opp tilleggsfunksjoner.

Når du er logget inn, vises kontoopplysningene dine:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>

## CLI Innlogging

Logg inn med Chloros+-påloggingsinformasjonen din for å aktivere CLI-behandling.

**Syntaks:**

```bash
chloros-cli login <email> <password>
```

**Eksempel:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Spesialtegn**: Bruk enkelt anførselstegn rundt passord som inneholder tegn som `$`, `!` eller mellomrom.
{% endhint %}

**Utdata:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>

### Planens utløpsdato

Planens utløpsdato i GUI viser når lisensen din blir ugyldig. For gjentakende månedlige abonnementer er utløpsdatoen ved slutten av måneden. For årlige abonnementer er det ett år etter at du startet abonnementet. Lisenskontrollen krever en månedlig internettforbindelse for å verifisere, med en 30 dagers nådeperiode.

### Enhetsbegrensning

Hver Chloros+-plan tilbyr et forskjellig antall registrerte enheter. Hver enhet du logger inn på med en Chloros+-konto, teller med i antall registrerte enheter. Du kan endre navn på og fjerne en enhet på MAPIR Cloud-kontosiden.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+-abonnement</th><th align="center">KOBBER</th><th align="center">BRONZE</th><th align="center">SØLV</th><th align="center">GULL</th></tr></thead><tbody><tr><td align="right">Støttede enheter</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
