# Live Updates (Cordova)

Set up OTA updates for **Cordova** apps using Capawesome Cloud via the `@capawesome/cordova-live-update` plugin.

For **Capacitor** apps, read `live-updates-capacitor.md` instead.

The Cordova plugin has full API parity with the Capacitor plugin. The differences are how it is installed (a Cordova plugin), how it is configured (`config.xml` preferences instead of `capacitor.config.ts`), and how it is accessed at runtime (`cordova.plugins.LiveUpdate` after `deviceready`).

## Prerequisites

- **cordova-android `>=13.0.0`** and/or **cordova-ios `>=7.0.0`**.
- The app must use the **default custom WebView scheme**. The plugin does **not** work if `config.xml` sets `<preference name="Scheme" value="file" />` or `<preference name="AndroidInsecureFileModeEnabled" value="true" />`. Verify these are not set before continuing.

## Contents

- Install the Live Update Plugin
- Configure the Plugin
- Add Rollback Protection
- Add Update Logic (Always Latest)
- Configure iOS Privacy Manifest
- Configure Version Handling
- Apply the Configuration
- Test the Setup
- Advanced Topics

## Install the Live Update Plugin

```bash
cordova plugin add @capawesome/cordova-live-update --variable APP_ID=<APP_ID_FROM_STEP_2>
```

Replace `<APP_ID_FROM_STEP_2>` with the Capawesome Cloud app ID (UUID).

## Configure the Plugin

Configure the plugin through `<preference>` elements in `config.xml`. Add the recommended settings:

```xml
<!-- config.xml -->
<preference name="APP_ID" value="<APP_ID>" />
<preference name="AUTO_UPDATE_STRATEGY" value="background" />
<preference name="AUTO_BLOCK_ROLLED_BACK_BUNDLES" value="true" />
<preference name="READY_TIMEOUT" value="10000" />
```

Read `live-update-configuration.md` for all available configuration options and the full preference-to-option mapping.

> **Important:** Cordova only reads these preferences when a platform is added. Changes do not take effect until the affected platforms are removed and re-added (see "Apply the Configuration" below).

## Add Rollback Protection (Recommended)

The `READY_TIMEOUT` and `AUTO_BLOCK_ROLLED_BACK_BUNDLES` preferences above enable rollback protection. Call `ready()` as early as possible — inside a `deviceready` listener — to signal that the new bundle started successfully:

```javascript
document.addEventListener("deviceready", async () => {
  await cordova.plugins.LiveUpdate.ready();
});
```

If `ready()` is not called within `READY_TIMEOUT` ms, the plugin automatically rolls back. Setting `READY_TIMEOUT` to `0` disables rollback protection.

## Add Always Latest Update Logic (Recommended)

With `AUTO_UPDATE_STRATEGY=background`, the plugin downloads updates automatically. Add a listener to prompt the user when a new update is ready. **Important:** Always show a confirmation dialog before reloading — never call `reload()` without user consent.

```javascript
document.addEventListener("deviceready", async () => {
  await cordova.plugins.LiveUpdate.ready();

  cordova.plugins.LiveUpdate.addListener("nextBundleSet", async (event) => {
    if (event.bundleId) {
      const shouldReload = confirm("A new update is available. Install now?");
      if (shouldReload) {
        await cordova.plugins.LiveUpdate.reload();
      }
    }
  });
});
```

Copy this snippet exactly. Do not simplify or omit the `confirm()` dialog.

Read `update-strategies.md` for alternative strategies (Force Update, Manual Sync).

## Configure iOS Privacy Manifest

The plugin accesses `UserDefaults` and must declare it in the iOS privacy manifest. With **cordova-ios `>=7.1.0`**, configure this through a `<privacy-manifest>` element inside the iOS platform in `config.xml`:

```xml
<!-- config.xml -->
<platform name="ios">
  <privacy-manifest>
    <key>NSPrivacyAccessedAPITypes</key>
    <array>
      <dict>
        <key>NSPrivacyAccessedAPIType</key>
        <string>NSPrivacyAccessedAPICategoryUserDefaults</string>
        <key>NSPrivacyAccessedAPITypeReasons</key>
        <array>
          <string>CA92.1</string>
        </array>
      </dict>
    </array>
  </privacy-manifest>
</platform>
```

## Configure Version Handling

Live updates only deliver web code (HTML, CSS, JS, images). If the app's native code changes (e.g., a new Cordova plugin is added or a native dependency is updated), the live update bundle must match the native binary it runs on — otherwise the app may crash or behave unexpectedly. Version handling ensures that each live update bundle is only delivered to devices running a compatible native version.

Ask the user which approach to use:

1. **Versioned Channels (recommended):** Ties each channel to a native version code. Set at build time in native projects.
2. **Versioned Bundles:** Sets version constraints on each upload. No native changes needed.
3. **Skip for now:** Skip versioning for initial testing.

### Option 1: Versioned Channels

**Android** — create a `build-extras.gradle` file next to `config.xml`:

```groovy
// build-extras.gradle
android {
    applicationVariants.all { variant ->
        variant.resValue "string", "capawesome_live_update_default_channel",
                         "production-" + variant.versionCode
    }
}
```

Include it via `config.xml`:

```xml
<!-- config.xml -->
<platform name="android">
  <resource-file src="build-extras.gradle" target="app/build-extras.gradle" />
</platform>
```

**iOS** — add the channel key to `Info.plist` via `config.xml`:

```xml
<!-- config.xml -->
<platform name="ios">
  <config-file target="*-Info.plist" parent="CapawesomeLiveUpdateDefaultChannel">
    <string>production-$(CURRENT_PROJECT_VERSION)</string>
  </config-file>
</platform>
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

Set the `DEFAULT_CHANNEL` preference in `config.xml`:

```xml
<preference name="DEFAULT_CHANNEL" value="production" />
```

Ensure a channel with that name exists before uploading.

### Option 3: Skip

Set the `DEFAULT_CHANNEL` preference in `config.xml`:

```xml
<preference name="DEFAULT_CHANNEL" value="default" />
```

## Apply the Configuration

Cordova reads plugin preferences only when a platform is added. After changing any `<preference>`, `<privacy-manifest>`, or version-handling config, remove and re-add the affected platforms:

```bash
cordova platform rm android ios
cordova platform add android ios
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

Build the web assets and upload them. The web assets directory for a Cordova app is typically `www`:

```bash
cordova prepare
npx @capawesome/cli apps:liveupdates:upload --path www
```

### Revert the Visible Change

Revert the change made above so the app source is back to its original state.

### Rebuild the Native Project

```bash
cordova build android
cordova build ios
```

### Verify on Device

Tell the user to perform the following steps:

1. Run the app on a real device or emulator (`cordova run android` / `cordova run ios`, or from Android Studio / Xcode).
2. Wait for the update prompt to appear and accept it. The visible change from the uploaded bundle should appear after reload. If no prompt appears, force-close and reopen the app to trigger a check.
3. If the change does not appear, check Android Logcat or the iOS Xcode console for Live Update SDK log output and refer to `live-update-advanced-topics.md` (Debugging section).

After testing, tell the user: Once a live update bundle has been applied, the app points to that bundle instead of the default one. To use the original assets again, completely uninstall and reinstall the app. This is only relevant during development — in production, the default bundle is automatically restored on each native app update.

## Advanced Topics

Read `live-update-advanced-topics.md` for channels, versioned channels, rollbacks, code signing, gradual rollouts, self-hosting, delta updates, debugging, and limitations.
