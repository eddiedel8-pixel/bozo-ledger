# The Bozo Ledger — Netlify + Firebase Setup

This package runs the Bozo Ledger as a fully independent website: your own
domain (or a free Netlify one), a real shared database everyone's phone
reads and writes to, and the AI slip-reading still working — routed through
a small serverless function so your API key never sits in the browser.

There are four things to set up, in this order. None of them require
coding — it's mostly clicking through free signup flows and copying values
into two files.

**Time estimate:** 20–30 minutes the first time.

---

## What's in this folder

```
bozo-ledger/
  index.html                    ← the app itself
  firebase-config.js            ← you'll paste your Firebase keys here
  netlify.toml                  ← tells Netlify where the function lives
  netlify/functions/read-slip.js ← proxies AI requests, keeps your API key secret
  README.md                     ← this file
```

---

## 1. Firebase — the shared database (free)

This is what makes everyone's phone see the same live ledger.

1. Go to **[console.firebase.google.com](https://console.firebase.google.com)** and sign in with any Google account.
2. Click **Add project**. Name it anything (e.g. "bozo-ledger"). You can decline Google Analytics — not needed.
3. Once the project's created, click the **Firestore Database** item in the left sidebar → **Create database**.
   - Choose any region close to you.
   - Start in **production mode** (we'll set our own rules next).
4. Go to the **Rules** tab inside Firestore and replace the contents with:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /bozoLedger/{document=**} {
         allow read, write: if true;
       }
       match /bozoLedgerPhotos/{document=**} {
         allow read, write: if true;
       }
     }
   }
   ```

   Click **Publish**.

   **Honest security note:** this app has no login system beyond the "who
   are you" name picker, so these rules leave the database open to anyone
   who has both your site's URL and enough know-how to find the Firebase
   project behind it. That's a low bar to worry about for a 5-person bet
   pool with no sensitive personal data in it, but it's not bank-grade
   security — don't put anything more sensitive in here than what's
   already in this app.

5. Back in the project overview (gear icon → **Project settings**), scroll
   to **Your apps** and click the **Web** icon (`</>`) to register a new
   web app. Name it anything, skip Firebase Hosting.
6. You'll see a code block with a `firebaseConfig` object — copy those
   values into **`firebase-config.js`** in this folder, replacing every
   `"REPLACE_ME"`.

---

## 2. Anthropic API key — for the AI slip-reading (costs a few cents per use)

This is separate from your claude.ai login — it's a developer account for
making direct API calls, and it's billed per use rather than a
subscription.

1. Go to **[console.anthropic.com](https://console.anthropic.com)** and sign up.
2. Add a payment method under **Billing** (a few dollars of credit covers a lot of slip reads — each read is roughly a cent or two with Claude Sonnet; check Anthropic's current pricing page for exact rates).
3. Go to **API Keys** → **Create Key**. Copy it somewhere safe — you'll paste it into Netlify, not into any file in this folder (never put a real API key directly into a file you'll upload publicly).

---

## 3. Put this folder on GitHub (free)

Netlify's serverless functions work most reliably when deployed from a
Git repository, so we'll use GitHub as the middleman. No command line
needed — GitHub's website lets you upload files directly.

1. Go to **[github.com](https://github.com)** and sign up if you don't have an account.
2. Click the **+** in the top right → **New repository**. Name it
   `bozo-ledger`, keep it Public or Private (either works), don't add a
   README (we already have one). Click **Create repository**.
3. On the new repo's page, click **uploading an existing file**.
4. Drag the entire contents of this `bozo-ledger` folder in — all the
   files and the `netlify` folder together (keep the folder structure
   intact; GitHub preserves it when you drag a whole folder in, or you
   can use "choose your files" and select everything at once).
5. Scroll down, click **Commit changes**.

---

## 4. Deploy on Netlify (free)

1. Go to **[app.netlify.com](https://app.netlify.com)** and sign up — use
   "Sign up with GitHub" so the two are connected automatically.
2. Click **Add new project** → **Import an existing project** → **GitHub**
   → authorize Netlify → select your `bozo-ledger` repo.
3. Leave the build settings as-is (this is a static site — no build
   command needed) and click **Deploy**.
4. Once it's deployed, go to **Site configuration → Environment
   variables** → **Add a variable**:
   - Key: `ANTHROPIC_API_KEY`
   - Value: the key you copied from console.anthropic.com in step 2
5. Go back to **Deploys** and click **Trigger deploy → Deploy site** once
   more so the function picks up the new environment variable.
6. Netlify gives you a free URL like `random-name-123.netlify.app`. Send
   that link to your group. You can also add a custom domain for free
   under **Domain management** if you own one.

---

## Testing it

Open your new Netlify URL. You should see the app with no red warning
banner at the top (that banner means `firebase-config.js` still has
placeholder values). Place a test bet, upload a photo, and try "Read Slip
with AI" — if it fails, check that `ANTHROPIC_API_KEY` is set correctly in
Netlify and that you redeployed after adding it.

Open the same URL on a second phone or browser tab — anything you do on
one should appear on the other within a second or two, no refresh needed.

---

## Updating the site later

Since it's on GitHub, updating is: edit the file on GitHub's website (or
re-upload a changed `index.html`), commit, and Netlify automatically
redeploys within a minute or two. If you come back and ask me to add or
change something, I'll hand you an updated `index.html` — just re-upload
that one file to your GitHub repo.

---

## Costs, roughly

- **Firebase:** free tier covers this easily (way under the daily free quota for 5 people).
- **Netlify:** free tier covers this easily.
- **Anthropic API:** the only real cost, and only when someone taps "Read Slip with AI" — a few cents per read. Nobody in the group needs a claude.ai subscription for this to work.
