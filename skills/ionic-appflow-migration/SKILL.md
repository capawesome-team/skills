---
name: ionic-appflow-migration
description: "Guides the agent through migrating an existing Ionic/Capacitor project from Ionic Appflow to Capawesome Cloud. Detects which Appflow features are in use (Live Updates, Native Builds, App Store Publishing) and provides step-by-step migration for each feature to its Capawesome Cloud equivalent. Covers SDK replacement, configuration mapping, API migration, CI/CD pipeline updates, and verification. References the capawesome-cloud skill for detailed Capawesome Cloud setup procedures. Do not use for setting up Capawesome Cloud from scratch without an existing Appflow project, for non-Capacitor mobile frameworks, or for migrating Ionic Enterprise plugins. Pure Cordova apps (without Capacitor) should migrate their live updates to the Cordova Live Update SDK (@capawesome/cordova-live-update) instead — this skill's Live Updates migration targets the Capacitor plugin."
metadata:
  author: capawesome-team
  source: https://github.com/capawesome-team/skills/tree/main/skills/ionic-appflow-migration
---

# Ionic Appflow Migration

Migrate an existing Ionic/Capacitor project from Ionic Appflow to Capawesome Cloud.

Ionic Appflow reaches **end of life on December 31, 2027**, and new customer sales already stopped in February 2025. Existing customers keep access until the shutdown date, but every Appflow project needs a migration plan — migrating early leaves room for a gradual, low-risk rollout.

## Prerequisites

