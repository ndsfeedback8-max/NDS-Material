# Deploying NDS Material — Vercel (hosting) + Firebase (shared database)

Good news: the app already has Firebase wiring built in — there's a `FIREBASE_CONFIG`
object near the top of the `<script>` tag pointing at a project called `nds-material`,
and the storage layer automatically switches to Firestore whenever the app isn't
running inside Claude.ai. So on Vercel it'll use Firebase automatically — nothing to
edit in the code. What's left is turning the right things on in the Firebase console,
and pushing the file to Vercel.

## 1. Firebase console setup

Go to https://console.firebase.google.com and open (or create, if it doesn't exist
yet) the project **nds-material** — that's the `projectId` already in the code.

1. **Firestore Database** → Create database → **Production mode** → pick a region
   (e.g. `asia-south1`). Skip if a database already exists.
2. **Build → Authentication → Sign-in method** → enable the **Anonymous** provider.
   The app signs every visitor in anonymously behind the scenes, then handles its own
   username/password + admin/staff roles on top of that — this checkbox is required or
   sign-in will fail silently.
3. **Firestore Database → Rules** → paste in the contents of `firestore.rules`
   (included alongside this guide) → **Publish**.

That's it on the Firebase side — no need to touch Storage, Hosting, or Functions.

## 2. Deploy to Vercel

**Option A — CLI, fastest for a one-off deploy:**
```bash
npm i -g vercel
cd deploy          # the folder containing index.html
vercel login
vercel --prod
```
Accept the defaults when prompted — it's a single static HTML file, so no build
command or framework is needed.

**Option B — GitHub + Vercel dashboard, better if you'll keep editing it:**
1. Create a new GitHub repo and push `index.html` to it.
2. In Vercel → **Add New → Project** → import that repo.
3. Framework preset: **Other**. Leave build/output/install commands blank.
4. Deploy. Every future push to the repo redeploys automatically.

## 3. Verify it's actually using Firebase

1. Open the deployed URL — since no users exist yet, you'll land on "Create admin
   account." Create yours.
2. Go to **Settings** → click **Test Firebase connection**. It should say connected.
3. Open the same URL on a second device/phone — you should see the same data. That
   confirms it's shared (Firestore), not per-browser `localStorage`.

## Notes

- Having the `apiKey` visible in the page source is normal — Firebase web API keys
  aren't secret; access is controlled by the Firestore rules above, not by hiding
  the key.
- Because everyone authenticates anonymously, `request.auth != null` is the
  strongest boundary Firestore itself can enforce here — real per-person permission
  (who's an admin, who can edit what) is handled by the app's own login system, which
  lives inside that same Firestore data. That matches the disclaimer already in the
  app's Settings page.
- If you ever want to lock the API key down further, Firebase Console →
  Project settings → your web app → restrict the key to your Vercel domain(s) under
  Google Cloud API key restrictions. Optional, not required for this to work.
