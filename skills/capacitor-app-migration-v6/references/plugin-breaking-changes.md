# Official Plugin Breaking Changes (v6)

For all plugins with listeners: `addListener` now only returns a `Promise`. If the call result was stored without `await`, the code will no longer compile.

## Action Sheet

- `androidxMaterialVersion` updated to `1.10.0`.

## Camera

- Version 6 uses the Photo Picker API. Camera permissions are no longer required unless `saveToGallery: true`. If `saveToGallery` is `false`, these permissions can be removed from `AndroidManifest.xml` (if no other plugin requires them):
  ```xml
  <uses-permission android:name="android.permission.READ_MEDIA_IMAGES"/>
  <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
  ```
- On Android, cancelling gallery picker now returns `"User cancelled photos app"` (matching other platforms).
- `androidxMaterialVersion` updated to `1.10.0`.

## Filesystem

- iOS now returns `ctime` and `mtime` as numbers instead of strings (matching other platforms).

## Geolocation

- `NSLocationAlwaysUsageDescription` is deprecated and can be removed from `Info.plist`.
- `playServicesLocationVersion` updated to `21.1.0`.

## Google Maps

- iOS native libraries updated.
- `NSLocationAlwaysUsageDescription` is deprecated and can be removed from `Info.plist`.
- `googleMapsPlayServicesVersion` updated to `18.2.0`.
- `googleMapsUtilsVersion` updated to `3.8.2`.
- `googleMapsKtxVersion` updated to `5.0.0`.
- `googleMapsUtilsKtxVersion` updated to `5.0.0`.
- `kotlinxCoroutinesVersion` updated to `1.7.3`.
- `androidxCoreKTXVersion` updated to `1.12.0`.
- `kotlin_version` updated to `1.9.10`.

## Local Notifications

- On Android 14, notifications are not exact by default even with `SCHEDULE_EXACT_ALARM` permission.

## Push Notifications

- `firebaseMessagingVersion` updated to `23.3.1`.

## Share

- `androidxCoreVersion` updated to `1.12.0`.

## Splash Screen

- `coreSplashScreenVersion` updated to `1.0.1`.

## Status Bar

- `androidxCoreVersion` updated to `1.12.0`.
