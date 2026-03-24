# Web Plugin Implementation

## Architecture

The web implementation provides PWA/browser functionality for the plugin. It extends `WebPlugin` from `@capacitor/core` and implements the plugin interface defined in `src/definitions.ts`.

The file lives at `src/web.ts`.

## Basic Implementation

```typescript
import { WebPlugin } from '@capacitor/core';

import type { EchoOptions, EchoResult, ExamplePlugin } from './definitions';

export class ExampleWeb extends WebPlugin implements ExamplePlugin {
  async echo(options: EchoOptions): Promise<EchoResult> {
    console.log('ECHO', options);
    return options;
  }
}
```

Every method defined in the plugin interface must be implemented. If a method cannot be implemented on the web, throw an error:

```typescript
async openNativeSettings(): Promise<void> {
  throw this.unimplemented('openNativeSettings is not available on the web.');
}
```

## Error Handling

### Unavailable

Use when the functionality requires a web API that the current browser does not support:

```typescript
async getOrientation(): Promise<OrientationResult> {
  if (!('screen' in window) || !('orientation' in window.screen)) {
    throw this.unavailable('Screen Orientation API is not available in this browser.');
  }
  // ... implementation
}
```

### Unimplemented

Use when the method has no meaningful web implementation:

```typescript
async vibrate(): Promise<void> {
  throw this.unimplemented('vibrate is not implemented on the web.');
}
```

## Events

Emit events to JavaScript listeners using `notifyListeners()`:

```typescript
window.screen.orientation.addEventListener('change', () => {
  this.notifyListeners('orientationChange', {
    type: window.screen.orientation.type,
  });
});
```

## Permissions

If the plugin requires permissions, implement `checkPermissions()` and `requestPermissions()`:

```typescript
async checkPermissions(): Promise<PermissionStatus> {
  if (typeof navigator === 'undefined' || !navigator.permissions) {
    throw this.unavailable('Permissions API not available in this browser.');
  }

  const permission = await navigator.permissions.query({ name: 'camera' as PermissionName });
  return {
    camera: permission.state === 'granted' ? 'granted' : permission.state === 'denied' ? 'denied' : 'prompt',
  };
}

async requestPermissions(): Promise<PermissionStatus> {
  throw this.unimplemented(
    'requestPermissions is not available on the web. Use the native browser permission prompt instead.',
  );
}
```

## Web-Only Behavior

Some plugins may have richer functionality on the web than on native. This is acceptable. The TypeScript interface is the contract, and each platform implements it according to platform capabilities.

If a method does significantly more or less on web vs. native, document the difference in JSDoc comments on the interface method.
