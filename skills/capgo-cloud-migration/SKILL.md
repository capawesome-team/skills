---
name: capgo-cloud-migration
description: "Guides the agent through migrating an existing Capacitor project from Capgo to Capawesome Cloud. Detects Capgo usage (the @capgo/capacitor-updater plugin, CapacitorUpdater configuration, and @capgo/cli commands in CI/CD) and provides step-by-step migration: replacing the plugin with @capawesome/capacitor-live-update, mapping configuration options and API methods, migrating update strategies, channels, code signing, and CI/CD pipelines. References the capawesome-cloud skill for detailed Capawesome Cloud setup procedures. Do not use for setting up Capawesome Cloud from scratch without an existing Capgo project, for migrating from Ionic Appflow (use the ionic-appflow-migration skill instead), or for non-Capacitor mobile frameworks."
metadata:
  author: capawesome-team
  source: https://github.com/capawesome-team/skills/tree/main/skills/capgo-cloud-migration
---

# Capgo Cloud Migration

Migrate an existing Capacitor project from Capgo to Capawesome Cloud.

## Prerequisites

1. A **Capacitor 6, 7, or 8** app currently using Capgo for live updates.
2. Node.js 18+ and npm installed.
3. Access to the project's source code repository.
4. A [Capawesome Cloud](https://console.cloud.capawesome.io) account and organization.

## General Rules

- Before running any `@capawesome/cli` command for the first time, run it with the `--help` flag to review all available options.
- Do not remove Capgo configuration, packages, or signing keys until the corresponding Capawesome Cloud feature is fully set up and verified.
- Determine the Capacitor version from `package.json` (`@capacitor/core`) before making any changes — it affects which plugin version and update strategy to use.
- The Capgo app ID is a native bundle ID (e.g. `com.example.app`), while the Capawesome Cloud app ID is a UUID. Never copy the Capgo app ID into the Capawesome configuration.

## Procedures

### Step 1: Detect Capgo Usage

Scan the project to determine which Capgo features are in use.

#### 1.1 Check for the Capgo Updater Plugin

Search for these signals:

1. `@capgo/capacitor-updater` in `package.json` (dependencies or devDependencies).
2. A `CapacitorUpdater` key inside the `plugins` object in `capacitor.config.ts` or `capacitor.config.json`.
3. Imports of `@capgo/capacitor-updater` in TypeScript/JavaScript source files.

If any signal is found, mark **Live Updates** as in use.

Record the current configuration values from the `CapacitorUpdater` config (all optional):

- `appId` (Capgo app ID, a bundle ID like `com.example.app`)
- `autoUpdate` (`true`, `false`, `'off'`, `'atBackground'`, `'atInstall'`, `'onLaunch'`, `'always'`, or `'onlyDownload'`; default `true`)
- `directUpdate` (`true`, `'atInstall'`, `'onLaunch'`, or `'always'`; default `false`)
- `defaultChannel`
- `appReadyTimeout` (milliseconds; default `10000`)
- `autoDeletePrevious` (default `true`)
- `publicKey` (indicates end-to-end encryption is in use)
- `periodCheckDelay` (seconds)
- `responseTimeout` (seconds; default `20`)
- `updateUrl` / `statsUrl` / `channelUrl` (indicates a self-hosted Capgo backend)
- `autoSplashscreen` (default `false`)

#### 1.2 Check for the Capgo CLI in CI/CD

Search for these signals:

1. `@capgo/cli` in `package.json` (usually devDependencies).
2. `capgo` or `@capgo/cli` commands (e.g. `bundle upload`, `channel set`) in CI/CD configuration files (e.g., `.github/workflows/*.yml`, `.gitlab-ci.yml`, `bitrise.yml`, `Jenkinsfile`, `azure-pipelines.yml`) and npm scripts.
3. A `CAPGO_API_KEY` (or similar) secret referenced in CI/CD files.

If any signal is found, mark **CI/CD** as in use and record every Capgo CLI command with its options.

#### 1.3 Check for Code Signing / Encryption

Search for these signals:

1. `.capgo_key_v2` or `.capgo_key_v2.pub` files in the project root (created by `npx @capgo/cli key create`).
2. A `publicKey` value in the `CapacitorUpdater` config.
3. `--key-v2` or `--key-data-v2` flags on `bundle upload` commands in CI/CD.

If any signal is found, mark **Code Signing** as in use.

#### 1.4 Check for Capgo Native Builds

Search for `capgo build` commands (e.g. `build request`, `build credentials`) in CI/CD files or npm scripts. If found, mark **Native Builds** as in use.

#### 1.5 Record Channels

List the existing Capgo channels so they can be recreated in Capawesome Cloud:

```bash
npx @capgo/cli channel list <CAPGO_APP_ID>
```

If the command fails (e.g. no API key available), ask the user to list their channels from the Capgo Console instead.

#### 1.6 Present Findings

Present the detected features (Live Updates, CI/CD, Code Signing, Native Builds) and recorded configuration to the user. Ask the user to confirm which features to migrate.

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

Save the returned **app ID** (UUID) for subsequent steps. This replaces the Capgo app ID everywhere, but has a different format — it is **not** the native bundle ID.

#### 2.3 Recreate Channels

Create one Capawesome Cloud channel for each Capgo channel recorded in Step 1.5:

```bash
npx @capawesome/cli apps:channels:create --app-id <APP_ID> --name <CHANNEL_NAME>
```

Capgo channel settings like `--self-assign`, `--disable-auto-update`, or device targeting (`--ios`/`--android`) have no channel-level equivalent in Capawesome Cloud:

- Device self-assignment is not required — the Live Update SDK can request any channel via `sync({ channel })` or `setChannel()` without a server-side opt-in.
- Update blocking by native version is configured per bundle at upload time (`--android-min`, `--ios-min`, and related flags).

### Step 3: Replace the Plugin

Skip this step if Live Updates was not detected or the user chose not to migrate it.

#### 3.1 Remove the Capgo Updater Plugin

```bash
npm uninstall @capgo/capacitor-updater
```

#### 3.2 Install the Capawesome Live Update Plugin

Install the version matching the project's Capacitor version:

- **Capacitor 8**: `npm install @capawesome/capacitor-live-update@latest`
- **Capacitor 7**: `npm install @capawesome/capacitor-live-update@v7-lts`
- **Capacitor 6**: `npm install @capawesome/capacitor-live-update@v6-lts`

### Step 4: Update the Capacitor Configuration

Replace the `CapacitorUpdater` plugin config with `LiveUpdate` in `capacitor.config.ts` (or `.json`). Map the configuration options as follows:

| Capgo (`CapacitorUpdater`) | Capawesome (`LiveUpdate`) | Notes |
|---|---|---|
| `appId` | `appId` | **Different value** — use the Capawesome Cloud app ID (UUID) from Step 2 |
| `appReadyTimeout` | `readyTimeout` | Both in milliseconds. Capgo defaults to `10000`, Capawesome to `0` (disabled) — set `readyTimeout: 10000` explicitly to keep rollback protection |
| `autoDeletePrevious` | `autoDeleteBundles` | Capgo defaults to `true`, Capawesome to `false` — set `autoDeleteBundles: true` explicitly |
| `autoDeleteFailed` | *(remove)* | Use `autoBlockRolledBackBundles: true` to prevent re-installing bundles that caused a rollback |
| `autoUpdate` | `autoUpdateStrategy` or code | See Step 6 for the strategy mapping |
| `autoSplashscreen` (+ `autoSplashscreenLoader`, `autoSplashscreenTimeout`) | *(remove)* | Handled in code — see the Force Update pattern in Step 6 |
| `defaultChannel` | `defaultChannel` | Same value |
| `directUpdate` | *(remove)* | Handled in code — see Step 6 |
| `periodCheckDelay` | *(remove)* | Handled in code — sync on the App `resume` event instead (Step 6) |
| `publicKey` | `publicKey` | **New value required** — generate a new key pair in Step 7. Do not reuse the Capgo key |
| `resetWhenUpdate` | *(remove)* | Handled automatically by the SDK |
| `responseTimeout` | `httpTimeout` | **Unit change**: Capgo uses seconds (default `20`), Capawesome uses milliseconds (default `60000`) — multiply by 1000 |
| `statsUrl` | *(remove)* | Not needed |
| `updateUrl` / `channelUrl` | `serverDomain` | Only set for EU hosting (`api.cloud.capawesome.eu`). Domain only, without scheme or path. Default is `api.cloud.capawesome.io` |
| `version` | *(remove)* | The native version code is used by the SDK |

All other Capgo options (`allowModifyUrl`, `allowModifyAppId`, `allowManualBundleError`, `allowPreview`, `persistCustomId`, `persistModifyUrl`, `allowSetDefaultChannel`, `keepUrlPathAfterReload`, `disableJSLogging`, `osLogging`, `shakeMenu`, `shakeMenuGesture`, `allowShakeChannelSelector`, `localS3`, `localHost`, `localWebHost`, `localSupa`, `localSupaAnon`, `localApi`, `localApiFiles`) have no equivalent — remove them.

**Example:**

```diff
 // capacitor.config.ts
 const config: CapacitorConfig = {
   plugins: {
-    CapacitorUpdater: {
-      appId: 'com.example.app',
-      autoUpdate: true,
-      defaultChannel: 'production',
-      appReadyTimeout: 10000,
-      autoDeletePrevious: true,
-      responseTimeout: 20
-    }
+    LiveUpdate: {
+      appId: '<CAPAWESOME_APP_ID>',
+      autoUpdateStrategy: 'background',
+      defaultChannel: 'production',
+      readyTimeout: 10000,
+      autoDeleteBundles: true,
+      autoBlockRolledBackBundles: true,
+      httpTimeout: 20000
+    }
   }
 };
```

### Step 5: Update Imports and API Calls

Search all TypeScript/JavaScript files for imports of `@capgo/capacitor-updater` and replace:

```diff
-import { CapacitorUpdater } from '@capgo/capacitor-updater';
+import { LiveUpdate } from '@capawesome/capacitor-live-update';
```

Replace **all** references to the `CapacitorUpdater` class with `LiveUpdate`, applying the method mapping below. Read `references/api-mapping.md` for the complete method, event, and result-field mapping. The most common replacements:

**App ready notification** — `notifyAppReady()` becomes `ready()`. Keep it as early as possible in app startup:

```diff
-import { CapacitorUpdater } from '@capgo/capacitor-updater';
+import { LiveUpdate } from '@capawesome/capacitor-live-update';

-CapacitorUpdater.notifyAppReady();
+void LiveUpdate.ready();
```

**Check for updates** — `getLatest()` becomes `fetchLatestBundle()`. The result fields change (`version` → `bundleId`, `url` → `downloadUrl`):

```diff
-const latest = await CapacitorUpdater.getLatest();
-if (latest.url) {
-  console.log('Update available:', latest.version);
+const result = await LiveUpdate.fetchLatestBundle();
+if (result.downloadUrl) {
+  console.log('Update available:', result.bundleId);
 }
```

**Download a bundle** — `download()` becomes `downloadBundle()`:

```diff
-const bundle = await CapacitorUpdater.download({
-  url: latest.url,
-  version: latest.version
-});
+await LiveUpdate.downloadBundle({
+  url: result.downloadUrl,
+  bundleId: result.bundleId
+});
```

**Set the next bundle** — `next()` becomes `setNextBundle()`. Capgo's `set()` (apply immediately) becomes `setNextBundle()` followed by `reload()`:

```diff
-await CapacitorUpdater.next({ id: bundle.id });
+await LiveUpdate.setNextBundle({ bundleId: result.bundleId });
```

```diff
-await CapacitorUpdater.set({ id: bundle.id });
+await LiveUpdate.setNextBundle({ bundleId: result.bundleId });
+await LiveUpdate.reload();
```

**Channel switching** — prefer passing the channel directly to `sync(...)` or `fetchLatestBundle(...)` instead of persisting it with `setChannel()`:

```diff
-await CapacitorUpdater.setChannel({ channel: 'beta', triggerAutoUpdate: true });
+await LiveUpdate.sync({ channel: 'beta' });
```

Use `setChannel({ channel })` only if the channel assignment must persist across app restarts, and `setChannel({ channel: null })` as the replacement for Capgo's `unsetChannel()`.

### Step 6: Migrate the Update Strategy

Map the recorded `autoUpdate` and `directUpdate` values to a Capawesome update strategy.

#### 6.1 Background Updates (Capgo default)

For `autoUpdate: true`, `'atBackground'`, `'atInstall'`, or `'onLaunch'` **without** `directUpdate`, use the background strategy: set `autoUpdateStrategy: 'background'` in the config (already done in Step 4) — no sync code is needed. Downloaded updates are applied on the next app start.

#### 6.2 Always Latest (Capgo `autoUpdate: 'always'`)

Combine the background strategy with a `nextBundleSet` listener that prompts the user to apply the update immediately. Add this code early in the app's initialization:

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

#### 6.3 Force Update (Capgo `directUpdate` / `autoSplashscreen`)

If Capgo was configured with `directUpdate` (any non-`false` value), replicate it by extending the splash screen until the sync completes. Omit `autoUpdateStrategy` from the config and add:

```typescript
import { SplashScreen } from '@capacitor/splash-screen';
import { LiveUpdate } from '@capawesome/capacitor-live-update';

const initializeApp = async () => {
  const { nextBundleId } = await LiveUpdate.sync();
  if (nextBundleId) {
    await LiveUpdate.reload();
  } else {
    await SplashScreen.hide();
  }
};

initializeApp();
```

This replaces both `directUpdate` and `autoSplashscreen` — no separate splash screen config is needed. This pattern may impact user experience on slow connections; consider the background strategy instead.

#### 6.4 Manual / Download-Only (Capgo `autoUpdate: 'off'` or `'onlyDownload'`)

Keep `autoUpdateStrategy` unset and migrate the existing manual `getLatest()`/`download()`/`next()` flow using the method mapping from Step 5, or replace the whole flow with a single `sync()` call. A common pattern is to sync on the App `resume` event:

```typescript
import { App } from '@capacitor/app';
import { LiveUpdate } from '@capawesome/capacitor-live-update';

void LiveUpdate.ready();

App.addListener('resume', async () => {
  await LiveUpdate.sync();
});
```

With this pattern, downloaded updates are applied on the next app start.

#### 6.5 Periodic Checks (Capgo `periodCheckDelay`)

There is no interval-based check option. The background strategy already checks for updates on app start and resume (throttled to once every 15 minutes). For a custom interval, sync on the App `resume` event (see 6.4) or implement a JavaScript timer that calls `sync()`.

#### 6.6 Sync the Capacitor Project

```bash
npx cap sync
```

### Step 7: Migrate Code Signing

Skip this step if Code Signing was not detected.

Capgo's key system (`.capgo_key_v2`) provides end-to-end encryption, while Capawesome Cloud uses RSA signing for bundle integrity verification. The keys are **not** interchangeable — generate a new key pair:

```bash
npx @capawesome/cli apps:liveupdates:generatesigningkey --app-type capacitor
```

This prints the `publicKey` config snippet for `capacitor.config.ts`. Store the private key securely (e.g. as a CI/CD secret) and never commit it. Sign every upload by passing `--private-key` (see Step 8).

Keep the `.capgo_key_v2` files until the migration is verified, then delete them (Step 9).

### Step 8: Update CI/CD Pipelines

Skip this step if CI/CD was not detected.

#### 8.1 Replace Authentication

Generate a token in the [Capawesome Cloud Console](https://console.cloud.capawesome.io) and store it as a CI/CD secret named `CAPAWESOME_CLOUD_TOKEN`. Then replace the Capgo API key:

```diff
-- run: npx @capgo/cli@latest login "$CAPGO_API_KEY"
+- run: npx @capawesome/cli login --token ${{ secrets.CAPAWESOME_CLOUD_TOKEN }}
```

Capgo commands often pass `--apikey` (or `-a`) inline instead of using `login` — remove those flags; the Capawesome CLI uses the token from `login --token`.

#### 8.2 Replace CLI Commands

Use the following command mapping (Capgo identifies apps by bundle ID as a positional argument; Capawesome uses `--app-id <UUID>`):

| Capgo CLI | Capawesome CLI Equivalent |
|---|---|
| `npx @capgo/cli login <API_KEY>` | `npx @capawesome/cli login --token <TOKEN>` |
| `npx @capgo/cli init <API_KEY> <BUNDLE_ID>` | No equivalent — follow this skill's steps instead |
| `npx @capgo/cli app add <BUNDLE_ID>` | `npx @capawesome/cli apps:create --name <NAME>` |
| `npx @capgo/cli app list` | `npx @capawesome/cli apps:list` |
| `npx @capgo/cli app delete <BUNDLE_ID>` | `npx @capawesome/cli apps:delete --app-id <APP_ID>` |
| `npx @capgo/cli bundle upload <BUNDLE_ID> --path <PATH> --channel <CH>` | `npx @capawesome/cli apps:liveupdates:upload --app-id <APP_ID> --path <PATH> --channel <CH>` |
| `bundle upload --delta` / `--delta-only` | `apps:liveupdates:upload --artifact-type manifest` |
| `bundle upload --key-v2 <KEY_PATH>` | `apps:liveupdates:upload --private-key <KEY_PATH>` |
| `bundle upload --rollout <PCT>` | `apps:liveupdates:upload --rollout-percentage <PCT>` |
| `bundle upload --min-update-version <V>` | `--android-min`/`--ios-min` flags (**native** version constraints, not bundle versions) |
| `bundle upload -e <URL>` (external/self-hosted) | `apps:liveupdates:register --url <URL>` |
| `bundle zip` | `apps:liveupdates:bundle` |
| `bundle list` / `bundle delete` / `bundle cleanup` | Manage bundles in the Console; set a bundle limit per channel for automatic cleanup |
| `channel add <CH> <BUNDLE_ID>` | `apps:channels:create --app-id <APP_ID> --name <CH>` |
| `channel delete <CH> <BUNDLE_ID>` | `apps:channels:delete --app-id <APP_ID> --name <CH>` |
| `channel list <BUNDLE_ID>` | `apps:channels:list --app-id <APP_ID>` |
| `channel set <CH> <BUNDLE_ID> --bundle <V>` | `apps:liveupdates:rollback --app-id <APP_ID> --channel <CH>` (previous bundle) or re-upload the desired bundle |
| `channel set <CH> <BUNDLE_ID> --rollout-percentage <PCT>` | `apps:liveupdates:rollout --app-id <APP_ID> --channel <CH> --percentage <PCT>` |
| `key create` | `apps:liveupdates:generatesigningkey` |
| `build request` (Capgo native builds) | `apps:builds:create` — use the `capawesome-cloud` skill's Native Builds section |
| `doctor` | `doctor` |

**Example upload replacement:**

```diff
-npx @capgo/cli@latest bundle upload com.example.app --apikey "$CAPGO_API_KEY" --path ./dist --channel production
+npx @capawesome/cli apps:liveupdates:upload --app-id <APP_ID> --path ./dist --channel production
```

There is no equivalent for Capgo's `--bundle <VERSION>` flag — Capawesome Cloud assigns a bundle ID automatically. Use `--git-ref` to associate a Git reference and `--custom-property key=value` for custom metadata instead.

### Step 9: Test and Verify

#### 9.1 Verify Live Updates

Ask the user whether to test live update functionality. If accepted:

1. Make a small, visible change in the app (e.g., append " - Migration Test" to a heading).
2. Build the web assets: `npm run build`.
3. Upload the bundle:
   ```bash
   npx @capawesome/cli apps:liveupdates:upload --app-id <APP_ID> --channel <CHANNEL_NAME>
   ```
4. Revert the visible change, rebuild, and sync:
   ```bash
   npm run build && npx cap sync
   ```
5. Open the native project (`npx cap open ios` or `npx cap open android`) and run on a device or emulator.
6. Verify the live update is applied. With `autoUpdateStrategy: "background"`, force-close and reopen the app, or switch away from the app and return after more than 15 minutes to trigger a check.

#### 9.2 Clean Up

After all features are verified:

1. Confirm `@capgo/capacitor-updater` and `@capgo/cli` are removed from `package.json`.
2. Confirm the `CapacitorUpdater` config is removed from `capacitor.config.ts`/`capacitor.config.json`.
3. Delete `.capgo_key_v2` and `.capgo_key_v2.pub` from the project root (if present).
4. Remove Capgo CI/CD secrets (e.g. `CAPGO_API_KEY`) and any remaining `@capgo/cli` commands.
5. Remind the user to cancel their Capgo subscription once all app versions using Capgo have been phased out.

## Error Handling

- `npx cap sync` fails after the plugin swap → Verify the installed `@capawesome/capacitor-live-update` version matches the Capacitor version in `package.json` (`@latest` for 8, `@v7-lts` for 7, `@v6-lts` for 6).
- `CapacitorUpdater is not defined` or similar runtime errors → Ensure all imports were updated from `@capgo/capacitor-updater` to `@capawesome/capacitor-live-update` and the class name changed from `CapacitorUpdater` to `LiveUpdate`.
- App reverts to the previous bundle after an update → `LiveUpdate.ready()` is not called early enough (Capgo's `notifyAppReady()` equivalent). Add it as the first call in app initialization, or increase `readyTimeout`.
- No update is applied although a bundle was uploaded → Verify the `appId` in the config is the Capawesome Cloud UUID (not the Capgo bundle ID) and that the upload targeted the channel the device resolves (check with `getChannel()`).
- Updates not detected with `autoUpdateStrategy: "background"` → Updates are only checked if the last check was more than 15 minutes ago. Force-close and restart the app.
- Signature verification fails after enabling code signing → The Capgo key pair was reused. Generate a new key pair with `apps:liveupdates:generatesigningkey`, update `publicKey` in the config, and re-upload with `--private-key`.
- `setConfig is not a function` → The installed plugin release is outdated. Update to the latest release of the matching dist-tag (`@latest` for Capacitor 8, `@v7-lts` for 7, `@v6-lts` for 6) — all current tag releases include `setConfig()`.
- `fetchChannels()` returns an error → Channel discovery must be enabled for the app in the Capawesome Cloud Console.
- CI/CD pipeline fails after migration → Ensure the `CAPAWESOME_CLOUD_TOKEN` secret is set, `login --token` runs before other commands, and every command passes `--app-id` with the Capawesome Cloud UUID.

## Related Skills

- **`capawesome-cloud`** — Detailed Capawesome Cloud setup procedures for Live Updates, Native Builds, and App Store Publishing.
- **`capawesome-cli`** — Installing, configuring, and using the Capawesome CLI.
- **`ionic-appflow-migration`** — Use this skill instead if the project migrates from Ionic Appflow rather than Capgo.
