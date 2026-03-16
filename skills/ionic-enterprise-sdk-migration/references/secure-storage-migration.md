# Secure Storage → Capawesome Secure Preferences + SQLite

Migrate from `@ionic-enterprise/secure-storage` to `@capawesome-team/capacitor-secure-preferences` (key-value) and/or `@capawesome-team/capacitor-sqlite` (SQLite databases).

Ionic Secure Storage offered both key-value storage and SQLite database functionality. Determine which features the project uses and migrate accordingly — one or both plugins may be needed.

## Feature Mapping

### Key-Value Storage

| Feature        | Ionic Secure Storage             | Capawesome Secure Preferences          |
| -------------- | -------------------------------- | -------------------------------------- |
| Create store   | `KeyValueStorage.create(...)`    | Not needed — ready to use immediately  |
| Set value      | `KeyValueStorage.set(key, val)`  | `SecurePreferences.set({ key, value })`|
| Get value      | `KeyValueStorage.get(key)`       | `SecurePreferences.get({ key })`       |
| Remove value   | `KeyValueStorage.remove(key)`    | `SecurePreferences.remove({ key })`    |
| Clear store    | `KeyValueStorage.clear()`        | `SecurePreferences.clear()`            |

### SQLite Database

| Feature          | Ionic Secure Storage                     | Capawesome SQLite                              |
| ---------------- | ---------------------------------------- | ---------------------------------------------- |
| Open database    | `SQLite.create({ name, location })`      | `Sqlite.open({ path, upgradeStatements })`     |
| Execute SQL      | `db.executeSql(statement, values)`        | `Sqlite.execute({ databaseId, statement, values })` |
| Query data       | `db.transaction(tx => tx.executeSql(...))` | `Sqlite.query({ databaseId, statement, values })` |
| Transactions     | `db.transaction(callback)`               | `Sqlite.beginTransaction` / `commitTransaction` |
| Schema migration | Manual                                   | Built-in via `upgradeStatements`               |
| Encryption       | Built-in (256-bit AES)                   | Optional via SQLCipher configuration           |

## Key Differences

- **No store creation for key-value**: Ionic Secure Storage requires `KeyValueStorage.create('encryption-key')`. Capawesome Secure Preferences is ready to use immediately — the encryption key is managed by the plugin.
- **Object-based API**: Ionic Secure Storage uses positional arguments (`set(key, value)`). Capawesome uses objects (`set({ key, value })`).
- **Built-in schema migrations**: Capawesome SQLite supports `upgradeStatements` with version numbers, replacing manual `CREATE TABLE IF NOT EXISTS` patterns.
- **Promise-based queries**: Ionic Secure Storage uses callback-based transactions. Capawesome SQLite uses `async/await` throughout.
- **Explicit encryption opt-in**: Capawesome SQLite encryption requires platform configuration (SQLCipher on Android/iOS). See the `capacitor-plugins` skill reference for details.

## Step-by-Step Migration: Key-Value Storage

### 1. Remove Ionic Secure Storage and install Capawesome Secure Preferences

```bash
npm uninstall @ionic-enterprise/secure-storage
```

Install the Capawesome Secure Preferences plugin using the `capacitor-plugins` skill or follow the installation instructions in the reference file `references/capawesome-secure-preferences.md` in the `capacitor-plugins` skill.

### 2. Remove store creation

```diff
-import { KeyValueStorage } from '@ionic-enterprise/secure-storage';
-
-await KeyValueStorage.create('my-secret-key');
+import { SecurePreferences } from '@capawesome-team/capacitor-secure-preferences';
+
+// No store creation needed — ready to use immediately
```

### 3. Replace set

**Before:**

```typescript
import { KeyValueStorage } from '@ionic-enterprise/secure-storage';

await KeyValueStorage.set('username', 'john_doe');
```

**After:**

```typescript
import { SecurePreferences } from '@capawesome-team/capacitor-secure-preferences';

await SecurePreferences.set({ key: 'username', value: 'john_doe' });
```

### 4. Replace get

**Before:**

```typescript
import { KeyValueStorage } from '@ionic-enterprise/secure-storage';

const value = await KeyValueStorage.get('username');
```

**After:**

```typescript
import { SecurePreferences } from '@capawesome-team/capacitor-secure-preferences';

const { value } = await SecurePreferences.get({ key: 'username' });
```

### 5. Replace remove

**Before:**

```typescript
import { KeyValueStorage } from '@ionic-enterprise/secure-storage';

await KeyValueStorage.remove('username');
```

**After:**

```typescript
import { SecurePreferences } from '@capawesome-team/capacitor-secure-preferences';

await SecurePreferences.remove({ key: 'username' });
```

### 6. Replace clear

**Before:**

```typescript
import { KeyValueStorage } from '@ionic-enterprise/secure-storage';

await KeyValueStorage.clear();
```

**After:**

```typescript
import { SecurePreferences } from '@capawesome-team/capacitor-secure-preferences';

await SecurePreferences.clear();
```

## Step-by-Step Migration: SQLite Database

### 1. Remove Ionic Secure Storage and install Capawesome SQLite

```bash
npm uninstall @ionic-enterprise/secure-storage
```

Install the Capawesome SQLite plugin using the `capacitor-plugins` skill or follow the installation instructions in the reference file `references/capawesome-sqlite.md` in the `capacitor-plugins` skill.

### 2. Replace database opening

**Before:**

```typescript
import { SQLite } from '@ionic-enterprise/secure-storage';

const db = await SQLite.create({
  name: 'database.db',
  location: 'default',
});
await db.executeSql(
  'CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT, email TEXT)',
  []
);
```

**After:**

```typescript
import { Sqlite } from '@capawesome-team/capacitor-sqlite';

const { databaseId } = await Sqlite.open({
  path: 'database.db',
  upgradeStatements: [
    {
      version: 1,
      statements: [
        'CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT, email TEXT)',
      ],
    },
  ],
});
```

### 3. Replace SQL execution

**Before:**

```typescript
import { SQLiteObject } from '@ionic-enterprise/secure-storage';

await db.executeSql('INSERT INTO users (name, email) VALUES (?, ?)', [name, email]);
```

**After:**

```typescript
import { Sqlite } from '@capawesome-team/capacitor-sqlite';

await Sqlite.execute({
  databaseId,
  statement: 'INSERT INTO users (name, email) VALUES (?, ?)',
  values: [name, email],
});
```

### 4. Replace queries

**Before:**

```typescript
import { SQLiteObject } from '@ionic-enterprise/secure-storage';

const result = await new Promise((resolve) => {
  db.transaction((tx) => {
    tx.executeSql('SELECT * FROM users WHERE name LIKE ?', ['%John%'], (tx, result) => {
      resolve(result.rows);
    });
  });
});
```

**After:**

```typescript
import { Sqlite } from '@capawesome-team/capacitor-sqlite';

const result = await Sqlite.query({
  databaseId,
  statement: 'SELECT * FROM users WHERE name LIKE ?',
  values: ['%John%'],
});
const rows = result.values;
```