1. A **Capacitor 6, 7, or 8** app currently using Ionic Appflow. For **pure Cordova apps** (without Capacitor), migrate live updates to the Cordova Live Update SDK ([`@capawesome/cordova-live-update`](https://github.com/capawesome-team/cordova-live-update)) instead — the SDK migration steps in this skill (Step 3) target the Capacitor plugin. The Capawesome Cloud, native build, publishing, and CI/CD steps apply to both.
2. Node.js 18+ and npm installed.
3. Access to the project's source code repository.
4. A [Capawesome Cloud](https://console.cloud.capawesome.io) account and organization.

## General Rules

- Before running any `@capawesome/cli` command for the first time, run it with the `--help` flag to review all available options.
- Do not remove Ionic Appflow configuration until the corresponding Capawesome Cloud feature is fully set up and verified.
- Determine the Capacitor version from `package.json` (`@capacitor/core`) before making any changes — it affects which plugin version and update strategy to use.

## Procedures

### Step 1: Detect Ionic Appflow Usage

Scan the project to determine which Appflow features are in use.

#### 1.1 Check for Live Updates

Search for these signals:

1. `@capacitor/live-updates` in `package.json` (dependencies or devDependencies).
2. `cordova-plugin-ionic` in `package.json` — this is the legacy Cordova SDK for Ionic Live Updates.
3. A `LiveUpdates` key inside the `plugins` object in `capacitor.config.ts` or `capacitor.config.json`.
4. Imports of `@capacitor/live-updates` or `cordova-plugin-ionic` in TypeScript/JavaScript source files.

If any signal is found, mark **Live Updates** as in use. Also record which SDK is in use (`@capacitor/live-updates` or `cordova-plugin-ionic`).

Record the current configuration values:

- `appId` (Ionic Appflow app ID)
- `autoUpdateMethod` (`background`, `always`, or `none`)
- `channel`
- `enabled`
- `maxVersions`

#### 1.2 Check for Native Builds

Search for these signals:

1. Appflow CLI build invocations in CI/CD configuration files (e.g., `.github/workflows/*.yml`, `.gitlab-ci.yml`, `bitrise.yml`, `Jenkinsfile`, `azure-pipelines.yml`). The Appflow CLI binary is `appflow` (legacy: `ionic-cloud`), so grep for `appflow build` and `ionic-cloud build`.
2. References to `dashboard.ionicframework.com` or `appflow.ionic.io` in CI/CD files or scripts.
3. An `appflow.config.json`, `.appflow.yaml`, or similar Appflow configuration file in the project root.

If any signal is found, mark **Native Builds** as in use.

#### 1.3 Check for App Store Publishing

Search for these signals:

1. Appflow CLI store deploy invocations in CI/CD configuration files — grep for `appflow deploy android`, `appflow deploy ios`, and the legacy `ionic-cloud deploy` (note: `appflow deploy web` deploys a live update to a channel and belongs to Live Updates, not App Store Publishing).
2. Appflow deploy destinations configured for app store submission (e.g., `appflow destination` commands or store destinations referenced via `--destination`).

If any signal is found, mark **App Store Publishing** as in use.

#### 1.4 Present Findings

Present the detected features to the user. Ask the user to confirm which features to migrate. The user may choose to migrate all detected features or only a subset.

### Step 2: Set Up Capawesome Cloud

#### 2.1 Authenticate

```bash
npx @capawesome/cli login
```

#### 2.2 Create an App

Skip if the user already has a Capawesome Cloud app ID.

```bash
npx @capawesome/cli apps:create
```

Save the returned **app ID** (UUID) for subsequent steps.

### Step 3: Migrate Live Updates

Skip this step if Live Updates was not detected or the user chose not to migrate it.

#### 3.1 Remove the Ionic Live Updates SDK

If the project uses `@capacitor/live-updates`:

```bash
npm uninstall @capacitor/live-updates
```

If the project uses the legacy Cordova SDK (`cordova-plugin-ionic`):

```bash
npm uninstall cordova-plugin-ionic
```

Read `references/cordova-sdk-migration.md` for the native configuration cleanup steps (removing legacy keys from `Info.plist`, `strings.xml`, and Capacitor config).

**Scope note**: The steps below target Capacitor apps (including Capacitor apps that still ship the legacy Cordova SDK). If the project is a **pure Cordova app** without Capacitor, migrate to the Cordova Live Update SDK (`@capawesome/cordova-live-update`) instead of the Capacitor plugin — see the [Capawesome Cloud Live Updates setup](https://capawesome.io/docs/cloud/live-updates/setup/) for the Cordova installation and configuration.

#### 3.2 Install the Capawesome Live Update Plugin

Install the version matching the project's Capacitor version:

- **Capacitor 8**: `npm install @capawesome/capacitor-live-update@latest`
- **Capacitor 7**: `npm install @capawesome/capacitor-live-update@v7-lts`
- **Capacitor 6**: `npm install @capawesome/capacitor-live-update@v6-lts`

#### 3.3 Update the Capacitor Configuration

Replace the `LiveUpdates` plugin config with `LiveUpdate` in `capacitor.config.ts` (or `.json`). Map the configuration options as follows:

| Ionic Appflow (`LiveUpdates`) | Capawesome Cloud (`LiveUpdate`) | Notes |
|---|---|---|
| `appId` | `appId` | Replace with the Capawesome Cloud app ID from Step 2 |
| `autoUpdateMethod: 'background'` | `autoUpdateStrategy: 'background'` | Same behavior |
| `autoUpdateMethod: 'always'` | `autoUpdateStrategy: 'background'` | Also add `nextBundleSet` listener (see Step 3.5) |
| `autoUpdateMethod: 'none'` | *(omit `autoUpdateStrategy`)* | Use manual sync code instead (see Step 3.7) |
| `channel` | `defaultChannel` | Same value |
| `enabled` | *(remove)* | Not needed — controlled in code |
| `maxVersions` | `autoDeleteBundles: true` | Boolean instead of number |

**Example — background strategy:**

```diff
 // capacitor.config.ts
 const config: CapacitorConfig = {
   plugins: {
-    LiveUpdates: {
-      appId: 'abc12345',
-      autoUpdateMethod: 'background',
-      channel: 'production',
-      maxVersions: 3
-    }
+    LiveUpdate: {
+      appId: '<CAPAWESOME_APP_ID>',
+      autoUpdateStrategy: 'background',
+      defaultChannel: 'production',
+      autoDeleteBundles: true
+    }
   }
 };
```

**Example — manual updates (`autoUpdateMethod: 'none'`, no `autoUpdateStrategy`):**

```diff
 // capacitor.config.ts
 const config: CapacitorConfig = {
   plugins: {
-    LiveUpdates: {
-      appId: 'abc12345',
-      autoUpdateMethod: 'none',
-      channel: 'production',
-      maxVersions: 3
-    }
+    LiveUpdate: {
+      appId: '<CAPAWESOME_APP_ID>',
+      defaultChannel: 'production',
+      autoDeleteBundles: true
+    }
   }
 };
```

#### 3.4 Update Import Statements and API Calls

Search all TypeScript/JavaScript files for imports of `@capacitor/live-updates` (or `cordova-plugin-ionic`) and replace. The Ionic SDK uses two import styles — replace both:

```diff
-import * as LiveUpdates from '@capacitor/live-updates';
+import { LiveUpdate } from '@capawesome/capacitor-live-update';
```

```diff
-import { LiveUpdates } from '@capacitor/live-updates';
+import { LiveUpdate } from '@capawesome/capacitor-live-update';
```

If the project uses the legacy Cordova SDK (`cordova-plugin-ionic`), read `references/cordova-sdk-migration.md` for the complete `Deploy` → `LiveUpdate` method mapping, including the granular check-download-extract-reload pattern and native config cleanup.

Replace **all** references to the `LiveUpdates` class (or `Deploy` class) with `LiveUpdate` (singular).

**`sync()` return value has changed.** The Ionic Capacitor SDK returns `{ activeApplicationPathChanged: boolean }`. The Capawesome SDK returns `{ nextBundleId: string | null }`. Update all code that checks the sync result:

```diff
 const result = await LiveUpdate.sync();
-if (result.activeApplicationPathChanged) {
+if (result.nextBundleId) {
   await LiveUpdate.reload();
 }
```

**`reload()` has the same signature** — no changes needed beyond the class name.

**`setConfig()`, `getConfig()`, and `resetConfig()` exist but have different signatures.** The Capawesome SDK splits config and channel management. For the channel, prefer passing it directly to `sync()` (or `fetchLatestBundle()`) — this keeps the selected channel explicit at the call site instead of relying on hidden, persisted state:

```diff
 // Setting config at runtime
-await LiveUpdates.setConfig({ appId: '456', channel: 'staging', maxVersions: 5 });
+await LiveUpdate.setConfig({ appId: '456' });
+// Pass the channel directly when fetching updates:
+await LiveUpdate.sync({ channel: 'staging' });
```

Only use `setChannel({ channel })` if the app needs a persistent channel subscription that applies to all subsequent update checks without passing the channel each time.

```diff
 // Getting config at runtime
-const config = await LiveUpdates.getConfig();
-console.log(config.channel);
+const config = await LiveUpdate.getConfig();   // { appId, autoUpdateStrategy }
+const { channel } = await LiveUpdate.getChannel(); // { channel }
```

```diff
 // Resetting config
-await LiveUpdates.resetConfig();
+await LiveUpdate.resetConfig();
```

`maxVersions` has no runtime equivalent — use `autoDeleteBundles: true` in the static Capacitor config instead.

#### 3.5 Add Always-Latest Update Logic

If the previous Ionic Appflow `autoUpdateMethod` was `always`, add a `nextBundleSet` listener to prompt the user when an update is ready. Add this code early in the app's initialization:

```typescript
import { LiveUpdate } from '@capawesome/capacitor-live-update';

LiveUpdate.addListener('nextBundleSet', async (event) => {
  if (event.bundleId) {
    const shouldReload = confirm('A new update is available. Install now?');
    if (shouldReload) {
      await LiveUpdate.reload();
    }
  }
});
```

Copy this snippet exactly. Do not simplify or omit the `confirm()` dialog.

#### 3.6 Migrate Force Update Logic (If Applicable)

If the project implements a "Force Update" pattern (extending the splash screen until the update completes), migrate it as follows:

```diff
 import { SplashScreen } from '@capacitor/splash-screen';
-import * as LiveUpdates from '@capacitor/live-updates';
+import { LiveUpdate } from '@capawesome/capacitor-live-update';

 const initializeApp = async () => {
-  const result = await LiveUpdates.sync();
-  if (result.activeApplicationPathChanged) {
-    await LiveUpdates.reload();
+  const { nextBundleId } = await LiveUpdate.sync();
+  if (nextBundleId) {
+    await LiveUpdate.reload();
   } else {
     await SplashScreen.hide();
   }
 };
```

This pattern may impact user experience on slow connections. Consider migrating to the background update strategy instead.

#### 3.7 Add Manual Update Logic (If Applicable)

If the previous Ionic Appflow `autoUpdateMethod` was `none`, keep `autoUpdateStrategy` unset and add manual sync logic:

```typescript
import { App } from '@capacitor/app';
import { LiveUpdate } from '@capawesome/capacitor-live-update';

void LiveUpdate.ready();

App.addListener('resume', async () => {
  const { nextBundleId } = await LiveUpdate.sync();
  if (nextBundleId) {
    const shouldReload = confirm('A new update is available. Install now?');
    if (shouldReload) {
      await LiveUpdate.reload();
    }
  }
});
```

#### 3.8 Add Rollback Protection (Recommended)

Add `readyTimeout` and `autoBlockRolledBackBundles` to the `LiveUpdate` config:

```typescript
LiveUpdate: {
  appId: '<CAPAWESOME_APP_ID>',
  autoUpdateStrategy: 'background',
  readyTimeout: 10000,
  autoBlockRolledBackBundles: true,
}
```

Call `ready()` as early as possible in app startup:

```typescript
import { LiveUpdate } from '@capawesome/capacitor-live-update';

void LiveUpdate.ready();
```

#### 3.9 Make Updates Version-Compatible (Do This Before Shipping to Production)

A live update can only change the web layer — it must stay compatible with the native binary already installed on the device. In Appflow this was handled with "minimum native version" semantics (`appflow live-update set-native-versions`). In Capawesome Cloud, the recommended approach is **versioned channels**: pin each native release to its own channel, derived from the version code, so a bundle only ever reaches compatible devices.

Configure the channel natively at build time. On Android, add to `android/app/build.gradle`:

```groovy
android {
    defaultConfig {
        resValue "string", "capawesome_live_update_default_channel",
                 "production-" + defaultConfig.versionCode
    }
}
```

On iOS, add to `ios/App/App/Info.plist`:

```xml
<key>CapawesomeLiveUpdateDefaultChannel</key>
<string>production-$(CURRENT_PROJECT_VERSION)</string>
```

Then upload each bundle to the matching `production-<versionCode>` channel. Alternatively, keep Appflow-style per-bundle native version constraints using the `--android-min`/`--android-max`/`--ios-min`/`--ios-max` flags on `apps:liveupdates:upload`, or set them after upload with `apps:liveupdates:setnativeversions` (see the mapping table in Step 6).

#### 3.10 Configure iOS Privacy Manifest

Add to `ios/App/PrivacyInfo.xcprivacy` inside the `NSPrivacyAccessedAPITypes` array:

```xml
<dict>
  <key>NSPrivacyAccessedAPIType</key>
  <string>NSPrivacyAccessedAPICategoryUserDefaults</string>
  <key>NSPrivacyAccessedAPITypeReasons</key>
  <array>
    <string>CA92.1</string>
  </array>
</dict>
```

#### 3.11 Sync the Capacitor Project

```bash
npx cap sync
```

### Step 4: Migrate Native Builds

Skip this step if Native Builds was not detected or the user chose not to migrate it.

Native build setup in Capawesome Cloud requires connecting a Git repository, uploading signing certificates, configuring environments, and triggering builds. Use the `capawesome-cloud` skill — follow the **Native Builds** section starting from "Connect Git Repository." Specifically, read the `capawesome-cloud` skill's `references/native-builds.md` for the full procedure.

Remove any Ionic Appflow build commands from CI/CD configuration files after verifying that Capawesome Cloud builds work correctly (see Step 6 for CI/CD migration).

### Step 5: Migrate App Store Publishing

Skip this step if App Store Publishing was not detected or the user chose not to migrate it.

App store publishing setup requires creating destinations and configuring credentials for Apple App Store and/or Google Play Store. Use the `capawesome-cloud` skill — follow the **App Store Publishing** section. Specifically, read the `capawesome-cloud` skill's `references/app-store-publishing.md` for the full procedure.

Remove any Ionic Appflow deploy commands from CI/CD configuration files after verifying that Capawesome Cloud deployments work correctly (see Step 6 for CI/CD migration).

### Step 6: Update CI/CD Pipelines

Skip this step if the project has no CI/CD pipeline or uses Ionic Appflow's built-in build/deploy features without external CI/CD.

Use the following CLI command mapping as a reference when replacing Appflow commands:

| Appflow CLI Command | Capawesome CLI Equivalent |
|---|---|
| `appflow live-update upload-artifact` | `npx @capawesome/cli apps:liveupdates:upload` |
| `appflow live-update upload-artifact --signing-key <key>` | `npx @capawesome/cli apps:liveupdates:upload --private-key <key>` |
| `appflow live-update generate-signing-key` | `npx @capawesome/cli apps:liveupdates:generatesigningkey` |
| `appflow live-update create-channel` | `npx @capawesome/cli apps:channels:create` |
| `appflow live-update delete-channel` | `npx @capawesome/cli apps:channels:delete` |
| `appflow live-update list-channels` | `npx @capawesome/cli apps:channels:list` |
| `appflow live-update download-artifact` | `npx @capawesome/cli apps:builds:download --zip --build-id <BUILD_ID>` — downloads a web build artifact by build ID; there is no channel-based download command. Find the build ID via `apps:builds:list` or the Console |
| `appflow live-update set-native-versions` | `npx @capawesome/cli apps:liveupdates:setnativeversions --build-id <BUILD_ID>` with `--android-min`/`--android-max`/`--android-eq` and `--ios-min`/`--ios-max`/`--ios-eq`. Prefer versioned channels for new setups (see Step 3.9) |
| `appflow build android` / `appflow build ios` | `npx @capawesome/cli apps:builds:create --platform android` / `--platform ios` |
| `appflow build web` + `appflow deploy web` | `npx @capawesome/cli apps:liveupdates:create` — builds the web assets and deploys them to a channel in one command via Cloud Runners |
| `appflow deploy android` / `appflow deploy ios` | `npx @capawesome/cli apps:deployments:create` |

#### 6.1 Replace Authentication

Replace Ionic Appflow authentication with Capawesome Cloud token-based auth:

```diff
-# Ionic Appflow authentication
-IONIC_TOKEN: ${{ secrets.IONIC_TOKEN }}
+# Capawesome Cloud authentication
+- run: npx @capawesome/cli login --token ${{ secrets.CAPAWESOME_CLOUD_TOKEN }}
```

Generate a token in the [Capawesome Cloud Console](https://console.cloud.capawesome.io) and store it as a CI/CD secret named `CAPAWESOME_CLOUD_TOKEN`.

#### 6.2 Replace Build Commands

```diff
-appflow build android release --app-id=<APPFLOW_APP_ID> --commit=<COMMIT_SHA> --signing-cert="Android Release"
+npx @capawesome/cli apps:builds:create --app-id <APP_ID> --platform android --type release --git-ref main --certificate "<CERTIFICATE_NAME>" --yes
```

```diff
-appflow build ios app-store --app-id=<APPFLOW_APP_ID> --commit=<COMMIT_SHA> --signing-cert="iOS Distribution"
+npx @capawesome/cli apps:builds:create --app-id <APP_ID> --platform ios --type app-store --git-ref main --certificate "<CERTIFICATE_NAME>" --yes
```

Legacy pipelines may invoke the same commands via the `ionic-cloud` binary instead of `appflow` — replace those the same way.

Add `--detached` for non-blocking builds in CI/CD pipelines.

#### 6.3 Replace Live Update Upload Commands

Appflow deploys live updates in two steps (`appflow build web` followed by `appflow deploy web`). Replace both with a single upload of locally built web assets:

```diff
-appflow build web --app-id=<APPFLOW_APP_ID> --commit=<COMMIT_SHA>
-appflow deploy web --app-id=<APPFLOW_APP_ID> --build-id=<BUILD_ID> --destination=production
+npx @capawesome/cli apps:liveupdates:upload --app-id <APP_ID> --channel production
```

Alternatively, to keep building the web assets in the cloud (the closest equivalent to Appflow's build-and-deploy pipeline), use `apps:liveupdates:create`, which builds and deploys in one command via Cloud Runners:

```diff
-appflow build web --app-id=<APPFLOW_APP_ID> --commit=<COMMIT_SHA>
-appflow deploy web --app-id=<APPFLOW_APP_ID> --build-id=<BUILD_ID> --destination=production
+npx @capawesome/cli apps:liveupdates:create --app-id <APP_ID> --channel production --git-ref main --yes
```

#### 6.4 Replace App Store Deploy Commands

```diff
-appflow deploy android --app-id=<APPFLOW_APP_ID> --build-id=<BUILD_ID> --destination="Google Play"
+npx @capawesome/cli apps:deployments:create --app-id <APP_ID> --build-number <BUILD_NUMBER> --destination "<DESTINATION_NAME>"
```

#### 6.5 Remove Ionic Appflow Dependencies

After verifying the new CI/CD pipeline works:

1. Remove any `@ionic/appflow` or Appflow-related npm packages from `package.json`.
2. Remove Ionic Appflow environment variables from CI/CD secrets (e.g., `IONIC_TOKEN`).
3. Remove `appflow.config.json`, `.appflow.yaml`, or similar Appflow configuration files from the project root.

### Step 7: Test and Verify

#### 7.1 Verify Live Updates (if migrated)

Ask the user whether to test live update functionality. If accepted:

1. Make a small, visible change in the app (e.g., append " - Migration Test" to a heading).
2. Build the web assets: `npm run build`.
3. Check existing channels:
   ```bash
   npx @capawesome/cli apps:channels:list --app-id <APP_ID> --json
   ```
4. Create the target channel if missing:
   ```bash
   npx @capawesome/cli apps:channels:create --app-id <APP_ID> --name <CHANNEL_NAME>
   ```
5. Upload the bundle:
   ```bash
   npx @capawesome/cli apps:liveupdates:upload --app-id <APP_ID> --channel <CHANNEL_NAME>
   ```
6. Revert the visible change, rebuild, and sync:
   ```bash
   npm run build && npx cap sync
   ```
7. Open the native project (`npx cap open ios` or `npx cap open android`) and run on a device or emulator.
8. Verify the live update is applied. With `autoUpdateStrategy: "background"`, wait for the update prompt or force-close and reopen the app, or switch away from the app and return after more than 15 minutes to trigger a check.

#### 7.2 Verify Native Builds (if migrated)

Trigger a test build via the Capawesome CLI and verify the build artifact is produced.

#### 7.3 Verify App Store Publishing (if migrated)

Deploy a test build to the configured destination and verify it appears in TestFlight or Google Play Console.

#### 7.4 Clean Up

After all features are verified:

1. Confirm all Ionic Appflow SDK packages are removed from `package.json`.
2. Confirm all Ionic Appflow configuration is removed from `capacitor.config.ts`/`capacitor.config.json`.
3. Confirm all Ionic Appflow CI/CD commands and environment variables are removed.

## Error Handling

### Live Updates

- `npm uninstall @capacitor/live-updates` fails → The package may be listed under a different name. Search `package.json` for any package containing `live-updates` from `@capacitor` scope.
- `npx cap sync` fails after plugin swap → Verify the installed `@capawesome/capacitor-live-update` version matches the Capacitor version in `package.json`.
- App reverts to default bundle after restart → `LiveUpdate.ready()` is likely not called early enough. Add it as the first call in app initialization.
- Updates not detected with `autoUpdateStrategy: "background"` → Updates only checked if last check was >15 minutes ago. Force-close and restart the app. Check device logs for Live Update SDK output.
- `LiveUpdates is not defined` or similar runtime errors → Ensure all imports were updated from `@capacitor/live-updates` to `@capawesome/capacitor-live-update` and the class name changed from `LiveUpdates` to `LiveUpdate`.
- `Deploy is not defined` → The project uses the legacy Cordova SDK (`cordova-plugin-ionic`). Read `references/cordova-sdk-migration.md` for the full method mapping.
- `activeApplicationPathChanged is not a property` or sync result checks fail → The Capawesome SDK returns `{ nextBundleId }` instead of `{ activeApplicationPathChanged }`. Update all sync result checks.
- `setConfig is not a function` → The installed plugin release is outdated. Update to the latest release of the matching dist-tag (`@latest` for Capacitor 8, `@v7-lts` for 7, `@v6-lts` for 6) — all current tag releases include `setConfig()`.

### Native Builds

- Build fails with authentication error → Re-run `npx @capawesome/cli login`. For CI/CD, verify the `CAPAWESOME_CLOUD_TOKEN` secret is set correctly.
- Build fails with missing signing certificate → Upload certificates via `npx @capawesome/cli apps:certificates:create`. Read the `capawesome-cloud` skill's `references/certificates-android.md` or `references/certificates-ios.md`.
- Build fails with `invalid source release` → Set `JAVA_VERSION` environment variable in the build environment. Read the `capawesome-cloud` skill's `references/build-troubleshooting.md`.

### App Store Publishing

- Destination creation fails → Verify credentials are correct. Read the `capawesome-cloud` skill's `references/apple-app-store-credentials.md` or `references/google-play-store-credentials.md`.
- Deployment fails with "build not found" → Ensure the build completed successfully before deploying.

### CI/CD

- CI/CD pipeline fails after migration → Compare the old and new pipeline configurations side by side. Ensure all Appflow command replacements use the correct Capawesome CLI flags. Check that `CAPAWESOME_CLOUD_TOKEN` is available as a secret in the CI/CD environment.

## Related Skills

- **`capawesome-cloud`** — Referenced throughout this skill for Native Builds and App Store Publishing setup.
- **`ionic-enterprise-sdk-migration`** — If the project also uses discontinued Ionic Enterprise SDK plugins (Auth Connect, Identity Vault, Secure Storage), use this skill to migrate them to Capawesome alternatives.
- **`capgo-cloud-migration`** — Use this skill instead if the project migrates from Capgo rather than Ionic Appflow.
