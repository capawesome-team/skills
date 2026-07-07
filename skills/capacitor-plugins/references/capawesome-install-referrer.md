# Install Referrer

Read install attribution data from the Play Install Referrer (Android) and Apple Ad Services (iOS).

**Package:** `@capawesome/capacitor-install-referrer`
**Platforms:** Android, iOS

## Installation

```bash
npm install @capawesome/capacitor-install-referrer
npx cap sync
```

## Configuration

### Android

#### Variables

Optionally define in `android/variables.gradle`:

- `$installReferrerVersion` version of `com.android.installreferrer:installreferrer` (default: `2.2`)

## Usage

### Read the Play Store install referrer (Android)

```typescript
import { InstallReferrer } from '@capawesome/capacitor-install-referrer';

const result = await InstallReferrer.getInstallReferrer();
console.log(result.referrerUrl);
console.log(result.referrerClickTimestampMillis);
console.log(result.installBeginTimestampMillis);
console.log(result.googlePlayInstantParam);
```

### Read the Apple Ad Services attribution token (iOS)

```typescript
import { InstallReferrer } from '@capawesome/capacitor-install-referrer';

const { token } = await InstallReferrer.getAttributionToken();
```

## Notes

- `getInstallReferrer()` is Android-only; `getAttributionToken()` is iOS-only. On all other platforms, the respective method rejects as unimplemented.
- Android: Google only guarantees the install referrer is available for a limited window after install. Fetch it once shortly after the first launch and persist it in your app.
- iOS: the attribution token is opaque and only the client half of the flow. Exchange it server-side against Apple's Ad Services API (`POST https://api-adservices.apple.com/api/v1/`, `Content-Type: text/plain`, token as the body). It expires after 24 hours.
- `ErrorCode` values: `DEVELOPER_ERROR`, `FEATURE_NOT_SUPPORTED`, `SERVICE_UNAVAILABLE` (retryable), `TOKEN_GENERATION_FAILED`.
