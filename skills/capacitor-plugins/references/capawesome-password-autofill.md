# Password Autofill

Save a username and password to the platform credential store (iCloud Keychain on iOS, Google Password Manager on Android) after a successful in-app login.

**Package:** `@capawesome/capacitor-password-autofill`
**Platforms:** Android, iOS

This plugin saves credentials into the platform autofill system. It does not fill forms — it solves the problem that WebView-based apps never trigger the native "Save password?" prompt.

## Installation

```bash
npm install @capawesome/capacitor-password-autofill
npx cap sync
```

On Web, all methods reject as unimplemented.

## Configuration

### Android

#### Variables

Optionally define in `android/variables.gradle`:

- `$androidxCredentialsVersion` version of `androidx.credentials:credentials` and `androidx.credentials:credentials-play-services-auth` (default: `1.5.0`)

No further setup is required. Saving a credential presents the system "Save password?" prompt from the active credential provider (e.g. Google Password Manager).

### iOS

The Shared Web Credentials API requires the Associated Domains capability and a matching `apple-app-site-association` (AASA) file. Without this setup, `savePassword(...)` cannot save the credential.

#### Associated Domains

Add a `webcredentials:` entry for each domain to the Associated Domains entitlement of your app (`ios/App/App/App.entitlements`):

```xml
<key>com.apple.developer.associated-domains</key>
<array>
  <string>webcredentials:example.com</string>
</array>
```

The `domain` passed to `savePassword(...)` must match one of these entries (without the `webcredentials:` prefix).

#### Apple App Site Association

Host an `apple-app-site-association` file at `https://example.com/.well-known/apple-app-site-association`:

```json
{
  "webcredentials": {
    "apps": ["TEAMID.com.example.app"]
  }
}
```

Replace `TEAMID` with your Apple Developer Team ID and `com.example.app` with your app's bundle identifier.

## Usage

```typescript
import { PasswordAutofill } from '@capawesome/capacitor-password-autofill';

// Call after a successful login.
await PasswordAutofill.savePassword({
  domain: 'example.com', // required on iOS; must match an entitlement entry
  username: 'jane.doe@example.com',
  password: 'super-secret',
});
```

## Notes

- The `domain` option is required on iOS and must match an entry in the Associated Domains entitlement (without the `webcredentials:` prefix).
- On iOS, the credential is saved to the iCloud Keychain via the Shared Web Credentials API. On Android, it is saved via the Credential Manager API, which presents the system "Save password?" prompt.
- Unlike the official Capacitor autofill guide, this plugin does not require changing `server.hostname`, so the WebView origin (and existing `localStorage`, `IndexedDB`, cookies) stays intact. It also covers non-form flows such as social signup with generated passwords.
- Autofill (reading credentials back) still benefits from the `autocomplete` attributes described in the official Capacitor guide.
- `ErrorCode` values: `CANCELED`, `SAVE_FAILED`.
