# Phone Dialer

Open the native phone dialer prefilled with a phone number.

**Package:** `@capawesome/capacitor-phone-dialer`
**Platforms:** Android, iOS

## Installation

```bash
npm install @capawesome/capacitor-phone-dialer
npx cap sync
```

On Web, all methods reject as unimplemented. Use an `<a href="tel:...">` link on the web instead.

## Configuration

### Android

No configuration is required. The plugin already declares the necessary `<queries>` element for the `tel` scheme in its `AndroidManifest.xml`, so the dialer can be detected on Android 11 (API level 30) and higher.

### iOS

To let `canDial()` detect whether the device can open the dialer, add the `tel` scheme to the `LSApplicationQueriesSchemes` array in `ios/App/App/Info.plist`:

```xml
<key>LSApplicationQueriesSchemes</key>
<array>
  <string>tel</string>
</array>
```

## Usage

```typescript
import { PhoneDialer } from '@capawesome/capacitor-phone-dialer';

// Check whether the device can open the dialer.
const { canDial } = await PhoneDialer.canDial();

// Open the dialer prefilled with the number.
await PhoneDialer.dial({ number: '+41791234567' });
```

## Notes

- The plugin only opens the dialer prefilled with the number; it never places the call itself. On Android it uses `Intent.ACTION_DIAL` (no `CALL_PHONE` permission needed); on iOS it opens the `tel:` URL, which shows the system Call/Cancel confirmation.
- The number is sanitized before dialing: only digits and the characters `+`, `*` and `#` are kept; all other characters (spaces, dashes) are removed. If nothing remains after sanitization, `dial(...)` rejects.
- On devices without telephony capability (Wi-Fi-only tablets, iPod touch), `canDial()` resolves with `canDial: false`, and `dial(...)` rejects as unavailable.
- `ErrorCode` values: `DIAL_FAILED`.
