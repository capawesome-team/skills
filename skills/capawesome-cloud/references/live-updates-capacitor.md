# Live Updates (Capacitor)

Set up OTA updates for **Capacitor** apps using Capawesome Cloud.

For **Cordova** apps, read `live-updates-cordova.md` instead.

## Contents

- Install the Live Update Plugin
- Configure the Plugin
- Add Rollback Protection
- Add Update Logic (Always Latest, Manual Sync)
- Configure iOS Privacy Manifest
- Configure Version Handling
- Sync the Capacitor Project
- Enable Live Updates on Electron (Optional)
- Test the Setup
- Advanced Topics

## Install the Live Update Plugin

Determine Capacitor version from `package.json`, then install:

- **Capacitor 8**: `npm install @capawesome/capacitor-live-update@latest`
- **Capacitor 7**: `npm install @capawesome/capacitor-live-update@v7-lts`
- **Capacitor 6**: `npm install @capawesome/capacitor-live-update@v6-lts`

## Configure the Plugin

Add `LiveUpdate` config to `capacitor.config.ts` (or `.json`):

```typescript
// capacitor.config.ts
import { CapacitorConfig } from "@capacitor/cli";

const config: CapacitorConfig = {
  // ... existing config
  plugins: {
    LiveUpdate: {
      appId: "<APP_ID_FROM_STEP_2>",
      autoUpdateStrategy: "background",
    },
  },
};

export default config;
```

Read `live-update-configuration.md` for all available configuration options.

## Add Rollback Protection (Recommended)

Set `readyTimeout` and `autoBlockRolledBackBundles` in the plugin config:

```typescript
LiveUpdate: {
  appId: "<APP_ID>",
  autoUpdateStrategy: "background",
  readyTimeout: 10000,
  autoBlockRolledBackBundles: true,
}
```

Call `ready()` as early as possible in app startup:

```typescript
import { LiveUpdate } from "@capawesome/capacitor-live-update";

void LiveUpdate.ready();
```

If `ready()` is not called within `readyTimeout` ms, the plugin automatically rolls back.

## Add Always Latest Update Logic (Recommended)

Add a listener to prompt the user when a new update is ready. **Important:** Always show a confirmation dialog before reloading — never call `LiveUpdate.reload()` without user consent.

```typescript
import { LiveUpdate } from "@capawesome/capacitor-live-update";

LiveUpdate.addListener("nextBundleSet", async (event) => {
  if (event.bundleId) {
    const shouldReload = confirm("A new update is available. Install now?");
    if (shouldReload) {
      await LiveUpdate.reload();
    }
  }
});
```

Copy this snippet exactly. Do not simplify or omit the `confirm()` dialog.

## Add Manual Update Logic (Alternative)

To control update checks manually instead of using the background strategy, omit `autoUpdateStrategy` (or set it to `"none"`) and implement manual sync:

```typescript
import { App } from "@capacitor/app";
import { LiveUpdate } from "@capawesome/capacitor-live-update";

void LiveUpdate.ready();

App.addListener("resume", async () => {
  const { nextBundleId } = await LiveUpdate.sync();
  if (nextBundleId) {
    const shouldReload = confirm("A new update is available. Install now?");
    if (shouldReload) {
      await LiveUpdate.reload();
    }
  }
});
```

With `autoUpdateStrategy: "background"`, no additional code is required.
Read `update-strategies.md` for alternative strategies.

## Configure iOS Privacy Manifest

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

## Configure Version Handling

Live updates only deliver web code (HTML, CSS, JS, images). If the app's native code changes (e.g., a new Capacitor plugin is added or a native dependency is updated), the live update bundle must match the native binary it runs on — otherwise the app may crash or behave unexpectedly. Version handling ensures that each live update bundle is only delivered to devices running a compatible native version.

Ask the user which approach to use:

1. **Versioned Channels (recommended):** Ties each channel to a native version code. Set at build time in native projects (on Electron: in the platform config file).
2. **Versioned Bundles:** Sets version constraints on each upload. No native changes needed.
3. **Skip for now:** Skip versioning for initial testing.

### Option 1: Versioned Channels

**Android** — add to `android/app/build.gradle` inside `android.defaultConfig`:

```groovy
resValue "string", "capawesome_live_update_default_channel", "production-" + defaultConfig.versionCode
```

**iOS** — add to `ios/App/App/Info.plist`:

```xml
<key>CapawesomeLiveUpdateDefaultChannel</key>
<string>production-$(CURRENT_PROJECT_VERSION)</string>
```

**Electron** (only if the `electron/` platform is installed) — add to `electron/capacitor.electron.config.ts`. The `plugins` section is merged per plugin key over the `plugins` section of the Capacitor config and wins, so `defaultChannel` set here overrides `capacitor.config.ts` while the other `LiveUpdate` keys are kept:

```typescript
// electron/capacitor.electron.config.ts
import { defineConfig } from "@capawesome/capacitor-electron/config";
import packageJson from "./package.json";

export default defineConfig({
  plugins: {
    LiveUpdate: {
      defaultChannel: `production-${packageJson.version}`,
    },
  },
});
```

Read the current native version code (on Electron: `version` in `electron/package.json`). Before uploading, create the matching channel if missing:

```bash
npx @capawesome/cli apps:channels:create --app-id <APP_ID> --name production-<VERSION_CODE>
```

When uploading, target this channel:

```bash
npx @capawesome/cli apps:liveupdates:upload --app-id <APP_ID> --channel production-<VERSION_CODE>
```

