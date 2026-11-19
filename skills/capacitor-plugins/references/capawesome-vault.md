# Vault

Store secrets behind biometric, device-passcode, or app-passcode authentication using hardware-backed encryption (Android Keystore, iOS Keychain). Unlock once, then perform many read/write operations until the vault locks again.

**Package:** `@capawesome-team/capacitor-vault`
**Platforms:** Android, iOS, Web
**Documentation:** https://capawesome.io/docs/sdks/capacitor/vault/
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

> The `DEVICE_PASSCODE` and `BIOMETRIC_OR_DEVICE_PASSCODE` vault types require Android API 30+. On lower versions, `initialize()` rejects with `AUTHENTICATOR_UNAVAILABLE`. `BIOMETRIC` and `CUSTOM_PASSCODE` work on all supported versions, and `CUSTOM_PASSCODE` does not need a screen lock.

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
import { Vault, VaultType, ErrorCode } from '@capawesome-team/capacitor-vault';

const options = {
  type: VaultType.Biometric,
  title: 'Unlock vault',
  cancelButtonText: 'Cancel',
  lockAfterBackgrounded: 30000,
};

const initialize = async () => {
  try {
    await Vault.initialize(options);
  } catch (error) {
    if (error.code === ErrorCode.AuthenticatorUnavailable) {
      // No biometric or device passcode set up.
    } else if (error.code === ErrorCode.KeyInvalidated) {
      // All values are lost. Start over.
      await Vault.destroy();
      await Vault.initialize(options);
    } else {
      throw error;
    }
  }
};
```

`InitializeOptions` fields: `type` (required `VaultType`), `title`, `subtitle`, `cancelButtonText`, `iosFallbackButtonText` (iOS only, ignored for `BIOMETRIC` vaults, which show no fallback button), `lockAfterBackgrounded` (milliseconds), `invalidateOnBiometricEnrollment` (default `false`), `vaultId` (default `'default'`).

`type` and `invalidateOnBiometricEnrollment` are fixed when the vault is created and cannot be changed later. See the `CUSTOM_PASSCODE` rules below for how a different `type` is handled.

### Unlock and store/retrieve values

```typescript
import { Vault, ErrorCode } from '@capawesome-team/capacitor-vault';

try {
  await Vault.unlock();
} catch (error) {
  if (error.code === ErrorCode.UnlockCanceled) {
    console.log('User canceled authentication.');
  } else if (error.code === ErrorCode.AuthenticatorLockedOut) {
    console.log('Biometrics are locked until the device is unlocked with its passcode.');
  } else if (error.code === ErrorCode.KeyInvalidated) {
    // All values are lost. Call Vault.destroy() and initialize the vault again.
  } else {
    throw error;
  }
}

await Vault.setValue({ key: 'token', value: 'abc123' });

try {
  const { value } = await Vault.getValue({ key: 'token' });
} catch (error) {
  if (error.code === ErrorCode.DecryptionFailed) {
    // Only this value is unreadable. Remove it.
    await Vault.removeValue({ key: 'token' });
  } else {
    throw error;
  }
}
```

### Use a custom passcode

A `CUSTOM_PASSCODE` vault is unlocked with a passcode that the app sets. It works without a device passcode or biometrics and on every supported Android version. The plugin never shows a prompt for this type. The app must build its own setup and unlock screens and pass the passcode in. Use `exists()` to decide which screen to show.

```typescript
import { Vault, VaultType, ErrorCode } from '@capawesome-team/capacitor-vault';

// Call once per session before any other method.
const initialize = async () => {
  await Vault.initialize({
    type: VaultType.CustomPasscode,
    lockAfterBackgrounded: 900000,
  });
};

// Show the setup screen if this returns false, otherwise the unlock screen.
const isSetUp = async () => {
  const { exists } = await Vault.exists();
  return exists;
};

// Create the vault with the passcode from the setup screen.
// The vault is unlocked afterwards. Three wrong passcodes in a row delete it.
const setPasscode = async (passcode: string) => {
  await Vault.setPasscode({ passcode, failedUnlockCountLimit: 3 });
};

// Verify the passcode from the unlock screen.
const unlock = async (passcode: string) => {
  try {
    await Vault.unlock({ passcode });
  } catch (error) {
    if (error.code === ErrorCode.PasscodeInvalid) {
      // remainingAttemptCount is only set when a limit was configured.
      console.log(`Attempts left: ${error.data?.remainingAttemptCount}`);
    } else if (error.code === ErrorCode.PasscodeLimitReached) {
      // The vault and all values were deleted. Show the setup screen again.
    } else {
      throw error;
    }
  }
};

