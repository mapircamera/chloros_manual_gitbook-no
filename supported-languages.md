# Støttede språk

Chloros tilbyr full grensesnittstøtte på **38 språk over hele verden**, noe som gjør det tilgjengelig for brukere over hele kloden. Du kan bytte språk umiddelbart både i skrivebordsgrensesnittet og i CLI.

Chloros støtter følgende språk:

| # | Språk | Opprinnelig navn | CLI-kode |
|---|----------|-------------|----------|
| 1 | 🇺🇸 Engelsk | English | `en` |
| 2 | 🇪🇸 Spansk | Español | `es` |
| 3 | 🇵🇹 Portugisisk | Português | `pt` |
| 4 | 🇫🇷 Fransk | Français | `fr` |
| 5 | 🇩🇪 Tysk | Deutsch | `de` |
| 6 | 🇮🇹 Italiensk | Italiano | `it` |
| 7 | 🇯🇵 Japansk | 日本語 | `ja` |
| 8 | 🇰🇷 Koreansk | 한국어 | `ko` |
| 9 | 🇨🇳 Kinesisk (forenklet) | 简体中文 | `zh` |
| 10 | 🇹🇼 Kinesisk (tradisjonell) | 繁體中文 | `zh-TW` |
| 11 | 🇷🇺 Russisk | Русский | `ru` |
| 12 | 🇳🇱 Nederlandsk | Nederlands | `nl` |
| 13 | 🇸🇦 Arabisk | العربية | `ar` |
| 14 | 🇵🇱 Polsk | Polski | `pl` |
| 15 | 🇹🇷 Tyrkisk | Türkçe | `tr` |
| 16 | 🇮🇳 Hindi | हिंदी | `hi` |
| 17 | 🇮🇩 Indonesisk | Bahasa Indonesia | `id` |
| 18 | 🇻🇳 Vietnamesisk | Tiếng Việt | `vi` |
| 19 | 🇹🇭 Thai | ไทย | `th` |
| 20 | 🇸🇪 Svensk | Svenska | `sv` |
| 21 | 🇩🇰 Dansk | Dansk | `da` |
| 22 | 🇳🇴 Norsk | Norsk | `no` |
| 23 | 🇫🇮 Finsk | Suomi | `fi` |
| 24 | 🇬🇷 Gresk | Ελληνικά | `el` |
| 25 | 🇨🇿 Tsjekkisk | Čeština | `cs` |
| 26 | 🇭🇺 Ungarsk | Magyar | `hu` |
| 27 | 🇷🇴 Rumensk | Română | `ro` |
| 28 | 🇺🇦 Ukrainsk | Українська | `uk` |
| 29 | 🇧🇷 Brasiliansk portugisisk | Português Brasileiro | `pt-BR` |
| 30 | 🇭🇰 Kantonesisk | 粵語 | `zh-HK` |
| 31 | 🇲🇾 Malajisk | Bahasa Melayu | `ms` |
| 32 | 🇸🇰 Slovakisk | Slovenčina | `sk` |
| 33 | 🇧🇬 Bulgarsk | Български | `bg` |
| 34 | 🇭🇷 Kroatisk | Hrvatski | `hr` |
| 35 | 🇱🇹 Litauisk | Lietuvių | `lt` |
| 36 | 🇱🇻 Lettisk | Latviešu | `lv` |
| 37 | 🇪🇪 Estisk | Eesti | `et` |
| 38 | 🇸🇮 Slovensk | Slovenščina | `sl` |

## Slik endrer du språk

### I Chloros Desktop

1. Åpne programinnstillingene
2. Gå til menyen for språkvalg
3. Velg ønsket språk fra listen
4. Grensesnittet oppdateres umiddelbart

### I Chloros CLI

Bruk kommandoen `language` for å vise eller endre språket i CLI-grensesnittet:

```bash
# View current language
chloros-cli language

# Change to Spanish
chloros-cli language es

# Change to Chinese (Simplified)
chloros-cli language zh

# Change to Brazilian Portuguese
chloros-cli language pt-BR

# List all available languages
chloros-cli language --list
```

For mer informasjon, se [CLI-dokumentasjonen](CLI.md).

## Dekning

Alle 38 språk støttes fullt ut i:

* **Chloros Desktop** – Fullstendig oversettelse av brukergrensesnittet
* **Chloros CLI** – Kommandolinjegrensesnitt og utdatameldinger

Python, SDK, API og [referansedokumentasjonen](reference/sdk-reference.md) tilhørende disse er tilgjengelig på engelsk.

Språkstøtten sikrer at brukere over hele verden kan jobbe effektivt på sitt morsmål uten hindringer.
