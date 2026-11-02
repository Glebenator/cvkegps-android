# Project rules: CVKE GPS for Android

CVKE GPS has three repositories. This repository is the native Android app.

- [cvkegps](https://github.com/Glebenator/cvkegps): the product. Decisions (ADRs), the glossary, the shared
  measurement rules, and the progress report. Product rules there apply to this app.
- [cvkegps-android](https://github.com/Glebenator/cvkegps-android): this repository.
- [cvkegps-ios](https://github.com/Glebenator/cvkegps-ios): the iOS app.

## Product

- Build a native Android app for publication. The iOS app in cvkegps-ios has the same product behaviour.
- Use ASD-STE100 Simplified Technical English for instructions and documentation.
- Implement only the agreed feature scope. Do not add inactive controls or simulated production data.
- Commit and push only when the user asks.

## Structure

- Use Kotlin, Jetpack Compose, and Material 3.
- Use a ViewModel with immutable UI state and one-way data flow.
- Put Android location access behind a small interface. Keep Android objects out of measurement models.
- Keep measurements in SI units. Convert them only for display.
- Use constructor injection. Add a dependency injection framework or module only when it solves a current problem.
- Keep platform behaviour native. Share measurement rules and product behaviour with iOS; do not force shared UI code.
- Keep dependencies stable, explicit, and versioned in the version catalog. Do not add unused libraries.
- Use coroutines with a defined scope. Do not use global jobs.

## Design and access

- Use Material components, theme colours, typography, and standard Android navigation.
- Support system light and dark themes, dynamic colour, edge-to-edge insets, and screen rotation.
- Support large fonts, screen readers, and touch targets of at least 48 dp.
- Put visible text in string resources. Format numbers for the user's locale.
- Explain permission use before requesting it. Provide a recovery action when permission or location services are off.
- Show missing or old measurements as unavailable. Never show an old speed as a current speed.

## Performance and privacy

- Run GPS only while a feature needs it. Release display listeners when the screen becomes inactive.
- A trip started by the user can record through a location foreground service after Home or screen lock.
- Recording needs a separate lifecycle, a visible notification, and device tests. Do not start or restart a trip silently in the background.
- Set an explicit update interval. Do not poll GPS, hold wake locks, or keep the screen on by default.
- Keep disk access, network access, and expensive calculations off the main thread.
- Avoid continuous animation and unnecessary UI updates. Measure before adding performance complexity.
- Do not log coordinates or include them in crash reports.
- Do not add analytics, network transmission, or cloud backup of location data without an agreed product requirement.
- Treat simulated GPS tests as functional evidence only. Measure battery use and GPS accuracy on a real device.

## Tests and delivery

- Prefer a small set of end-to-end device tests for important user flows.
- Test permission recovery, GPS delivery, signal loss, lifecycle cleanup, and relevant configuration changes.
- Use the production location path with Android test locations where possible. Do not replace the whole app with a fake.
- Add unit tests only for calculations or failure cases that device tests cannot cover well.
- Do not add template tests, coverage targets, or tests that only repeat the implementation.
- Restore device settings and remove test providers after tests.
- Run the debug build, Android lint, and relevant device tests for functional changes.
- Check the release build before release. Inspect light mode, dark mode, and large text after layout changes.
- Report what was tested, what failed, and what still needs a real device.
- Keep generated output, local SDK paths, signing keys, IDE state, and design exports out of Git.
- Before publication, replace the development application ID, add release signing, and complete store and privacy requirements.

## Progress report

- The progress report is `PROGRESS.html` in the cvkegps repository. Its rules are in the `AGENTS.md` file there.
- After a completed Android change, update the report in a separate change in cvkegps. Name the cvkegps-android commit.
- The workflow `.github/workflows/android.yml` builds the app, runs lint and the unit tests, and builds the release
  APK for each push to `main` and each pull request. Check its result before you report a change as verified.

## Agent skills

### Issue tracker

Issues for the Android app are in GitHub Issues for `Glebenator/cvkegps-android`. Use the `gh` CLI. A feature that both apps need has a parent issue in `Glebenator/cvkegps` and one sub-issue for each app. See `docs/agents/issue-tracker.md` in the cvkegps repository.

### Triage labels

Use the five default triage labels: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. See `docs/agents/triage-labels.md` in the cvkegps repository.

### Domain docs

The glossary (`GLOSSARY.md`) and the decisions (`docs/adr/`) are in the cvkegps repository. See `docs/agents/domain.md` there.
