# Singular

Unofficial Capacitor plugin for the Singular Mobile SDK to attribute installs, track events and revenue, and handle Singular Links for mobile measurement.

**Package:** `@capawesome/capacitor-singular`
**Platforms:** Android, iOS
**Documentation:** https://capawesome.io/docs/sdks/capacitor/singular/

## Installation

```bash
npm install @capawesome/capacitor-singular
npx cap sync
```

Requires a [Singular](https://www.singular.net/) account. The **SDK Key** and the **SDK Secret** are listed in the Singular dashboard under **Developer Tools → SDK Integration → SDK Keys**. Do not use the Singular Reporting API key, otherwise no SDK data is received.

Use of the SDKs is governed by the [Singular Terms & Conditions of Service](https://www.singular.net/terms/), which limit the use of the service to your own advertising and promotions and prohibit sending Protected Health Information (as defined under HIPAA) to Singular.

## Configuration

### Android

#### Repositories

The [Singular SDK for Android](https://support.singular.net/hc/en-us/articles/360037581952-Android-SDK-Basic-Integration) is not published on Maven Central. Add Singular's Maven repository to the `allprojects` section of `android/build.gradle`:

```groovy
allprojects {
    repositories {
        google()
        mavenCentral()
        maven { url 'https://maven.singular.net/' }
    }
}
```

If the app declares its repositories in `android/settings.gradle` instead (via `dependencyResolutionManagement`), add the repository there.

#### Variables

Defined in your app's `android/variables.gradle`:

- `$singularSdkVersion` version of `com.singular.sdk:singular_sdk` (default: `12.16.0`)

#### Permissions

The plugin already declares the `INTERNET`, `ACCESS_NETWORK_STATE` and `com.google.android.gms.permission.AD_ID` permissions in its `AndroidManifest.xml`, so no manual configuration is required.

Apps that participate in the [Google Play Families program](https://support.google.com/googleplay/android-developer/answer/9893335) must not request the advertising ID. Remove the permission by adding the following element to `android/app/src/main/AndroidManifest.xml` before or after the `application` tag:

```xml
<!-- Required for apps in the Google Play Families program. -->
<uses-permission android:name="com.google.android.gms.permission.AD_ID" tools:node="remove" />
```

Make sure the `tools` namespace is declared on the `manifest` element (`xmlns:tools="http://schemas.android.com/tools"`).

#### Proguard

If Proguard is used, add the following rules to `android/app/proguard-rules.pro`:

```
-keep class com.singular.sdk.** { *; }
-keep public class com.android.installreferrer.** { *; }
```

#### Singular Links

To open [Singular Links](https://support.singular.net/hc/en-us/articles/35356520601755-Android-SDK-Supporting-Deep-Links) in the app, add an App Links intent filter to the `MainActivity` in `android/app/src/main/AndroidManifest.xml`:

```xml
<intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="https" android:host="YOUR_SUBDOMAIN.sng.link" android:pathPrefix="/A" />
    <data android:scheme="https" android:host="YOUR_SUBDOMAIN.sng.link" android:pathPrefix="/B" />
    <data android:scheme="https" android:host="YOUR_SUBDOMAIN.sng.link" android:pathPrefix="/E" />
    <data android:scheme="https" android:host="YOUR_SUBDOMAIN.sng.link" android:pathPrefix="/F" />
</intent-filter>
```

Replace `YOUR_SUBDOMAIN` with the subdomain of the Singular Links domain. Singular hosts the required `assetlinks.json` file, but the SHA256 fingerprints of the signing keys must be entered in the Singular dashboard under **Settings → Apps** so that Android can verify the App Links.

### iOS

The [Singular SDK for iOS](https://github.com/singular-labs/Singular-iOS-SDK) is resolved via Swift Package Manager or CocoaPods, so no manual dependency setup is required. The plugin requires iOS 15 or later.

#### App Tracking Transparency

The SDK only collects the advertising identifier (IDFA) if the user granted tracking permission. Request it with `@capawesome/capacitor-app-tracking-transparency` (see `references/capawesome-app-tracking-transparency.md`) and add the `NSUserTrackingUsageDescription` key to `ios/App/App/Info.plist`:

```xml
<key>NSUserTrackingUsageDescription</key>
<string>The advertising identifier is used to measure the performance of our advertising campaigns.</string>
```

Set `iosWaitForTrackingAuthorizationTimeout` to a value greater than `0` when calling `initialize(...)` so that the SDK waits for the user's decision before sending the first session. Skip this section entirely if the app does not request tracking authorization.

#### Singular Links

To open Singular Links in the app, add the **Associated Domains** capability to the app in Xcode with one entry per Singular Links domain in the format `applinks:YOUR_SUBDOMAIN.sng.link`.

#### SKAdNetwork

[SKAdNetwork](https://support.singular.net/hc/en-us/articles/360047448611-Introduction-to-Singular-s-SKAdNetwork-Solution) support is enabled by default and the SDK manages the conversion value. Set `iosManualSkanConversionManagement` to `true` when calling `initialize(...)` only if the app manages the conversion value itself.

## Usage

### Initialize

Add the listeners before calling `initialize(...)` so that no deferred deep link or attribution event is missed:

```typescript
import { Singular } from '@capawesome/capacitor-singular';

await Singular.addListener('singularLinkResolved', (event) => {
  console.log(event.deepLink, event.isDeferred, event.passthrough, event.urlParameters);
});
await Singular.addListener('deviceAttributionInfoReceived', (event) => {
  console.log(event.network, event.campaignName, event.creativeName);
});

await Singular.initialize({
  apiKey: 'YOUR_SDK_KEY',
  secret: 'YOUR_SDK_SECRET',
  customSdid: 'YOUR_CUSTOM_SDID', // Optional. Custom Singular Device ID.
  customUserId: 'user-123', // Optional.
  globalProperties: { plan: 'premium' }, // Optional. At most 5 properties.
  brandedDomains: ['links.example.com'], // Optional.
  espDomains: ['click.mail.example.com'], // Optional. Email service provider domains.
  sessionTimeout: 60, // Seconds in the background before a new session starts. Default: 60
  shortLinkResolveTimeout: 10, // Default: 10
  limitAdvertisingIdentifiers: false, // Blocks collection of the Google Advertising ID / IDFA. Default: false
  limitDataSharing: false, // Default: false
  loggingEnabled: false, // Default: false
  androidFacebookAppId: 'YOUR_FACEBOOK_APP_ID', // Android only. Meta Install Referrer attribution.
  iosSkAdNetworkEnabled: true, // iOS only. Default: true
  iosManualSkanConversionManagement: false, // iOS only. Default: false
  iosWaitForTrackingAuthorizationTimeout: 0, // iOS only. Seconds. Default: 0
});
```

### Track events and revenue

```typescript
import { Singular } from '@capawesome/capacitor-singular';

await Singular.trackEvent({ name: 'sng_login' });
await Singular.trackEvent({
  name: 'level_completed', // Standard or custom event name.
  attributes: { level: 3, character: 'warrior' },
});

await Singular.trackRevenue({
  amount: 9.99,
  currency: 'USD',
  eventName: 'subscription_purchase', // Optional. Tracks a custom revenue event.
});

await Singular.trackAdRevenue({
  adPlatform: 'AdMob',
  adType: 'Rewarded',
  currency: 'USD',
  revenue: 0.05,
});
```

### Identify the user

```typescript
import { Singular } from '@capawesome/capacitor-singular';

await Singular.setCustomUserId({ customUserId: 'user-123' });
await Singular.unsetCustomUserId();
```

### Global properties

Global properties are attached to all events. At most 5 can be set:

```typescript
import { Singular } from '@capawesome/capacitor-singular';

await Singular.setGlobalProperty({ key: 'plan', value: 'premium', overrideExisting: true });
const { properties } = await Singular.getGlobalProperties();
await Singular.unsetGlobalProperty({ key: 'plan' });
await Singular.clearGlobalProperties();
```

### Privacy

```typescript
import { Singular } from '@capawesome/capacitor-singular';

await Singular.trackingOptIn(); // Notifies the SDK that the user gave consent.
await Singular.trackingUnder13(); // Notifies the SDK that the user is under 13 years old.
await Singular.stopAllTracking();
await Singular.resumeAllTracking();
const { stopped } = await Singular.isAllTrackingStopped();

await Singular.setLimitDataSharing({ limit: true });
const { limit } = await Singular.getLimitDataSharing();
await Singular.setLimitAdvertisingIdentifiers({ limit: true });
```

### Create a referrer short link

```typescript
import { Singular } from '@capawesome/capacitor-singular';

const { link } = await Singular.createReferrerShortLink({
  baseLink: 'https://myapp.sng.link/A1b2c/d3e4',
  passthroughParameters: { campaign: 'friend-invite' },
  referrerId: 'user-123',
  referrerName: 'Jane Doe',
});
```

### SKAdNetwork

Only required on iOS with `iosManualSkanConversionManagement: true`:

```typescript
import { Singular, SkanCoarseConversionValue } from '@capawesome/capacitor-singular';

await Singular.addListener('skanConversionValueUpdated', (event) => {
  console.log(event.value, event.coarseValue, event.lockWindow);
});

await Singular.skanRegisterAppForAdNetworkAttribution();
await Singular.skanUpdateConversionValue({
  value: 10, // Fine-grained conversion value (0 - 63).
  coarseValue: SkanCoarseConversionValue.Medium, // iOS 16.1+. 'LOW' | 'MEDIUM' | 'HIGH'
  lockWindow: false, // iOS 16.1+. Default: false
});
const { value } = await Singular.skanGetConversionValue();
```

### Uninstall tracking

Pass the push notification device token to the SDK — the FCM registration token on Android, the hex-encoded APNs device token on iOS:

```typescript
import { PushNotifications } from '@capacitor/push-notifications';
import { Singular } from '@capawesome/capacitor-singular';

await PushNotifications.addListener('registration', async (token) => {
  await Singular.setDeviceToken({ token: token.value });
});
await PushNotifications.register();
```

### Singular Device ID

The Singular Device ID (SDID) is resolved by Singular during the first session. Add the listener before calling `initialize(...)`:

```typescript
import { Singular } from '@capawesome/capacitor-singular';

await Singular.addListener('sdidReceived', (event) => {
  console.log(event.sdid);
});
await Singular.addListener('sdidSet', (event) => {
  console.log(event.sdid); // Emitted once a custom SDID has been stored.
});
```

## Notes

- Only available on Android and iOS; all methods reject with `unimplemented` on the Web.
- Requires Capacitor 8 or later.
- `initialize(...)` must be called before every other method; otherwise the call rejects with the `NOT_INITIALIZED` error code.
- Always add the listeners before calling `initialize(...)`. Deferred deep links and the device attribution info are reported during the first session, so listeners attached afterwards miss those events.
- Limits: event names 32 characters, attribute keys and values 500 characters, at most 5 global properties. Currencies must be upper case ISO 4217 codes (e.g. `USD`).
- Standard event names (e.g. `sng_login`, `sng_tutorial_complete`) are listed under [Singular Standard Events](https://support.singular.net/hc/en-us/articles/7648172966299-Singular-Standard-Events-Full-List-and-Recommended-Events-by-Vertical). Any other name is tracked as a custom event.
- `stopAllTracking()` persists across app restarts until `resumeAllTracking()` is called. Query the current state with `isAllTrackingStopped()`.
- Deep links that open the app while it is already running are handled by the plugin: it re-initializes the SDK with the new link so that `singularLinkResolved` is emitted. On Android this requires the default `singleTask` launch mode of Capacitor's `MainActivity`.
- Events: `singularLinkResolved`, `deviceAttributionInfoReceived`, `sdidReceived`, `sdidSet`, `skanConversionValueUpdated`.
- The `skan*` methods and the `skanConversionValueUpdated` event are only available on iOS.
- `ErrorCode` values: `INITIALIZATION_FAILED`, `NOT_INITIALIZED`.
- This project is not affiliated with Singular Labs, Inc.
