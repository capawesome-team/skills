---
name: capacitor-live-update
description: Guides the agent through setting up and managing Capacitor Live Updates using Capawesome Cloud, the @capawesome/capacitor-live-update plugin, and the @capawesome/cli. Covers creating apps in Capawesome Cloud, installing and configuring the Live Update SDK, publishing bundles, configuring channels and update strategies, rollbacks, code signing, and CI/CD integration. Do not use for native build management, app store submissions, or non-Capacitor mobile frameworks.
---

# Capacitor Live Update

Set up and manage Over-the-Air (OTA) updates for Capacitor apps using Capawesome Cloud.
Live Updates deliver web asset changes (HTML, CSS, JS, images) to users instantly without app store review.

## Prerequisites

Before proceeding, verify:

1. The project is a **Capacitor 6, 7, or 8** app.
2. The user has a [Capawesome Cloud](https://console.cloud.capawesome.io) account and organization.
3. Node.js and npm are installed.

## General Rules

Before running any `@capawesome/cli` command for the first time, run it with the `--help` flag to review all available options. This ensures the correct flags are used and avoids unnecessary prompts.

## Procedures

### Step 1: Authenticate with Capawesome Cloud

Run the Capawesome CLI login command:

```bash
npx @capawesome/cli login
```

This opens a browser for interactive authentication. For CI/CD, use token-based auth:

```bash
npx @capawesome/cli login --token <token>
```

### Step 2: Create an App in Capawesome Cloud

Skip this step if the user already has an app ID.

```bash
npx @capawesome/cli apps:create
```

The CLI prompts for the organization and app name, then outputs the **app ID** (a UUID like `6e351b4f-69a7-415e-a057-4567df7ffe94`). Save this ID for the next step.

### Step 3: Install the Live Update Plugin

Determine the Capacitor version from the project's `package.json`, then install the appropriate plugin version:

- **Capacitor 8**: `npm install @capawesome/capacitor-live-update@latest`
- **Capacitor 7**: `npm install @capawesome/capacitor-live-update@^7.3.0`
- **Capacitor 6**: `npm install @capawesome/capacitor-live-update@^6.0.0`

### Step 4: Configure the Plugin

Add the `LiveUpdate` plugin configuration to the Capacitor config file (`capacitor.config.ts` or `capacitor.config.json`).

**For Capacitor 7 and 8** (with auto-update):

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

**For Capacitor 6** (no auto-update support, manual sync required):

```typescript
// capacitor.config.ts
import { CapacitorConfig } from "@capacitor/cli";

const config: CapacitorConfig = {
  // ... existing config
  plugins: {
    LiveUpdate: {
      appId: "<APP_ID_FROM_STEP_2>",
    },
  },
};

export default config;
```

Replace `<APP_ID_FROM_STEP_2>` with the actual app ID.

Read `references/configuration.md` for all available configuration options (e.g., `readyTimeout`, `publicKey`, `defaultChannel`, `serverDomain`).

### Step 5: Add Rollback Protection (Recommended)

Set `readyTimeout` in the plugin config to enable automatic rollback if a bad bundle is deployed:

```typescript
LiveUpdate: {
  appId: "<APP_ID>",
  autoUpdateStrategy: "background",
  readyTimeout: 10000,
}
```

Then call `ready()` as early as possible in the app startup:

```typescript
import { LiveUpdate } from "@capawesome/capacitor-live-update";

void LiveUpdate.ready();
```

If `ready()` is not called within `readyTimeout` ms, the plugin automatically rolls back to the previous bundle.

### Step 6: Add Manual Update Logic (Capacitor 6 Only)

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
Read `references/update-strategies.md` for alternative strategies (Always Latest, Force Update, Instant).

### Step 7: Configure iOS Privacy Manifest

Add the `NSPrivacyAccessedAPICategoryUserDefaults` entry to `ios/App/PrivacyInfo.xcprivacy`:

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

Add this as a new entry inside the `NSPrivacyAccessedAPITypes` array if the file already exists.

### Step 8: Configure Version Handling

Live updates can only deliver web asset changes. When native code changes (new plugins, SDK updates, etc.), the app must be resubmitted to the stores. To prevent incompatible bundles from being delivered to older native versions, Capawesome Cloud offers two approaches.

Ask the user which approach to use:

1. **Versioned Channels (recommended):** Ties each channel to a native version code. The channel is set at build time in the native project, so no TypeScript code is needed. This is the safest approach.
2. **Versioned Bundles:** Sets version constraints on each upload. Simpler to start with but requires specifying version ranges on every upload.
3. **Skip for now:** Skip versioning for initial testing. Can be added later.

#### Option 1: Versioned Channels

Add the default channel configuration to both native projects.

**Android** — add to `android/app/build.gradle` inside the `android.defaultConfig` block:

```groovy
resValue "string", "capawesome_live_update_default_channel", "production-" + defaultConfig.versionCode
```

**iOS** — add to `ios/App/App/Info.plist`:

```xml
<key>CapawesomeLiveUpdateDefaultChannel</key>
<string>production-$(CURRENT_PROJECT_VERSION)</string>
```

Then read the current native version code from the project files (e.g., `versionCode` in `android/app/build.gradle` or `CURRENT_PROJECT_VERSION` in the iOS project). Before uploading a bundle, create the matching channel if it does not exist:

```bash
npx @capawesome/cli apps:channels:create --app-id <APP_ID> --name production-<VERSION_CODE>
```

When uploading, always target this channel:

```bash
npx @capawesome/cli apps:liveupdates:upload --app-id <APP_ID> --channel production-<VERSION_CODE>
```

#### Option 2: Versioned Bundles

No native project changes are needed. Instead, read the current native version code from the project files and pass version constraints on every upload:

```bash
npx @capawesome/cli apps:liveupdates:upload --android-min <CODE> --android-max <CODE> --ios-min <CODE> --ios-max <CODE>
```

Use `--android-eq` and `--ios-eq` to exclude a specific version code if the bundle was built from that exact native version (meaning the native binary already contains the same web assets).

Since versioned channels are not used, set `defaultChannel` in the Capacitor config so the plugin knows which channel to fetch updates from:

```typescript
LiveUpdate: {
  appId: "<APP_ID>",
  defaultChannel: "production",
}
```

Make sure a channel with that name exists before uploading (see Step 10).

#### Option 3: Skip

Since versioned channels are not used, set `defaultChannel` in the Capacitor config so the plugin knows which channel to fetch updates from:

```typescript
LiveUpdate: {
  appId: "<APP_ID>",
  defaultChannel: "default",
}
```

### Step 9: Sync the Capacitor Project

```bash
npx cap sync
```

### Step 10: Test the Setup

Setup is now complete. Ask the user whether they would like to test the live update functionality. If declined, skip to the **Advanced Topics** section. If confirmed, continue with Steps 11 through 14.

### Step 11: Build and Upload the Initial Bundle

First, check which channels exist for the app:

```bash
npx @capawesome/cli apps:channels:list --app-id <APP_ID> --json
```

If no channel exists or the desired channel (including versioned channels from Step 8) is missing, create one:

```bash
npx @capawesome/cli apps:channels:create --app-id <APP_ID> --name <NAME>
```

Then build the web assets and upload them:

First, check which channels exist for the app:

```bash
npx @capawesome/cli apps:channels:list --app-id <APP_ID> --json
```

If no channel exists or the desired channel (including versioned channels from Step 9) is missing, create one:

```bash
npx @capawesome/cli apps:channels:create --app-id <APP_ID> --name <NAME>
```

Then build the web assets and upload them:

```bash
npm run build
npx @capawesome/cli apps:liveupdates:upload
```

The CLI prompts for the web assets path (e.g., `dist` or `www`), the app, and the channel. After upload, the bundle is instantly available.

### Step 12: Make a Visible Change

Pick an easily noticeable element in the app's web source — for example, a heading, a label, or a background color. Apply a small, obvious change so the update is easy to spot after it arrives on the device. For example, append a text like " - Live Update Test" to a heading.

### Step 13: Build and Upload the Updated Bundle

Build the modified web assets and upload them again:

```bash
npm run build
npx @capawesome/cli apps:liveupdates:upload
```

### Step 14: Verify on Device

Tell the user to perform the following steps:

1. Open the app on a real device or emulator.
2. **For `autoUpdateStrategy: "background"` (Capacitor 7/8):** Force-close the app and reopen it. The update downloads silently in the background. After roughly 15–30 seconds, force-close and reopen the app once more — the visible change should now appear.
3. **For manual sync (Capacitor 6):** Switch away from the app and return to it. Accept the update prompt when it appears, and the change should be visible immediately after reload.
4. If the change does not appear, check Android Logcat or iOS Xcode console for Live Update SDK log output and refer to the **Debugging** section below.

## Advanced Topics

### Channels

Channels distribute different bundles to different user groups (e.g., `production`, `staging`, `beta`). A `default` channel is created automatically.

Create a channel:

```bash
npx @capawesome/cli apps:channels:create
```

Set a channel in the app:

```typescript
await LiveUpdate.setChannel({ channel: "beta" });
```

Or pass it to `sync()`:

```typescript
await LiveUpdate.sync({ channel: "production" });
```

Read `references/cli-commands.md` for all channel management commands.

### Versioned Channels

Restrict bundles to specific native versions by tying channel names to version codes.

**Android** (`android/app/build.gradle`):

```groovy
android {
    defaultConfig {
        resValue "string", "capawesome_live_update_default_channel", "production-" + defaultConfig.versionCode
    }
}
```

**iOS** (`ios/App/App/Info.plist`):

```xml
<key>CapawesomeLiveUpdateDefaultChannel</key>
<string>production-$(CURRENT_PROJECT_VERSION)</string>
```

### Bundle Versioning

Restrict bundles to native version ranges when uploading:

```bash
npx @capawesome/cli apps:liveupdates:upload --android-min 1 --android-max 5 --ios-min 1.0.0 --ios-max 1.0.5
```

### Rollbacks

Manual rollback via CLI:

```bash
npx @capawesome/cli apps:liveupdates:rollback --channel production --steps 1
```

Enable auto-blocking of rolled-back bundles:

```typescript
LiveUpdate: {
  autoBlockRolledBackBundles: true,
  readyTimeout: 10000,
}
```

### Gradual Rollouts

Upload a bundle with a rollout percentage:

```bash
npx @capawesome/cli apps:liveupdates:upload --rollout-percentage 10
```

Update the rollout:

```bash
npx @capawesome/cli apps:liveupdates:rollout --channel production --percentage 50
```

### Code Signing

Generate a signing key pair:

```bash
npx @capawesome/cli apps:liveupdates:generatesigningkey
```

Upload a signed bundle:

```bash
npx @capawesome/cli apps:liveupdates:upload --private-key private.pem
```

Configure the plugin with the public key:

```typescript
LiveUpdate: {
  publicKey: "-----BEGIN PUBLIC KEY-----\n...\n-----END PUBLIC KEY-----",
}
```

### Self-Hosting

Register a self-hosted bundle URL:

```bash
npx @capawesome/cli apps:liveupdates:register --url https://example.com/bundle.zip
```

### CI/CD Integration

Read `references/ci-cd-integrations.md` for GitHub Actions, GitLab CI, Azure DevOps, and Bitbucket Pipelines examples.

### Delta Updates

Use manifest artifact type to download only changed files:

```bash
npx @capawesome/cli apps:liveupdates:upload --artifact-type manifest
```

Generate a manifest first if needed:

```bash
npx @capawesome/cli apps:liveupdates:generatemanifest --path dist
```

## Error Handling

* If `npx cap sync` fails, verify the plugin version matches the Capacitor version in `package.json`.
* If bundles are not applied, check that `LiveUpdate.ready()` is called before `readyTimeout` expires.
* If the app reverts to the default bundle after every restart, `ready()` is likely not being called — add it as early as possible in app initialization.
* If uploads fail with authentication errors, re-run `npx @capawesome/cli login`.
* If updates are not detected on resume with `autoUpdateStrategy: "background"`, note that updates are only checked if the last check was more than 15 minutes ago. Force-close and restart the app to trigger an immediate check.
* Read `references/plugin-api.md` for the full SDK API reference.
* Read `references/faq.md` for answers to common questions about compliance, billing, and limitations.

## Debugging

The Live Update SDK logs useful information to Android Logcat and iOS Xcode console. View server-side logs in the [Capawesome Cloud Console](https://console.cloud.capawesome.io) under the "Logs" section of the app.

## Limitations

* Live updates only support **binary-compatible changes** (HTML, CSS, JS, images). Native code changes (Java, Swift, CocoaPods, Gradle) require a full app store submission.
* Maximum bundle size on Capawesome Cloud is **1 GB**. Use `manifest` artifact type for larger bundles.
* Live updates are compliant with both Apple App Store and Google Play policies.
