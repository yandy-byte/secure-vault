# SECURE – iPhone-style Password Vault

A single-file web app (`index.html`, plain HTML/CSS/JS, no build step) styled like an iOS app, meant for iPhone Safari and "Add to Home Screen".

**This repository contains no data.** The app ships empty. Everything you store stays only in your phone's browser storage (`localStorage`) on that one device — nothing is sent to a server, and nothing is in this repo.

## Features
- **Google sign-in (step 1)** – official Google Identity Services button. Only `yandy@romanprovisions.com` is accepted (your Google account's 2-Step Verification applies at Google's sign-in). Any other account gets "This Google account isn't allowed."
- **PIN lock (step 2)** – on first launch you create a 4–8 digit PIN (enter + confirm). Only a salted SHA-256 hash of the PIN is stored (Web Crypto). Repeated wrong PINs trigger a temporary lockout.
- **Auto-lock** whenever the app is backgrounded or the tab is hidden. After any lock, both Google sign-in and the PIN are required again.
- **Import** a SECURE backup file or a Keeper JSON export (merged, duplicates skipped).
- **Backup** – export all entries as a JSON file.
- Search, categories (Work, Banking & Cards, PINs & Alarms, Personal), tap-to-copy username / password / notes.
- "Forgot PIN?" erases the vault on this device so you can start over (restore from a backup afterwards).

## Sign-in flow
1. **Google sign-in.** Tap the Google button and choose the account (auto sign-in is off, so it asks every time). The app checks the returned ID token: issuer is Google, audience is this app's client ID, not expired, email verified, email equals `ALLOWED_EMAIL`, and the `hd` domain (if present) is `romanprovisions.com`.
2. **PIN.** First time: create a 4–8 digit PIN. After that: enter it to unlock.
3. Lock / backgrounding forgets the Google session (it is kept only in memory, never saved) and returns to step 1.

Without an internet connection the Google step can't load, so the vault stays locked ("Google sign-in needs an internet connection").

### One-time setup: Google client ID
1. In [Google Cloud Console](https://console.cloud.google.com/apis/credentials) create an **OAuth client ID** of type **Web application** (configure the OAuth consent screen first if asked).
2. Under **Authorized JavaScript origins** add your Pages origin, e.g. `https://<your-username>.github.io` (no path). Add `http://localhost` too if you test locally.
3. Copy the client ID and paste it into `index.html`: `const GOOGLE_CLIENT_ID = '...';` (the client ID is not a secret).

## Security notes (please read)
- **The Google check runs in the browser.** This is a static site with no server, so the ID token is checked by the page's own JavaScript (its signature is not verified server-side). It stops anyone who doesn't have your Google account from getting past the app's lock screen on the phone, but someone with access to the phone's browser storage or developer tools could bypass the app entirely — the Google step is a gate on the app, not encryption of the data.
- Likewise the PIN gates the app's screen; entries are stored **unencrypted** in the browser's localStorage. Anyone with access to the unlocked phone/browser profile (or its backups) could read them. A short numeric PIN is also easy to brute-force against its hash. For high-value secrets, a dedicated password manager (Keeper, Apple Passwords, etc.) is stronger.
- Backup files you export are plain JSON containing your passwords. Store them carefully and **never commit them to this repo**.
- Clearing Safari website data deletes the vault — keep a current backup.

## Host on GitHub Pages
1. Push `index.html` (and this README) to the repository root.
2. GitHub → **Settings → Pages** → Source: *Deploy from a branch* → branch `main`, folder `/ (root)` → **Save**.
3. After a minute the site is live at `https://<your-username>.github.io/<repo-name>/`.

GitHub Pages on a **private** repo requires a paid GitHub plan (Pro/Team/Enterprise). A **public** repo works on the free plan and is fine here, because the repo holds only code — no credentials.

The page must be served over HTTPS (GitHub Pages is), since the PIN hashing uses Web Crypto.

## Add to Home Screen (iPhone)
Open the Pages URL in **Safari** → tap **Share** → **Add to Home Screen** → **Add**. It launches full-screen like an app.

Note: Safari and the Home Screen app keep **separate** storage. Set your PIN and import your data from the Home Screen app itself. Google sign-in opens a Google window; if it doesn't return to the app in Home Screen mode on your iPhone, use the site in Safari instead.

## Import a Keeper export
1. In Keeper (desktop/web vault): **Settings → Export → JSON**, and save the `.json` file. Get the file onto your iPhone (e.g. AirDrop to Files — avoid emailing or uploading it anywhere).
2. Open SECURE, unlock, tap **Import**, and choose the file.

Keeper records (`title`, `login`, `password`, `login_url`, `notes`, `custom_fields`, `folders`) are converted to SECURE entries: custom fields are appended to the notes, and each entry gets a category from its Keeper folder name or, failing that, keywords in its title. A SECURE backup file (from the **Backup** button) can be imported the same way.

After importing, delete the export file from your phone.
