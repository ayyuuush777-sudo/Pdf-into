# Getting PDF N-Up Arranger onto the Play Store

Everything below happens in a browser — no Android Studio, no Java,
no CLI. Budget an hour or two, plus Google's review time afterward
(usually a few days to a couple weeks for a new app).

## 0. What needs to be true first

- The HTML file, `manifest.json`, and the `icons/` folder all need to
  sit **in the same folder** at your real hosting domain — e.g.:
  ```
  yourdomain.com/index.html        (rename your HTML file to this, or
                                     adjust manifest.json's start_url
                                     if you keep the original name)
  yourdomain.com/manifest.json
  yourdomain.com/icons/icon-192.png
  yourdomain.com/icons/icon-512.png
  yourdomain.com/icons/icon-192-maskable.png
  yourdomain.com/icons/icon-512-maskable.png
  ```
- That domain needs to already be in your OAuth Client's **Authorized
  JavaScript origins** and Firebase Auth's **Authorized domains** —
  same requirement as always, nothing new here.

## 1. Get a Google Play Developer account

play.google.com/console → sign up. **One-time $25 fee**, paid once,
covers all apps you ever publish. Google's identity verification can
take a day or two, so worth starting this early — you can do the
rest of these steps while it's pending.

## 2. Package the app with PWABuilder

1. Go to **pwabuilder.com**
2. Enter your site's URL (e.g. `https://yourdomain.com`) → **Start**
3. PWABuilder scans it and shows a score for manifest, service
   worker, and security. The manifest and icons are handled by what's
   already in this folder. If it flags a missing service worker,
   that's fine to ignore for a TWA specifically — it's a "nice to
   have" score, not a requirement for packaging, and adding one
   carelessly risks caching stale versions of the app, so I left it
   out on purpose. Say so if you want one added properly later.
4. Click **Package for stores → Android**
5. Signing: choose **"New"** — PWABuilder generates and holds a
   fresh signing key for you, no separate tool needed. Fill in the
   package name it suggests (or customize it, e.g.
   `com.yourname.pdfnup`) and your app's display name.
6. **Generate Package** → downloads a zip containing:
   - an `.apk` (for testing on a real device first, if you want)
   - an `.aab` (**this is what you upload to Play Console**)
   - your signing key details, including a **SHA-256 fingerprint**

## 3. Finish Digital Asset Links

Without this step the app still works — it just shows a browser
address bar at the top, which looks less like a native app.

1. Open `.well-known/assetlinks.json` from this folder
2. Replace `PASTE_YOUR_PACKAGE_NAME_HERE` with the package name from
   step 2.5
3. Replace `PASTE_YOUR_SHA256_FINGERPRINT_HERE` with the fingerprint
   PWABuilder gave you
4. Upload the whole `.well-known` folder to your host so the file
   ends up at exactly `https://yourdomain.com/.well-known/assetlinks.json`
5. Reinstall/reopen the app — the address bar should be gone

## 4. Submit to Play Console

play.google.com/console → **Create app** → fill in name, description,
category. You'll also need, before it'll let you publish:
- A **privacy policy URL** (required even for simple apps — a single
  hosted page describing what data you collect is enough; since you
  collect Google sign-in + Firestore data, this genuinely needs to
  say so)
- A few **screenshots** of the app running
- A **feature graphic** (1024×500) for the store listing
- Content rating questionnaire (a few clicks, generates automatically)

Upload the `.aab` from step 2.6 under **Production → Create release**,
fill in release notes, and submit for review.

## Notes

- **Updating the app later**: re-run PWABuilder with signing set to
  **"Mine"** instead of "New," uploading the same key file from your
  first package — reusing a NEW key would make Play Store treat it as
  a different app entirely.
- **The app's actual content still lives on your website.** Nothing
  about this process copies your HTML/JS into the Android package —
  the installed app just opens your hosted URL in a chrome-less view.
  Any update you push to your hosted file appears in the installed
  app automatically, no re-submission needed, *unless* the update
  changes the manifest/icons/package identity.
