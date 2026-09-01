# Intune

Integrate the Microsoft Intune App SDK for Mobile Application Management (MAM): enroll accounts, read app protection policies and app configuration, and react to selective wipes.

**Package:** `@capawesome/capacitor-intune`
**Platforms:** Android, iOS
**Documentation:** https://capawesome.io/docs/sdks/capacitor/intune/

## Installation

```bash
npm install @capawesome/capacitor-intune
npx cap sync
```

**Requires third-party setup:** a Microsoft Intune tenant with Intune licenses and an app registration in [Microsoft Entra ID](https://entra.microsoft.com/). Note the **Application (client) ID** and register the platform-specific redirect URIs (see below). The scopes passed to `acquireToken(...)` must be exposed or granted on that app registration. The Intune App SDKs are proprietary Microsoft software; the plugin does not bundle them but downloads them from Microsoft's repositories at build/install time.

## Configuration

### Android

#### Variables

Optionally define in `android/variables.gradle`:

- `$intuneMamSdkVersion` version of the Microsoft Intune App SDK for Android (default: `12.4.0`)
- `$msalVersion` version of `com.microsoft.identity.client:msal` (default: `8.4.0`)

When overriding `$intuneMamSdkVersion`, update the `com.microsoft.intune.mam.build` classpath in `android/build.gradle` to the same version — the build plugin and the SDK must match.

#### MAM Build Plugin

**Required.** The SDK rewrites the Android base classes of the app and all Capacitor plugins. Only the app module can apply it. Add to the `buildscript` block of `android/build.gradle`:

```groovy
repositories {
    ivy {
        url 'https://raw.githubusercontent.com/microsoftconnect/ms-intune-app-sdk-android'
        patternLayout { artifact '[revision]/GradlePlugin/[artifact].[ext]' }
        metadataSources { artifact() }
        content { includeGroup 'com.microsoft.intune.mam.build' }
    }
}
dependencies {
    classpath 'org.javassist:javassist:3.29.2-GA'
    classpath 'com.microsoft.intune.mam.build:com.microsoft.intune.mam.build:12.4.0@jar'
}
```

Then in `android/app/build.gradle`: `apply plugin: 'com.microsoft.intune.mam'`.

#### Application Class

Set the ready-made class in the `application` tag of `android/app/src/main/AndroidManifest.xml`:

```xml
<application
    android:name="io.capawesome.capacitorjs.plugins.intune.IntuneApplication"
    android:enableOnBackInvokedCallback="false"
    ...>
```

If the app already has a custom `Application` class, call `Intune.initialize(this)` in `onCreate()` instead. Disabling predictive back gestures is recommended — the SDK does not support them yet.

#### MSAL Configuration

Create `android/app/src/main/res/raw/auth_config.json` with `client_id`, `authorization_user_agent: "DEFAULT"`, `redirect_uri: "msauth://YOUR_PACKAGE_NAME/YOUR_BASE64_URL_ENCODED_PACKAGE_SIGNATURE"`, `account_mode: "MULTIPLE"`, `broker_redirect_uri_registered: true` and an `AAD` authority. Generate the signature hash with `keytool -exportcert -alias YOUR_KEY_ALIAS -keystore YOUR_KEYSTORE | openssl sha1 -binary | openssl base64`.

Register the same redirect URI as an **Android** platform redirect URI in Microsoft Entra, and add the MSAL activity to the `application` tag:

```xml
<activity android:name="com.microsoft.identity.client.BrowserTabActivity" android:exported="true">
  <intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="msauth" android:host="YOUR_PACKAGE_NAME" android:path="/YOUR_BASE64_ENCODED_PACKAGE_SIGNATURE" />
  </intent-filter>
</activity>
```

#### Gradle Properties

Add `android.enableResourceOptimizations=false` to `android/gradle.properties` — without it, release builds may report `MAM Enabled: No` in the diagnostic console.

#### Company Portal

App protection policies are only applied when the Company Portal app is installed on the device (the user does not need to sign in). Without it, the app behaves as unmanaged.

### iOS

**Requires iOS 17+** as deployment target: replace every `IPHONEOS_DEPLOYMENT_TARGET` entry in `ios/App/App.xcodeproj/project.pbxproj` with `IPHONEOS_DEPLOYMENT_TARGET = 17.0;` and, for SPM, raise the platform in `ios/App/CapApp-SPM/Package.swift`. Both CocoaPods and Swift Package Manager are supported.

#### Info.plist

Add to `ios/App/App/Info.plist` (the keys keep their legacy `ADAL` names). The plugin also configures MSAL from these values, so no separate MSAL config is needed:

```xml
<key>IntuneMAMSettings</key>
<dict>
    <key>ADALAuthority</key>
    <string>https://login.microsoftonline.com/YOUR_TENANT_ID</string>
    <key>ADALClientId</key>
    <string>YOUR_CLIENT_ID</string>
    <key>ADALRedirectUri</key>
    <string>msauth.YOUR_BUNDLE_ID://auth</string>
</dict>
```

Also add to the same file:

- `CFBundleURLTypes` with the URL name `MSAL` and the scheme `msauth.YOUR_BUNDLE_ID`
- `LSApplicationQueriesSchemes` with `msauthv2`, `msauthv3`, `http-intunemam` and `https-intunemam`
- `NSFaceIDUsageDescription`, since app protection policies may require biometric unlock

Register `msauth.YOUR_BUNDLE_ID://auth` as an **iOS/macOS** platform redirect URI in Microsoft Entra. No `AppDelegate` changes are required.

#### Keychain Sharing

Enable the **Keychain Sharing** capability and add these groups in this order: your bundle ID, `com.microsoft.intune.mam`, `com.microsoft.adalcache`.

## Usage

### Sign in and enroll an account

```typescript
import { Intune } from '@capawesome/capacitor-intune';

await Intune.addListener('enrollmentChange', (event) => {
  console.log('Enrollment status:', event.status);
});
const { accountId } = await Intune.acquireToken({
  scopes: ['https://graph.microsoft.com/.default'],
});
await Intune.registerAndEnrollAccount({ accountId });
```

### Read the app protection policy and app configuration

```typescript
import { Intune } from '@capawesome/capacitor-intune';

const { account } = await Intune.getEnrolledAccount();
if (account) {
  const policy = await Intune.getPolicy({ accountId: account.accountId });
  // pinRequired, screenCaptureAllowed, saveToPersonalStorageAllowed, ...
  const { values } = await Intune.getAppConfig({ accountId: account.accountId });
  console.log('Server URL:', values['com.example.serverUrl']);
}
```

### Handle selective wipe

```typescript
import { Intune } from '@capawesome/capacitor-intune';

await Intune.addListener('wipeRequested', async () => {
  localStorage.clear();
  sessionStorage.clear();
  const databases = await indexedDB.databases();
  for (const database of databases) {
    if (database.name) indexedDB.deleteDatabase(database.name);
  }
});
```

## Notes

- Most host app changes can be automated with [Trapeze](https://trapeze.dev/); the README ships a ready-made `trapeze.yaml`. Trapeze cannot edit `gradle.properties`, and the Gradle insertions are not idempotent — run them only once.
- Policy **enforcement** (PIN, encryption, copy/paste, screenshots) happens automatically inside the SDK once the native integration is in place. The JS API only covers enrollment, introspection, and web storage cleanup.
- `loginAndEnrollAccount()` and the `wipe` option of `unenrollAccount(...)` are iOS-only. On Android, use `acquireToken(...)` + `registerAndEnrollAccount(...)` instead.
- The `wipeRequested` event is persisted and replayed on the next launch if no listener was registered, and may be delivered more than once — keep the handler idempotent.
- Microsoft blocks apps that ship outdated Intune App SDK versions, which is why the plugin tracks the current SDK line and requires iOS 17.
- This plugin covers the **MAM channel** (no device enrollment). For the MDM channel use the [Managed Configurations](https://capawesome.io/docs/sdks/capacitor/managed-configurations/) plugin.
- Microsoft's `IntuneMAMConfigurator` tool (shipped with the iOS SDK repo) applies the minimum required `Info.plist`/entitlements changes and is idempotent — recommended before shipping.
- No web implementation; all methods reject with `unimplemented` on the web. This project is not affiliated with or endorsed by Microsoft Corporation.
