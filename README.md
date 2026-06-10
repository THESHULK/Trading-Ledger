# Trading Ledger

A self-contained trading journal — stocks, options, sell-side spreads, and futures — with
per-tab P/L summaries, account filters, interactive charts, and an Orangutan Army theme.

It's a **local-first Progressive Web App**: a single `index.html` plus a service worker and
icons. All trade data lives in **your own browser** (and optionally **your own Google Drive**).
There is **no backend** — nobody hosts your data but you.

---

## Features

- **Trades tab** — stocks/options/futures executions + sell-side spreads in one view, with an
  **Account** column/filter, **DTE** for options, per-trade realized P/L, and a filtered totals bar.
- **Positions** — open positions netted from your trades, with mark prices for unrealized P/L.
- **Sell-Side** — multi-leg spreads with their own filters (including Account) and totals.
- **Prop Firm** — cost/payout tracking that flows into the dashboard.
- **Dashboard** — cumulative realized P/L (with a mouse crosshair), P/L by instrument and by ticker.
- **Themes** — Orangutan Army (default), Terminal, Midnight, Slate, Solarized, Amber, Light.
- **Installable + offline** — add to your phone/desktop home screen; works without a connection.
- **Your data, your control** — browser storage by default; optional sync to *your* Google Drive;
  manual JSON/CSV import & export anytime.

---

## Run locally

Service workers and Google sign-in both require `http(s)` (and allow `localhost`):

```bash
cd Trading-Ledger
python3 -m http.server 8000
# open http://localhost:8000/
```

Opening `index.html` directly from disk (`file://`) works for the app itself, but the PWA
install and Google Drive sync will not — use a local server or deploy.

---

## Deploy (free static hosting)

The app is just static files, so any static host works. **GitHub Pages:**

1. Push this repo to GitHub.
2. Repo **Settings → Pages → Build and deployment → Source: Deploy from a branch**.
3. Branch: `main`, folder: `/ (root)` → **Save**.
4. Your app is live at `https://<your-username>.github.io/Trading-Ledger/`.

(Netlify, Vercel, and Cloudflare Pages work the same way.)

> After each update, bump `CACHE_VERSION` in `sw.js` (e.g. `ledger-v2`) so installed
> clients pick up the new version instead of serving the cached one.

---

## Google Drive sync (optional, bring-your-own-cloud)

Each user signs into **their own** Google Drive; the ledger is stored in a hidden,
app-private folder (`drive.appdata` scope). The app can only see the one file it creates —
not the rest of the user's Drive — and the data never touches any server you run.

### One-time setup: create an OAuth Client ID

1. Go to the [Google Cloud Console](https://console.cloud.google.com/) and **create a project**.
2. **APIs & Services → Library →** enable **Google Drive API**.
3. **APIs & Services → OAuth consent screen:**
   - User type: **External** → Create.
   - Fill in app name, your support email, developer email.
   - **Scopes:** add `.../auth/drive.appdata` (search "appdata").
   - **Test users:** add the Google account(s) that will use the app (up to 100 while in
     "Testing"). Test users see a one-time "Google hasn't verified this app" screen — that's
     expected; click **Continue**.
4. **APIs & Services → Credentials → Create credentials → OAuth client ID:**
   - Application type: **Web application**.
   - **Authorized JavaScript origins** — add every origin the app runs on, e.g.:
     - `http://localhost:8000`
     - `https://<your-username>.github.io`
       (origin only — no path; GitHub Pages origin is the `github.io` host)
   - Create, then **copy the Client ID** (`xxxx.apps.googleusercontent.com`).

### Wire it into the app

Open `index.html`, find:

```js
const GDRIVE_CLIENT_ID="";  // <-- paste your "...apps.googleusercontent.com" Client ID here
```

Paste your Client ID, save, redeploy. The **Data → Sync across devices** panel's
"Connect Google Drive" button will now work. Until a Client ID is set, the panel shows
"Cloud sync isn't set up for this build yet" and everything else still works locally.

### How sync behaves

- **Connect** pulls your Drive copy (if any), then keeps it updated.
- **On save**, changes are pushed to Drive (debounced ~1.5s).
- **On open / every ~2 min / "Sync now"**, it pulls and adopts the newer copy.
- **Conflict rule:** last-write-wins by timestamp (`updatedAt`) on the whole file. Fine for one
  person across devices; if you edit two devices *simultaneously while offline*, the later save wins.
- Access is per-session (Google tokens last ~1 hour and silently refresh while signed in).

> Going beyond ~100 users, or removing the "unverified app" screen, requires submitting the
> app for Google verification (the `drive.appdata` scope is "sensitive"). For personal or
> small-community use, Testing mode + test users is enough.

---

## Data & privacy

- Default storage is your browser's `localStorage` (per device).
- Optional Google Drive sync stores one JSON file in **your** Drive's hidden app folder.
- **Backups:** Data tab → *Export backup (JSON)* / *Export trades (CSV)*; restore via *Import*.
- **Never commit your data to git.** `.gitignore` excludes `*.json`, `*.csv`, `trading-ledger*`,
  and `Ledger.html` so your trade history stays off GitHub. Keep it that way.

---

## Files

| File | Purpose |
|---|---|
| `index.html` | The entire app (UI + logic, single file) |
| `sw.js` | Service worker — offline cache, network-first updates |
| `manifest.webmanifest` | PWA metadata (name, theme, icons) |
| `icon-192.png`, `icon-512.png`, `icon-maskable-512.png` | App icons |
| `.gitignore` | Keeps data files out of the repo |

🤖 Built with [Claude Code](https://claude.com/claude-code)
