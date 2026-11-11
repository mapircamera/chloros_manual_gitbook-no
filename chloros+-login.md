# Chloros+ Innlogging

## Innlogging via grafisk brukergrensesnitt

Brukermenyen i sidefeltet <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> lar deg logge inn på Chloros+-kontoen din og få tilgang til flere funksjoner.

**Du trenger bare å logge inn én gang per maskin.** GUI, CLI og Python-SDK deler samme bufret økt — når du logger inn via GUI på skrivebordet, aktiveres også CLI og SDK på den maskinen (og omvendt via `chloros-cli login`).

Når du er logget inn, vises kontoopplysningene dine:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.3.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->

## Abonnementsnivåer

| Abonnement | `plan_id` | Type |
| --- | --- | --- |
| Iron | `0` | Gratis |
| Copper | `1` | Betalt (Chloros+) |
| Bronze | `2` | Betalt (Chloros+) |
| Silver | `3` | Betalt (Chloros+) |
| Gull | `4` | Betalt (Chloros+) |

Se [abonnementer og priser](https://cloud.mapir.camera/pricing) for å se hva hvert betalt nivå inkluderer.

### Tilgang til CLI/SDK krever et betalt nivå

Tilgang til CLI og Python SDK krever **et hvilket som helst betalt Chloros+-nivå (Copper eller høyere)**. Dette håndheves**på serversiden** — hver CLI/SDK-forespørsel må inneholde både en aktiv økt og et betalt abonnement:

| HTTP-status | `error_code` | Betydning | Løsning |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Ikke logget inn på denne maskinen | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Logget inn, men abonnementet er for lavt (gratis Iron-nivå) | Oppgrader til et hvilket som helst betalt Chloros+-abonnement |

`chloros-cli status` er fortsatt tilgjengelig på gratisnivået, så du kan alltid se ditt nåværende abonnement og hvorfor tilgang nektes.

### Begrensninger for tilkoblet maskinvare per abonnement

Hvert abonnement har en øvre grense for hvor mange LATTICE-kameraer og DAQ-lyssensorer som kan være tilkoblet live samtidig:

| Abonnement | LATTICE-kameraer | DAQ-lyssensorer |
| --- | --- | --- |
| Iron (gratis / ikke logget inn) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## CLI-pålogging

Logg inn med dine Chloros+-påloggingsopplysninger for å aktivere CLI-behandling. På Linux (uten grafisk brukergrensesnitt) er dette den eneste måten å aktivere lisensen din på.

**Syntaks:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDK-brukere**: Python-SDK-et tilbyr også en programmatisk `logout()`-metode for å tømme bufret påloggingsinformasjon. Se [SDK-referansen](reference/sdk-reference.md) for detaljer.
{% endhint %}

**Eksempel:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Spesialtegn**: Bruk enkle anførselstegn rundt passord som inneholder tegn som `$`, `!` eller mellomrom.
{% endhint %}

**Utdata:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.3.0"; capture a successful login with the current output format. -->

### Lagring av påloggingsinformasjon

Bufret påloggingsinformasjon og konfigurasjon lagres i mappen `.chloros` i brukerens hjemmekatalog på **alle plattformer**:

| Plattform | Sti til påloggingsinformasjonsbuffer |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

Hvis du installerer en nyere versjon over en eksisterende, beholdes påloggingsinformasjonen din, både på Windows (installasjonsprogrammet) og på Linux (`.deb`, inkludert `chloros-cli update`). Hvis du avinstallerer Chloros – eller fjerner Linux-pakken – slettes den bufret påloggingen, slik at du må logge inn på nytt etter en ny installasjon.

### Utløp av abonnement og offline-toleranseperiode

Utløpsdatoen for abonnementet i brukergrensesnittet viser når lisensen din blir ugyldig. For gjentakende månedlige abonnementer utløper lisensen ved månedsslutt; for årlige abonnementer er det ett år etter at du startet abonnementet.

Chloros validerer lisensen din online, men det er mulig å jobbe offline innenfor en overgangsperiode:

* Vellykkede servervalideringer lagres i hurtigbufferen i **5 minutter**, så ved normal bruk foretas det svært få lisensforespørsler.
* En signert, maskinbundet lisensbuffer dekker lengre perioder uten nettilgang: **30 dager for månedlige abonnementer**, og**frem til abonnementets utløpsdato (maksimalt 365 dager) for årlige abonnementer**.
* Når overgangsperioden utløper, går abonnementet over til det gratis Iron-nivået inntil maskinen kan opprette kontakt med lisensserveren én gang; tilgangen gjenopptas ved neste vellykkede sjekk.

### Enhetsbegrensning

Hvert Chloros+-abonnement tilbyr et forskjellig antall registrerte enheter. Hver enhet du logger inn på med en Chloros+-konto, teller med i antallet registrerte enheter. Du kan gi en enhet nytt navn og fjerne den på MAPIR Cloud-kontosiden din.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros-+-abonnement</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Støttede enheter</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>

Den nøyaktige antall enheter som er tillatt for kontoen din, vises på MAPIR Cloud-kontosiden din. Når du logger ut av en enhet, frigjøres plassen til den, og en enhet som allerede er registrert, kan alltid logge inn igjen, selv når kontoen har nådd grensen for antall enheter.
