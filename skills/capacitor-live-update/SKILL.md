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

### Step 7: Sync the Capacitor Project

```bash
npx cap sync
```

### Step 8: Build and Upload the First Bundle

Build the web assets:

```bash
npm run build
```

Upload to Capawesome Cloud:

```bash
npx @capawesome/cli apps:liveupdates:upload
```

The CLI prompts for the web assets path (e.g., `dist` or `www`), the app, and the channel. After upload, the bundle is instantly available.

### Step 9: Verify the Setup

1. Make a visible change to the web assets (e.g., change text or a color).
2. Rebuild: `npm run build`
3. Upload again: `npx @capawesome/cli apps:liveupdates:upload`
4. **For `autoUpdateStrategy: "background"`**: Force-close and restart the app. Wait 15-30 seconds, then restart again to see changes.
5. **For manual sync (Capacitor 6)**: Resume the app, accept the update prompt, and verify changes.

### Step 10: Configure iOS Privacy Manifest

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
