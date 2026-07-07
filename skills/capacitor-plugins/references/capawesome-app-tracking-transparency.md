# App Tracking Transparency

Capacitor plugin for Apple's [App Tracking Transparency](https://developer.apple.com/documentation/apptrackingtransparency) framework. Read the tracking authorization status, present the system prompt, and read the advertising identifier (IDFA).

**Package:** `@capawesome/capacitor-app-tracking-transparency`

**Platforms:** iOS

## Installation

```bash
npm install @capawesome/capacitor-app-tracking-transparency
npx cap sync
```

All methods reject as unimplemented on Android and Web.

## Configuration

### iOS

The `NSUserTrackingUsageDescription` key must be added to `ios/App/App/Info.plist`. Otherwise `requestPermission()` rejects with an error.

```xml
<key>NSUserTrackingUsageDescription</key>
<string>Your data will be used to deliver personalized ads to you.</string>
```

The purpose string must clearly explain why your app requests permission to track the user.

## Usage

```typescript
import { AppTrackingTransparency } from '@capawesome/capacitor-app-tracking-transparency';

const getStatus = async () => {
  const { status } = await AppTrackingTransparency.getStatus();
  return status; // 'authorized' | 'denied' | 'notDetermined' | 'restricted'
};

const requestPermission = async () => {
  const { status } = await AppTrackingTransparency.requestPermission();
  return status;
};

const getAdvertisingIdentifier = async () => {
  const { advertisingIdentifier } =
    await AppTrackingTransparency.getAdvertisingIdentifier();
  return advertisingIdentifier; // null unless status is 'authorized'
};
```

## Notes

- The system prompt is only shown once per install while the status is `notDetermined`. Calling `requestPermission()` again resolves with the existing status without showing the prompt.
- Call `requestPermission()` only in a context where tracking makes sense to avoid App Review rejections. The purpose string must accurately describe how collected data is used.
- The advertising identifier (IDFA) is only available while the status is `authorized`; otherwise `getAdvertisingIdentifier()` returns `null`.
- The iOS Simulator never provides an advertising identifier (always `null`, even when authorized). Use a real device to test.
