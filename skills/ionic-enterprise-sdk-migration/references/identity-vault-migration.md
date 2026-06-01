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
| Check lock state             | `vault.isLocked()`              | `Vault.isLocked()` → `{ locked }`                      |
| Check if empty               | `vault.isEmpty()`               | `Vault.isEmpty()` → `{ empty }`                        |
| Check if vault exists        | `vault.doesVaultExist()`        | `Vault.exists()` → `{ exists }`                        |
| Store value                  | `vault.setValue(key, value)`    | `Vault.setValue({ key, value })`                       |
| Retrieve value               | `vault.getValue(key)`           | `Vault.getValue({ key })` → `{ value }`                |
| Remove value                 | `vault.removeValue(key)`        | `Vault.removeValue({ key })`                           |
| List keys                    | `vault.getKeys()`               | `Vault.getKeys()` → `{ keys }`                         |
| Clear all values             | `vault.clear()`                 | `Vault.clear()`                                        |
| Export data                 | `vault.exportVault()`           | `Vault.exportData()` → `{ data }`                      |
| Import data                  | `vault.importVault(data)`       | `Vault.importData({ data })`                           |
| Auto-lock after background    | `lockAfterBackgrounded` config  | `lockAfterBackgrounded` option (ms)                    |
| Lock event                   | `vault.onLock(cb)`              | `Vault.addListener('lock', cb)`                        |
| Unlock event                 | `vault.onUnlock(cb)`            | `Vault.addListener('unlock', cb)`                      |
| Update configuration         | `vault.updateConfig(config)`    | Re-call `Vault.initialize(options)`                    |
| Custom passcode              | `setCustomPasscode` / `onPasscodeRequested` | Application logic (no direct equivalent)   |

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

## Key Differences

- **No vault instance**: Identity Vault uses a `Vault` instance created with `new Vault(...)`. The Capawesome plugin is a singleton called via static methods after a one-time `Vault.initialize(...)`.
- **Options objects**: Identity Vault passes positional arguments (`setValue(key, value)`); the Capawesome plugin uses options objects (`setValue({ key, value })`) and returns results wrapped in objects (`getValue` → `{ value }`).
- **String values only**: `Vault.setValue` stores strings. Identity Vault's `setValue<T>` accepts any serializable value. Serialize non-string values with `JSON.stringify` on write and `JSON.parse` on read.
- **Built-in session management**: Auto-lock (`lockAfterBackgrounded`) and `lock`/`unlock` events are supported natively — no need to rebuild them with `@capacitor/app`.
- **Custom passcode vaults**: Identity Vault's custom-passcode flow (`setCustomPasscode`, `onPasscodeRequested`) has no direct equivalent and must be rebuilt with application logic if used.
- **Two plugins only when needed**: Add the Biometrics plugin only if the app uses the Identity Vault `Device` class. For pure vault storage/unlock, the Vault plugin alone is sufficient.

## Choose a Migration Strategy First

Before changing any code, ask the user which strategy they want. The answer determines whether step 2 runs and when Identity Vault is uninstalled.

- **Preserve existing data (gradual cutover).** Carry users' stored values over to the new vault. Both plugins stay installed for a transition period: you ship a release that runs the runtime data migration (step 2), let users open the app so the migration runs on their devices, then remove Identity Vault in a later release (step 9). Choose this when the vault holds data users must not lose — long-lived session tokens, credentials, or anything the app cannot simply re-fetch.

- **Hard cutover (clean break).** Drop the old data entirely and switch straight to the Capawesome Vault. Identity Vault is uninstalled immediately and no data is carried over, so users start with an empty vault and must re-authenticate or re-enter anything that was stored. Choose this when the stored data is disposable — e.g. a cache, or a token the app can transparently obtain again on next login.

Confirm the choice, then follow the steps below. **Skip step 2 entirely for a hard cutover.**

## Step-by-Step Migration

### 1. Install the Capawesome plugin(s)

Install the Capawesome Vault plugin (and the Biometrics plugin if the `Device` class is used) via the `capacitor-plugins` skill, or follow the installation instructions in `references/capawesome-vault.md` and `references/capawesome-biometrics.md` in the `capacitor-plugins` skill.

- **Preserve existing data:** keep `@ionic-enterprise/identity-vault` installed for now — it is needed to export the old data in step 2 and is removed later in step 9.
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

  // 4. Initialize the new vault and import the exported data.
  await Vault.initialize({
    vaultId: 'com.example.vault',
    type: VaultType.BiometricOrDevicePasscode,
    title: 'Authenticate',
  });
  await Vault.importData({ data });

  // 5. Clear the old vault and mark the migration complete.
  await legacyVault.clear();
  localStorage.setItem(MIGRATION_FLAG, 'true');
}
```

Match the `key`/`vaultId`, `type`, and `deviceSecurityType` to the app's original Identity Vault configuration. Run `migrateIdentityVaultData()` early in app startup, before any code reads from the new vault.

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
  vaultId: 'com.example.vault',
  type: VaultType.BiometricOrDevicePasscode,
  title: 'Authenticate',
  subtitle: 'Verify your identity to continue',
  lockAfterBackgrounded: 2000,
});
```

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
  }
}

await Vault.lock();
const { locked } = await Vault.isLocked();
```

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
