# UTM Builder – CLAUDE.md

## Co tento projekt je

Interní single-page appka pro marketingový tým Zoner Studio. Generuje správně naformátované UTM parametry podle firemní dokumentace.

Celá logika žije v **jednom souboru: `index.html`** (inline HTML + CSS + JS).

---

## Struktura kódu v index.html

| Sekce (komentář) | Co obsahuje |
|---|---|
| `DATA` | Objekt `RULES` – pravidla pro každý medium |
| `STAV` | Proměnné `curMedium`, `appLinkType` |
| `SANITIZACE` | Funkce `sanitize()`, `sanitizeEl()`, `showToast()` |
| `LOCALSTORAGE` | Paměť partnerů pro affiliate/sponsored_content/referral |
| `RENDER` | Funkce `renderDynamic()` – kreslí pole podle medium |
| `BUILD URL` | Funkce `buildURL()` – sestavuje URL |
| `AKTUALIZACE` | Funkce `updateURL()` – aktualizuje výstup |
| `KOPÍROVAT` | Event listener na tlačítku Kopírovat |

---

## Pravidla UTM parametrů

Jsou zakódována v objektu `RULES` v `index.html`. Zdrojová dokumentace: `UTM parametry - úravidla a dokumentace.pdf`.

### Kanály a jejich logika

| Medium | Source | Campaign | Poznámka |
|---|---|---|---|
| `application` | `ZPS` (fixed) | volitelná / povinná | Toggle: Marketingová zpráva vs. Standardní odkaz |
| `cpc` | select: google/facebook/instagram/sklik/bing | auto (disabled) | Generuje reklamný systém |
| `social` | select: facebook/instagram/youtube/linkedin/tiktok/x | volitelná | — |
| `affiliate` | free-text + localStorage | volitelná | Partner/ambasador |
| `sponsored_content` | free-text + localStorage | volitelná | Web/médium |
| `referral` | free-text + localStorage | volitelná | Web/zdroj |
| `owned_media` | select: magaziny/napoveda | **povinná** (slug) | — |

---

## Co NELZE měnit bez konzultace

- Hodnoty `medium` – mailing záměrně chybí (generuje CRM automaticky)
- Hodnoty `source` pro fixed/select kanály (ZPS, google, facebook atd.)
- Logika sanitizace – zlatá pravidla GA4 (lowercase, no diacritics, spaces→_)
- Pořadí parametrů v URL: `utm_source`, `utm_medium`, `utm_campaign`, `utm_content`

---

## Jak přidat nový medium

1. Přidat záznam do objektu `RULES` v `index.html`
2. Přidat `<option>` do `<select id="medium">`
3. V `renderDynamic()` funguje automaticky přes `RULES` objekt – pokud source type je `fixed`, `select` nebo `memory` a campaign type je `disabled`, `optional`, `required` nebo `app_toggle`
4. Otestovat: vybrat medium, zkontrolovat vygenerovanou URL

---

## Jak funguje localStorage pro partnery

- Klíče: `utm_sources_affiliate`, `utm_sources_sponsored_content`, `utm_sources_referral`
- Ukládá se automaticky při kliknutí "Kopírovat URL" (pokud je source nový)
- Hodnoty se zobrazují jako chips pod polem + jako datalist autocomplete
- Mazání: kliknutím na × u každého chipu

---

## Sanitizace vstupů

Funkce `sanitize(v)` aplikuje zlatá pravidla GA4:
1. Převod na malá písmena
2. Odstranění diakritiky (Unicode NFD normalization)
3. Mezery → podtržítko `_`
4. Odstranění všech ostatních znaků kromě `a-z 0-9 _ - .`

Probíhá live při každém `oninput` eventu + zobrazí toast upozornění.

---

## Testovací scénáře

```
1. application / Marketingová zpráva → ?utm_source=ZPS&utm_medium=application&utm_campaign=...
2. application / Standardní odkaz   → ?utm_source=ZPS&utm_medium=application  (bez campaign)
3. cpc / google                      → campaign pole disabled, v URL není
4. social / instagram                → ?utm_source=instagram&utm_medium=social
5. owned_media / napoveda            → ?utm_source=napoveda&utm_medium=owned_media&utm_campaign=faq-platby
6. affiliate – nový partner          → uloží se, navržen při příštím otevření
7. Sanitizace "Zoner Studio"         → auto-oprava na "zoner_studio" + toast
8. Prázdná URL                       → tlačítko Kopírovat je disabled
```
