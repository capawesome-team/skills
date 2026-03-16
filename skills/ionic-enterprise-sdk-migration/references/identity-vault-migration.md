# Identity Vault → Capawesome Biometrics + Secure Preferences

Migrate from `@ionic-enterprise/identity-vault` to `@capawesome-team/capacitor-biometrics` and `@capawesome-team/capacitor-secure-preferences`.

## Feature Mapping

| Feature                      | Identity Vault           | Capawesome                                                   |
| ---------------------------- | ------------------------ | ------------------------------------------------------------ |
| Biometric authentication     | `Vault.unlock()`         | `Biometrics.authenticate(...)`                               |
| Store values                 | `Vault.setValue(...)`    | `SecurePreferences.set(...)`                                 |
| Retrieve values              | `Vault.getValue(...)`    | `SecurePreferences.get(...)`                                 |
| Remove values                | `Vault.removeValue(...)` | `SecurePreferences.remove(...)`                              |
| List keys                    | `Vault.getKeys()`        | `SecurePreferences.keys()`                                   |
| Clear all data               | `Vault.clear()`          | `SecurePreferences.clear()`                                  |
| Check biometric availability | Device API               | `Biometrics.isAvailable()`                                   |
| Check biometric enrollment   | Device API               | `Biometrics.isEnrolled()`                                    |
| Device credential fallback   | Vault config             | `authenticate(...)` with `allowDeviceCredential: true`       |
| Auto-lock on timeout         | Built-in                 | Application logic (see session management section below)     |
| Custom passcode              | Built-in vault type      | Application logic                                            |
| Lock/unlock events           | `onLock` / `onUnlock`    | Application logic                                            |

## Key Differences

- **Two plugins instead of one**: Identity Vault bundles biometric auth, encrypted storage, and session management into a single class. Capawesome separates these into Biometrics (authentication) and Secure Preferences (storage), giving more flexibility.
- **No vault object**: Identity Vault uses a `Vault` instance with config. Capawesome plugins are called directly as static methods.
- **Session management is manual**: Identity Vault has built-in auto-lock, timeout, and lock/unlock events. With Capawesome, implement this using the `@capacitor/app` plugin's `appStateChange` listener (example below).

## Step-by-Step Migration

### 1. Remove Identity Vault and install the Capawesome plugins

```bash
npm uninstall @ionic-enterprise/identity-vault
```

Install the Capawesome Biometrics plugin and Secure Preferences plugin using the `capacitor-plugins` skill or follow the installation instructions in the reference files `references/capawesome-biometrics.md` and `references/capawesome-secure-preferences.md` in the `capacitor-plugins` skill.

### 2. Replace biometric authentication

**Before:**

```typescript
import { Vault, DeviceSecurityType, VaultType } from '@ionic-enterprise/identity-vault';

const vault = new Vault({
  key: 'com.example.vault',
  type: VaultType.DeviceSecurity,
  deviceSecurityType: DeviceSecurityType.Both,
  lockAfterBackgrounded: 2000,
});

await vault.unlock();
```

**After:**

```typescript
import { Biometrics } from '@capawesome-team/capacitor-biometrics';

await Biometrics.authenticate({
  title: 'Authenticate',
  subtitle: 'Verify your identity to continue',
  allowDeviceCredential: true,
});
```

### 3. Replace value storage

**Before:**

```typescript
import { Vault } from '@ionic-enterprise/identity-vault';

await vault.setValue('session_token', 'eyJhbGciOiJIUzI1NiIs...');
```

**After:**

```typescript
import { SecurePreferences } from '@capawesome-team/capacitor-secure-preferences';

await SecurePreferences.set({
  key: 'session_token',
  value: 'eyJhbGciOiJIUzI1NiIs...',
});
```

### 4. Replace value retrieval

**Before:**

```typescript
import { Vault } from '@ionic-enterprise/identity-vault';

const token = await vault.getValue('session_token');
```

**After:**

```typescript
import { SecurePreferences } from '@capawesome-team/capacitor-secure-preferences';

const { value } = await SecurePreferences.get({ key: 'session_token' });
```

### 5. Replace value removal

**Before:**

```typescript
import { Vault } from '@ionic-enterprise/identity-vault';

await vault.removeValue('session_token');
```

**After:**

```typescript
import { SecurePreferences } from '@capawesome-team/capacitor-secure-preferences';

await SecurePreferences.remove({ key: 'session_token' });
```

### 6. Replace clear all

**Before:**

```typescript
import { Vault } from '@ionic-enterprise/identity-vault';

await vault.clear();
```

**After:**

```typescript
import { SecurePreferences } from '@capawesome-team/capacitor-secure-preferences';

await SecurePreferences.clear();
```

### 7. Implement session management (replaces auto-lock)

Identity Vault's built-in `lockAfterBackgrounded` and lock/unlock events must be rebuilt using `@capacitor/app`:

```typescript
import { App } from '@capacitor/app';
import { Biometrics } from '@capawesome-team/capacitor-biometrics';
import { SecurePreferences } from '@capawesome-team/capacitor-secure-preferences';

let locked = false;

App.addListener('appStateChange', async ({ isActive }) => {
  if (!isActive) {
    locked = true;
  }
  if (isActive && locked) {
    try {
      await Biometrics.authenticate({
        title: 'Welcome back',
        subtitle: 'Authenticate to unlock',
        allowDeviceCredential: true,
      });
      locked = false;
    } catch (error) {
      // Authentication failed — keep locked or sign out
    }
  }
});
```

Adapt the logic to match the previous `lockAfterBackgrounded` timeout and any custom behavior.
