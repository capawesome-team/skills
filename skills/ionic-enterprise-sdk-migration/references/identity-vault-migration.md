# Identity Vault → Capawesome Vault (+ Biometrics)

Migrate from `@ionic-enterprise/identity-vault` to `@capawesome-team/capacitor-vault`. The Capawesome Vault plugin is a near drop-in replacement for Identity Vault: it combines biometric/passcode-gated unlock, hardware-backed key/value storage, and auto-lock with `lock`/`unlock` events in a single API.

Use `@capawesome-team/capacitor-biometrics` in addition only when the app calls Identity Vault's `Device` class directly to inspect biometric hardware, enrollment, or lockout state.

## Feature Mapping

### Vault class → Vault plugin

| Feature                      | Identity Vault (`Vault`)        | Capawesome (`Vault`)                                   |
| ---------------------------- | ------------------------------- | ------------------------------------------------------ |
| Configure / initialize       | `new Vault(config)` / `initialize(config)` | `Vault.initialize(options)`                 |
| Unlock                       | `vault.unlock()`                | `Vault.unlock()`                                       |
| Lock                         | `vault.lock()`                  | `Vault.lock()`                                         |
| Check lock state             | `vault.isLocked()`              | `Vault.isLocked()` → `{ isLocked }`                    |
| Check if empty               | `vault.isEmpty()`               | `Vault.isEmpty()` → `{ isEmpty }`                      |
| Check if vault exists        | `vault.doesVaultExist()`        | `Vault.exists()` → `{ exists }`                        |
| Store value                  | `vault.setValue(key, value)`    | `Vault.setValue({ key, value })`                       |
| Retrieve value               | `vault.getValue(key)`           | `Vault.getValue({ key })` → `{ value }`                |
| Remove value                 | `vault.removeValue(key)`        | `Vault.removeValue({ key })`                           |
| List keys                    | `vault.getKeys()`               | `Vault.getKeys()` → `{ keys }`                         |
| Clear all values             | `vault.clear()`                 | `Vault.clear()`                                        |
| Destroy vault (values + config) | *(no direct equivalent — `clear()` removes values only)* | `Vault.destroy()` — must reinitialize before reuse |
| Export data                 | `vault.exportVault()`           | `Vault.exportData()` → `{ data }`                      |
| Import data                  | `vault.importVault(data)`       | `Vault.importData({ data })`                           |
| Auto-lock after background    | `lockAfterBackgrounded` config  | `lockAfterBackgrounded` option (ms)                    |
| Lock event                   | `vault.onLock(cb)`              | `Vault.addListener('lock', cb)`                        |
| Unlock event                 | `vault.onUnlock(cb)`            | `Vault.addListener('unlock', cb)`                      |
| Update configuration         | `vault.updateConfig(config)`    | Re-call `Vault.initialize(options)`. `type` is fixed at creation and cannot change. Switching to or from `CustomPasscode` rejects with `VaultTypeMismatch` (0.2.5 or later) once the vault holds values or a passcode. |
| Set custom passcode          | `vault.setCustomPasscode(passcode)` in `onPasscodeRequested` | `Vault.setPasscode({ passcode })` |
| Unlock with custom passcode  | `vault.unlock()` → `onPasscodeRequested` | `Vault.unlock({ passcode })`                  |
| Failed passcode limit        | `shouldClearVaultAfterTooManyFailedAttempts: true` + `customPasscodeInvalidUnlockAttempts` (default `5`) | `failedUnlockCountLimit` option of `Vault.setPasscode` |

### Device class → Biometrics plugin

| Feature                       | Identity Vault (`Device`)        | Capawesome (`Biometrics`)                              |
| ----------------------------- | -------------------------------- | ------------------------------------------------------ |
| Biometrics supported          | `Device.isBiometricsSupported()` | `Biometrics.isAvailable()` → `{ isAvailable }`         |
| Biometrics enabled/enrolled   | `Device.isBiometricsEnabled()`   | `Biometrics.isEnrolled()` → `{ isEnrolled }`           |
| Biometric permission granted  | `Device.isBiometricsAllowed()`   | `Biometrics.isAllowed()` → `{ isAllowed }`             |
| Locked out of biometrics      | `Device.isLockedOutOfBiometrics()` | `Biometrics.isLockedOut()` → `{ isLockedOut }`       |
| System passcode set           | `Device.isSystemPasscodeSet()`   | `Biometrics.hasDeviceCredential()` → `{ hasDeviceCredential }` |
| Available hardware            | `Device.getAvailableHardware()`  | `Biometrics.getBiometricTypes()` → `{ types }`         |
| Biometric strength level      | `Device.getBiometricStrengthLevel()` | `Biometrics.getBiometricStrengthLevel()` → `{ strengthLevel }` |
| Standalone biometric prompt   | `Device.showBiometricPrompt(config)` | `Biometrics.authenticate(options)`                 |

