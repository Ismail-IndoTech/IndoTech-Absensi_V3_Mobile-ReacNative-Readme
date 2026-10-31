> 📚 **Dokumentasi:** [Indeks lengkap ID/EN](DOCS.md) · [English documentation index](DOCS.en.md)

# GSG Presensi Mobile


GSG Presensi Mobile (versi `1.14.0`) adalah aplikasi native Android/iOS berbasis **Expo SDK 57** dan **React Native 0.86**. Aplikasi ini merupakan WebView terintegrasi untuk memakai aplikasi web Presensi PT Graha Sarana Gresik, dengan lapisan native untuk permission, keamanan perangkat, notifikasi, offline queue, dan Sharing PDF.

## Stack

> Migrasi Expo 54 → 57 (Agustus 2026): versi dinaikkan ke 1.14.0 (versionCode 25)
> karena sisi native berubah besar. `newArchEnabled` dihapus (New Architecture kini
> satu-satunya opsi) dan konfigurasi splash dipindahkan ke plugin `expo-splash-screen`.

- Expo SDK 57 / React Native 0.86.3 / React 19.2.3
- Expo Router dan TypeScript strict
- `react-native-webview` 13.16.1
- Expo Location, Camera, Device, Notifications, File System, dan Sharing
- TanStack React Query dan AsyncStorage

## Menjalankan project

Prasyarat: Node.js dan npm yang kompatibel dengan Expo SDK 54. Push notification hanya dapat diuji pada perangkat fisik; Expo Go dan web tidak dapat menghasilkan token push perangkat.

```bash
npm install
npx expo start
```

Untuk development Android/iOS lokal:

```bash
npx expo run:android
npx expo run:ios
```

Verifikasi:

```bash
npx tsc --noEmit
npx expo lint
npm audit --production
```

## Konfigurasi endpoint

Default endpoint aplikasi berada di `app.json`:

```text
expo.extra.apiBaseUrl = https://presensi.grahasaranagresik.com
```

Untuk override pada build atau development, gunakan environment variable public berikut sebelum menjalankan Expo:

```bash
EXPO_PUBLIC_API_BASE_URL=https://presensi.grahasaranagresik.com npx expo start
```

`utils/config.ts` memvalidasi konfigurasi tersebut. Produksi wajib memakai HTTPS. HTTP hanya diizinkan secara konfigurasi untuk host development lokal (`localhost`, `127.0.0.1`, atau `10.0.2.2`); konfigurasi native tidak mengaktifkan cleartext traffic secara global. Jangan menaruh API key, secret, password, atau token privat di source code maupun `app.json`.

Token perangkat disimpan secara lokal dan tidak dicetak ke log.

## Build EAS

Repository ini memiliki script build, tetapi profil EAS harus tersedia di project EAS/`eas.json` sebelum build production:

```bash
npx eas-cli login
npx eas-cli build:configure
npx eas-cli build --platform android --profile production
```

Shortcuts yang tersedia:

```bash
npm run build:android
npm run build:android:beta
```

iOS dapat dibangun dengan EAS dari macOS/remote build:

```bash
npx eas-cli build --platform ios --profile production
```

## Struktur folder

- `app/` — route Expo Router, layout, dan entry aplikasi.
- `components/` — WebView, splash, onboarding, permissions, dan state screen native.
- `services/` — onboarding, offline queue/sync, storage, dan push notification.
- `utils/` — konfigurasi endpoint, security check, dan navigator WebView.
- `contexts/` — app state dan theme.
- `constants/` — warna, template HTML, dan halaman error/offline.
- `assets/images/` — icon dan splash image yang dibundel aplikasi.

## Catatan WebView dan keamanan

- Endpoint production dimuat melalui `API_BASE_URL`, bukan URL yang disalin di beberapa komponen.
- WebView mengizinkan JavaScript karena aplikasi web membutuhkan bridge native, tetapi navigasi luar first-party diarahkan ke browser sistem.
- `mixedContentMode` disetel `never`; tidak ada `allowFileAccess` atau `allowUniversalAccessFromFileURLs`.
- Tidak ada exception cleartext/ATS untuk production. Jika memakai backend HTTP lokal, gunakan hanya konfigurasi development dan perhatikan keterbatasan cleartext native.
- Icon dan splash image utama berada di `assets/`; source project tidak memakai CDN asset pihak ketiga. Logo pada tampilan splash berasal dari endpoint HTTPS first-party.
- Monitoring mock location, permission, offline cache, dan log error tidak mencetak token, NIK, koordinat, atau respons API mentah.

