# NEXT — setup

A daily-ops app for the training and eating plan: meals with reminders,
the fortnightly training rotation built around your custody schedule,
tracking, and the Sunday admin.

Everything is stored on your own phone. Nothing is sent anywhere.

---

## 1. Use it right now (30 seconds)

Open `index.html` in Chrome on your phone. It works immediately.

Limitation: opened as a local file it cannot register the offline worker
or install to your home screen. For that, do step 2.

---

## 2. Install it as an app on your phone (5 minutes)

The app needs to be served over HTTPS. Any of these work:

**GitHub Pages** — free, permanent
1. Create a repo, upload the contents of this folder to the root
2. Settings → Pages → Source: `main` branch, `/root`
3. Wait a minute, then open `https://<you>.github.io/<repo>/`

**Netlify Drop** — no account needed
Drag this folder onto https://app.netlify.com/drop

**Your own hosting** — upload the folder anywhere that serves HTTPS

Then on your Android phone:
1. Open the URL in Chrome
2. Menu (⋮) → **Add to Home screen** → **Install**

It now has its own icon, opens fullscreen with no browser chrome, and
works with no signal. This is a real installed app as far as Android is
concerned — it just was not delivered through the Play Store.

---

## 3. Turn it into an actual .apk file

### Route A — PWABuilder (no tools, about 5 minutes)

Needs the app hosted from step 2.

1. Go to https://www.pwabuilder.com
2. Paste your URL, click **Start**
3. Click **Package for stores** → **Android**
4. Choose **Signed APK** (not the app bundle, that is for Play Store only)
5. Download the zip. It contains `app-release-signed.apk` plus your
   signing key — **keep the key file**, you need it to publish updates
6. Copy the APK to your phone, tap it, allow "install unknown apps"

This produces a Trusted Web Activity: a genuine APK wrapping the app.

### Route B — Capacitor (about 20 minutes, gives real alarms)

Worth doing if you want notifications that fire when the app is fully
closed. Route A cannot do that; the browser suspends timers.

Requires Node.js and Android Studio on your machine.

```bash
npm create @capacitor/app next-app     # package: co.za.juan.next
cd next-app
npm install @capacitor/android @capacitor/local-notifications
rm -rf www/* && cp /path/to/this/folder/* www/
npx cap add android
npx cap sync
npx cap open android                   # opens Android Studio
```

In Android Studio: **Build → Build Bundle(s)/APK(s) → Build APK(s)**.
The file lands in `android/app/build/outputs/apk/debug/`.

To get true background alarms, replace the `new Notification(...)` calls
in `index.html` with the Capacitor plugin:

```js
import { LocalNotifications } from '@capacitor/local-notifications';
await LocalNotifications.schedule({
  notifications: [{
    id: 1, title: 'NEXT', body: 'Psyllium and water',
    schedule: { on: { hour: 5, minute: 45 }, repeats: true }
  }]
});
```

---

## Reminders: what actually works

| Method | Fires when app is closed | Setup |
|---|---|---|
| Calendar export (in the app, Meals tab) | Yes, always | 1 tap |
| Browser notifications | Only while backgrounded | 1 tap |
| Capacitor local notifications | Yes, always | Route B |

**Use the calendar export.** It is one tap, it puts every meal, session
and Sunday task into your phone's calendar with alarms attached, and
Android will never kill it. The in-app reminders are a convenience on
top, not the primary system.

---

## First run

The app assumes the rotation starts on the coming Monday and that
**week 1** is the fortnight where the boys are with you for the weekend.

If that is the wrong way round, open the **Sunday** tab and tap
**Flip to week 2**. One tap, it sticks.

---

## Files

```
index.html                 the whole app, no dependencies
manifest.webmanifest       makes it installable
sw.js                      offline caching
icon-192/512.png           launcher icons
icon-maskable-512.png      adaptive icon for Android
```

To change anything — meal times, exercises, motivation lines, the
roster — it is all plain data near the top of the `<script>` block in
`index.html`. Bump `CACHE` in `sw.js` after any edit so phones pick up
the new version.