### Vault type mapping

| Identity Vault config                                            | Capawesome `VaultType`                 |
| ---------------------------------------------------------------- | -------------------------------------- |
| `VaultType.DeviceSecurity` + `DeviceSecurityType.Biometrics`     | `VaultType.Biometric`                  |
| `VaultType.DeviceSecurity` + `DeviceSecurityType.SystemPasscode` | `VaultType.DevicePasscode`             |
| `VaultType.DeviceSecurity` + `DeviceSecurityType.Both`           | `VaultType.BiometricOrDevicePasscode`  |
| `VaultType.CustomPasscode`                                       | `VaultType.CustomPasscode`             |

## Key Differences

- **No vault instance**: Identity Vault uses a `Vault` instance created with `new Vault(...)`. The Capawesome plugin is a singleton called via static methods after a one-time `Vault.initialize(...)`.
- **Options objects**: Identity Vault passes positional arguments (`setValue(key, value)`); the Capawesome plugin uses options objects (`setValue({ key, value })`) and returns results wrapped in objects (`getValue` → `{ value }`).
- **String values only**: `Vault.setValue` stores strings. Identity Vault's `setValue<T>` accepts any serializable value. Serialize non-string values with `JSON.stringify` on write and `JSON.parse` on read.
- **Built-in session management**: Auto-lock (`lockAfterBackgrounded`) and `lock`/`unlock` events are supported natively — no need to rebuild them with `@capacitor/app`.
- **Vault id**: This guide uses the `'default'` vault, so no call passes `vaultId`. If the app uses more than one Identity Vault, use each legacy `key` as the `vaultId` and pass that `vaultId` to every `Vault` call for that vault. A call without `vaultId` targets the `'default'` vault.
- **Custom passcode vaults**: The Capawesome plugin never asks for the passcode. The app passes it in. See step 4.
- **Two plugins only when needed**: Add the Biometrics plugin only if the app uses the Identity Vault `Device` class. For pure vault storage/unlock, the Vault plugin alone is sufficient.

## Choose a Migration Strategy First

Before changing any code, ask the user which strategy they want. The answer determines whether step 2 runs and when Identity Vault is uninstalled.

- **Preserve existing data (gradual cutover).** Carry users' stored values over to the new vault. Both plugins stay installed for a transition period: you ship a release that runs the runtime data migration (step 2), let users open the app so the migration runs on their devices, then remove Identity Vault in a later release (step 9). Choose this when the vault holds data users must not lose — long-lived session tokens, credentials, or anything the app cannot simply re-fetch.

- **Hard cutover (clean break).** Drop the old data entirely and switch straight to the Capawesome Vault. Identity Vault is uninstalled immediately and no data is carried over, so users start with an empty vault and must re-authenticate or re-enter anything that was stored. Choose this when the stored data is disposable — e.g. a cache, or a token the app can transparently obtain again on next login.

Confirm the choice, then follow the steps below. **Skip step 2 entirely for a hard cutover.**

## Step-by-Step Migration

### 1. Install the Capawesome plugin(s)

Install the Capawesome Vault plugin (and the Biometrics plugin if the `Device` class is used) via the `capacitor-plugins` skill, or follow the installation instructions in `references/capawesome-vault.md` and `references/capawesome-biometrics.md` in the `capacitor-plugins` skill.

- **Preserve existing data:** keep `@ionic-enterprise/identity-vault` installed for now. It is needed to export the old data in step 2 and is removed later in step 9. Install `@capawesome-team/capacitor-vault` version 0.2.2 or later. Earlier versions crash the app at launch on iOS when Identity Vault is installed too.
- **Hard cutover:** uninstall Identity Vault now and skip step 2:

  ```bash
  npm uninstall @ionic-enterprise/identity-vault
  ```

### 2. Migrate existing stored data (preserve-data strategy only)

> Skip this step for a hard cutover.

Migrate the data once at runtime while both plugins are installed. Identity Vault's `exportVault()` returns a key/value map that is directly compatible with the Capawesome Vault's `importData()`. The user must authenticate once to unlock the old vault — this is unavoidable, since the data is protected by the device's biometric/passcode authentication by design.