## Deployment

Jangan melakukan `git init`, commit, atau push dari proses ini. Repository Laravel utama mengabaikan folder mobile ini; folder ini disiapkan sebagai repository GitHub private terpisah dengan nama proyek `IndoTech-Absensi_V3_Mobile-ReacNative` sesuai keputusan owner.

## Firebase & Push Notification

Project `presensi-online-pt-gsg` (nomor `108247760935`), package
`com.gsgpresensi.indotech`.

| Platform | App ID | Berkas konfigurasi |
|----------|--------|--------------------|
| Android | `1:108247760935:android:ae0ae9fc05d8da21e00e02` | `google-services.json` |
| iOS | `1:108247760935:ios:6d29bbdffc2467eee00e02` | `GoogleService-Info.plist` |

- `app.json` menunjuk kedua berkas tersebut lewat
  `android.googleServicesFile` / `ios.googleServicesFile`, sehingga EAS
  prebuild menyalinnya ke project native.
- Token diambil dengan `Notifications.getDevicePushTokenAsync()` (FCM native).
  **Expo Go tidak bisa menghasilkan token FCM** — wajib development build
  (`eas build --profile development`) atau production build.
- Token disuntikkan ke WebView lewat
  `window.__GSG_PUSH_TOKEN` / `window.__onNativePushTokenReady()`, lalu halaman
  web mendaftarkannya ke `POST /device-token/register` memakai sesi login
  yang sedang aktif. Reinjeksi dilakukan saat halaman selesai dimuat, setiap
  30 detik, dan saat aplikasi kembali aktif.
- Channel Android: `alarm_channel` (sinkron dengan `FIREBASE_ANDROID_CHANNEL`
  di server), `schedule_reminders` (alarm absen), `reminder`, `announcement`,
  `default`.
- Pengingat lokal jam kerja & pulang kerja arrive dari web app melalui bridge
  `SCHEDULE_REMINDERS`; `services/localReminderService.ts` menjadwalkannya
  dengan `expo-notifications` sehingga berbunyi saat aplikasi tertutup.

## Verifikasi rilis

```bash
npx tsc --noEmit            # harus 0 error
npx expo lint               # 0 error / 0 warning (termasuk aturan React Compiler)
npx expo-doctor             # 21/21 checks passed
npm audit                   # 0 high/critical
```

Status saat push terakhir: **tsc 0 error, lint 0, doctor 21/21,
bundle Metro sukses, audit 0 high/critical** (16 advisory moderate tersisa
seluruhnya di tooling build Expo CLI).

---

## Akses & Lisensi

Repositori ini adalah **karya saya sendiri**, **Mochamad Ismail Soleh** (Ismail-IndoTech), dan dilisensikan secara khusus di bawah **[MIT License](LICENSE)**.

Untuk memperoleh akses ke **repositori sumber (privat)** dari proyek ini, silakan hubungi saya sebagai **Creator Repo** melalui WhatsApp di nomor yang terdaftar pada akun DANA saya: **08121600058** — [wa.me/628121600058](https://wa.me/628121600058). Akses diberikan sesuai kesepakatan.
## 💖 Kontribusi & Donasi

Jika aplikasi ini mempermudah operasional IT, DevOps, dan pengelolaan server di perusahaan atau proyek pribadi Anda, dukung pengembangannya dengan menjadi kontributor atau **Donatur Spesial**.  
*(Donatur dan Sponsor resmi akan kami cantumkan logo perusahaannya di repositori ini).*

Dukung pemeliharaan berkelanjutan, riset modul bypass panel baru, dan pembaruan sistem dengan berdonasi melalui:

<div align="center">

**DANA — 08121600058** (a.n. Mochamad Ismail Soleh)

<img src="docs/qris.jpg" alt="QRIS DANA IndoTech" width="280">

*Pindai dengan aplikasi DANA, BCA, Mandiri, GoPay, OVO, atau aplikasi pembayaran QRIS apa pun di Indonesia.*

</div>

Semua dana dialokasikan untuk biaya infrastruktur riset server, lisensi alat pengujian, dan pengembangan modul open-source IndoTech. Jika Anda atau institusi Anda ingin menjadi mitra resmi atau sponsor, silakan hubungi kami. Terima kasih atas dukungan Anda!