// Change the passcode. The stored values are kept.
const changePasscode = async (oldPasscode: string, newPasscode: string) => {
  await Vault.unlock({ passcode: oldPasscode });
  await Vault.setPasscode({ passcode: newPasscode, failedUnlockCountLimit: 3 });
};
```

Rules for `CUSTOM_PASSCODE` vaults:

- `failedUnlockCountLimit` must be at least `1`. If omitted, failed attempts are counted but the vault is never deleted. Omitting it on a later `setPasscode()` call removes an existing limit. The counter survives app restarts and resets after a correct passcode.
- The prompt options of `initialize()` (`title`, `subtitle`, `cancelButtonText`, `iosFallbackButtonText`) are ignored.
- A passcode passed to an already unlocked vault is verified too. A wrong one counts toward the limit.
- The plugin enforces no passcode policy, so the app decides length and format. A short passcode protects less than the device credential. Set `failedUnlockCountLimit` and keep `lockAfterBackgrounded` short.
- Once its passcode is set, a `CUSTOM_PASSCODE` vault rejects `initialize()` with any other type. A vault of another type rejects `CUSTOM_PASSCODE` once it holds values.
- The iOS simulator accepts any passcode. Test the wrong-passcode handling on a real device.

### Manage values and lock state

```typescript
import { Vault } from '@capawesome-team/capacitor-vault';

const { keys } = await Vault.getKeys();
await Vault.removeValue({ key: 'token' });
await Vault.clear();

await Vault.lock();
const { isLocked } = await Vault.isLocked();
const { isEmpty } = await Vault.isEmpty();
const { exists } = await Vault.exists();
await Vault.destroy();
```

### Export / import data

The vault must be unlocked before calling `exportData()` or `importData()`:

```typescript
import { Vault } from '@capawesome-team/capacitor-vault';

await Vault.unlock();
const { data, skippedCount } = await Vault.exportData();
await Vault.importData({ data });
```

`exportData()` leaves out values that cannot be decrypted and reports their number in `skippedCount`. `importData()` replaces all existing values.

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

- **`VaultType`**: `Biometric` (`'BIOMETRIC'`), `DevicePasscode` (`'DEVICE_PASSCODE'`), `BiometricOrDevicePasscode` (`'BIOMETRIC_OR_DEVICE_PASSCODE'`), `CustomPasscode` (`'CUSTOM_PASSCODE'`)
- **`LockTrigger`**: `Manual` (`'MANUAL'`), `Timeout` (`'TIMEOUT'`)
- **`ErrorCode`**:
  - `AuthenticatorLockedOut` (`'AUTHENTICATOR_LOCKED_OUT'`): biometrics are locked after too many failed attempts. Only `BIOMETRIC` vaults are affected. Reported by `unlock()`, and on iOS also by an `initialize()` that creates a new vault. The lock ends when the user unlocks the device with its passcode, on Android also after a short time. The vault stays usable.
  - `AuthenticatorUnavailable` (`'AUTHENTICATOR_UNAVAILABLE'`): the authentication method the vault type needs is missing, for example no enrolled biometric, no device passcode, or Android below API 30.
  - `DecryptionFailed` (`'DECRYPTION_FAILED'`): one stored value cannot be decrypted. Other values stay readable.
  - `KeyInvalidated` (`'KEY_INVALIDATED'`): the encryption key is gone or can no longer be used. Call `destroy()` and initialize the vault again.
  - `PasscodeInvalid` (`'PASSCODE_INVALID'`): the passcode passed to `unlock()` is wrong. `error.data.remainingAttemptCount` is set when a limit is configured.
  - `PasscodeLimitReached` (`'PASSCODE_LIMIT_REACHED'`): `failedUnlockCountLimit` was reached. The key and all values were deleted. Call `setPasscode()` to start over.
  - `UnlockCanceled` (`'UNLOCK_CANCELED'`): the user canceled or failed the authentication prompt. The caller may retry `unlock()`.
  - `VaultLocked` (`'VAULT_LOCKED'`): the vault must be unlocked before accessing values.
  - `VaultNotFound` (`'VAULT_NOT_FOUND'`): `initialize()` was not called in the current session.

## Notes

- The vault must be unlocked before calling `setValue`, `getValue`, `removeValue`, `getKeys`, `clear`, `exportData`, or `importData`.
- Multiple independent vaults are supported via the `vaultId` option.
- `lockAfterBackgrounded` auto-locks the vault a configurable number of milliseconds after the app is backgrounded.
- `invalidateOnBiometricEnrollment` invalidates the encryption key when the device's enrolled biometrics change.
- On Android, encryption keys live in the Android Keystore and encrypted values in SharedPreferences. On iOS, both are stored in the Keychain with `*ThisDeviceOnly` accessibility, so they are never synced to iCloud Keychain.
- On Android, uninstalling the app deletes all vaults. On iOS, Keychain items survive an uninstall and are available again after a reinstall with the same bundle identifier. To start with empty vaults after a reinstall, keep a marker in `@capacitor/preferences` and call `Vault.destroy({ vaultId })` for every vault on the first launch when the marker is missing.
- On Web, values are stored unencrypted in `localStorage`, and the passcode of a `CUSTOM_PASSCODE` vault is stored as plain text. This is for development purposes only and should not be used in production.
- `CustomPasscode`, `setPasscode()`, `PasscodeInvalid` and `PasscodeLimitReached` need version 0.2.2 or later. `AuthenticatorLockedOut` needs 0.2.4 or later.
- A near drop-in replacement for the discontinued Ionic Identity Vault. Version 0.2.2 or later can be installed next to Identity Vault, which a runtime data migration needs. Pair with the Biometrics plugin for standalone biometric availability/enrollment checks.
