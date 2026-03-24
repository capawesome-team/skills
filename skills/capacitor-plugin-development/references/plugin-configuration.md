# Plugin Configuration Values

## Overview

Plugins can define runtime configuration values that app developers set in their Capacitor configuration file (`capacitor.config.ts` or `capacitor.config.json`). These are read-only values available at plugin load time.

## App Developer Configuration

App developers configure plugin values under `plugins.<PluginJSName>` in their Capacitor config:

```json
{
  "plugins": {
    "Example": {
      "style": "dark",
      "maxRetries": 3,
      "iconColor": "#FF0000"
    }
  }
}
```

Or in TypeScript:

```typescript
import { CapacitorConfig } from '@capacitor/cli';

const config: CapacitorConfig = {
  plugins: {
    Example: {
      style: 'dark',
      maxRetries: 3,
      iconColor: '#FF0000',
    },
  },
};

export default config;
```

## Providing Type Definitions

Extend the `PluginsConfig` interface from `@capacitor/cli` so app developers get autocomplete and type checking:

```typescript
declare module '@capacitor/cli' {
  export interface PluginsConfig {
    /**
     * Configuration options for the Example plugin.
     */
    Example?: {
      /**
       * The visual style to use.
       *
       * @default "light"
       * @example "dark"
       * @since 1.0.0
       */
      style?: 'light' | 'dark';

      /**
       * Maximum number of retries.
       *
       * @default 3
       * @since 1.0.0
       */
      maxRetries?: number;

      /**
       * The hex color code for icons.
       *
       * @default "#000000"
       * @example "#FF0000"
       * @since 1.0.0
       */
      iconColor?: string;
    };
  }
}
```

Add this declaration to `src/definitions.ts` or a separate `src/config.ts` file that is re-exported from `src/index.ts`.

## Reading Configuration on iOS

```swift
let style = getConfig().getString("style") ?? "light"
let maxRetries = getConfig().getInt("maxRetries") ?? 3
let iconColor = getConfig().getString("iconColor") ?? "#000000"
```

## Reading Configuration on Android

```java
String style = getConfig().getString("style", "light");
int maxRetries = getConfig().getInt("maxRetries", 3);
String iconColor = getConfig().getString("iconColor", "#000000");
```

## Important Notes

- Configuration values are **optional**. Plugin consumers may not provide any configuration. Always supply default values.
- Configuration values are **not validated** by Capacitor. Plugin consumers can pass invalid data. Handle gracefully.
- Document all configuration options, their types, defaults, and valid values in the plugin README.
