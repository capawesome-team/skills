# Zeroconf

Discover and advertise services on the local network using mDNS/DNS-SD (Zeroconf).

**Package:** `@capawesome-team/capacitor-zeroconf`
**Platforms:** Android, iOS
**Documentation:** https://capawesome.io/docs/sdks/capacitor/zeroconf/
**Capawesome Insiders:** Yes (requires license key)

## Installation

Set up the Capawesome npm registry:

```bash
npm config set @capawesome-team:registry https://npm.registry.capawesome.io
npm config set //npm.registry.capawesome.io/:_authToken <YOUR_LICENSE_KEY>
```

Install the package:

```bash
npm install @capawesome-team/capacitor-zeroconf
npx cap sync
```

## Configuration

### Android

#### Permissions

No configuration is required. The plugin declares `INTERNET`, `ACCESS_NETWORK_STATE` and `ACCESS_LOCAL_NETWORK` itself.

`ACCESS_LOCAL_NETWORK` is only enforced on Android 16+ for apps that target SDK 37 or higher. Use `requestPermissions()` before starting a discovery, or use the system picker (see below), which requires no permission at all.

#### Proguard

If using Proguard, add to `android/app/proguard-rules.pro`:

```
-keep class io.capawesome.capacitorjs.plugins.** { *; }
```

### iOS

#### Privacy Descriptions

**Required.** Add to `ios/App/App/Info.plist`. Every service type the app discovers or advertises must be declared in `NSBonjourServices`; wildcards are not supported. Without these entries, iOS silently returns no results:

```xml
<key>NSLocalNetworkUsageDescription</key>
<string>We need access to the local network to discover services.</string>
<key>NSBonjourServices</key>
<array>
  <string>_http._tcp</string>
</array>
```

The plugin validates both entries at runtime and rejects with an actionable error naming the exact entry to add.

## Usage

### Discover services

```typescript
import { Zeroconf } from '@capawesome-team/capacitor-zeroconf';

await Zeroconf.addListener('serviceFound', (event) => {
  console.log('Found:', event.service.name);
});
await Zeroconf.addListener('serviceResolved', (event) => {
  const { hostname, ipv4Addresses, port } = event.service;
  console.log('Resolved:', hostname, ipv4Addresses, port);
});
await Zeroconf.addListener('serviceLost', (event) => {
  console.log('Lost:', event.service.name);
});

const { id } = await Zeroconf.startDiscovery({ type: '_http._tcp' });
await Zeroconf.stopDiscoveryById({ id });
```

### Resolve a service manually

```typescript
import { Zeroconf } from '@capawesome-team/capacitor-zeroconf';

const { id } = await Zeroconf.startDiscovery({
  type: '_http._tcp',
  autoResolve: false,
});

const { service } = await Zeroconf.resolveServiceById({ id: serviceId });
```

### Advertise a service

```typescript
import { Zeroconf } from '@capawesome-team/capacitor-zeroconf';

const { id, name } = await Zeroconf.startAdvertising({
  name: 'My Service',
  port: 8080,
  type: '_http._tcp',
  txtRecord: { path: '/api' },
});

await Zeroconf.addListener('advertisingNameChange', (event) => {
  console.log('Renamed to:', event.name);
});

await Zeroconf.stopAdvertisingById({ id });
```

### Permissions

```typescript
import { Zeroconf } from '@capawesome-team/capacitor-zeroconf';

const { localNetwork } = await Zeroconf.checkPermissions();
if (localNetwork !== 'granted') {
  await Zeroconf.requestPermissions();
}
```

### Use the system picker (Android)

```typescript
import { Zeroconf } from '@capawesome-team/capacitor-zeroconf';

const { id } = await Zeroconf.startDiscovery({
  type: '_http._tcp',
  androidUsePicker: true,
});
```

## Notes

- Discovery and advertising only work while the app is in the foreground on both platforms. On iOS, `serviceLost` is emitted for all services when the app enters the background; discovery and advertising are restored automatically when it returns to the foreground.
- Service types must be of the form `_<service>._tcp` or `_<service>._udp` (a trailing dot is optional). Service type enumeration (`_services._dns-sd._udp`) and subtypes are not supported.
- `startAdvertising()` returns the actually registered name, which may differ from the requested one if the responder renamed the service because of a conflict. Later renames are delivered via `advertisingNameChange`.
- With `autoResolve: true` (default), a resolved service replaces the one from `serviceFound`; both carry the same `id`.
- On iOS, there is no API to read the local network permission, so `checkPermissions()` returns `prompt` until a denial is observed and never returns `granted`.
- `androidUsePicker` requires Android 17 (SDK 37) or higher, returns only the service the user selected, and needs no permission. It is ignored on iOS.
- The Android emulator does not support mDNS, and the iOS Simulator does not enforce local network privacy. Test on real devices on the same network.
