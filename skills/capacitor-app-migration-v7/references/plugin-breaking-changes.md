# Official Plugin Breaking Changes (v7)

## Action Sheet

- `androidxMaterialVersion` updated to `1.12.0`.

## App

- Deprecated type `AppRestoredResult` removed, use `RestoredListenerEvent`.
- Deprecated type `AppUrlOpen` removed, use `URLOpenListenerEvent`.

## Browser

- `androidxBrowserVersion` updated to `1.8.0`.

## Camera

- `androidxExifInterfaceVersion` updated to `1.3.7`.
- `androidxMaterialVersion` updated to `1.12.0`.

## Device

- `getInfo()` no longer returns `diskFree`, `diskTotal`, `realDiskFree`, and `realDiskTotal`. `PrivacyInfo.xcprivacy` entries for this plugin can be removed.
- Deprecated type `DeviceBatteryInfo` removed, use `BatteryInfo`.
- Deprecated type `DeviceLanguageCodeResult` removed, use `GetLanguageCodeResult`.

## Geolocation

- `playServicesLocationVersion` updated to `21.3.0`.

## Haptics

- Deprecated type `HapticsImpactOptions` removed, use `ImpactOptions`.
- Deprecated type `HapticsNotificationOptions` removed, use `NotificationOptions`.
- Deprecated type `HapticsNotificationType` removed, use `NotificationType`.
- Deprecated type `HapticsImpactStyle` removed, use `ImpactStyle`.

## Push Notifications

- `firebaseMessagingVersion` updated to `24.1.0`.

## Share

- `androidxCoreVersion` updated to `1.15.0`.

## Splash Screen

- Deprecated type `SplashScreenShowOptions` removed, use `ShowOptions`.
- Deprecated type `SplashScreenHideOptions` removed, use `HideOptions`.

## Status Bar

- `setOverlaysWebView()` and `setBackgroundColor()` are now supported on iOS.
- `androidxCoreVersion` updated to `1.15.0`.
