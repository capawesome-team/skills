# Cross-Platform Best Practices

Guidelines for building Capacitor apps that work well across Android, iOS, and the web.

## Platform Detection

Use `Capacitor.getPlatform()` and `Capacitor.isNativePlatform()` to handle platform differences:

```typescript
import { Capacitor } from '@capacitor/core';

const platform = Capacitor.getPlatform(); // 'android', 'ios', or 'web'

if (Capacitor.isNativePlatform()) {
  // Native-only code (Android or iOS)
}
```

## Plugin Availability

Not all plugins are available on all platforms. Check availability before calling a plugin method:

```typescript
import { Capacitor } from '@capacitor/core';

if (Capacitor.isPluginAvailable('Camera')) {
  // Camera plugin is available on this platform
  const photo = await Camera.getPhoto({ ... });
} else {
  // Provide a fallback (e.g., file input on web)
}
```

## Responsive Design

- Design for mobile-first, then adapt for larger screens (tablets, desktop via Electron or PWA).
- Use CSS media queries and flexible layouts.
- Test on multiple screen sizes and orientations.
- Use `env(safe-area-inset-*)` CSS variables for safe area handling (see `references/safe-area.md`).

## Performance

### Minimize Bridge Calls

Each call across the native bridge has overhead (serialization, async context switch). Batch operations when possible rather than making many individual calls.

```typescript
// BAD — many individual bridge calls
for (const key of keys) {
  await Preferences.set({ key, value: data[key] });
}

// BETTER — batch into a single operation if the plugin supports it
// Or serialize all data into one value
await Preferences.set({ key: 'all_settings', value: JSON.stringify(data) });
```

### Avoid Large Data Transfers

Do not pass large binary data (images, files) across the bridge as base64 strings. Use file paths instead (see `references/file-handling.md`).

### Lazy Load Plugins

Import plugins only where they are used. Most bundlers (Vite, Webpack) tree-shake unused imports, but dynamic imports ensure plugin code is only loaded when needed:

```typescript
async function takePhoto() {
  const { Camera } = await import('@capacitor/camera');
  return Camera.getPhoto({ ... });
}
```

## Error Handling

Always wrap plugin calls in try-catch blocks. Native operations can fail due to permission denial, hardware unavailability, or user cancellation:

```typescript
import { Camera, CameraResultType } from '@capacitor/camera';

try {
  const photo = await Camera.getPhoto({
    resultType: CameraResultType.Uri,
  });
} catch (error) {
  if (error.message === 'User cancelled photos app') {
    // User cancelled — not an error
  } else {
    // Handle actual error
    console.error('Camera error:', error);
  }
}
```

## Permissions

Follow the check-then-request pattern for all plugins that require permissions:

```typescript
import { Camera } from '@capacitor/camera';

// 1. Check current status
const status = await Camera.checkPermissions();

if (status.camera !== 'granted') {
  // 2. Request if not granted
  const requested = await Camera.requestPermissions();

  if (requested.camera === 'denied') {
    // 3. Handle permanent denial — guide user to app settings
    return;
  }
}

// 4. Proceed with the operation
const photo = await Camera.getPhoto({ ... });
```

On iOS, if a permission is denied, it cannot be re-requested programmatically. The user must enable it in device Settings > app > Permissions.

## Configuration Management

- Use `capacitor.config.ts` (TypeScript) over `capacitor.config.json` for environment-specific configuration (see `references/app-configuration.md`).
- Never commit development server URLs to version control.
- Use environment variables for build-time configuration.

## Native Project Management

- Commit `android/` and `ios/` directories to version control. They are source-of-truth native projects.
- Run `npx cap sync` after every plugin install/uninstall and after every web build.
- Keep native dependencies up to date alongside web dependencies.
- Avoid modifying files in `android/app/src/main/assets/public/` or `ios/App/App/public/` directly — these are overwritten by `npx cap copy`.
