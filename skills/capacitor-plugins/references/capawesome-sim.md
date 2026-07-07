# SIM

Read SIM card and carrier information.

**Package:** `@capawesome/capacitor-sim`
**Platforms:** Android

## Installation

```bash
npm install @capawesome/capacitor-sim
npx cap sync
```

## Configuration

### Android

The `READ_PHONE_STATE` permission is declared in the plugin's `AndroidManifest.xml` and merged into your app automatically. You must request it at runtime via `requestPermissions()` before calling `getSimCards()`.

## Usage

### Request permission and read SIM cards

```typescript
import { Sim } from '@capawesome/capacitor-sim';

const { readSimCards } = await Sim.checkPermissions();
if (readSimCards !== 'granted') {
  await Sim.requestPermissions();
}

const { simCards } = await Sim.getSimCards();
for (const simCard of simCards) {
  console.log(simCard.carrierName, simCard.isoCountryCode, simCard.slotIndex);
}
```

## Notes

- This plugin is only available on **Android**. On iOS and Web, all methods reject as unimplemented.
- **iOS is not supported**: Apple deprecated the `CTCarrier` Core Telephony APIs with iOS 16, and they return placeholder values (e.g. `"--"` and `65535`) on iOS 16.4 and later, so there is no reliable system API left.
- On devices with multiple SIM slots, all active SIM cards are returned.
- Each `SimCard` exposes `carrierName`, `displayName`, `isEmbedded` (eSIM), `isoCountryCode`, `mobileCountryCode` (MCC), `mobileNetworkCode` (MNC), `phoneNumber`, and `slotIndex`. All fields except `slotIndex` may be `null` when unavailable.
- `phoneNumber` is often `null` because carriers do not reliably store the phone number on the SIM card.
