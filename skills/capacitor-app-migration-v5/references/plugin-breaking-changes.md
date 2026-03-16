# Official Plugin Breaking Changes (v5)

## Action Sheet

- `androidxMaterialVersion` updated to `1.8.0`.

## Browser

- `androidxBrowserVersion` updated to `1.5.0`.

## Camera

- Android 13 requires `<uses-permission android:name="android.permission.READ_MEDIA_IMAGES"/>` in `AndroidManifest.xml`.
- `androidxMaterialVersion` updated to `1.8.0`.
- `androidxExifInterfaceVersion` updated to `1.3.6`.

## Device

- `DeviceId.uuid` renamed to `DeviceId.identifier`.
- On iOS 16+, `DeviceInfo.name` returns a generic name unless the appropriate [entitlements](https://developer.apple.com/documentation/bundleresources/entitlements/com_apple_developer_device-information_user-assigned-device-name) are added.

## Geolocation

- `playServicesLocationVersion` updated to `21.0.1`.

## Google Maps

- `googleMapsPlayServicesVersion` updated to `18.1.0`.
- `googleMapsUtilsVersion` updated to `3.4.0`.
- `googleMapsKtxVersion` updated to `3.4.0`.
- `googleMapsUtilsKtxVersion` updated to `3.4.0`.
- `kotlinxCoroutinesVersion` updated to `1.6.4`.
- `androidxCoreKTXVersion` updated to `1.10.0`.
- `kotlin_version` updated to `1.8.20`.

## Local Notifications

- Android 13 requires runtime permission check via `checkPermissions()` and `requestPermissions()` when targeting SDK 33.

## Push Notifications

- Android 13 requires runtime permission check via `checkPermissions()` and `requestPermissions()` when targeting SDK 33.
- `firebaseMessagingVersion` updated to `23.1.2`.

## Status Bar

- On iOS, the default status bar animation changed to `FADE`.
