# Live Updates

Set up OTA updates for Capacitor apps using Capawesome Cloud.

## Contents

- Install the Live Update Plugin
- Configure the Plugin
- Add Rollback Protection
- Add Update Logic (Always Latest, Manual Sync)
- Configure iOS Privacy Manifest
- Configure Version Handling
- Sync the Capacitor Project
- Test the Setup
- Advanced Topics

## Install the Live Update Plugin

Determine Capacitor version from `package.json`, then install:

- **Capacitor 8**: `npm install @capawesome/capacitor-live-update@latest`
- **Capacitor 7**: `npm install @capawesome/capacitor-live-update@^7.3.0`
- **Capacitor 6**: `npm install @capawesome/capacitor-live-update@^6.0.0`

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
      autoUpdateStrategy: "background", // Capacitor 7/8 only — omit for Capacitor 6
    },
  },
};

export default config;
```

Read `live-update-configuration.md` for all available configuration options.

## Add Rollback Protection (Recommended)

Set `readyTimeout` in the plugin config:

```typescript
LiveUpdate: {
  appId: "<APP_ID>",
  autoUpdateStrategy: "background",
  readyTimeout: 10000,
}
```

Call `ready()` as early as possible in app startup:

```typescript
import { LiveUpdate } from "@capawesome/capacitor-live-update";

void LiveUpdate.ready();
```

If `ready()` is not called within `readyTimeout` ms, the plugin automatically rolls back.

## Add Always Latest Update Logic (Recommended, Capacitor 7/8 Only)

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

## Add Manual Update Logic (Capacitor 6 Only)

Capacitor 6 does not support `autoUpdateStrategy`. Implement manual sync:

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

For Capacitor 7/8 with `autoUpdateStrategy: "background"`, no additional code is required.
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

Ask the user which approach to use:

1. **Versioned Channels (recommended):** Ties each channel to a native version code. Set at build time in native projects.
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

Read the current native version code. Before uploading, create the matching channel if missing:

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

### Verify on Device

Tell the user to perform the following steps:

1. Run the app on a real device or emulator from Xcode or Android Studio.
2. **For Always Latest / `autoUpdateStrategy: "background"` (Capacitor 7/8):** Wait for the update prompt to appear and accept it. The visible change from the uploaded bundle should appear after reload. If no prompt appears, force-close and reopen the app to trigger a check.
3. **For manual sync (Capacitor 6):** Switch away from the app and return to it. Accept the update prompt when it appears, and the change should be visible immediately after reload.
4. If the change does not appear, check Android Logcat or iOS Xcode console for Live Update SDK log output and refer to `live-update-advanced-topics.md` (Debugging section).

After testing, tell the user: Once a live update bundle has been applied, the app points to that bundle instead of the default one. To use the development server again, completely uninstall and reinstall the app. This is only relevant during development — in production, the default bundle is automatically restored on each native app update.

## Advanced Topics

Read `live-update-advanced-topics.md` for channels, versioned channels, rollbacks, code signing, gradual rollouts, self-hosting, delta updates, debugging, and limitations.
