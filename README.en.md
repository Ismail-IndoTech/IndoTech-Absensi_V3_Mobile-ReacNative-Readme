> 📚 **Documentation:** [Full ID/EN index](DOCS.md) · [English index](DOCS.en.md)

# GSG Presensi Mobile


GSG Presensi Mobile (version `1.14.0`) is a native Android/iOS app based on **Expo SDK 57** and **React Native 0.86**. This app is an integrated WebView for using the Presensi PT Graha Sarana Gresik web application, with native layers for permissions, device security, notifications, offline queue, and PDF Sharing.

## Stack

> Expo 54 → 57 migration (August 2026): the version was raised to 1.14.0 (versionCode 25)
> because the native side changed substantially. `newArchEnabled` was removed (New Architecture is now
> the only option) and the splash configuration was moved to the `expo-splash-screen` plugin.

- Expo SDK 57 / React Native 0.86.3 / React 19.2.3
- Expo Router and TypeScript strict
- `react-native-webview` 13.16.1
- Expo Location, Camera, Device, Notifications, File System, and Sharing
- TanStack React Query and AsyncStorage

## Running the project

Prerequisites: Node.js and npm compatible with Expo SDK 54. Push notifications can only be tested on physical devices; Expo Go and web cannot generate device push tokens.

```bash
npm install
npx expo start
```

For local Android/iOS development:

```bash
npx expo run:android
npx expo run:ios
```

Verification:

```bash
npx tsc --noEmit
npx expo lint
npm audit --production
```

## Endpoint configuration

The default app endpoint is in `app.json`:

```text
expo.extra.apiBaseUrl = https://presensi.grahasaranagresik.com
```

To override for a build or development, use the following public environment variable before running Expo:

```bash
EXPO_PUBLIC_API_BASE_URL=https://presensi.grahasaranagresik.com npx expo start
```

`utils/config.ts` validates that configuration. Production must use HTTPS. HTTP is only allowed by configuration for local development hosts (`localhost`, `127.0.0.1`, or `10.0.2.2`); the native configuration does not enable cleartext traffic globally. Do not put API keys, secrets, passwords, or private tokens in the source code or `app.json`.

Device tokens are stored locally and are not printed to logs.

## EAS Build

This repository has build scripts, but the EAS profile must be available in the EAS project/`eas.json` before a production build:

```bash
npx eas-cli login
npx eas-cli build:configure
npx eas-cli build --platform android --profile production
```

Available shortcuts:

```bash
npm run build:android
npm run build:android:beta
```

iOS can be built with EAS from macOS/remote build:

```bash
npx eas-cli build --platform ios --profile production
```

## Folder structure

- `app/` — Expo Router routes, layouts, and app entry.
- `components/` — WebView, splash, onboarding, permissions, and native screen state.
- `services/` — onboarding, offline queue/sync, storage, and push notification.
- `utils/` — endpoint configuration, security check, and WebView navigator.
- `contexts/` — app state and theme.
- `constants/` — colors, HTML templates, and error/offline pages.
- `assets/images/` — icons and splash images bundled with the app.

## WebView and security notes

- The production endpoint is loaded via `API_BASE_URL`, not a URL copied across multiple components.
- The WebView allows JavaScript because the web app needs the native bridge, but external first-party navigation is directed to the system browser.
- `mixedContentMode` is set to `never`; there is no `allowFileAccess` or `allowUniversalAccessFromFileURLs`.
- There are no cleartext/ATS exceptions for production. If using a local HTTP backend, use development configuration only and be aware of native cleartext limitations.
- The main icons and splash images are in `assets/`; the project source does not use third-party CDN assets. The logo on the splash screen comes from a first-party HTTPS endpoint.
- Mock location, permission, offline cache, and error log monitoring do not print tokens, NIK, coordinates, or raw API responses.

## Deployment

Do not run `git init`, commit, or push from this process. The main Laravel repository ignores this mobile folder; this folder is prepared as a separate private GitHub repository named `IndoTech-Absensi_V3_Mobile-ReacNative` per the owner's decision.

## Firebase & Push Notification

Project `presensi-online-pt-gsg` (number `108247760935`), package
`com.gsgpresensi.indotech`.

| Platform | App ID | Configuration file |
|----------|--------|--------------------|
| Android | `1:108247760935:android:ae0ae9fc05d8da21e00e02` | `google-services.json` |
| iOS | `1:108247760935:ios:6d29bbdffc2467eee00e02` | `GoogleService-Info.plist` |

- `app.json` points to both files via
  `android.googleServicesFile` / `ios.googleServicesFile`, so the EAS
  prebuild copies them into the native project.
- The token is obtained with `Notifications.getDevicePushTokenAsync()` (native FCM).
  **Expo Go cannot generate an FCM token** — a development build
  (`eas build --profile development`) or production build is required.
- The token is injected into the WebView via
  `window.__GSG_PUSH_TOKEN` / `window.__onNativePushTokenReady()`, then the web
  page registers it to `POST /device-token/register` using the currently active
  login session. Reinjection is done when the page finishes loading, every
  30 seconds, and when the app becomes active again.
- Android channels: `alarm_channel` (synchronized with `FIREBASE_ANDROID_CHANNEL`
  on the server), `schedule_reminders` (attendance alarms), `reminder`, `announcement`,
  `default`.
- Local work clock-in & clock-out reminders arrive from the web app via the
  `SCHEDULE_REMINDERS` bridge; `services/localReminderService.ts` schedules them
  with `expo-notifications` so they ring when the app is closed.

## Release verification

```bash
npx tsc --noEmit            # must be 0 error
npx expo lint               # 0 error / 0 warning (including React Compiler rules)
npx expo-doctor             # 21/21 checks passed
npm audit                   # 0 high/critical
```

Status at the last push: **tsc 0 error, lint 0, doctor 21/21,
Metro bundle successful, audit 0 high/critical** (16 moderate advisories remain,
all in the Expo CLI build tooling).

---

## Access & License

This repository is **my own work**, **Mochamad Ismail Soleh** (Ismail-IndoTech), and is specifically licensed under the **[MIT License](LICENSE)**.

To obtain access to the **private source repository** for this project, contact me as the **Repository Creator** via WhatsApp using the number registered to my DANA account: **08121600058** — [wa.me/628121600058](https://wa.me/628121600058). Access is granted by agreement.
## 💖 Contribution & Donation

If this application makes IT operations, DevOps, and server management easier for your company or personal project, support its development by becoming a contributor or a **Special Donor**.
*(We will list the company logos of official Donors and Sponsors in this repository).*

Support ongoing maintenance, research into new panel bypass modules, and system updates by donating through:

<div align="center">

**DANA — 08121600058** (a.n. Mochamad Ismail Soleh)

<img src="docs/qris.jpg" alt="IndoTech DANA QRIS" width="280">

*Scan with the DANA, BCA, Mandiri, GoPay, OVO app, or any QRIS payment app in Indonesia.*

</div>

All funds are allocated for server research infrastructure costs, testing tool licenses, and the development of IndoTech open-source modules. If you or your institution would like to become an official partner or sponsor, please contact us. Thank you for your support!