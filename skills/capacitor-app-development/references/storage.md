# Storage Solutions

Choose the right storage solution for a Capacitor app based on data size, complexity, and security requirements.

## Storage Options

### 1. Capacitor Preferences API (Small Key-Value Data)

**Best for**: User settings, feature flags, small tokens, simple app state.

The `@capacitor/preferences` plugin provides a native key-value store that persists across app restarts and is not subject to OS eviction (unlike `localStorage`).

```bash
npm install @capacitor/preferences
npx cap sync
```

```typescript
import { Preferences } from '@capacitor/preferences';

// Store
await Preferences.set({ key: 'user_id', value: '12345' });

// Retrieve
const { value } = await Preferences.get({ key: 'user_id' });

// Remove
await Preferences.remove({ key: 'user_id' });

// Clear all
await Preferences.clear();
```

**Limitations**: Values must be strings. Not suitable for large datasets or complex queries.

For full plugin setup, use the `capacitor-plugins` skill.

### 2. SQLite (Large Data, Complex Queries)

**Best for**: Relational data, offline-first apps, large datasets, apps requiring complex queries.

Options:
- `@capawesome-team/capacitor-sqlite` — Capawesome SQLite plugin.
- `@capacitor-community/sqlite` — Community-maintained SQLite plugin.

Both provide full SQLite database access on Android and iOS.

For plugin installation and configuration, use the `capacitor-plugins` skill.

### 3. Secure Storage (Sensitive Data)

**Best for**: Authentication tokens, encryption keys, credentials, biometric-protected data.

Options:
- `@capawesome-team/capacitor-secure-preferences` — Encrypted key-value storage using platform Keychain (iOS) and Keystore (Android).

For plugin installation and configuration, use the `capacitor-plugins` skill.

### 4. Filesystem (Files, Binary Data)

**Best for**: Documents, images, downloads, cached files.

The `@capacitor/filesystem` plugin provides file read/write access.

For plugin installation and configuration, use the `capacitor-plugins` skill. See also `references/file-handling.md` for file handling best practices.

## What NOT to Use

### `localStorage`

- **Problem**: The OS can evict `localStorage` data when device storage runs low. This is especially aggressive on iOS.
- **Use instead**: `@capacitor/preferences` for the same key-value pattern with native persistence.

### `IndexedDB`

- **Problem**: Also subject to OS eviction, particularly on iOS. Data may be lost unexpectedly.
- **Use instead**: SQLite for structured data that must persist reliably.

### Cookies

- **Problem**: Cookies in a WebView have limited storage, expiration constraints, and inconsistent behavior across platforms.
- **Use instead**: `@capacitor/preferences` or a secure storage plugin.

## Decision Guide

| Requirement | Recommended Solution |
| ----------- | -------------------- |
| App settings, user preferences | `@capacitor/preferences` |
| Authentication tokens, sensitive keys | `@capawesome-team/capacitor-secure-preferences` |
| Relational data, offline-first | SQLite (`@capawesome-team/capacitor-sqlite` or `@capacitor-community/sqlite`) |
| Files, images, documents | `@capacitor/filesystem` |
| Large binary blobs | `@capacitor/filesystem` (write to disk, reference by path) |
