# Passkeys

Create and authenticate with passkeys based on the WebAuthn standard.

**Package:** `@capawesome/capacitor-passkeys`
**Platforms:** Android, iOS, Web

## Installation

```bash
npm install @capawesome/capacitor-passkeys
npx cap sync
```

## Configuration

Your app must be associated with the domain of the relying party (`rp.id` / `rpId`). Otherwise the plugin methods reject with the `DOMAIN_NOT_ASSOCIATED` error code.

### Android

Passkeys require Android 9 (API level 28) or higher with an available credential provider (e.g. Google Password Manager).

#### Variables

Optionally define in `android/variables.gradle`:

- `$androidxCredentialsVersion` version of `androidx.credentials:credentials` and `androidx.credentials:credentials-play-services-auth` (default: `1.5.0`)

#### Digital Asset Links

Host a JSON file at `https://<your-domain>/.well-known/assetlinks.json` that delegates the `common.get_login_creds` permission to your app:

```json
[
  {
    "relation": [
      "delegate_permission/common.handle_all_urls",
      "delegate_permission/common.get_login_creds"
    ],
    "target": {
      "namespace": "android_app",
      "package_name": "com.example.app",
      "sha256_cert_fingerprints": [
        "01:23:45:67:89:AB:CD:EF:01:23:45:67:89:AB:CD:EF:01:23:45:67:89:AB:CD:EF:01:23:45:67:89:AB:CD:EF"
      ]
    }
  }
]
```

Replace `com.example.app` with your app's application ID and the fingerprint with the SHA-256 fingerprint of your app's signing certificate.

### iOS

Passkeys require iOS 15 or higher.

#### Associated Domains

Add the `webcredentials` service to the Associated Domains entitlement of your app (`ios/App/App/App.entitlements`):

```xml
<key>com.apple.developer.associated-domains</key>
<array>
  <string>webcredentials:example.com</string>
</array>
```

Additionally, host an `apple-app-site-association` file at `https://<your-domain>/.well-known/apple-app-site-association`:

```json
{
  "webcredentials": {
    "apps": ["TEAMID.com.example.app"]
  }
}
```

Replace `TEAMID` with your Apple Developer Team ID and `com.example.app` with your app's bundle identifier.

## Usage

The options mirror the WebAuthn JSON serialization, so values from any WebAuthn server library can be passed through unchanged. In production, generate the options and verify the results with a WebAuthn server library.

### Create (register) a passkey

```typescript
import { Passkeys } from '@capawesome/capacitor-passkeys';

const result = await Passkeys.createPasskey({
  challenge: 'dGhpc2lzYWNoYWxsZW5nZQ', // provided by your server
  rp: { id: 'example.com', name: 'Example Inc.' },
  user: {
    id: 'anVzdGFyYW5kb21pZA',
    name: 'jane.doe@example.com',
    displayName: 'Jane Doe',
  },
  pubKeyCredParams: [
    { alg: -7, type: 'public-key' },
    { alg: -257, type: 'public-key' },
  ],
  attestation: 'none',
  authenticatorSelection: { residentKey: 'required', userVerification: 'required' },
});
// Send `result` (RegistrationResponseJSON) to your server for verification.
```

### Get (authenticate with) a passkey

```typescript
import { Passkeys } from '@capawesome/capacitor-passkeys';

const result = await Passkeys.getPasskey({
  challenge: 'dGhpc2lzYWNoYWxsZW5nZQ', // provided by your server
  rpId: 'example.com',
  userVerification: 'required',
});
// Send `result` (AuthenticationResponseJSON) to your server for verification.
```

### Check availability

```typescript
import { Passkeys } from '@capawesome/capacitor-passkeys';

const { available } = await Passkeys.isAvailable();
```

## Notes

- `isAvailable()`: on Android returns `true` on API level 28+; on iOS always `true`; on Web `true` if the browser supports WebAuthn and a user-verifying platform authenticator is available.
- Test on a real device with a screen lock configured. Domain association changes can take time to propagate because the files are cached by Apple and Google.
- Conditional UI (passkey autofill) is not supported, because OS autofill targets native text fields, not HTML inputs in a web view.
- `ErrorCode` values: `CANCELED`, `CREATE_FAILED`, `DOMAIN_NOT_ASSOCIATED`, `GET_FAILED`, `NO_CREDENTIAL`, `NOT_SUPPORTED`.
