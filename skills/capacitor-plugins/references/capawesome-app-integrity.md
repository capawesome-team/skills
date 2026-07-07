# App Integrity

Capacitor plugin to verify app and device integrity using the Play Integrity API (Android) and App Attest (iOS).

**Package:** `@capawesome/capacitor-app-integrity`

**Platforms:** Android, iOS

## Installation

```bash
npm install @capawesome/capacitor-app-integrity
npx cap sync
```

## Configuration

### Android

The Play Integrity API must be set up in the Google Play Console before you can request integrity tokens. Follow the [official setup instructions](https://developer.android.com/google/play/integrity/setup) to link a Google Cloud project to your app. You need the Google Cloud project number of the linked project for the standard request flow.

#### Variables

This project variable can be set in `android/app/variables.gradle`:

- `$playIntegrityVersion` version of `com.google.android.play:integrity` (default: `1.6.0`)

### iOS

App Attest requires the App Attest capability. Add the following entitlement to `ios/App/App/App.entitlements`:

```xml
<key>com.apple.developer.devicecheck.appattest-environment</key>
<string>development</string>
```

Use `development` to attest against Apple's sandbox servers during development and `production` for App Store builds. App Attest is not supported on the iOS Simulator; use a real device for testing.

## Usage

### Android (Play Integrity)

```typescript
import { AppIntegrity } from '@capawesome/capacitor-app-integrity';

// Call once, e.g. at app start, well ahead of the first standard request.
await AppIntegrity.prepareIntegrityToken({
  cloudProjectNumber: 123456789012,
});

// Standard request (recommended, requires the prepare call above).
const { token } = await AppIntegrity.requestIntegrityToken({
  requestHash: '2cp24z...',
});

// Classic request (for infrequent, high-value actions).
const { token: classicToken } = await AppIntegrity.requestIntegrityToken({
  nonce: 'R2Rra24fVm5xa2Mg',
});
// Send the token to your server for verification.
```

### iOS (App Attest)

```typescript
import { AppIntegrity } from '@capawesome/capacitor-app-integrity';

const { available } = await AppIntegrity.isAvailable();

// 1. Generate a new key pair and store the key identifier.
const { keyId } = await AppIntegrity.generateKey();

// 2. Attest the key with a one-time challenge from your server.
const { attestationObject } = await AppIntegrity.attestKey({
  keyId,
  challenge: 'dGhpc2lzYWNoYWxsZW5nZQ==',
});

// 3. Later, generate assertions for signed client data.
const { assertion } = await AppIntegrity.generateAssertion({
  keyId,
  clientData: 'eyJjaGFsbGVuZ2UiOiJkR2hwYzJsellXTm9ZV3hzWlc1blpRPT0ifQ==',
});
// Send attestationObject / assertion to your server for verification.
```

## Notes

- `isAvailable()` checks Google Play Services on Android and App Attest support on iOS (not supported on the iOS Simulator).
- `prepareIntegrityToken(...)` and `requestIntegrityToken(...)` are Android only; `generateKey()`, `attestKey(...)`, and `generateAssertion(...)` are iOS only.
- This plugin only provides the client-side part of the flow. Tokens, attestation objects, and assertions are opaque and must be verified on your own server (Google's Play Integrity API on Android; Apple's App Attest validation on iOS). Never make security decisions from the client-side result alone.
- On Android, use the standard request flow for frequent checks and the classic request flow only for infrequent, high-value actions. `cloudProjectNumber` is required for classic requests if the app is distributed outside Google Play.
- App Attest private keys are stored in the Secure Enclave; challenges and client data are hashed with SHA-256 on device.
- Provides detailed error codes such as `PLAY_SERVICES_NOT_FOUND`, `TOO_MANY_REQUESTS`, `CLIENT_TRANSIENT_ERROR` (Android) and `INVALID_KEY`, `ATTESTATION_FAILED`, `ASSERTION_FAILED`, `SERVER_UNAVAILABLE` (iOS).
