# Network

Read the current network status and listen for connectivity changes.

**Package:** `@capawesome/capacitor-network`
**Platforms:** Android, iOS, Web

## Installation

```bash
npm install @capawesome/capacitor-network
npx cap sync
```

The plugin already declares the `android.permission.ACCESS_NETWORK_STATE` permission in its manifest, so no additional permissions or configuration are required on any platform.

## Usage

```typescript
import { Network, ConnectionType } from '@capawesome/capacitor-network';

const status = await Network.getStatus();
// status.connected: boolean
// status.connectionType: ConnectionType
// status.internetReachable: boolean | null (Android only)

// Android only.
const { enabled } = await Network.isAirplaneModeEnabled();
```

```typescript
// Listen for status changes (device is only observed while a listener is attached).
const handle = await Network.addListener('networkStatusChange', (status) => {
  console.log('Connected:', status.connected, 'Type:', status.connectionType);
});

await handle.remove();
// or: await Network.removeAllListeners();
```

`ConnectionType` values: `WIFI`, `CELLULAR`, `ETHERNET`, `VPN`, `NONE`, `UNKNOWN`.

## Notes

- `internetReachable` reports verified internet access and is only available on Android. It is `null` on iOS and Web, where connectivity does not guarantee reachability.
- `isAirplaneModeEnabled()` is only available on Android.
- The device is only observed while at least one `networkStatusChange` listener is attached.
