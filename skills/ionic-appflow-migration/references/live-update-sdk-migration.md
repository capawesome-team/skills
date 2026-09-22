# Live Update SDK Migration

Replace the Ionic Appflow Live Updates SDK with the Capawesome Live Update SDK in the app's source code.

This guide covers the Capacitor plugin (`@capawesome/capacitor-live-update`). For **pure Cordova apps**, install `@capawesome/cordova-live-update` instead and set the options as `<preference>` entries in `config.xml` (see the [Live Updates setup](https://capawesome.io/docs/cloud/live-updates/setup/)) — the API and the option names are the same. For **Capacitor apps that still ship the legacy Cordova SDK** (`cordova-plugin-ionic`), read `cordova-sdk-migration.md` for the full `Deploy` → `LiveUpdate` method mapping and the native configuration cleanup.

## 1. Update Import Statements and API Calls

Search all TypeScript/JavaScript files for imports of `@capacitor/live-updates` (or `cordova-plugin-ionic`) and replace them. The Ionic SDK uses two import styles — replace both:

```diff
-import * as LiveUpdates from '@capacitor/live-updates';
+import { LiveUpdate } from '@capawesome/capacitor-live-update';
```

```diff
-import { LiveUpdates } from '@capacitor/live-updates';
+import { LiveUpdate } from '@capawesome/capacitor-live-update';
```

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

## 2. Migrate the Update Strategy

Apply the section that matches the app's previous `autoUpdateMethod`.

### `autoUpdateMethod: 'background'`

No extra code required. Set `autoUpdateStrategy: 'background'` in the Capacitor config.

### `autoUpdateMethod: 'always'`

Set `autoUpdateStrategy: 'background'` and add a `nextBundleSet` listener to prompt the user when an update is ready. Add this code early in the app's initialization:

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

### `autoUpdateMethod: 'none'`

Keep `autoUpdateStrategy` unset and add manual sync logic:

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

### Force Update Pattern (If Applicable)

If the project extends the splash screen until the update completes, migrate it as follows:

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

## 3. Add Rollback Protection (Recommended)

Add `readyTimeout` and `autoBlockRolledBackBundles` to the `LiveUpdate` config:

```typescript
LiveUpdate: {
  appId: '<CAPAWESOME_APP_ID>',
  autoUpdateStrategy: 'background',
  readyTimeout: 10000,
  autoBlockRolledBackBundles: true,
}
```

Call `ready()` as early as possible in app startup, otherwise the app rolls back to the previous bundle after `readyTimeout`:

```typescript
import { LiveUpdate } from '@capawesome/capacitor-live-update';

void LiveUpdate.ready();
```

See [Rollbacks](https://capawesome.io/docs/cloud/live-updates/rollbacks/) for details.

## 4. Make Updates Version-Compatible (Do This Before Shipping to Production)

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

Then upload each bundle to the matching `production-<versionCode>` channel. Alternatively, keep Appflow-style per-bundle native version constraints using the `--android-min`/`--android-max`/`--ios-min`/`--ios-max` flags on `apps:liveupdates:upload`, or set them after upload with `apps:liveupdates:setnativeversions` (see `ci-cd-migration.md`).

## 5. Configure the iOS Privacy Manifest

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

## 6. Sync the Capacitor Project

```bash
npx cap sync
```
