# AdMob

Capacitor plugin for monetizing apps with Google AdMob ads. Supports banner, interstitial, rewarded, rewarded interstitial and app open ads, ad revenue events, and the User Messaging Platform (UMP) consent flow.

**Package:** `@capawesome-team/capacitor-admob`
**Platforms:** Android, iOS
**Capawesome Insiders:** Yes (requires license key)

## Installation

Set up the Capawesome npm registry:

```bash
npm config set @capawesome-team:registry https://npm.registry.capawesome.io
npm config set //npm.registry.capawesome.io/:_authToken <YOUR_LICENSE_KEY>
```

Install the package:

```bash
npm install @capawesome-team/capacitor-admob
npx cap sync
```

## Configuration

### Android

#### Meta-data

Add the AdMob app ID **inside** the `application` tag in `android/app/src/main/AndroidManifest.xml`:

```xml
<meta-data
    android:name="com.google.android.gms.ads.APPLICATION_ID"
    android:value="ca-app-pub-3940256099942544~3347511713" />
```

The value above is Google's public **test** app ID. Replace it with the real AdMob app ID before release. If the `meta-data` element is missing, `initialize(...)` rejects with `APPLICATION_ID_MISSING`.

#### Variables

Optionally override the dependency versions in `android/variables.gradle`:

- `adsMobileSdkVersion` — version of `com.google.android.libraries.ads.mobile.sdk:ads-mobile-sdk` (default: `1.2.1`)
- `userMessagingPlatformVersion` — version of `com.google.android.ump:user-messaging-platform` (default: `3.2.0`)

#### Proguard

If using Proguard, add to `android/app/proguard-rules.pro`:

```
-keep class io.capawesome.capacitorjs.plugins.** { *; }
```

### iOS

#### Info.plist Keys

Add the AdMob app ID and the SKAdNetwork identifiers to `ios/App/App/Info.plist`:

```xml
<key>GADApplicationIdentifier</key>
<string>ca-app-pub-3940256099942544~1458002511</string>
<key>SKAdNetworkItems</key>
<array>
  <dict>
    <key>SKAdNetworkIdentifier</key>
    <string>cstr6suwn9.skadnetwork</string>
  </dict>
</array>
```

`GADApplicationIdentifier` above is Google's public **test** app ID; replace it with the real AdMob app ID before release. If the key is missing, `initialize(...)` rejects with `APPLICATION_ID_MISSING`. Only Google's own SKAdNetwork identifier is listed; add the complete list from https://developers.google.com/admob/ios/quick-start#update_your_infoplist.

## Usage

### Gather consent and initialize the SDK

Call `requestConsent(...)` on **every** app launch before initializing the SDK. If `privacyOptionsRequired` is `true`, offer `Admob.showPrivacyOptionsForm()` from a privacy settings page.

```typescript
import { Admob } from '@capawesome-team/capacitor-admob';

const { canRequestAds, privacyOptionsRequired } = await Admob.requestConsent();
if (canRequestAds) {
  await Admob.initialize();
}
```

### Show a banner ad

```typescript
import { Admob, BannerSize } from '@capawesome-team/capacitor-admob';

const { id } = await Admob.showBanner({
  adUnitId: 'ca-app-pub-3940256099942544/6300978111', // Test ad unit ID
  size: BannerSize.AdaptiveBanner,
  position: 'bottom',
  mode: 'resize',
});
```

Use the returned `id` with `hideBanner({ id })`, `resumeBanner({ id })`, `removeBanner({ id })` and `setBannerFrame({ id, frame })`.

### Show an interstitial ad

```typescript
import { Admob } from '@capawesome-team/capacitor-admob';

const { id } = await Admob.loadInterstitialAd({
  adUnitId: 'ca-app-pub-3940256099942544/1033173712', // Test ad unit ID
});
await Admob.showInterstitialAd({ id });
```

### Show a rewarded ad

```typescript
import { Admob } from '@capawesome-team/capacitor-admob';

await Admob.addListener('rewardEarned', (event) => {
  console.log(`User earned ${event.amount} ${event.type}`);
});
const { id } = await Admob.loadRewardedAd({
  adUnitId: 'ca-app-pub-3940256099942544/5224354917', // Test ad unit ID
});
await Admob.showRewardedAd({ id });
```

### Show app open ads automatically and track revenue

```typescript
import { Admob } from '@capawesome-team/capacitor-admob';

await Admob.enableAppOpenAutoShow({
  adUnitId: 'ca-app-pub-3940256099942544/9257395921', // Test ad unit ID
  minInterval: 14400, // Seconds, default: 14400
});
await Admob.addListener('adRevenuePaid', (event) => {
  console.log(`Ad revenue: ${event.value} ${event.currencyCode} (${event.precision})`);
});
```

## Notes

- All methods are only available on Android and iOS. On the web they reject with an unimplemented error.
- Always use Google's public test ad unit IDs during development. Using production ad units during development can get the AdMob account suspended.
- Banner placement: `mode: 'overlay'` (default) draws the banner on top of the web view, `mode: 'resize'` shrinks the web view. Passing `frame: { x, y, width, height }` (CSS pixels, e.g. from `getBoundingClientRect()`) places the banner inline and ignores `mode`/`position`; update it with `setBannerFrame(...)` on layout changes.
- `BannerSize`: `AdaptiveBanner` (default), `InlineAdaptiveBanner`, `Banner`, `LargeBanner`, `MediumRectangle`, `FullBanner`, `Leaderboard`. `position`: `'bottom'` (default) or `'top'`. Set `collapsible: true` for collapsible banners.
- `showBanner(...)` and the `load*` methods take an optional `id` (generated if omitted), so multiple ads of the same format can exist at once. They return the `id` that the matching `show*` method requires.
- `initialize(...)` options: `maxAdContentRating`, `tagForChildDirectedTreatment`, `tagForUnderAgeOfConsent`, `testDeviceIds`. Rewarded and rewarded interstitial ads support `serverSideVerification: { userId, customData }`.
- Error codes (`error.code`, `ErrorCode` enum): `AD_ALREADY_SHOWING`, `AD_NOT_LOADED`, `APPLICATION_ID_MISSING`, `CONSENT_FORM_UNAVAILABLE`, `CONSENT_NOT_GATHERED`, `CONSENT_REQUEST_FAILED`, `LOAD_FAILED`, `NOT_INITIALIZED`.
- Events: `adClicked`, `adDismissed`, `adFailedToLoad`, `adFailedToShow`, `adImpressionRecorded`, `adLoaded`, `adRevenuePaid`, `adShowed`, `bannerSizeChanged`, `rewardEarned`. The `errorCode` of `adFailedToLoad`/`adFailedToShow` is the numeric Google Mobile Ads SDK code, not an `ErrorCode`.
- Test the consent flow with `resetConsent()` plus the `debugGeography` (`DebugGeography` enum) and `testDeviceIds` options of `requestConsent(...)`.
- For personalized ads on iOS, request the tracking permission with the App Tracking Transparency plugin **after** `requestConsent(...)` and **before** `initialize(...)`, otherwise App Store rejections are likely.
- Other methods: `disableAppOpenAutoShow()`, `setApplicationMuted({ muted })`, `setApplicationVolume({ volume })`, `removeAllListeners()`.
