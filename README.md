# expense.io

A single-file personal expense tracker with GitHub Gist sync, offline-first storage, and multi-tab conflict resolution. No backend, no build step, no dependencies to install — open the HTML file and it works.

## Features

- **Add & delete transactions** — merchant, amount (₹), date, category
- **5 analytics views** — Monthly, Weekly, Daily, Categories, All Transactions
- **Charts** — bar, line, doughnut (Chart.js 4.4.1, inlined)
- **GitHub Gist sync** — push/pull across devices using a Personal Access Token
- **Auto-sync** — debounced 3-second push after every add/delete, with tab-close guard
- **Conflict resolution modal** — content-fingerprint detection with merge, keep-local, or use-remote options
- **JSON export / import** — with pre-import backup download and record-level validation
- **Offline-first** — all data in `localStorage`; Gist sync is optional
- **Up to 5,000 transactions**, max ₹99,99,999 per entry

## Quick Start

1. Download `expense_tracker_v4.html`
2. Open it in any modern browser — no server needed
3. Add transactions immediately; data saves to `localStorage` automatically

### Optional: GitHub Gist Sync

1. Go to [GitHub → Settings → Developer Settings → Personal Access Tokens → Tokens (classic)](https://github.com/settings/tokens/new)
2. Create a token with only the **gist** scope; set an expiration (30–90 days recommended)
3. Click **⚙ Sync Settings** in the app, paste the token, and save
4. Click **☁ Push to GitHub** — a private Gist is created and the ID is stored locally
5. On other devices, enter the same token + Gist ID to sync

---

## Changelog

### v4 — Security hardening & reliability (current)

**Security fixes:**

| # | Issue | Fix |
|---|-------|-----|
| S1 | Chart.js loaded from CDN with no integrity check | Inlined Chart.js 4.4.1 directly from the npm package — no CDN request at all |
| S2 | CSP `<meta>` tag missing `object-src`, `form-action`, `base-uri` | Added all three; `cdnjs.cloudflare.com` removed from allowlist since CDN is no longer used |
| S3 | Gist ID stored in `localStorage` permanently with no rotation mechanism | Added **🔄 Rotate Gist ID** button — creates a fresh private Gist and orphans the old ID |

**Bug fixes:**

| # | Issue | Fix |
|---|-------|-----|
| B1 | Auto-sync race condition: tab closed within 3s window → data never pushed | `beforeunload` listener cancels the debounce timer and fires `fetch(..., { keepalive: true })`, which browsers complete even after page unload |
| B2 | GitHub API rate limit (5,000 req/hour) hit silently with no user feedback | `gistRequest()` inspects `X-RateLimit-Remaining` and `X-RateLimit-Reset` headers; 403/429 with zero remaining surfaces a human-readable toast with the reset time |

**Note on CDN inlining:** The browser's SRI check compares the hash of the exact bytes served by the CDN. The cdnjs-hosted minified build differs from the npm package build, making a correct pre-computed hash impractical to maintain. Inlining eliminates the entire CDN supply-chain risk — no external script request means no opportunity for CDN compromise or URL hijacking.

---

### v3 — XSS protection, PAT security, smarter conflict detection

**Security fixes:**

| # | Issue | Fix |
|---|-------|-----|
| S1 | PAT stored in `localStorage` — persists after tab close | Moved PAT to `sessionStorage` (key `expense_tracker_v3_pat`) — auto-cleared when the tab closes |
| S2 | All dynamic content injected via `innerHTML` without escaping | Universal `esc()` function applied to every dynamic value inserted into the DOM |
| S3 | No Content Security Policy | Added `<meta http-equiv="Content-Security-Policy">` tag |

**Bug fixes:**

| # | Issue | Fix |
|---|-------|-----|
| B1 | Conflict detection compared only transaction counts — delete-one/add-one went undetected | Replaced count comparison with a content fingerprint: each transaction serialised as `date\|merchant\|amount\|cat`, array sorted and joined, then compared as a string |
| B2 | `isSyncing` flag not reset on conflict → push/pull buttons deadlocked | Flag reset explicitly before the conflict modal opens |
| B3 | Import used `window.confirm()` with no undo | Replaced with a preview modal showing the first 10 records; pre-import data auto-exported as a timestamped backup file |
| B4 | No amount validation — NaN/0/negative/Infinity could corrupt Chart.js | `addTransaction()` validates `amount > 0`, `amount <= 9999999`, `!isNaN(amount)`; same rules applied in `validateTransaction()` for remote and imported data |
| B5 | Stale index bug: delete button captured index at render time; list could shift before confirm | Delete opens a modal with the captured index; re-validates index is still in-bounds before splicing |

---

### v2 — GitHub Gist sync, multi-tab support

- GitHub Gist push/pull via Personal Access Token
- Auto-sync (3-second debounce after add/delete)
- Basic conflict detection (count-based — later improved in v3)
- Sync status indicator (dot + label in header)
- `<meta>` CSP tag added
- JSON export and import (direct replace, no preview)
- PAT stored in `localStorage` (later fixed in v3)

---

### v1 — Initial release

- Core transaction entry: merchant, amount, date, category
- `localStorage` persistence
- Monthly, Weekly, Daily, Categories, All Transactions tabs
- Chart.js charts: monthly bar, trend line, daily bar, category doughnut
- Top 10 merchants bar list
- JSON export
- No sync, no input validation, no XSS protection

---

## Security Model

| Concern | Mitigation |
|---------|-----------|
| XSS via merchant/category names | All dynamic DOM insertions pass through `esc()` (HTML entity encoding) |
| PAT exposure in storage | PAT lives in `sessionStorage` only — cleared automatically on tab/browser close |
| PAT exposure via script injection | Chart.js inlined; no external scripts loaded at runtime |
| Gist ID exposure (long-lived) | **🔄 Rotate Gist ID** creates a new Gist and abandons the old ID |
| Malicious import file | Every record validated through `validateTransaction()` before any data is touched; invalid records counted and skipped |
| CDN supply-chain attack | Eliminated — Chart.js bundled directly, no CDN requests |
| GitHub API abuse / rate limits | 5,000 req/hour limit detected via response headers; user notified with reset time |
| Tab-close data loss | `beforeunload` + `fetch keepalive` flushes any pending sync before unload |
| Plugin/object injection | CSP `object-src 'none'` |
| `<base>` tag hijacking | CSP `base-uri 'self'` |
| Form action redirection | CSP `form-action 'none'` |

**Remaining known limitation:** The `<meta>` CSP cannot enforce HTTP-header-level restrictions (e.g. it cannot block navigation-level attacks). A proper deployment on a server with `Content-Security-Policy` HTTP headers would be stronger. On GitHub Pages, HTTP headers are not configurable.

---

## Data Format

Transactions are stored in `localStorage` under the key `expense_tracker_v4_data` as a JSON array:

```json
[
  {
    "merchant": "Swiggy",
    "amount": 349.00,
    "date": "2026-03-15",
    "cat": "Food"
  }
]
```

When pushed to Gist, the file `expense_tracker_data.json` wraps the array in a metadata envelope:

```json
{
  "transactions": [...],
  "meta": {
    "pushedAt": "2026-03-15T17:30:00.000Z",
    "count": 42,
    "version": "v4"
  }
}
```

Valid categories: `Food`, `Recharge`, `Shopping`, `Entertainment`, `Transport`, `Health`, `Transfer`, `Other`

---

## Browser Support

Any modern browser (Chrome 89+, Firefox 90+, Safari 15+, Edge 89+). Requires `localStorage`, `sessionStorage`, `fetch`, and `keepalive` fetch support for the tab-close sync guard.
