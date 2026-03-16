# Official Plugin Breaking Changes (v8)

## Action Sheet

- `androidxMaterialVersion` updated to `1.13.0`.

## Barcode Scanner

- `scanOrientation` has no effect on large screens (tablets) on Android 16+. Opt-out temporarily by adding to `AndroidManifest.xml` inside `<application>` or `<activity>`:
  ```xml
  <property android:name="android.window.PROPERTY_COMPAT_ALLOW_RESTRICTED_RESIZABILITY" android:value="true" />
  ```
  This opt-out will stop working on Android 17. Regular phones are unaffected.

## Browser

- `androidxBrowserVersion` updated to `1.9.0`.

## Camera

- `androidxExifInterfaceVersion` updated to `1.4.1`.
- `androidxMaterialVersion` updated to `1.13.0`.

## Geolocation

- `kotlinxCoroutinesVersion` updated to `1.10.2`.
- `timeout` now applies to all requests on Android and iOS (previously only web and `getCurrentPosition` on Android). Increase `timeout` if experiencing timeouts. For `watchPosition` on Android, use the new `interval` parameter.

## Google Maps

- `googleMapsPlayServicesVersion` updated to `19.2.0`.
- `googleMapsUtilsVersion` updated to `3.19.1`.
- `googleMapsKtxVersion` updated to `5.2.1`.
- `googleMapsUtilsKtxVersion` updated to `5.2.1`.
- `kotlinxCoroutinesVersion` updated to `1.10.2`.
- `androidxCoreKTXVersion` updated to `1.17.0`.
- `kotlin_version` updated to `2.2.20`.

## Push Notifications

- `firebaseMessagingVersion` updated to `25.0.1`.

## Screen Orientation

- `lock` method has no effect on large screens (tablets) on Android 16+. Same opt-out as Barcode Scanner applies.

## Splash Screen

- `coreSplashScreenVersion` updated to `1.2.0`.

## Status Bar

- `CAPNotifications.swift` and `CAPBridgeViewController.swift` that emitted `.capacitorViewDidAppear` and `.capacitorViewWillTransition` events have been removed. Listen for these events from `@capacitor/ios` instead.
