# SMS Composer

Open the native SMS composer prefilled with recipients and a message body.

**Package:** `@capawesome/capacitor-sms-composer`
**Platforms:** Android, iOS

## Installation

```bash
npm install @capawesome/capacitor-sms-composer
npx cap sync
```

## Usage

### Check capability and compose an SMS

```typescript
import { SmsComposer } from '@capawesome/capacitor-sms-composer';

const { canCompose } = await SmsComposer.canComposeSms();
if (!canCompose) {
  return;
}

const { status } = await SmsComposer.composeSms({
  recipients: ['+41791234567'],
  body: 'Hello from Capacitor!',
});
// status: 'sent' | 'canceled' | 'unknown'
```

## Notes

- The user always reviews and sends the message. The plugin **never sends SMS on its own**.
- `composeSms(...)` resolves once the composer is dismissed. If the device cannot compose SMS messages, it rejects as unavailable — gate the call with `canComposeSms()`.
- **Status**: On iOS, `status` reflects the actual outcome (`sent` or `canceled`). On **Android**, `status` is always `unknown` because the system does not report whether the message was sent.
- **Android**: The plugin declares the required `<queries>` element for the `smsto` scheme in its `AndroidManifest.xml`, so capability can be detected on Android 11 (API level 30) and higher. No additional configuration is required.
- Multiple recipients are joined with a semicolon (`;`) in the underlying `smsto:` URI on Android. Some messaging apps expect a comma instead, so multi-recipient prefilling may not work reliably across all Android messaging apps.
- On Web, all methods reject as unimplemented.
