# Plugin Configuration

All options are set under `plugins.LiveUpdate` in `capacitor.config.ts` or `capacitor.config.json`.

| Option                         | Type                          | Default                        | Since | Description                                                                                                                                                                                                          |
| ------------------------------ | ----------------------------- | ------------------------------ | ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `appId`                        | `string`                      | —                              | 5.0.0 | The Capawesome Cloud app ID (UUID). Not the native app identifier.                                                                                                                                                   |
| `autoBlockRolledBackBundles`   | `boolean`                     | `false`                        | 7.3.0 | Automatically block bundles that caused a rollback (up to 100). No effect if `readyTimeout` is `0`. Android/iOS only.                                                                                               |
| `autoDeleteBundles`            | `boolean`                     | `false`                        | 5.0.0 | Automatically delete unused bundles after `ready()` is called.                                                                                                                                                       |
| `autoUpdateStrategy`           | `'none'` \| `'background'`   | `'none'`                       | 7.3.0 | `none`: No automatic updates. `background`: Auto-download and apply at startup and resume (if last check >15 min ago). Android/iOS only.                                                                            |
| `defaultChannel`               | `string`                      | —                              | 6.3.0 | Default channel. Can be overridden by `setChannel()`, `sync()` channel param, or native config.                                                                                                                     |
| `httpTimeout`                  | `number`                      | `60000`                        | 6.4.0 | Timeout in milliseconds for HTTP requests.                                                                                                                                                                           |
| `publicKey`                    | `string`                      | —                              | 6.1.0 | PEM-encoded RSA public key for bundle integrity verification.                                                                                                                                                        |
| `readyTimeout`                 | `number`                      | `0`                            | 5.0.0 | Timeout in ms to wait for `ready()` before rolling back. Set to e.g. `10000` for rollback protection. `0` disables.                                                                                                 |
| `serverDomain`                 | `string`                      | `'api.cloud.capawesome.io'`    | 7.0.0 | API domain without scheme or path. Use `'api.cloud.capawesome.eu'` for EU servers.                                                                                                                                   |

## Example Configuration

```typescript
import { CapacitorConfig } from "@capacitor/cli";

const config: CapacitorConfig = {
  plugins: {
    LiveUpdate: {
      appId: "6e351b4f-69a7-415e-a057-4567df7ffe94",
      autoUpdateStrategy: "background",
      readyTimeout: 10000,
      autoDeleteBundles: true,
      autoBlockRolledBackBundles: true,
      defaultChannel: "production",
    },
  },
};

export default config;
```

## Channel Priority

The channel is resolved in the following order (highest priority first):

1. **Forced channel** (set via Capawesome Cloud Console or CLI per device — highest priority)
2. **`sync()` channel parameter** (not persisted, single-call override)
3. **`setChannel()`** (persisted in SharedPreferences/UserDefaults)
4. **Native config** (`CapawesomeLiveUpdateDefaultChannel` in `Info.plist` on iOS, `capawesome_live_update_default_channel` in `strings.xml` on Android)
5. **Capacitor config** `defaultChannel` (lowest priority)
