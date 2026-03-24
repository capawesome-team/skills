---
name: capacitor-app-development
description: "Guides the agent through general Capacitor app development topics. Covers core concepts (native bridge, plugins, web layer), Capacitor CLI usage, app configuration (capacitor.config.ts, splash screens, app icons, deep links), platform management (Android, iOS, Electron, PWA), edge-to-edge and safe area handling on Android, live reload setup, storage solutions, file handling, security best practices, CI/CD references, iOS package managers (SPM, CocoaPods), and troubleshooting for Android and iOS. Do not use for creating new Capacitor apps, Capacitor plugin APIs, creating Capacitor plugins, in-app purchases, upgrading Capacitor versions, Cordova or PhoneGap migration, or framework-specific patterns (Angular, React, Vue)."
metadata:
  author: capawesome-team
  source: https://github.com/capawesome-team/skills/tree/main/skills/capacitor-app-development
---

# Capacitor App Development

General guidance for developing cross-platform apps with Capacitor, covering core concepts, CLI usage, app configuration, platform management, and troubleshooting.

## Prerequisites

1. **Capacitor 6, 7, or 8** app already created and initialized.
2. Node.js and npm installed (Node 18+ for Cap 6, Node 20+ for Cap 7, Node 22+ for Cap 8).
3. For **Android**: Android Studio installed.
4. For **iOS**: Xcode installed on macOS.

## Agent Behavior

- **Auto-detect before asking.** Inspect the project for platforms (`android/`, `ios/`), framework (`vite.config.ts`, `angular.json`, `webpack.config.js`, `next.config.js`), Capacitor version (`@capacitor/core` in `package.json`), and iOS dependency manager (`ios/App/Podfile` for CocoaPods vs. SPM).
- **Route to the right reference file.** Use the topic index below to identify the correct reference file and read it before providing guidance.
- **One topic at a time.** Address the user's specific question. Do not provide a general overview unless explicitly requested.
- **Prefer actionable instructions.** Provide exact file paths, commands, and diff blocks rather than conceptual explanations.

## Procedures

### Step 1: Identify the Topic

Match the user's request to a topic from the index below. If the request spans multiple topics, address them sequentially.

If the request matches a topic covered by a **different skill**, redirect the user to that skill (see Related Skills at the bottom).

### Step 2: Analyze the Project

Auto-detect the following by reading project files:

1. **Capacitor version**: Read `@capacitor/core` version from `package.json`.
2. **Platforms**: Check which directories exist (`android/`, `ios/`).
3. **Framework**: Check for build tool config files.
4. **Capacitor config format**: Check for `capacitor.config.ts` or `capacitor.config.json`.
5. **iOS dependency manager**: Check for `ios/App/Podfile` (CocoaPods) or SPM usage.

### Step 3: Read the Reference File and Apply Guidance

Read the reference file for the matched topic and apply the instructions to the user's project.

## Topic Index

| Topic | Reference |
| ----- | --------- |
| Core concepts (native bridge, how Capacitor works) | `references/core-concepts.md` |
| Capacitor platforms (Android, iOS, Electron, PWA) | `references/platforms.md` |
| Capacitor CLI commands | `references/cli.md` |
| App configuration (`capacitor.config.ts`) | `references/app-configuration.md` |
| Splash screens and app icons | `references/splash-screens-and-icons.md` |
| Deep links and universal links | `references/deep-links.md` |
| Android edge-to-edge support | `references/edge-to-edge.md` |
| Android safe area handling | `references/safe-area.md` |
| Live reload setup | `references/live-reload.md` |
| Storage solutions | `references/storage.md` |
| File handling best practices | `references/file-handling.md` |
| Security best practices | `references/security.md` |
| iOS package managers (SPM, CocoaPods) | `references/ios-package-managers.md` |
| CI/CD for Capacitor apps | `references/ci-cd.md` |
| Testing (unit and E2E) | `references/testing.md` |
| Cross-platform best practices | `references/cross-platform-best-practices.md` |
| Android troubleshooting | `references/troubleshooting-android.md` |
| iOS troubleshooting | `references/troubleshooting-ios.md` |

## Error Handling

- **`npx cap sync` fails**: Verify `@capacitor/core` and `@capacitor/cli` versions match. On iOS with CocoaPods, run `cd ios/App && pod install`. On Android, sync Gradle files in Android Studio.
- **Android build fails after configuration changes**: Clean the build with `cd android && ./gradlew clean`, then rebuild.
- **iOS build fails after configuration changes**: Clean the build folder in Xcode (Product > Clean Build Folder) or delete `ios/App/Pods` and run `cd ios/App && pod install`.
- **Plugin not found at runtime**: Run `npx cap sync` after any plugin installation. Verify the plugin appears in `capacitor.config` or the native project.
- **Live reload not connecting**: Verify the device and development machine are on the same network. Check that the server URL in `capacitor.config` uses the correct LAN IP address.
- **Deep links not working**: Verify the site association file is accessible at `https://<domain>/.well-known/apple-app-site-association` (iOS) or `https://<domain>/.well-known/assetlinks.json` (Android). Verify the app is signed with the correct certificate.

## Related Skills

- **`capacitor-app-creation`** — Create a new Capacitor app.
- **`capacitor-angular`** — Angular-specific Capacitor development.
- **`capacitor-react`** — React-specific Capacitor development.
- **`capacitor-vue`** — Vue-specific Capacitor development.
- **`capacitor-plugins`** — Install and configure Capacitor plugins.
- **`capacitor-plugin-development`** — Create custom Capacitor plugins.
- **`capacitor-app-upgrades`** — Upgrade Capacitor to a newer major version.
- **`capacitor-in-app-purchases`** — Set up in-app purchases.
- **`capawesome-cloud`** — Cloud builds, live updates, and app store publishing.
