# Device Info

Read device information such as model, manufacturer, operating system, memory, and a per-install identifier.

**Package:** `@capawesome/capacitor-device-info`
**Platforms:** Android, iOS, Web

## Installation

```bash
npm install @capawesome/capacitor-device-info
npx cap sync
```

No additional permissions or configuration are required on any platform.

## Usage

```typescript
import { DeviceInfo } from '@capawesome/capacitor-device-info';

const { identifier } = await DeviceInfo.getId();

const info = await DeviceInfo.getInfo();
// info.model, info.manufacturer, info.operatingSystem, info.osVersion, info.platform,
// info.deviceType, info.isVirtual, info.name, info.androidSdkVersion, info.iosVersion,
// info.totalMemory, info.usedMemory, info.webViewVersion

const { uptime } = await DeviceInfo.getUptime(); // milliseconds since last boot
```

Type values: `deviceType` is `'phone' | 'tablet' | 'desktop' | 'tv' | 'unknown'`; `operatingSystem` is `'android' | 'ios' | 'windows' | 'mac' | 'unknown'`; `platform` is `'android' | 'ios' | 'web'`.

## Notes

- `getInfo()` returns `null` for any field a platform cannot determine. `androidSdkVersion` is Android-only; `iosVersion`, `manufacturer`, `name`, `totalMemory`, and `usedMemory` are unavailable on Web.
- `getUptime()` is only available on Android and iOS.
- `getId()` identifier source: Android `ANDROID_ID` (unique per app-signing key, user, and device; reset on signing-key change or factory reset); iOS `identifierForVendor` (reset when all vendor apps are uninstalled); Web a random UUID persisted in `localStorage` (reset when browser storage is cleared).
- On iOS, `model` is the internal model identifier (e.g. `iPhone13,4`), not the marketing name. On iOS 16+, `name` returns a generic value (e.g. `iPhone`) unless the app has the required entitlement.
- Replacement for `@capacitor/device` `getId()`/`getInfo()`; use the Battery and Localization plugins for the battery and language methods.
