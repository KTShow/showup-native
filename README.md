# showup-native

Native iOS shell for **ShowUp**. It's a [Capacitor](https://capacitorjs.com/) WebView
that loads the live web app at **https://showuproom.com** — it does **not** bundle the
web code.

## How this fits with the web app

| Repo | What it is | How it ships |
|------|-----------|--------------|
| `showup-app` | the whole product (`index.html` + Supabase) | edit → `git push` → Vercel auto-deploys → **live for web *and* app on next launch** |
| `showup-native` (this) | ~a dozen files: the iOS wrapper | only changes when the native shell or a plugin changes; a change here needs a new TestFlight build |

Day-to-day product work never touches this repo. `capacitor.config.json > server.url`
points the WebView at the production URL, so a Vercel deploy reaches the app with no
rebuild.

## Key facts

- **Bundle ID:** `com.showuproom.app`
- **Display name:** ShowUp
- **Min deployment / Xcode / CocoaPods:** Capacitor 6 defaults
- **Signing & build:** [Codemagic](https://codemagic.io) (Tracey is on Windows — no local
  Xcode). See `codemagic.yaml`.

## First-time setup (after the Apple Developer account is approved)

1. **Apple Developer portal** → Certificates, Identifiers & Profiles → **Identifiers** →
   register an App ID `com.showuproom.app` with the **Push Notifications** capability
   ticked (we don't wire push yet, but registering it now avoids a profile regen later).
2. **App Store Connect** → **Apps** → **+** → new app, bundle id `com.showuproom.app`,
   name "ShowUp". Note the numeric **Apple ID** it gets assigned.
3. **App Store Connect** → Users and Access → Integrations → **App Store Connect API** →
   generate a key with role **App Manager**. Download the `.p8` (once only).
4. **Codemagic**: sign up with `admin@showuproom.com`, add this repo, add the App Store
   Connect API key as an integration named `showup-asc-key`, paste the app's numeric
   Apple ID into `codemagic.yaml` (`APP_STORE_APPLE_ID`).
5. In App Store Connect → TestFlight, create an **internal** beta group called
   "ShowUp Beta" (matches `codemagic.yaml`).
6. Push to `main` → Codemagic builds a signed `.ipa` and submits it to TestFlight.

## Local commands (Windows-safe)

```
npm install          # once
npx cap sync ios      # after changing capacitor.config.json or adding a plugin
```

`pod install`, `xcodebuild`, and the actual archive run on Codemagic's Mac, not here.

## Roadmap

1. ✅ Bare shell scaffolded
2. ⬜ First signed build to TestFlight (proves the pipeline)
3. ⬜ Push notifications: add `@capacitor/push-notifications`, a `device_tokens` table
   in Supabase, and a provider (OneSignal planned). The daily "new season" job then
   fires a push alongside the in-app bell row. See the `showup-app` project notes.

## Note on `npm audit`

`@capacitor/cli` pulls an old `node-tar` transitively (flagged high/critical). It's a
build-time-only dev dependency (downloads/extracts native templates); it isn't in the
shipped app. Left as-is rather than force-upgrading the CLI past a major version.
