# expense.io — Personal Expense Tracker

A fully offline-capable, single HTML file personal expense tracker with GitHub Gist cloud sync. No server, no backend, no subscription. Just open the URL and use it.

---

## Tech Stack

- Single `index.html` file — no framework, no build step
- Chart.js 4.4.1 (from CDN) for all charts
- Google Fonts — Syne (headings) + DM Mono (numbers/body)
- localStorage for instant offline data persistence
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
|-------|------|---------|
| localStorage | All transactions, settings, PAT, Gist ID | On your device only |
| GitHub Gist | JSON backup of all transactions | Private Gist — only you |
| GitHub Pages | The HTML file (code only, no data) | Public URL, but no data |

Your expense data never lives in the GitHub repository. The repo only contains the HTML file. Data lives in your private Gist only.

---

## Hosting

Hosted on GitHub Pages at:
```
https://adrajpt07.github.io/expense-tracker
```

Repository is public (required for free GitHub Pages) but contains zero personal data — only the app code.

---

## Sync Setup (one-time per device)

1. Go to `github.com` → Settings → Developer Settings → Personal Access Tokens → Tokens (classic)
2. Generate new token — name it `expense-tracker`, tick only the **gist** scope, set No expiration
3. Copy the `ghp_...` token and save it somewhere safe
4. Open the app URL → click **⚙ Sync Settings** → paste the token → Save
5. Click **☁ Push to GitHub** — Gist is created automatically, Gist ID auto-saves
6. On any other device: same URL → **⚙ Sync Settings** → paste same PAT + Gist ID → Pull

---

## Daily Usage

- Open the app on any device — it auto-pulls latest data on load
- Add transactions normally — auto-syncs to Gist after 3 seconds
- No manual action needed after first setup

---

## Adding to Phone Home Screen

**Android (Chrome):** 3-dot menu → Add to Home Screen

**iPhone (Safari):** Share button → Add to Home Screen

Opens fullscreen like a native app.

---

## Security

- No login system needed — PAT token acts as your private key
- Private Gist cannot be seen, edited or accessed by anyone without the token
- PAT is stored only in your browser's localStorage — never in the repo
- No third party has access to your data at any point

---

## Files in this Repo

```
index.html   — the entire application (HTML + CSS + JS, self-contained)
README.md    — this file
```

---

*Built with help from Claude (Anthropic) — spec, code, hosting, sync and deployment all done in one session.*
