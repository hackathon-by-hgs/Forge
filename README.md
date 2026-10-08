# Forge mobile

The Forge worker app. Workers find jobs near them, clock in with GPS, get paid the moment they clock out, and build a credit record that banks recognise. It is built for Nigeria's blue-collar gig workforce: loaders, drivers, unloaders, welders and general labour.

This repository is the `mobile` branch of the Forge remote. The API is the sibling `backend` clone and the web dashboards are the sibling `frontend` clone.

Android build: [app-arm64-v8a-release.apk](https://github.com/Ferousco-dev/Forge/releases/download/v1.0.0/app-arm64-v8a-release.apk). All builds are on the [releases page](https://github.com/Ferousco-dev/Forge/releases).

## What it does

| Area | What is in it |
| --- | --- |
| Sign up | Phone code by SMS, WhatsApp or push, then a liveness selfie, name, photo and bank link. |
| Jobs | A map and list of jobs near the worker, refreshed quietly every 40 seconds, filtered by trade or best pay. |
| Apply | One tap to apply, live status, and withdraw before the job starts. |
| Work | Clock in inside the job's GPS fence (each job sets its radius), a live timer, then clock out with a proof photo and a GPS accuracy check. |
| Pay | The employer signs off in a payout window, then the money lands in the wallet. Withdraw to a bank by NIBSS transfer. |
| Loans | An AI credit score, eligibility after 3 completed jobs, and repayment taken from future earnings. |
| Profile | Work history, saved jobs, notifications, help and support, settings and a data export. |

A work session moves through `accepted`, `arriving`, `working`, `reviewing`, `submitting`, `pending_review` and `done`. Each phase is saved in secure storage, so a restart picks up where the worker left off. Sessions are keyed by job, so a worker can have several in flight at once.

## Stack

Flutter with Dart 3.10.8 or newer, one codebase for Android and iOS. Riverpod for state and go_router for navigation, with a stateful shell so each tab keeps its own history. Firebase Cloud Messaging with local notifications for push, Google Maps with an OpenStreetMap fallback (flutter_map), geolocator and geocoding for location, image_picker for photos, flutter_secure_storage for tokens, and printing for receipts.

## Setup

You need Flutter, Android Studio for Android builds, and Xcode with CocoaPods for iOS builds.

```bash
git clone --branch mobile --single-branch https://github.com/hackathon-by-hgs/Forge.git mobile
cd mobile
flutter pub get
flutter run
```

The Firebase files for the Forge project are committed: `android/app/google-services.json`, `ios/Runner/GoogleService-Info.plist` and `lib/firebase_options.dart`. To use your own Firebase project, replace them (or run `flutterfire configure`) and enable Cloud Messaging.

## Configuration

| Setting | Where | Notes |
| --- | --- | --- |
| API address | `lib/core/api/api_config.dart` | Points at the deployed API, `https://forgebe-production.up.railway.app/v1`. For the local `backend` clone use `http://localhost:3000/v1`, or `http://10.0.2.2:3000/v1` from an Android emulator. |
| Google Maps key | `android/app/src/main/AndroidManifest.xml` | A development key is committed. Replace it with your own restricted key before a release. |

## API contract

`endpoint_resources/` has one file per screen with the exact request and response shape. Start with `00_README.md` for the envelope, paging and error format. The app reads both snake_case and camelCase fields.

## Permissions

| Permission | Used for |
| --- | --- |
| Location | The jobs feed, the clock-in fence and the clock-out GPS proof. Needed to clock in. |
| Camera | The liveness selfie and the clock-out proof photo. Needed to finish a shift. |
| Photo library | Choosing a profile photo. Optional. |
| Notifications | Job alerts, payment confirmations and loan decisions. Optional. |

## Code layout

```text
lib/app/          router, bottom navigation shell, theme
lib/core/         API client, notifications, location, storage, uploads, AI hooks, mock data
lib/features/     auth, jobs, work, earnings, loans, profile, bookmarks, employers, splash
lib/shared/       shared widgets
endpoint_resources/   API contract, one file per screen
```

## Commands

| Command | What it does |
| --- | --- |
| `flutter run` | Runs on the connected device or emulator. |
| `flutter analyze` | Lints. Keep it at zero warnings. |
| `flutter test` | Runs `test/widget_test.dart`, which opens every route to check nothing crashes. |
| `dart format lib/` | Formats the code. |
| `flutter build appbundle --release` | Android bundle for the Play Console. |
| `flutter build apk --release` | Android APK for sideloading. |
| `flutter build ipa --release` | iOS archive for App Store Connect. |

Set the version in `pubspec.yaml` before each release, in the form `<semver>+<build number>`.

## Known gaps

- Voice search is not built yet. The mic icon says it is coming soon.
- Only English is shipped.
- This app is for workers. Employers use the web dashboards in the `frontend` branch.