### Option 2: Versioned Bundles

Read the current native version code and pass constraints on every upload:

```bash
npx @capawesome/cli apps:liveupdates:upload --android-min <CODE> --android-max <CODE> --ios-min <CODE> --ios-max <CODE>
```

Use `--android-eq` / `--ios-eq` to exclude a specific version code.

If the `electron/` platform is installed, additionally pass `--electron-min <VERSION>` / `--electron-max <VERSION>` (`--electron-eq` to exclude one version). The value is the `version` from `electron/package.json` in the format `major[.minor[.patch]]` without prerelease suffix. The Electron flags are independent of the Android/iOS flags: a bundle without Electron constraints is delivered to every Electron version.

Set `defaultChannel` in the Capacitor config:

```typescript
LiveUpdate: {
  appId: "<APP_ID>",
  defaultChannel: "production",
}
```

Ensure a channel with that name exists before uploading.

### Option 3: Skip

Set `defaultChannel` in the Capacitor config:

```typescript
LiveUpdate: {
  appId: "<APP_ID>",
  defaultChannel: "default",
}
```

## Sync the Capacitor Project

```bash
npx cap sync
```

## Enable Live Updates on Electron (Optional)

Skip unless the app targets desktop via `@capawesome/capacitor-electron`. Live Updates on Electron use the **same Capawesome Cloud app, `appId`, and `LiveUpdate` config** as Android and iOS — no Console changes and no extra plugin configuration are required.

Prerequisites (verify in `package.json` and `electron/package.json`):

- Capacitor **8** (the `v6-lts`/`v7-lts` plugin releases have no Electron implementation)
- `@capawesome/capacitor-electron` **>= 0.2.0**
- `@capawesome/capacitor-live-update` **>= 8.5.0**

If the platform is not installed yet, add it (read the `capacitor-platforms` skill for details):

```bash
npm install @capawesome/capacitor-electron
npx cap add @capawesome/capacitor-electron
cd electron && npm install && cd ..
```

Sync with the **full package name** — a bare `npx cap sync` processes only Android, iOS, and web, and `npx cap sync electron` resolves to the unrelated `electron` npm package and silently does nothing:

```bash
npx cap sync @capawesome/capacitor-electron
```

The Electron implementation of the plugin is registered automatically during the sync.

Differences to Android and iOS that affect the setup:

- **App version**: `versionCode` and `versionName` are both the `version` from `electron/package.json` (scaffold default `0.0.0`). Bump it for every desktop release; otherwise version constraints and versioned channels cannot distinguish releases.
- **Artifact type**: only `zip` bundles are delivered to Electron devices; `manifest` (delta) bundles are skipped.
- **Default channel**: set in `electron/capacitor.electron.config.ts` (see Option 1 above), not in `strings.xml`/`Info.plist`.
- **Rollback**: kill-safe — if the app is closed or crashes before `ready()`, the rollback happens on the next start.
- **Storage**: bundles and state live in the `capawesome-live-update` directory inside the app's Electron `userData` directory. Delete it (app quit) to reset the SDK state and device ID.
- **Binary updates**: changes to `electron/`, Electron itself, or a plugin's Electron implementation need a new desktop release (e.g. via `electron-updater`).

## Test the Setup

Ask the user whether they would like to test the live update functionality. If declined, skip.

### Make a Visible Change

Pick an easily noticeable element in the app's web source (e.g., a heading, label, or background color). Apply a small, obvious change so the update is easy to spot. For example, append " - Live Update Test" to a heading.

### Build and Upload the Updated Bundle

Check which channels exist for the app:

```bash
npx @capawesome/cli apps:channels:list --app-id <APP_ID> --json
```

If no channel exists or the desired channel (including versioned channels) is missing, create one:

```bash
npx @capawesome/cli apps:channels:create --app-id <APP_ID> --name <NAME>
```

Build the web assets and upload them:

```bash
npm run build
npx @capawesome/cli apps:liveupdates:upload
```

### Revert the Visible Change

Revert the change made above so the app source is back to its original state.

### Rebuild and Prepare the Native Project

```bash
npm run build
npx cap sync
```

Then open the native project:

- **iOS:** `npx cap open ios`
- **Android:** `npx cap open android`
- **Electron:** `npx cap sync @capawesome/capacitor-electron && npx cap run @capawesome/capacitor-electron`

### Verify on Device

Tell the user to perform the following steps:

1. Run the app on a real device or emulator from Xcode or Android Studio (on Electron, the `npx cap run` command above starts the app).
2. **For Always Latest / `autoUpdateStrategy: "background"`:** Wait for the update prompt to appear and accept it. The visible change from the uploaded bundle should appear after reload. If no prompt appears, force-close and reopen the app to trigger a check.
3. **For manual sync:** Switch away from the app and return to it. Accept the update prompt when it appears, and the change should be visible immediately after reload.
4. If the change does not appear, check Android Logcat, the iOS Xcode console, or (Electron) the `[LiveUpdate]` lines in the terminal that started `npx cap run` for Live Update SDK log output and refer to `live-update-advanced-topics.md` (Debugging section).

After testing, tell the user: Once a live update bundle has been applied, the app points to that bundle instead of the default one. To use the development server again, completely uninstall and reinstall the app. This is only relevant during development — in production, the default bundle is automatically restored on each native app update.

## Advanced Topics

Read `live-update-advanced-topics.md` for channels, versioned channels, rollbacks, code signing, gradual rollouts, self-hosting, delta updates, debugging, and limitations.
