# Wallet

Add passes to Apple Wallet and Google Wallet.

**Package:** `@capawesome/capacitor-wallet`
**Platforms:** Android, iOS

## Installation

```bash
npm install @capawesome/capacitor-wallet
npx cap sync
```

## Configuration

### iOS

Adding passes to Apple Wallet via `PKAddPassesViewController` does **not** require any special entitlement. The `com.apple.developer.pass-type-identifiers` entitlement is only needed to read or manage passes your app owns, which is out of scope for this plugin.

### Android

No additional permissions or Gradle variables are required. The plugin opens the Google Wallet "Save to Wallet" flow, handled by the Google Wallet app or the device's browser.

## Usage

### Add passes to Apple Wallet (iOS)

```typescript
import { Wallet } from '@capawesome/capacitor-wallet';

const { canAdd } = await Wallet.canAddPasses();
if (canAdd) {
  // `passes` are base64-encoded `.pkpass` files signed on your server.
  await Wallet.addPasses({ passes });
}
```

### Save a pass to Google Wallet (Android)

```typescript
import { Wallet } from '@capawesome/capacitor-wallet';

// `jwt` is a signed Google Wallet JWT created on your server.
await Wallet.saveToGoogleWallet({ jwt });
```

## Notes

- This plugin only **presents** passes. Passes must be **created and signed on your server**, because signing requires private keys and certificates that must never ship inside your app.
- `addPasses()` and `canAddPasses()` are only available on **iOS**. `saveToGoogleWallet()` is only available on **Android**.
- **Apple Wallet**: Each pass is a base64-encoded `.pkpass` bundle signed on your server with your Pass Type ID certificate. Use `canAddPasses()` to gate the UI, as adding passes may be unavailable on some devices (e.g. certain iPads) or when restricted.
- **Google Wallet**: A pass is a signed JWT created on your server with your Google Wallet service account. With the URL-based "Save to Wallet" flow, there is no completion signal, so `saveToGoogleWallet()` resolves as soon as the flow is launched — not when the pass is actually saved.
