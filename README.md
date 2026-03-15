# expense.io — Personal Expense Tracker

A fully offline-capable, single HTML file personal expense tracker with GitHub Gist cloud sync. No server, no backend, no subscription. Just open the URL and use it.

---

## What's New in v2

| Area | Change |
| --- | --- |
| 🔐 PAT Storage | Token is now **AES-256-GCM encrypted** before being written to `localStorage` — raw PAT is never stored on disk |
| 🔑 Passphrase unlock | On page load, a non-blocking inline banner asks for your passphrase to decrypt the token for the session |
| 🛡️ XSS protection | All user-controlled fields (merchant name, category, date) are HTML-escaped before being injected into the DOM |
| 📋 Content Security Policy | Strict CSP locks external connections to `api.github.com` only — limits blast radius of any future XSS |
| ⚠️ Token expiry guidance | Setup instructions now explicitly advise setting a **30 or 90 day expiry** on the PAT — "No expiration" is discouraged |
| 🔒 onclick hardening | Delete button indices are cast to `Number()` to prevent injection via crafted transaction data |

---

## Tech Stack

- Single `index.html` file — no framework, no build step
- Chart.js 4.4.1 (from CDN) for all charts
- Google Fonts — Syne (headings) + DM Mono (numbers/body)
- Web Crypto API (built into every modern browser) for AES-256-GCM PAT encryption
- `localStorage` for instant offline data persistence
- GitHub Pages for free hosting
- GitHub Gist API for private cloud sync

---

## Features

- Add transactions with merchant, amount, date, category
- 8 categories: Food, Recharge, Shopping, Entertainment, Transport, Health, Transfer, Other
- Delete any transaction with confirmation
- 5 tabs: Monthly, Weekly, Daily, Categories, All Transactions
- Charts: bar, line, donut — all dark themed, responsive
- Export data as JSON backup
- Import JSON backup
- GitHub Gist sync — push and pull data across all devices
- Auto-sync: every add/delete triggers a push to Gist after 3 seconds
- Auto-pull on page load — always starts with latest data
- Conflict resolution modal — merge, use remote, or keep local
- Toast notifications for every action
- Dark theme with lime/pink/teal/amber accents

---

## How Data is Stored

| Layer | What | Privacy |
| --- | --- | --- |
| `localStorage` | All transactions, Gist ID, **encrypted** PAT blob | On your device only |
| GitHub Gist | JSON backup of all transactions | Private Gist — only you |
| GitHub Pages | The HTML file (code only, no data) | Public URL, but no data |

Your expense data never lives in the GitHub repository. The repo only contains the HTML file. Data lives in your private Gist only.

The raw PAT **never** touches `localStorage`. Only an AES-256-GCM encrypted blob (salt + IV + ciphertext) is written, derived from a passphrase you choose. Without the passphrase the blob is useless.

---

## Hosting

Hosted on GitHub Pages at:

```
https://adrajput07.github.io/expense-tracker
```

The repository is public (required for free GitHub Pages) but contains zero personal data — only the app code.

---

## Sync Setup (one-time per device)

1. Go to `github.com` → Settings → Developer Settings → Personal Access Tokens → Tokens (classic)
2. Generate a new token — name it `expense-tracker`, tick only the **gist** scope
3. ⚠️ **Set an expiration of 30 or 90 days** — avoid "No expiration" to limit risk if the token leaks
4. Copy the `ghp_...` token
5. Open the app URL → click **⚙ Sync Settings** → paste the token + choose a passphrase (min 8 chars) → Save
6. Click **☁ Push to GitHub** — Gist is created automatically, Gist ID auto-saves
7. On any other device: same URL → **⚙ Sync Settings** → paste same PAT + same passphrase + Gist ID → Pull

**On each page load:** if you've previously saved encrypted settings, a small unlock banner appears at the top — enter your passphrase to activate sync. The rest of the app works fully offline while the banner is present.

---

## Migrating from v1

If you used v1 (plain PAT in `localStorage`), the app automatically clears the old raw token on first load. You'll see a message prompting you to re-enter your PAT in **⚙ Sync Settings** with a passphrase. Your transaction data is unaffected.

---

## Daily Usage

- Open the app on any device — enter your passphrase when prompted, then it auto-pulls the latest data
- Add transactions normally — auto-syncs to Gist after 3 seconds
- No manual action needed after first setup

---

## Adding to Phone Home Screen

**Android (Chrome):** 3-dot menu → Add to Home Screen

**iPhone (Safari):** Share button → Add to Home Screen

Opens fullscreen like a native app.

---

## Security

| Threat | Mitigation in v2 |
| --- | --- |
| XSS via merchant name | All user fields HTML-escaped via `esc()` before DOM insertion |
| PAT theft via `localStorage` read | PAT stored as AES-256-GCM encrypted blob; passphrase never stored |
| Token leak via long-lived PAT | Setup now requires a token expiry date (30–90 days recommended) |
| Data exfiltration via XSS | CSP restricts outbound connections to `api.github.com` only |
| Injection via delete button index | Index values cast to `Number()` before use in `onclick` |

**Remaining limitation:** client-side encryption is not a silver bullet. If an attacker can execute arbitrary JavaScript in the page (e.g. via a supply-chain attack on the CDN), they could hook the decrypt step when you type your passphrase. For very high-sensitivity use, a backend OAuth token-exchange flow would eliminate this risk entirely.

---

## Files in this Repo

```
index.html   — the entire application (HTML + CSS + JS, self-contained)
README.md    — this file
```

---

## Changelog

### v2.0 — Security hardening
- AES-256-GCM encryption of PAT via Web Crypto API + PBKDF2 (200k iterations, SHA-256)
- XSS protection: `esc()` sanitizer on all `innerHTML` insertions of user data
- Content Security Policy meta tag added
- Token expiry guidance updated (discourage "No expiration")
- Non-blocking passphrase unlock banner replaces blocking `prompt()`
- `onclick` index injection hardened with `Number()` cast
- Backwards compatible: v1 plain-text PAT automatically migrated on first load

### v1.0 — Initial release
- Single-file expense tracker with GitHub Gist sync
- Monthly / Weekly / Daily / Categories / All Transactions views
- Chart.js charts, dark theme, export/import JSON
