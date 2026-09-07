# TikTok App Events

Unofficial Capacitor plugin for the TikTok App Events SDK to track installs, purchases and custom events for TikTok ad campaigns.

**Package:** `@capawesome/capacitor-tiktok-app-events`
**Platforms:** Android, iOS
**Documentation:** https://capawesome.io/docs/sdks/capacitor/tiktok-app-events/

## Installation

```bash
npm install @capawesome/capacitor-tiktok-app-events
npx cap sync
```

Requires a [TikTok for Business](https://ads.tiktok.com/) account. Connect the app in the **TikTok Events Manager** and generate a **TikTok App ID** and an **Access Token** (see [TikTok's guide](https://ads.tiktok.com/help/article/how-to-integrate-tiktok-app-events-sdk)).

Use of the SDK is governed by the [TikTok for Business Commercial Terms of Service](https://ads.tiktok.com/i18n/official/policy/commercial-terms-of-service) and the [TikTok Business Products (Data) Terms](https://ads.tiktok.com/i18n/official/policy/business-products-terms), which prohibit sharing sensitive data with TikTok.

## Configuration

### Android

#### Repositories

The SDK is resolved from [JitPack](https://jitpack.io/). The plugin already declares the repository in its `build.gradle`. If the app restricts repositories to the settings file (`dependencyResolutionManagement` with `FAIL_ON_PROJECT_REPOS`), add it to `android/settings.gradle`:

```groovy
dependencyResolutionManagement {
    repositories {
        maven { url 'https://jitpack.io' }
    }
}
```

#### Variables

Defined in your app's `android/variables.gradle`:

- `$tiktokBusinessSdkVersion` version of `com.github.tiktok:tiktok-business-android-sdk` (default: `1.7.1`)
- `$installReferrerVersion` version of `com.android.installreferrer:installreferrer` (default: `2.2`)

### iOS

- The SDK collects the IDFA only after the user granted tracking permission. Request it with `@capawesome/capacitor-app-tracking-transparency` (see `references/capawesome-app-tracking-transparency.md`).
- For SKAdNetwork attribution, add TikTok's SKAdNetwork identifiers to `ios/App/App/Info.plist` as listed in the [TikTok App Events SDK documentation](https://business-api.tiktok.com/portal/docs?id=1739585432134657).

## Usage

### Initialize

```typescript
import { TiktokAppEvents } from '@capawesome/capacitor-tiktok-app-events';

await TiktokAppEvents.initialize({
  accessToken: 'YOUR_ACCESS_TOKEN',
  tiktokAppId: 'YOUR_TIKTOK_APP_ID', // Comma-separated list for multiple IDs.
  iosAppId: 'YOUR_APP_STORE_ID', // Required on iOS. Android uses the package name.
  automaticTracking: true, // InstallApp, LaunchAPP and 2Dretention events. Default: true
  automaticPurchaseTracking: true, // Google Play Billing / StoreKit purchases. Default: true
  debugMode: false, // Sends events to the test pipeline. Default: false
  limitedDataUse: false, // Limited Data Use mode. Default: false
  iosSkAdNetworkSupport: true, // Set to false if an MMP updates the conversion value. Default: true
});
```

### Track events

```typescript
import { TiktokAppEvents } from '@capawesome/capacitor-tiktok-app-events';

await TiktokAppEvents.trackEvent({
  name: 'Purchase', // Standard or custom event name.
  id: 'order-123', // Optional. Deduplicates against events sent via the Events API.
  properties: {
    currency: 'USD',
    value: 9.99,
    contents: [{ content_id: 'sku-123', content_type: 'product', quantity: 1, price: 9.99 }],
  },
});

await TiktokAppEvents.trackEvent({ name: 'LevelCompleted', properties: { level: 3 } });
```

### Identify the user

```typescript
import { TiktokAppEvents } from '@capawesome/capacitor-tiktok-app-events';

await TiktokAppEvents.identify({
  externalId: 'user-123',
  externalUserName: 'jane',
  phoneNumber: '+491234567890',
  email: 'jane.doe@example.com',
});

await TiktokAppEvents.logout();
```

### Flush queued events

```typescript
import { TiktokAppEvents } from '@capawesome/capacitor-tiktok-app-events';

await TiktokAppEvents.flush();
```

## Notes

- Only available on Android and iOS; all methods reject with `unimplemented` on the Web.
- `initialize(...)` must be called before every other method; otherwise the call rejects with the `NOT_INITIALIZED` error code. Call it on app start or after the user gave consent.
- `iosAppId` is the numeric App Store ID and is required on iOS. On Android the SDK uses the app's package name automatically.
- Standard event names (e.g. `Purchase`, `AddToCart`, `Registration`, `Search`) and properties (`currency`, `value`, `contents`, `content_type`, `content_id`, `description`) are listed under [Supported In-App Events](https://ads.tiktok.com/help/article/all-supported-in-app-events). Any other name is tracked as a custom event.
- The SDK hashes `email`, `phoneNumber` and `externalId` with SHA-256 on the device before sending them.
- With `debugMode: true`, events go to the test pipeline and appear in the **Test Events** tab of the Events Manager. Disable it before release.
- Set `iosSkAdNetworkSupport: false` when a mobile measurement partner or another SDK already updates the SKAdNetwork conversion value.
- `ErrorCode` values: `INITIALIZATION_FAILED`, `NOT_INITIALIZED`.
- This project is not affiliated with TikTok Inc.
