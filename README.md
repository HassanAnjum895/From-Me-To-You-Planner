# 🌸 Todoke Planner

A private, shared school-week planner for two people. One static `index.html` —
no build step, no npm, no server code. Firebase Auth + Firestore are loaded
straight from Google's CDN.

**How access works:** there is exactly **one** Firebase account. Its *password* is
the shared passcode you both type on the lock screen. Its *email* is only a
username and never receives mail. Firestore security rules then hard-limit all
data to that account's UID, so even if someone else creates a Firebase account
in your project, they can read nothing.

Ordinary sharing = just tell the other person the passcode. Nothing else to do.

---

## 1. Create the Firebase project

1. Go to <https://console.firebase.google.com/> → **Add project**.
2. Name it something like `todoke-planner`. Google Analytics is not needed.
3. Once it's ready you'll land on the project overview.

## 2. Turn on the database

1. Left sidebar → **Build → Firestore Database** → **Create database**.
2. Choose **Production mode** (starts locked — we replace the rules next).
3. Pick a region close to you. `eur3` / `nam5` style multi-region or a single
   region both have a free tier; anything is fine for two people.
4. Click **Create**.

## 3. Turn on login

1. Left sidebar → **Build → Authentication** → **Get started**.
2. **Sign-in method** tab → **Email/Password** → toggle **Enable** → **Save**.

## 4. Create the one shared account

1. Still in Authentication, open the **Users** tab → **Add user**.
2. **Email:** make up a username-style address, e.g. `us@todoke-planner.app`.
   It does not need to exist or be yours — nothing is ever emailed to it.
3. **Password:** this is the passcode you'll both type. Use something decent
   (12+ characters) — it is the only thing standing between the internet and
   your planner.
4. Click **Add user**, then **copy the User UID** in the table on the left.
   It looks like `aB1cD2eF3gH4iJ5kL6mN7oP8qR9s`.

## 5. Lock the database to that account

1. **Firestore Database → Rules** tab.
2. Paste the contents of [`firestore.rules`](./firestore.rules), replacing
   `PASTE_UID_HERE` with the UID you just copied. Keep the quotes.
3. **Publish**.

It should end up looking like:

```
function isUs() {
  return request.auth != null && request.auth.uid == 'aB1cD2eF3gH4iJ5kL6mN7oP8qR9s';
}
```

> Skipping the UID check (e.g. using `if request.auth != null`) would let anyone
> who makes their own account into your planner. The UID is what makes it
> two-people-only.

## 6. Grab the web config

1. ⚙️ **Project settings → General**, scroll to **Your apps**.
2. Click the **`</>`** (Web) icon, register the app with any nickname (no
   Hosting needed), and copy the `firebaseConfig` object it shows you.

## 7. Paste it into `index.html`

Near the bottom of the file, fill in both placeholders:

```js
const firebaseConfig={
 apiKey:"AIza...",                                  // <- from the console
 authDomain:"todoke-planer.firebaseapp.com",
 projectId:"todoke-planer",
 storageBucket:"todoke-planer.firebasestorage.app",
 messagingSenderId:"874297732911",
 appId:"1:874297732911:web:..."
};
const SHARED_EMAIL='us@todoke-planer.app';         // <- the email from step 4
```

These values are **not secrets** — Google ships them in every Firebase web app.
They're safe to commit publicly. Security comes from the rules in step 5.

⚠️ Never paste a *service account key* or `private_key` into this file.

## 8. Try it locally

ES modules and Firebase need a real `http://` origin, so don't double-click the
file. From this folder run one of:

```bash
npx serve .
# or
python -m http.server 8080
```

Open the shown address, type the passcode, and add an entry. Reload — it should
still be there.

## 9. Put it on GitHub

```bash
git init
git add .
git commit -m "Private two-person school planner"
git branch -M main
git remote add origin https://github.com/YOUR-NAME/todoke-planner.git
git push -u origin main
```

(Create the empty repo on GitHub first — don't let it add a README, or the
push will be rejected.)

## 10. Deploy on Vercel

1. Go to <https://vercel.com/new> and **Import** the repo you just pushed.
2. **Framework Preset:** `Other`.
3. Leave **Build Command** and **Output Directory** empty — it's a static file.
4. **Deploy**. You'll get a URL like `https://todoke-planner.vercel.app`.

Every future `git push` to `main` redeploys automatically.

## 11. Tell Firebase your new domain

1. Firebase console → **Authentication → Settings → Authorized domains**.
2. **Add domain** → `todoke-planner.vercel.app` (yours from step 10).

Email/password sign-in usually works without this, but adding it keeps the
domain allowlisted and prevents surprises later.

## 12. Final check

Open the Vercel URL in a private/incognito window:

- You should see only the passcode card, no planner behind it.
- The right passcode lets you in; a wrong one shows "That passcode is not right".
- Open it in two browsers/devices and confirm an entry added on one appears on
  the other within a second or two.
- The 🚪 button in the toolbar locks the device back to the passcode screen.

---

## Facts worth knowing

- **Free tier:** Firestore's Spark plan gives 1 GiB storage, 50k reads and 20k
  writes per day. Two people will not come close.
- **Both of you share one identity.** The 💙/🌸 "I am" toggle (not the login) is
  what marks whose entry is whose.
- **`🚪` only locks that device.** Anyone with the passcode can unlock anywhere;
  that's the point of a shared passcode.
- **Changing the passcode:** Authentication → Users → the account → reset
  password. No file changes needed.
- **Removing access:** disable or delete the user account in the console. The
  rules stop accepting the old session immediately.
- **Data lives in Firestore**, not in `localStorage`. Clearing your browser does
  not erase the planner.
