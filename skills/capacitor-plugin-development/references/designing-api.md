# Designing the Plugin API

## Overview

The TypeScript API definition is the contract between JavaScript and native code. Define it before writing any native implementation.

## Check for Existing Web APIs

Before building a plugin, check whether the desired functionality already exists as a Web API. If a Web API covers the use case, a plugin may not be needed. Resources:

- [MDN Web APIs](https://developer.mozilla.org/en-US/docs/Web/API)
- [What Web Can Do Today](https://whatwebcando.today/)

If the Web API is missing on one or more platforms (e.g., iOS Safari), the plugin provides the cross-platform bridge.

## Defining Types in `src/definitions.ts`

All plugin types, interfaces, and the plugin interface itself are defined in `src/definitions.ts`.

### Plugin Interface

Define an interface that describes every method the plugin exposes:

```typescript
export interface ExamplePlugin {
  echo(options: EchoOptions): Promise<EchoResult>;
  getStatus(): Promise<StatusResult>;
  openSettings(): Promise<void>;
  addListener(
    eventName: 'statusChange',
    listenerFunc: (event: StatusChangeEvent) => void,
  ): Promise<PluginListenerHandle>;
  removeAllListeners(): Promise<void>;
}
```

### Options and Result Interfaces

Define separate interfaces for method options and return values:

```typescript
export interface EchoOptions {
  /**
   * The value to echo back.
   */
  value: string;
}

export interface EchoResult {
  /**
   * The echoed value.
   */
  value: string;
}

export interface StatusResult {
  /**
   * The current status.
   */
  status: 'active' | 'inactive';
}

export interface StatusChangeEvent {
  /**
   * The new status.
   */
  status: 'active' | 'inactive';
}
```

### Enums and Union Types

Prefer string union types over TypeScript enums for better compatibility:

```typescript
// Preferred: string union type
export type OrientationType =
  | 'portrait-primary'
  | 'portrait-secondary'
  | 'landscape-primary'
  | 'landscape-secondary';

// Avoid: TypeScript enum
// export enum OrientationType { ... }
```

## Registering the Plugin in `src/index.ts`

The `src/index.ts` file registers the plugin with Capacitor and re-exports all types:

```typescript
import { registerPlugin } from '@capacitor/core';

import type { ExamplePlugin } from './definitions';

const Example = registerPlugin<ExamplePlugin>('Example', {
  web: () => import('./web').then((m) => new m.ExampleWeb()),
});

export * from './definitions';
export { Example };
```

The first argument to `registerPlugin()` is the **JavaScript plugin name**. This must match:
- The `jsName` property in the iOS `CAPBridgedPlugin` conformance.
- The `name` property in the Android `@CapacitorPlugin` annotation.

## JSDoc Tags

Every interface, method, and property should have JSDoc comments. These are used by `npm run docgen` to auto-generate README documentation. Use the following tags:

- `@since` — Version when the interface, method, or property was introduced. Must always be the next version after the last release (semver).
- `@deprecated` — Marks deprecated interfaces, methods, or properties.
- `@example` — Provides an example value for properties.
- `@default` — Indicates the default value of a property.

### Platform Availability

If a method is only available on certain platforms, document it with a comment above the `@since` tag:

```typescript
export interface ExamplePlugin {
  /**
   * Take a photo using the device camera.
   *
   * Only available on Android and iOS.
   *
   * @since 1.0.0
   */
  takePhoto(options: TakePhotoOptions): Promise<TakePhotoResult>;
}
```

### Error Codes

Define error codes as a string enum in `definitions.ts` so consumers can match against them:

```typescript
/**
 * @since 1.0.0
 */
export enum ErrorCode {
  /**
   * The required value is missing.
   *
   * @since 1.0.0
   */
  ValueMissing = 'VALUE_MISSING',
}
```

## API Design Principles

1. **Use `undefined` instead of `null`** for missing or absent values.
2. **Methods must return a `Promise`** and may have a maximum of one parameter.
3. **Options parameter**: Name it `options` with a type named `<MethodName>Options`.
4. **Result type**: Name it `<MethodName>Result`.
5. **Standardize measurement units** across platforms (e.g., always use meters, not feet on one platform and meters on another).
6. **Use ISO 8601 datetime strings** with timezone information instead of platform-specific date objects or Unix timestamps.
7. **Keep plugins small in scope.** A plugin should do one thing well. Bundling unrelated features increases app size and may trigger App Store rejections for unused API permissions.