```typescript
import {
  Vault as IdentityVault,
  VaultType as IdentityVaultType,
  DeviceSecurityType,
} from '@ionic-enterprise/identity-vault';
import { Vault, VaultType } from '@capawesome-team/capacitor-vault';

const MIGRATION_FLAG = 'identity-vault-migrated';

async function migrateIdentityVaultData(): Promise<void> {
  // 1. Skip if the migration already ran on this device.
  if (localStorage.getItem(MIGRATION_FLAG)) {
    return;
  }

  const legacyVault = new IdentityVault({
    key: 'com.example.vault',
    type: IdentityVaultType.DeviceSecurity,
    deviceSecurityType: DeviceSecurityType.Both,
  });

  // 2. Skip (and mark done) if there is nothing to migrate.
  if (await legacyVault.isEmpty()) {
    localStorage.setItem(MIGRATION_FLAG, 'true');
    return;
  }

  // 3. Unlock the legacy vault (prompts the user once) and export its data.
  await legacyVault.unlock();
  const data = await legacyVault.exportVault();

  // 4. Initialize the new vault, unlock it, and import the exported data.
  //    The vault must be unlocked before calling importData().
  await Vault.initialize({
    type: VaultType.BiometricOrDevicePasscode,
    title: 'Authenticate',
  });
  await Vault.unlock();
  await Vault.importData({ data });

  // 5. Clear the old vault and mark the migration complete.
  await legacyVault.clear();
  localStorage.setItem(MIGRATION_FLAG, 'true');
}
```

Match the `key`, `type`, and `deviceSecurityType` of the legacy vault to the app's original Identity Vault configuration, and map the new `type` with the vault type mapping table. Pass the same options to `Vault.initialize()` as in step 3. Run `migrateIdentityVaultData()` early in app startup, before any code reads from the new vault. If it rejects, for example because the user canceled or entered a wrong passcode, do not continue to the new vault. Run it again instead. A later successful run replaces everything stored in the new vault in the meantime.

**If the Identity Vault uses `VaultType.CustomPasscode`**, create the legacy vault with `type: IdentityVaultType.CustomPasscode` and without `deviceSecurityType`, and replace steps 3 and 4 of the function above. Keep the passcode the user enters to unlock the legacy vault, and create the new vault with the same passcode. The user keeps their passcode, and `setPasscode()` leaves the new vault unlocked for `importData()`:

```typescript
  // 3. Unlock the legacy vault with the app's existing passcode screen and keep the passcode.
  let passcode = '';
  legacyVault.onPasscodeRequested(async () => {
    passcode = await showPasscodeScreen();
    await legacyVault.setCustomPasscode(passcode);
  });
  await legacyVault.unlock();
  const data = await legacyVault.exportVault();

  // 4. Create the new vault with the same passcode and import the exported data.
  //    An earlier run may have stopped halfway, so start the new vault over.
  const { exists } = await Vault.exists();
  if (exists) {
    await Vault.destroy();
  }
  await Vault.initialize({ type: VaultType.CustomPasscode });
  await Vault.setPasscode({ passcode });
  await Vault.importData({ data });
```

Replace `showPasscodeScreen()` with the screen the app already shows in its `onPasscodeRequested` handler. If the legacy vault sets `shouldClearVaultAfterTooManyFailedAttempts: true`, also pass `failedUnlockCountLimit` to `setPasscode()` with the legacy `customPasscodeInvalidUnlockAttempts` value (default `5`).

### 3. Initialize the vault

**Before:**

```typescript
import { Vault, DeviceSecurityType, VaultType } from '@ionic-enterprise/identity-vault';

const vault = new Vault({
  key: 'com.example.vault',
  type: VaultType.DeviceSecurity,
  deviceSecurityType: DeviceSecurityType.Both,
  lockAfterBackgrounded: 2000,
});
```

**After:**

```typescript
import { Vault, VaultType } from '@capawesome-team/capacitor-vault';

await Vault.initialize({
  type: VaultType.BiometricOrDevicePasscode,
  title: 'Authenticate',
  subtitle: 'Verify your identity to continue',
  lockAfterBackgrounded: 2000,
});
```

Map `type` and `deviceSecurityType` with the vault type mapping table. For a custom passcode vault, pass `type: VaultType.CustomPasscode` and leave out the prompt options (`title`, `subtitle`), which this type ignores.

### 4. Replace unlock / lock

**Before:**

```typescript
await vault.unlock();
await vault.lock();
const locked = await vault.isLocked();
```

**After:**

```typescript
import { Vault, ErrorCode } from '@capawesome-team/capacitor-vault';

try {
  await Vault.unlock();
} catch (error) {
  if (error.code === ErrorCode.UnlockCanceled) {
    // User canceled authentication — keep locked or sign out
  } else {
    throw error;
  }
}

await Vault.lock();
const { isLocked } = await Vault.isLocked();
```

For all error codes, see `references/capawesome-vault.md` in the `capacitor-plugins` skill.

**Custom passcode vaults.** Identity Vault calls `onPasscodeRequested` during `unlock()`, and `isPasscodeSetRequest` tells whether to show the setup or the unlock screen. Replace this with an explicit `Vault.exists()` check. Reuse the app's existing setup and unlock screens. Initialize the vault as in step 3 with `type: VaultType.CustomPasscode` first.

