# UTM Builder – CLAUDE.md

## What this project is

Internal single-page app for the Zoner Studio marketing team. Generates correctly
formatted UTM parameter URLs according to company documentation.

**All logic lives in one file: `index.html`** (inline HTML + CSS + JS, no build step).

**Live URL:** `https://matuslong.github.io/utm-builder/`  
**Repo:** `https://github.com/matuslong/utm-builder`  
**Docs:** Google Doc with UTM rules (linked in the app header)

---

## Code structure in index.html

| Section (comment)      | Contents |
|------------------------|----------|
| `DATA`                 | `RULES` object (channel logic), `TIPS` object (tooltip texts), `SHEETS_URL` constant |
| `STAV`                 | `curMedium`, `appLinkType` variables |
| `SANITIZACE`           | `sanitize()`, `sanitizeEl()`, `showToast()` |
| `STAV A SHEETS API`    | `state` object, `sheetsApi()`, `loadState()`, saved-sources helpers |
| `HELPERS`              | `esc()`, `getVal()`, `tipHtml()` |
| `RENDER SKELETON`      | `renderSkeleton()` – placeholder fields before medium is selected |
| `RENDER DYNAMIC`       | `renderDynamic()` – renders fields based on selected medium |
| `BUILD URL`            | `buildURL()` – assembles the final URL |
| `AKTUALIZACE`          | `updateURL()` – updates output display + CMS info box |
| `CHANGELOG – Sheets`   | `addToLog()`, `deleteLogEntry()`, `clearLog()`, `renderChangelog()` |
| `COPY`                 | btn-copy event listener |
| `INICIALIZACE`         | Event listeners, `renderSkeleton()`, `loadState()` |

---

## UTM channels and their logic (RULES object)

| Medium              | Source type                                          | Campaign      | Notes |
|---------------------|------------------------------------------------------|---------------|-------|
| `application`       | fixed: `ZPS`                                         | app_toggle    | Toggle: marketing message (campaign required) vs. standard link (no campaign) |
| `cpc`               | select: google/facebook/instagram/sklik/bing         | disabled      | Generated automatically by the ad platform |
| `social`            | select: facebook/instagram/youtube/linkedin/tiktok/x | optional      | — |
| `affiliate`         | memory (free-text, saved cross-device)               | optional      | Partner / ambassador |
| `sponsored_content` | memory (free-text, saved cross-device)               | optional      | Sponsored article web |
| `referral`          | memory (free-text, saved cross-device)               | optional      | Organic mention source |
| `owned_media`       | select: magaziny/napoveda                            | **required**  | Slug/section name |

**`utm_content`** is always optional and present for all channels.

---

## Do NOT change without consultation

- `medium` values – `mailing` intentionally missing (CRM generates it automatically)
- `source` values for fixed/select channels (ZPS, google, facebook, etc.)
- Sanitization logic – GA4 golden rules (lowercase, no diacritics, spaces→`_`)
- Parameter order in URL: `utm_source` → `utm_medium` → `utm_campaign` → `utm_content`

---

## Google Sheets integration

### Architecture
The app uses **Google Apps Script** as a serverless backend for cross-device persistence.
All data (history + saved sources) lives in Google Sheets, not in the browser.

- **Spreadsheet:** `https://docs.google.com/spreadsheets/d/11g0P-9ExTMSRMJLe_U3M9J4gy22T7VOsvZhZ_hK9sho/`
- **Apps Script web app URL:** stored in `const SHEETS_URL` at the top of the JS section
- **Two sheet tabs:**
  - `Historia` – columns: id | date | time | url | source | medium | campaign | content | fidl | note
  - `Zdroje` – columns: medium | value (saved partner sources)

### API actions (GET params)
| `action`           | Description |
|--------------------|-------------|
| `read`             | Returns `{ history: [...], sources: {...} }` JSON on startup |
| `write_history`    | Appends a history entry (deduplicates by URL) |
| `delete_history`   | Deletes a row by `id` |
| `clear_history`    | Deletes all data rows in Historia |
| `save_source`      | Adds a medium+value pair to Zdroje |
| `delete_source`    | Removes a medium+value pair from Zdroje |

### In-memory state
```javascript
const state = {
  loaded:  false,                         // false until loadState() resolves
  history: [],                            // array of entry objects, newest first
  sources: { affiliate: [], sponsored_content: [], referral: [] },
};
```
On startup, `loadState()` fetches from Sheets and populates `state`. All subsequent
reads are from `state` (synchronous). Writes update `state` immediately and call
`sheetsApi()` asynchronously.

### Redeploying Apps Script
After changing Apps Script code: **Deploy → Manage deployments → Edit (pencil) →
Version: New version → Deploy**. The URL stays the same.

---

## How to add a new medium

1. Add an entry to `RULES` in `index.html`
2. Add an `<option>` to `<select id="medium">` — format: `Label (medium_value)`
3. Add entries to `TIPS.src[medium]` and `TIPS.camp[medium]` (text + ex for tooltip)
4. `renderDynamic()` handles it automatically via `RULES` — source types: `fixed`, `select`, `memory`; campaign types: `disabled`, `optional`, `required`, `app_toggle`
5. Test: select the medium, check generated URL and tooltip examples

---

## Input sanitization

`sanitize(v)` applies GA4 golden rules:
1. Lowercase
2. Remove diacritics (Unicode NFD normalization)
3. Spaces → underscore `_`
4. Remove all characters except `a-z 0-9 _ - .`

Runs live on every `oninput` event. Shows a toast warning when a value was modified.

---

## Field value persistence across medium changes

`renderDynamic()` saves current `utm-source`, `utm-campaign`, `utm-content` values
before re-rendering, then restores them after:
- `source`: restored if new medium uses a text input (`memory` type) or if the value
  exists in a `select`'s options; skipped for `fixed` type
- `campaign`: restored if new field is not disabled
- `content`: always restored

---

## Deployment

- Hosted on **GitHub Pages** (branch: `main`, root `/`)
- Auto-deploys on every `git push origin main` (~1 min delay)
- No build step – `index.html` is served directly

---

## Test scenarios

```
1.  application / Marketingová zpráva  → ?utm_source=ZPS&utm_medium=application&utm_campaign=...
2.  application / Standardní odkaz    → ?utm_source=ZPS&utm_medium=application (no campaign)
3.  cpc / google                       → campaign field disabled, absent from URL
4.  social / instagram                 → ?utm_source=instagram&utm_medium=social
5.  owned_media / napoveda             → ?utm_source=napoveda&utm_medium=owned_media&utm_campaign=faq-platby
6.  affiliate – new partner            → saved to Sheets Zdroje, suggested on next load on any device
7.  Sanitize "Zoner Studio"            → auto-corrected to "zoner_studio" + toast
8.  Empty URL                          → Kopírovat button is disabled
9.  social – source tooltip            → example "utm_source=instagram nebo linkedin"
10. owned_media – campaign tooltip     → example "utm_campaign=faq-platby nebo magazin-tipy-portret"
11. Valid URL → CMS info box visible; delete URL → info box hidden
12. FIDL placeholder                   → "napr. 2026-04-cpc-en-meta-rvw-sk"
13. Changelog mobile (<520px)          → 2-row layout, no URL overflow
14. Switch medium with filled fields   → source/campaign/content values preserved
15. Delete history entry               → removed from UI and from Sheets Historia tab
16. Open on second device              → same history and saved sources visible
```
