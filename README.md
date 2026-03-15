# expense.io — Personal Expense Tracker `v3`

A fully offline-capable, single HTML file personal expense tracker with GitHub Gist cloud sync. No server, no backend, no subscription. Just open the URL and use it.

> **v3** is a security-hardened rewrite. If you're on v1 or v2, see the [migration note](#migrating-from-v1--v2) below.

---

## Tech Stack

- Single `index.html` file — no framework, no build step
- Chart.js 4.4.1 (from CDN) for all charts
- Google Fonts — Syne (headings) + DM Mono (numbers/body)
- `localStorage` for transaction data persistence
- `sessionStorage` for PAT (cleared automatically on tab close)
- GitHub Pages for free hosting
- GitHub Gist API for private cloud sync

---

## Features

- Add transactions with merchant, amount, date, and category
- 8 categories: Food, Recharge, Shopping, Entertainment, Transport, Health, Transfer, Other
- Delete any transaction with confirmation modal (no accidental deletes)
- 5 tabs: Monthly, Weekly, Daily, Categories, All Transactions
- Charts: bar, line, donut — all dark themed, responsive
- Export data as JSON backup
- Import JSON backup with preview modal + auto-backup before replacing
- GitHub Gist sync — push and pull data across all devices
- Auto-sync: every add/delete triggers a push to Gist after 3 seconds
- Auto-pull on page load — always starts with latest data
- Conflict resolution modal — merge, use remote, or keep local
- Toast notifications for every action
- Dark theme with lime / pink / teal / amber accents
- Enter key submits the add form from any field

---

## Security (v3)

v3 fixes all known security vulnerabilities from v1/v2:

| Fix | What changed |
|-----|-------------|
| **PAT storage** | Token is stored in `sessionStorage` only — it is automatically wiped when the browser tab is closed. It never touches `localStorage` or the repo. |
| **XSS protection** | Every piece of user-supplied data (merchant names, dates, categories) is HTML-escaped via `esc()` before being inserted into the DOM. Malicious merchant names like `<script>` are rendered as plain text, not executed. |
| **Input validation** | All transaction fields are validated on add and on import — type, length, range, and category are all checked. Invalid records are rejected with a count shown to the user. |
| **Import safety** | Import shows a preview modal before replacing data, and automatically downloads a timestamped backup of current data before overwriting. |
| **Sync deadlock fix** | `isSyncing` is now always reset correctly, including when the conflict modal is shown. Push/Pull buttons can never get stuck disabled. |
| **Conflict detection** | Conflict detection now uses a content fingerprint (sorted hash of all transactions) instead of just comparing counts. Identical data across devices no longer triggers a false conflict. |
| **Consistent formatting** | All amounts use a single `fmt()` function throughout — no more rounded totals in summaries vs. decimal amounts in tables. |
| **Tooltip callbacks** | Chart options are built fresh per-render via `makeBaseOpts()` — the old `JSON.parse/JSON.stringify` clone silently stripped tooltip formatting functions. |
| **PAT expiry guidance** | Setup instructions now explicitly warn against "No expiration" tokens. 30–90 day expiry is recommended. |
| **Delete UX** | Deletes now use a modal instead of `confirm()`, which is suppressible in some browsers. |
| **Content Security Policy** | A `Content-Security-Policy` meta header restricts which origins can load scripts, styles, and make network requests. |

---

## How Data is Stored

| Layer | What | Privacy |
|-------|------|---------|
| `localStorage` | Transactions, Gist ID | On your device only |
| `sessionStorage` | GitHub PAT | Cleared when tab closes |
| GitHub Gist | JSON backup of all transactions | Private Gist — only you |
| GitHub Pages | The HTML file (code only, no data) | Public URL, no personal data |

Your expense data never lives in the GitHub repository. The repo only contains the HTML file. Data lives in your private Gist only.

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
3. ⚠️ **Set an expiration of 30–90 days** — never use "No expiration"
4. Copy the `ghp_...` token and keep it somewhere safe
5. Open the app → click **⚙ Sync Settings** → paste the token → Save
6. Click **☁ Push to GitHub** — Gist is created automatically, Gist ID auto-saves
7. On any other device: same URL → **⚙ Sync Settings** → paste same PAT + Gist ID → Pull

> The PAT is kept in `sessionStorage` and cleared when you close the tab. You'll need to re-enter it each session. This is intentional — it means the token is never persistently exposed in browser storage.

---

## Daily Usage

- Open the app on any device — it auto-pulls latest data on load (if PAT is set for the session)
- Add transactions normally — auto-syncs to Gist 3 seconds after each change
- No manual action needed after daily setup

---

## Import / Export

**Export:** Clicking **⬇ Export JSON** downloads a JSON file of all transactions to your device.

**Import:** Clicking **⬆ Import JSON** opens a file picker. After selecting a file, v3 will:
1. Validate every record in the file (invalid rows are skipped and counted)
2. Show a preview of the first 10 transactions
3. **Automatically download a backup** of your current data before replacing
4. Only replace data after you confirm in the modal

---

## Migrating from v1 / v2

Your transaction data stored in `localStorage` under the key `expense_tracker_v1` is **not automatically migrated**. To keep your data:

1. Open your existing v1/v2 app
2. Click **⬇ Export JSON** to download a backup
3. Open the v3 app
4. Click **⬆ Import JSON** and select the backup file
5. Confirm in the preview modal

v3 stores data under the key `expense_tracker_v3_data` and the Gist ID under `expense_tracker_v3_gist_id`, so v1/v2 and v3 can coexist in the same browser without interfering.

---

## Adding to Phone Home Screen

**Android (Chrome):** 3-dot menu → Add to Home Screen

**iPhone (Safari):** Share button → Add to Home Screen

Opens fullscreen like a native app.

---

## Files in This Repo

```
index.html   — the entire application (HTML + CSS + JS, self-contained)
README.md    — this file
```

---

## Changelog

### v3 (current)
- 🔒 PAT moved to `sessionStorage` (never persists beyond tab close)
- 🔒 Full XSS protection via `esc()` on all dynamic HTML
- 🔒 Import validation — every field type/range/category checked
- 🐛 Fixed `isSyncing` deadlock after conflict modal
- 🐛 Fixed false-positive conflict detection (now uses content fingerprint)
- 🐛 Fixed chart tooltip functions being silently stripped by `JSON.clone`
- 🐛 Fixed inconsistent amount formatting across views
- ✨ Delete confirmation now uses a modal instead of `confirm()`
- ✨ Import now shows preview + auto-downloads backup before replacing
- ✨ Weekly tab now has delete buttons (consistent with All Transactions)
- ✨ PAT show/hide toggle in settings
- ✨ Clear Token button to wipe credentials on demand
- ✨ Enter key submits add form
- ✨ Content Security Policy header added

### v2
- AES-256 PAT encryption with passphrase (superseded by sessionStorage approach in v3)
- XSS escaping added in weekly/daily/all views (incomplete — v3 completes this)

### v1
- Initial release
