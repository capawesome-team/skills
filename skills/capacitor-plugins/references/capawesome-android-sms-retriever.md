# Android SMS Retriever

Capacitor plugin for OTP autofill on Android via the SMS User Consent and Phone Number Hint APIs. Requires no SMS permissions.

**Package:** `@capawesome/capacitor-android-sms-retriever`

**Platforms:** Android

## Installation

```bash
npm install @capawesome/capacitor-android-sms-retriever
npx cap sync
```

## Configuration

### Android

#### Variables

These project variables can be set in `android/app/variables.gradle`:

- `$playServicesAuthVersion` version of `com.google.android.gms:play-services-auth` (default: `21.5.0`)
- `$playServicesAuthApiPhoneVersion` version of `com.google.android.gms:play-services-auth-api-phone` (default: `18.3.0`)

### iOS

This plugin has no iOS implementation and does not need one. iOS already autofills one-time codes from incoming SMS messages in `WKWebView` when the input field uses `autocomplete="one-time-code"`:

```html
<input autocomplete="one-time-code" />
```

All methods reject as unimplemented on iOS and Web.

## Usage

```typescript
import { AndroidSmsRetriever } from '@capawesome/capacitor-android-sms-retriever';

const requestPhoneNumber = async () => {
  const { phoneNumber } = await AndroidSmsRetriever.requestPhoneNumber();
  return phoneNumber;
};

const retrieveSms = async () => {
  const { message } = await AndroidSmsRetriever.retrieveSms();
  // Extract the one-time code from the message yourself.
  const code = message.match(/\d{6}/)?.[0];
  return code;
};

// Optionally filter incoming messages by sender.
const retrieveSmsFromSender = async () => {
  const { message } = await AndroidSmsRetriever.retrieveSms({
    senderPhoneNumber: '+12025550123',
  });
  return message;
};
```

## Notes

- `requestPhoneNumber()` shows a system bottom sheet for the user to pick one of the device's phone numbers.
- `retrieveSms(...)` shows a system consent dialog when a matching SMS arrives and resolves with the full message text; your app extracts the code.
- For the SMS User Consent API to detect a message it must contain a one-time code, be no longer than 140 bytes, and not originate from a number in the user's contacts.
- The underlying broadcast waits up to 5 minutes for a matching SMS. If none arrives, the promise rejects with error code `TIMEOUT`.
- Error codes: `CANCELED`, `PHONE_NUMBER_HINT_FAILED`, `RETRIEVE_FAILED`, `TIMEOUT`, `USER_DENIED`.
