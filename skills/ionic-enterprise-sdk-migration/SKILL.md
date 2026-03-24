---
name: ionic-enterprise-sdk-migration
description: "Guides the agent through migrating Capacitor apps from discontinued Ionic Enterprise SDK plugins (Auth Connect, Identity Vault, Secure Storage) to their Capawesome alternatives (OAuth, Biometrics, Secure Preferences, SQLite). Covers dependency detection, side-by-side API mapping, code replacement, and platform-specific configuration for each plugin pair. Do not use for migrating Capacitor apps or plugins to a newer version, setting up Capawesome Cloud, or non-Capacitor mobile frameworks."
metadata:
  author: capawesome-team
  source: https://github.com/capawesome-team/skills/tree/main/skills/ionic-enterprise-sdk-migration
---

# Ionic Enterprise SDK Migration

Migrate Capacitor apps from discontinued Ionic Enterprise SDK plugins to Capawesome alternatives.

## Plugin Mapping

| Ionic Enterprise Plugin | Package | Capawesome Replacement | Package(s) |
| ----------------------- | ------- | ---------------------- | ---------- |
| Auth Connect | `@ionic-enterprise/auth` | OAuth | `@capawesome-team/capacitor-oauth` |
| Identity Vault | `@ionic-enterprise/identity-vault` | Biometrics + Secure Preferences | `@capawesome-team/capacitor-biometrics` + `@capawesome-team/capacitor-secure-preferences` |
| Secure Storage (key-value) | `@ionic-enterprise/secure-storage` | Secure Preferences | `@capawesome-team/capacitor-secure-preferences` |
| Secure Storage (SQLite) | `@ionic-enterprise/secure-storage` | SQLite | `@capawesome-team/capacitor-sqlite` |

All Capawesome replacement plugins require a [Capawesome Insiders](https://capawesome.io/insiders/) license.

## Prerequisites

1. **Capacitor 6, 7, or 8** app.
2. Node.js and npm installed.
3. A [Capawesome Insiders](https://capawesome.io/insiders/) license key.

## Agent Behavior

- **Auto-detect dependencies.** Scan `package.json` for Ionic Enterprise packages before asking the user which plugins to migrate.
- **One plugin at a time.** Complete the full migration for one plugin before starting the next.
- **Use the `capacitor-plugins` skill for installation.** Delegate plugin installation and platform configuration to the `capacitor-plugins` skill. Only handle migration-specific steps (uninstall, code replacement) in this skill.
- **Preserve existing behavior.** When replacing API calls, maintain the same functional behavior (e.g., same scopes, same stored keys, same database schema).

## Procedures

### Step 1: Detect Ionic Enterprise Dependencies

Read `package.json` and check for these packages:

- `@ionic-enterprise/auth` → Auth Connect
- `@ionic-enterprise/identity-vault` → Identity Vault
- `@ionic-enterprise/secure-storage` → Secure Storage

If none are found, inform the user that no Ionic Enterprise plugins were detected.

If multiple are found, list them and ask the user which to migrate first, or migrate all sequentially.

### Step 2: Set Up the Capawesome npm Registry

Check if the `@capawesome-team` npm registry is already configured:

```bash
npm config get @capawesome-team:registry
```

If not configured, guide the user through setup:

```bash
npm config set @capawesome-team:registry https://npm.registry.capawesome.io
npm config set //npm.registry.capawesome.io/:_authToken <YOUR_LICENSE_KEY>
```

Ask the user for their license key if needed. Wait for confirmation before continuing.

### Step 3: Migrate Each Plugin

For each detected Ionic Enterprise plugin, read the corresponding reference file and follow the migration steps:

- **Auth Connect** → Read `references/auth-connect-migration.md`
- **Identity Vault** → Read `references/identity-vault-migration.md`
- **Secure Storage** → Read `references/secure-storage-migration.md`

Each reference file contains:

1. Feature mapping table (Ionic API → Capawesome API)
2. Key differences to be aware of
3. Step-by-step code replacement with before/after examples

For plugin installation and platform-specific configuration (Android permissions, iOS plist entries, Proguard rules, etc.), use the `capacitor-plugins` skill.

### Step 4: Search for Remaining Ionic Enterprise Imports

After completing all migrations, search the codebase for any remaining Ionic Enterprise imports:

```bash
grep -r "@ionic-enterprise" --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx"
```

If any are found, replace them with the corresponding Capawesome imports.

### Step 5: Clean Up and Verify

1. Remove all Ionic Enterprise packages from `package.json`:

```bash
npm uninstall @ionic-enterprise/auth @ionic-enterprise/identity-vault @ionic-enterprise/secure-storage
```

Only uninstall packages that were actually installed.

2. Sync the project:

```bash
npx cap sync
```

3. Verify the project builds successfully on all platforms.

## Error Handling

- **Capawesome registry not configured**: If `npm install` fails with a 404 or authentication error for `@capawesome-team/*` packages, verify the npm registry is configured correctly (Step 2).
- **Missing Capawesome Insiders license**: All replacement plugins require a Capawesome Insiders license. Direct the user to [capawesome.io/insiders](https://capawesome.io/insiders/) to obtain one.
- **Identity Vault session management**: Identity Vault's built-in auto-lock, timeout, and lock/unlock events have no direct equivalent. These must be rebuilt using application logic. See the session management section in `references/identity-vault-migration.md`.
- **Secure Storage encryption**: Ionic Secure Storage has built-in encryption. Capawesome SQLite encryption requires additional platform configuration (SQLCipher). If the user needs database encryption, guide them through the SQLite encryption setup in the `capacitor-plugins` skill reference.
- **Web platform limitations**: Capawesome Secure Preferences stores values unencrypted in `localStorage` on the web. This is for development only and should not be used in production.

## Related Skills

- **`capacitor-plugins`** — Referenced throughout this skill for plugin installation and platform configuration.
- **`ionic-appflow-migration`** — If the project also uses Ionic Appflow (Live Updates, Native Builds, App Store Publishing), use this skill to migrate to Capawesome Cloud.
