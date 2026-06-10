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

This is free and takes ~10 minutes. You're creating a Google Cloud project, turning on the
Drive API, telling Google what your app is allowed to ask for (the consent screen), and
generating a **Client ID** the app uses to sign users in. Do this once.

> **Note on the console UI:** Google recently moved these settings under
> **APIs & Services → "Google Auth Platform"** (sections: *Overview, Branding, Audience,
> Clients, Data Access*). Older accounts still show **"OAuth consent screen"** and
> **"Credentials"** directly. The steps below name both so you can follow either layout.

#### Step 1 — Create a project
1. Open the [Google Cloud Console](https://console.cloud.google.com/).
2. Top bar: click the **project dropdown** (left of the search box) → **New project**.
3. Name it (e.g. `trading-ledger`) → **Create**. Wait for it to finish, then make sure that
   project is selected in the dropdown for every step below.

#### Step 2 — Enable the Drive API
1. Left menu (☰) → **APIs & Services → Library**.
2. Search **"Google Drive API"** → click it → **Enable**.

#### Step 3 — Configure the consent screen
Left menu → **APIs & Services → OAuth consent screen** (or **Google Auth Platform**).

1. **Get started / Branding:**
   - **App name:** e.g. `Trading Ledger`
   - **User support email:** your email
   - **Audience / User type:** **External**
   - **Developer contact email:** your email
   - Save / Continue through the steps until the app is created.
2. **Audience** (or "Publishing status" on the old UI):
   - Leave **Publishing status = Testing**.
   - Under **Test users → + Add users**, add every Google account that will use the app
     (your own first). **Only these accounts can connect** until you publish/verify.
     Up to 100 test users.
3. **Data Access** (or "Scopes" on the old UI):
   - Click **Add or remove scopes**.
   - In the filter box type **`drive.appdata`**, check
     **`.../auth/drive.appdata`** ("View and manage its own configuration data in your
     Google Drive"), → **Update** → **Save**.
   - *Do not* add broader Drive scopes — `appdata` is all the app needs and keeps it
     limited to its own hidden file.

#### Step 4 — Create the Client ID
Left menu → **APIs & Services → Credentials** (or **Google Auth Platform → Clients**).

1. **+ Create credentials → OAuth client ID**.
2. **Application type:** **Web application**.
3. **Name:** anything, e.g. `ledger-web`.
4. **Authorized JavaScript origins → + Add URI.** Add one entry per place the app runs.
   This must be the **origin only** — scheme + host (+ port), **no path, no trailing slash:**

   | Where you run it | Add this exact value |
   |---|---|
   | Local testing | `http://localhost:8000` |
   | GitHub Pages | `https://YOUR-USERNAME.github.io` |
   | Custom domain | `https://your-domain.com` |

   ✅ `https://janedoe.github.io`  ❌ `https://janedoe.github.io/Trading-Ledger/`
   (the `/Trading-Ledger/` path is **not** part of the origin — don't include it)
5. **Authorized redirect URIs:** leave **empty**. This app uses the token (popup) flow, so
   redirect URIs are not needed.
6. **Create** → a dialog shows your **Client ID** like
   `123456789-abcd….apps.googleusercontent.com`. **Copy it.** (You can reopen it anytime
   from the Credentials list.)

#### Step 5 — Put the Client ID in the app

You create **one** Client ID and bake it into the app — it's shared by everyone. The Client ID
is **not a secret** (it's safe in public code); it just identifies *the app* to Google and only
works for the origins you authorized in Step 4. Each user signs into their own Google account
and syncs to their own Drive — they never need a Client ID of their own.

1. Open `index.html`, find this line near the top of the `GOOGLE DRIVE SYNC` section:

   ```js
   const GDRIVE_CLIENT_ID="";  // <-- paste your "...apps.googleusercontent.com" Client ID here
   ```
2. Paste your Client ID between the quotes:

   ```js
   const GDRIVE_CLIENT_ID="123456789-abcd….apps.googleusercontent.com";
   ```
3. Save, **bump `CACHE_VERSION` in `sw.js`** (so installed clients refresh), and redeploy
   (or just reload on `localhost`).

Until the Client ID is set, the **Data → Sync across devices** panel shows
"Cloud sync isn't set up for this build yet" and the rest of the app works locally as normal.

#### Step 6 — Verify it works
1. Open the app over `http(s)` (e.g. `http://localhost:8000`, **not** `file://`).
2. Go to **Data → Sync across devices → Connect Google Drive**.
3. A Google popup appears → pick your account → you'll see **"Google hasn't verified this
   app"** (expected while in Testing) → **Continue** → grant access.
4. The status line should switch to **"● Synced to your Google Drive."** Add a trade, then
   open the app on another device (signed into the same Google account) and **Sync now** —
   your trades appear.

#### Troubleshooting

| Symptom | Fix |
|---|---|
| Panel says "Cloud sync isn't set up" | `GDRIVE_CLIENT_ID` is still empty in `index.html` — set it (Step 5) and redeploy. |
| **Connect** does nothing | You're on `file://`, or Google sign-in is still loading. Serve over http(s) and retry. |
| Popup error: *"origin is not allowed for the given client ID"* / `idpiframe_initialization_failed` | The page's origin isn't in **Authorized JavaScript origins**. Add the exact origin (no path/slash). Changes can take a few minutes. |
| *"Access blocked: app is being tested" / "hasn't completed verification"* | Add that Google account under **Test users** (Step 3.2). |
| *"Access blocked: this app is blocked"* | The `drive.appdata` scope isn't added, or you signed in with a non-test user. Check Step 3.2–3.3. |
| Connects but data doesn't sync | Confirm the **Drive API** is enabled (Step 2) and you granted access when prompted. Use **Sync now** and watch the status line. |

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
