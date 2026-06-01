# Vault

Store secrets behind biometric or device-passcode authentication using hardware-backed encryption (Android Keystore, iOS Keychain). Unlock once, then perform many read/write operations until the vault locks again.

**Package:** `@capawesome-team/capacitor-vault`
**Platforms:** Android, iOS, Web
**Capawesome Insiders:** Yes (requires license key)

## Installation

Set up the Capawesome npm registry:

```bash
npm config set @capawesome-team:registry https://npm.registry.capawesome.io
npm config set //npm.registry.capawesome.io/:_authToken <YOUR_LICENSE_KEY>
```

Install the package:

```bash
npm install @capawesome-team/capacitor-vault
npx cap sync
```

## Configuration

### Android

#### Proguard

If using Proguard, add to `android/app/proguard-rules.pro`:

```
-keep class io.capawesome.capacitorjs.plugins.** { *; }
```

#### Variables

Optionally define in `android/variables.gradle`:

- `$androidxBiometricVersion` version of `androidx.biometric:biometric` (default: `1.1.0`)
- `$androidxLifecycleProcessVersion` version of `androidx.lifecycle:lifecycle-process` (default: `2.9.4`)

> The `DEVICE_PASSCODE` and `BIOMETRIC_OR_DEVICE_PASSCODE` vault types require Android API 30+. `BIOMETRIC` works on all supported versions.

### iOS

#### Privacy Descriptions

Add to `ios/App/App/Info.plist`:

```xml
<key>NSFaceIDUsageDescription</key>
<string>This app uses Face ID to unlock your data.</string>
```

## Usage

### Initialize the vault

Call `initialize()` once per session before any other method.

```typescript
import { Vault, VaultType } from '@capawesome-team/capacitor-vault';

await Vault.initialize({
  type: VaultType.Biometric,
  title: 'Unlock vault',
  cancelButtonText: 'Cancel',
  iosFallbackButtonText: 'Use Passcode',
  lockAfterBackgrounded: 30000,
});
```

`InitializeOptions` fields: `type` (required `VaultType`), `title`, `subtitle`, `cancelButtonText`, `iosFallbackButtonText`, `lockAfterBackgrounded` (milliseconds), `invalidateOnBiometricEnrollment` (default `false`), `vaultId` (default `'default'`).

### Unlock and store/retrieve values

```typescript
import { Vault, ErrorCode } from '@capawesome-team/capacitor-vault';

try {
  await Vault.unlock();
} catch (error) {
  if (error.code === ErrorCode.UnlockCanceled) {
    console.log('User canceled authentication.');
  }
}

await Vault.setValue({ key: 'token', value: 'abc123' });
const { value } = await Vault.getValue({ key: 'token' });
```

### Manage values and lock state

```typescript
import { Vault } from '@capawesome-team/capacitor-vault';

const { keys } = await Vault.getKeys();
await Vault.removeValue({ key: 'token' });
await Vault.clear();

await Vault.lock();
const { locked } = await Vault.isLocked();
const { empty } = await Vault.isEmpty();
const { exists } = await Vault.exists();
await Vault.destroy();
```

### Export / import data

```typescript
import { Vault } from '@capawesome-team/capacitor-vault';

const { data } = await Vault.exportData();
await Vault.importData({ data });
```

### Listen for lock / unlock events

```typescript
import { Vault } from '@capawesome-team/capacitor-vault';

await Vault.addListener('lock', ({ vaultId, trigger }) => {
  // trigger is LockTrigger.Manual or LockTrigger.Timeout
  console.log(`Vault ${vaultId} locked: ${trigger}`);
});

await Vault.addListener('unlock', ({ vaultId }) => {
  console.log(`Vault ${vaultId} unlocked`);
});

await Vault.removeAllListeners();
```

## Enums

- **`VaultType`**: `Biometric` (`'BIOMETRIC'`), `DevicePasscode` (`'DEVICE_PASSCODE'`), `BiometricOrDevicePasscode` (`'BIOMETRIC_OR_DEVICE_PASSCODE'`)
- **`LockTrigger`**: `Manual` (`'MANUAL'`), `Timeout` (`'TIMEOUT'`)
- **`ErrorCode`**: includes `UnlockCanceled`, `KeyInvalidated`, and others for handling authentication failures.

## Notes

- The vault must be unlocked before calling `setValue`, `getValue`, `removeValue`, `getKeys`, or `clear`.
- Multiple independent vaults are supported via the `vaultId` option.
- `lockAfterBackgrounded` auto-locks the vault a configurable number of milliseconds after the app is backgrounded.
- `invalidateOnBiometricEnrollment` invalidates the encryption key when the device's enrolled biometrics change.
- On Android, encryption keys live in the Android Keystore and encrypted values in SharedPreferences. On iOS, both are stored in the Keychain with `*ThisDeviceOnly` accessibility, so they are excluded from cloud backups.
- On Web, values are stored unencrypted in `localStorage`. This is for development purposes only and should not be used in production.
- A drop-in replacement for the discontinued Ionic Identity Vault. Pair with the Biometrics plugin for standalone biometric availability/enrollment checks.
