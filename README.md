# Bavarchi

A shared, login-protected ledger for a two-partner food business — menu, rates, cost, sales orders, and a monthly P&L dashboard. Single self-contained file: `index.html`.

## Before you deploy — read this

This file was originally built to run inside Claude, where it used a built-in storage feature. That feature **does not exist** once you host the file yourself (Netlify, GitHub Pages, your own server, etc.).

To make it actually save data when self-hosted, this copy is wired to optionally use **Firebase Firestore** — a free database from Google. If you skip the setup below, the site will still open and look normal, but it'll silently run in "local demo mode": nothing you type is saved, and a refresh wipes it. You'll see a banner at the top of the app saying so.

**5-minute setup, no cost for this scale of use:**

1. Go to [console.firebase.google.com](https://console.firebase.google.com) and create a free project (any name).
2. In the left sidebar, open **Build → Firestore Database** → **Create database**. Choose any region close to you. Start in **test mode** for now (see the security note below).
3. In the left sidebar, click the gear icon → **Project settings**. Under **Your apps**, click the **`</>`** (web) icon to register a new web app (any nickname, no need to check "Firebase Hosting").
4. Firebase will show you a `firebaseConfig` object with your keys (`apiKey`, `authDomain`, `projectId`, etc.). Copy it.
5. Open `index.html` in a text editor, find this block near the top (search for `REPLACE_ME`), and paste your values in:
   ```js
   window.firebaseConfig = {
     apiKey: "REPLACE_ME",
     authDomain: "REPLACE_ME.firebaseapp.com",
     projectId: "REPLACE_ME",
     storageBucket: "REPLACE_ME.appspot.com",
     messagingSenderId: "REPLACE_ME",
     appId: "REPLACE_ME"
   };
   ```
6. Save the file. That's it — the app will now save real data to your Firestore database.

## Security note (please read)

Bavarchi's login is a lightweight, app-level password check (hashed client-side) — it is **not** wired to Firebase's own authentication system. That means Firestore's security rules can't tell "your partner" apart from "a stranger who found the URL" — they can only allow or block *everyone*.

For this to work at all without a lot of extra engineering, your Firestore rules need to allow open read/write access:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if true;
    }
  }
}
```

Set this under **Firestore Database → Rules** in the Firebase console. In practice this means: anyone who has both your site's URL *and* knows how to open browser dev tools could technically read or write your ledger data directly, bypassing the login screen. For a private tool you only share with your business partner, this is a reasonable trade-off — it's the same trust model the app already had. If you outgrow that (e.g. want real per-user security), the next step would be wiring up Firebase Authentication properly, which is a bigger change — ask if you want help with that later.

## Deploying

Any static hosting works, since this is one HTML file with no build step. A few free options:

**Netlify (easiest)**
1. Go to [app.netlify.com/drop](https://app.netlify.com/drop)
2. Drag the folder containing `index.html` onto the page
3. Done — you get a live URL immediately (and can add a custom domain in site settings)

**GitHub Pages**
1. Push this repo to a new GitHub repository
2. In the repo, go to **Settings → Pages**, set source to the `main` branch, root folder
3. Your site will be live at `https://<your-username>.github.io/<repo-name>/`

**Vercel**
1. Go to [vercel.com/new](https://vercel.com/new) and import this repo (or drag-drop the folder)
2. Deploy with default settings — no build command needed

## Local testing

Just open `index.html` directly in a browser, or run a tiny local server:
```
python3 -m http.server 8000
```
then visit `http://localhost:8000`. Without Firebase configured, it runs in local demo mode (data resets on refresh) — useful for checking the UI, not for real use.

## What's inside

- `index.html` — the entire app (HTML, CSS, and JS in one file)
- Data model: `users/{username}` for accounts, `workspaces/{workspaceId}/menu` and `workspaces/{workspaceId}/entries` for ledger data — `workspaceId` is `"main"` for real accounts and `"demo"` for the built-in demo login, so demo data never touches real data
- Rates and costs are snapshotted onto each sales entry at the time it's logged, so past P&L stays accurate even if you update a rate or cost later
