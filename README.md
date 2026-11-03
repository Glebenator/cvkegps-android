# CVKE GPS for Android

[![Android CI](https://github.com/Glebenator/cvkegps-android/actions/workflows/android.yml/badge.svg?branch=main)](https://github.com/Glebenator/cvkegps-android/actions/workflows/android.yml?query=branch%3Amain)

A native Android speedometer and trip recorder. It shows your GPS speed, records trips with the screen locked, and
replays them on a map with speed, height, and acceleration charts. Built with Kotlin, Jetpack Compose, Material 3, and
the Google Maps SDK.

CVKE GPS has three repositories:

| Repository | Contents |
|---|---|
| [cvkegps](https://github.com/Glebenator/cvkegps) | The product: decisions (ADRs), glossary, shared measurement rules, and the [progress report](https://github.com/Glebenator/cvkegps/blob/main/PROGRESS.html) |
| **cvkegps-android** | This repository: the Android app |
| [cvkegps-ios](https://github.com/Glebenator/cvkegps-ios) | The iOS app, with the same measurement rules and product behaviour |

## Status

- **Before the first release.** The app is not published yet. The application ID is `com.cvke.gps`. The debug build
  has the ID `com.cvke.gps.debug` and the name CVKE GPS Debug, so it can be on the same phone as the Play build.
- **CI.** The badge above shows the result of the newest run on `main`. Select it to see all runs.
- **Verification.** The device tests run on an emulator with simulated GPS locations. GPS accuracy, battery use, and
  TalkBack are not yet checked on a real phone. The progress report has the dated test results and the known limits.

### What CI checks

The workflow [`android.yml`](.github/workflows/android.yml) runs for each push to `main` and each pull request, on
`ubuntu-latest` with JDK 21.

| Check | Gradle task | In CI |
|---|---|---|
| Debug build | `:app:assembleDebug` | Yes |
| Device test build | `:app:assembleDebugAndroidTest` | Yes (compiled, not run) |
| Lint, warnings as errors | `:app:lintDebug` | Yes |
| Unit tests | `:app:testDebugUnitTest` | Yes |
| Release APK, minified and unsigned | `:app:assembleRelease` | Yes |
| Release bundle for Google Play, unsigned | `:app:bundleRelease` | Yes |
| Device tests on an emulator | `:app:connectedDebugAndroidTest` | Not yet. Run them locally. |

The lint and test reports are in the `android-reports` artifact of each run.

## Features

- **Speed.** The current GPS speed in a large dial, with altitude, maximum speed, and a position accuracy bar. An old or
  poor reading shows as unavailable, never as a current speed.
- **GPS details.** Reading quality, receiver events, satellite signals, and a sky map. A satellite ring around the
  speed shows the satellites in their directions.
- **Trip recording.** Start, pause, resume, and stop. A trip records through a location foreground service after Home
  or screen lock, with a notification. After Stop and save, you can resume the trip for 2 minutes. An optional setting
  records approximate network positions when GPS is lost.
- **Trips.** Saved trips grouped by day, each with its route on a Google map: map types, map styles (Standard, Minimal,
  High contrast), speed colours, and start and end markers.
- **Replay.** A player for a saved trip, with the moving position on the map, moments such as the fastest speed, and
  linked speed, height, and acceleration charts.
- **Activity.** An estimate from GPS speed (bike or ski, car or transit, plane), which you can correct.
- **GPX export.** Share a trip as a GPX file.
- **Settings.** Units, altitude reference, keep screen on, colours (including wallpaper colours), map options, and trip
  recording accuracy.
- **Layouts.** Light and dark themes, dynamic colour, large text, landscape, tablets, and foldables.

## Screenshots

| Speed | Current trip | Replay | Trips, dark |
|---|---|---|---|
| ![Speed tab with 22.4 mph in the dial, altitude and maximum speed cards, the accuracy bar, and Start trip](https://raw.githubusercontent.com/Glebenator/cvkegps/main/docs/progress/images/2026-10-31-ui-speed-start-row.webp) | ![Current trip with the live map, the Recording status, speed, time, distance, and the Pause and Stop buttons](https://raw.githubusercontent.com/Glebenator/cvkegps/main/docs/progress/images/2026-10-31-ui-current-trip.webp) | ![Trip replay with the map, the player, a moment chip, and the speed chart](https://raw.githubusercontent.com/Glebenator/cvkegps/main/docs/progress/images/2026-10-31-ui-replay-details.webp) | ![Trips list in dark mode with one trip under Today](https://raw.githubusercontent.com/Glebenator/cvkegps/main/docs/progress/images/2026-10-31-ui-trips-dark.webp) |

31 October 2026, commit `551642b`. Pixel 10 Pro emulator, Android 17 (API 37). All locations are simulated GPS test
locations.

## Build and run

1. Open this folder in Android Studio.
2. Use JDK 21 or later for Gradle. The wrapper uses Gradle 9.6.0 and Android Gradle Plugin 9.4.1.
3. Install Android SDK platform 37. The minimum supported API is 29.
4. Add `MAPS_API_KEY=<your key>` to `local.properties`. Get the key from a Google Cloud project with the Maps SDK for
   Android turned on. Restrict it to that API and to each application ID with its signing certificate SHA-1:
   `com.cvke.gps.debug` with your debug certificate, and `com.cvke.gps` with the upload certificate and the Play app
   signing certificate. Git ignores `local.properties`. Without a key, the app builds and runs, but the map is empty.
5. Run the `app` configuration on a device or an emulator with Google Play services.
6. Select **Allow precise location**, then allow location while using the app.

From a terminal:

```sh
./gradlew :app:assembleDebug :app:lintDebug :app:testDebugUnitTest
```

On Windows, use `.\gradlew.bat`.

## Release build

Google Play needs an Android App Bundle (AAB) signed with an upload key. Google Play keeps the app signing key
(Play App Signing) and signs the APKs that it delivers.

1. Make an upload key once. Keep it outside this folder, and keep a backup copy and the passwords in a safe place:

   ```sh
   keytool -genkeypair -v -keystore ~/keys/cvkegps-upload.jks -alias upload -keyalg RSA -keysize 4096 -validity 10000
   ```

2. Make `keystore.properties` in this folder. Git ignores it.

   ```properties
   storeFile=/Users/you/keys/cvkegps-upload.jks
   storePassword=...
   keyAlias=upload
   keyPassword=...
   ```

   You can also use the environment variables `CVKE_UPLOAD_STORE_FILE`, `CVKE_UPLOAD_STORE_PASSWORD`,
   `CVKE_UPLOAD_KEY_ALIAS`, and `CVKE_UPLOAD_KEY_PASSWORD`. They have priority over the file.
3. Increase `versionCode` in `app/build.gradle.kts` for each upload.
4. Build the bundle, with `MAPS_API_KEY` set to the release key:

   ```sh
   ./gradlew :app:bundleRelease
   ```

   The bundle is `app/build/outputs/bundle/release/app-release.aab`. Upload it to a testing track in Play Console.
5. In Play Console, open **Test and release > App integrity**. Add the SHA-1 of the upload certificate and of the app
   signing certificate to the Maps API key.

Without an upload key, `bundleRelease` and `assembleRelease` make unsigned output. CI builds them this way.

## Tests

- **Unit tests** (`app/src/test`) cover the calculations: fix interpretation, route math, the activity estimate, GPX,
  the sensor record, speed colours, replay data, settings, and display format.
- **Device tests** (`app/src/androidTest`) run the real app with Android test locations. They do not replace the
  location source or the ViewModels.

  | Class | Covers |
  |---|---|
  | `SpeedometerDeviceTest` | Permission and its recovery, live speed, poor and old fixes, location off, GPS details, and GPS request cleanup |
  | `TrackingDeviceTest` | Recording with screen lock and rotation, pause segments, signal loss, network positions, resume after stop, the activity, maps, speed colours, replay, GPX, and delete |
  | `SettingsDeviceTest` | Units, altitude, keep screen on, colours, map options, and map styles |
  | `SatelliteDisplayDeviceTest` | Satellite reports: signal bands, expiry, recovery, and both themes |
  | `SkyMapDeviceTest` | The sky map in GPS details, and the satellite ring that turns with the phone |
  | `CompassDeviceTest` | The compass display axes |
  | `DisplayRotationDeviceTest` | The compass axes in reverse landscape |

  Use a dedicated emulator with an English locale. Android Test Orchestrator clears the app data between tests. The
  tests change the GPS test provider and restore it after each test.

  ```sh
  ./gradlew :app:connectedDebugAndroidTest
  ```

Emulator results are functional evidence only. GPS accuracy, battery use, and performance need a real phone.

## Location, battery, and privacy

- GPS runs only while a screen or a recording trip needs it. It stops when the app is not visible, unless a trip
  records. A paused trip stops GPS.
- Standard recording uses one fix each second. High accuracy uses up to 10 fixes each second and more battery.
- The app does not hold wake locks or poll for location, and keeps the screen on only when you choose it in Settings.
- Trips stay on the phone in a local database. The app does not upload trip records and has no analytics.
- Google receives the map area that you look at. Share as GPX sends positions and times only to the app that you select.

## Project structure

```text
app/src/main/java/com/cvke/gps/
  location/      GPS and GNSS sources, the fix rules, and the compass
  tracking/      Trip recorder, foreground service, Room database, maps, replay, and the trip screens
  ui/            Navigation, the Speed tab, GPS details, Settings, display format, and the theme
  settings/      Stored settings and units
baselineprofile/ Baseline Profile generator for startup
scripts/         Process-loss recovery check on an emulator
```

State flows from the location source through the ViewModels to the screens. Measurements stay in SI units until
display. The measurement models contain no Android location objects.

## Documents

- [Progress report](https://github.com/Glebenator/cvkegps/blob/main/PROGRESS.html): status, screenshots, test evidence,
  and future work for both apps.
- [Decisions (ADRs)](https://github.com/Glebenator/cvkegps/tree/main/docs/adr) and the
  [glossary](https://github.com/Glebenator/cvkegps/blob/main/GLOSSARY.md).
- [AGENTS.md](AGENTS.md): the rules for work on this app.
- [Development notes](docs/development-notes.md): the dated notes from before 2 November 2026.
- [Performance review of 30 October 2026](https://github.com/Glebenator/cvkegps-android/blob/clanker/performance-review-71251c/docs/performance-review-2026-10-30.md),
  on the branch `clanker/performance-review-71251c`.

Report a problem or a request in [GitHub Issues](https://github.com/Glebenator/cvkegps-android/issues).