**Before:**

```typescript
import { Vault, VaultType } from '@ionic-enterprise/identity-vault';

const vault = new Vault({
  key: 'com.example.vault',
  type: VaultType.CustomPasscode,
  shouldClearVaultAfterTooManyFailedAttempts: true,
  customPasscodeInvalidUnlockAttempts: 3,
});

vault.onPasscodeRequested(async (isPasscodeSetRequest) => {
  const passcode = isPasscodeSetRequest
    ? await showSetupScreen()
    : await showUnlockScreen();
  await vault.setCustomPasscode(passcode);
});

await vault.unlock();
```

**After:**

```typescript
import { Vault, ErrorCode } from '@capawesome-team/capacitor-vault';

const { exists } = await Vault.exists();
if (!exists) {
  // Creates the vault and leaves it unlocked.
  const passcode = await showSetupScreen();
  await Vault.setPasscode({ passcode, failedUnlockCountLimit: 3 });
} else {
  const passcode = await showUnlockScreen();
  try {
    await Vault.unlock({ passcode });
  } catch (error) {
    if (error.code === ErrorCode.PasscodeInvalid) {
      // Wrong passcode. Show the unlock screen again.
    } else if (error.code === ErrorCode.PasscodeLimitReached) {
      // The vault and all values were deleted. Show the setup screen.
    } else {
      throw error;
    }
  }
}
```

To change the passcode, replace `vault.setCustomPasscode(newPasscode)` on an unlocked vault with `Vault.setPasscode({ passcode: newPasscode, failedUnlockCountLimit: 3 })`. Pass `failedUnlockCountLimit` on every `setPasscode()` call, because omitting it removes the limit.

### 5. Replace value storage

**Before:**

```typescript
await vault.setValue('session_token', 'eyJhbGciOiJIUzI1NiIs...');
const token = await vault.getValue('session_token');
```

**After:**

```typescript
import { Vault } from '@capawesome-team/capacitor-vault';

await Vault.setValue({ key: 'session_token', value: 'eyJhbGciOiJIUzI1NiIs...' });
const { value: token } = await Vault.getValue({ key: 'session_token' });
```

For non-string values, serialize them:

```typescript
await Vault.setValue({ key: 'user', value: JSON.stringify(user) });
const { value } = await Vault.getValue({ key: 'user' });
const user = value ? JSON.parse(value) : null;
```

### 6. Replace value removal, key listing, and clear

**Before:**

```typescript
await vault.removeValue('session_token');
const keys = await vault.getKeys();
await vault.clear();
```

**After:**

```typescript
import { Vault } from '@capawesome-team/capacitor-vault';

await Vault.removeValue({ key: 'session_token' });
const { keys } = await Vault.getKeys();
await Vault.clear();
```

### 7. Replace lock/unlock event listeners

**Before:**

```typescript
vault.onLock(() => {
  // Handle lock
});
vault.onUnlock(() => {
  // Handle unlock
});
```

**After:**

```typescript
import { Vault } from '@capawesome-team/capacitor-vault';

await Vault.addListener('lock', ({ vaultId, trigger }) => {
  // trigger is LockTrigger.Manual or LockTrigger.Timeout
});
await Vault.addListener('unlock', ({ vaultId }) => {
  // Handle unlock
});
```

Auto-lock on backgrounding is handled by the `lockAfterBackgrounded` option passed to `Vault.initialize` — no `@capacitor/app` listener is required.

### 8. Replace Device class checks (if used)

Only needed if the app uses the Identity Vault `Device` class.

**Before:**

```typescript
import { Device } from '@ionic-enterprise/identity-vault';

const supported = await Device.isBiometricsSupported();
const enabled = await Device.isBiometricsEnabled();
const passcodeSet = await Device.isSystemPasscodeSet();
const lockedOut = await Device.isLockedOutOfBiometrics();
```

**After:**

```typescript
import { Biometrics } from '@capawesome-team/capacitor-biometrics';

const { isAvailable: supported } = await Biometrics.isAvailable();
const { isEnrolled: enabled } = await Biometrics.isEnrolled();
const { hasDeviceCredential: passcodeSet } = await Biometrics.hasDeviceCredential();
const { isLockedOut: lockedOut } = await Biometrics.isLockedOut();
```

### 9. Remove Identity Vault (preserve-data strategy only)

> For a hard cutover, Identity Vault was already removed in step 1 — skip this step.

Only after the data migration (step 2) has shipped and run on devices, remove the Identity Vault package and the temporary migration code:

```bash
npm uninstall @ionic-enterprise/identity-vault
```

Then delete the `migrateIdentityVaultData()` function and its imports, and run `npx cap sync`.
