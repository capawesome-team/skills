# Watch

Capacitor plugin for communicating with native Apple Watch and Wear OS apps. Provides messaging with optional replies, latest-wins state sync, queued transfers and reachability info through a single API.

**Package:** `@capawesome-team/capacitor-watch`
**Platforms:** Android, iOS
**Capawesome Insiders:** Yes (requires license key)

## Installation

Set up the Capawesome npm registry:

```bash
npm config set @capawesome-team:registry https://npm.registry.capawesome.io
npm config set //npm.registry.capawesome.io/:_authToken <YOUR_LICENSE_KEY>
```

Install the package:

```bash
npm install @capawesome-team/capacitor-watch
npx cap sync
```

## Configuration

### Android

The plugin registers its own `WearableListenerService`, so no changes to `android/app/src/main/AndroidManifest.xml` and no `MainActivity` modifications are required. Google Play services are required; without them all methods reject as unavailable.

#### Wear OS companion app

A Wear OS app module is required to exchange data. Two hard requirements of the Google Play services Data Layer: the module must use the **same `applicationId`** as the phone app, and both apps must be signed with the **same signing certificate**.

1. Create the module folder `android/wear` (copy the minimal example from `https://github.com/capawesome-team/capacitor-plugins/tree/main/packages/watch/example/wearos` and change the `applicationId` to the phone app's id).
2. Add to `android/settings.gradle`:

```groovy
include ':wear'
include ':capawesome-watch-sdk'
project(':capawesome-watch-sdk').projectDir = new File('../node_modules/@capawesome-team/capacitor-watch/sdks/wearos')
```

3. Add to the `dependencies` block in `android/wear/build.gradle`: `implementation project(':capawesome-watch-sdk')`
4. Declare the capability in `android/wear/src/main/res/values/wear.xml`:

```xml
<resources>
    <string-array name="android_wear_capabilities">
        <item>capawesome_watch</item>
    </string-array>
</resources>
```

5. Create a listener service in the watch module that extends `WatchListenerService` (overriding `onMessageReceived`, `onStateReceived`, `onUserInfoReceived`) and register it **inside** the `application` tag of the watch module's `AndroidManifest.xml`:

```xml
<service android:name=".MyWatchListenerService" android:exported="true">
    <intent-filter>
        <action android:name="com.google.android.gms.wearable.MESSAGE_RECEIVED" />
        <action android:name="com.google.android.gms.wearable.DATA_CHANGED" />
        <data android:scheme="wear" android:host="*" android:pathPrefix="/capawesome/watch" />
    </intent-filter>
</service>
```

6. Send data from the watch with the `CapawesomeWatch(context)` class. Its methods (`sendMessage`, `sendMessageForReply`, `updateState`, `transferUserInfo`) are `suspend` functions and must be called from a coroutine.

#### Capability

Optionally set `plugins.Watch.capability` (string) in `capacitor.config.ts` to change the capability used to discover the watch app. Android only, default: `capawesome_watch`.

#### Variables

Optionally set in `android/variables.gradle`:

- `playServicesWearableVersion`: version of `com.google.android.gms:play-services-wearable` (default: `20.0.1`)

#### Proguard

If using Proguard, add to `android/app/proguard-rules.pro`:

```
-keep class io.capawesome.capacitorjs.plugins.** { *; }
```

### iOS

No phone-side changes are required. The plugin activates the `WCSession` on load and handles all session delegate callbacks internally, so `ios/App/App/AppDelegate.swift` does **not** need to be modified.

#### watchOS companion app

A watchOS app target is required to exchange data. It **cannot be created by editing files** and must be created in Xcode:

1. Open `ios/App/App.xcodeproj` (or `ios/App/App.xcworkspace` if the app uses CocoaPods) in Xcode.
2. Select **File > New > Target…**, choose the **watchOS** tab and the **Watch App for Existing iOS App** template. The **Project** dropdown at the bottom must point to the app's project.
3. Enter a product name (e.g. `watch`) and select **SwiftUI** as interface. The bundle identifier of the watch app must be prefixed with the bundle identifier of the iOS app (e.g. `com.example.app.watchkitapp`); Xcode derives this automatically.
4. Select **File > Add Package Dependencies… > Add Local…** and choose the folder `node_modules/@capawesome-team/capacitor-watch/sdks/watchos`. Add the `CapawesomeWatchSDK` product to the **watch app target**, not the iOS app target. `node_modules` must be installed before opening the project.
5. Use the SDK in the watch app's SwiftUI view (complete example: `https://github.com/capawesome-team/capacitor-plugins/tree/main/packages/watch/example/watchos`):

```swift
import SwiftUI
import CapawesomeWatchSDK

struct ContentView: View {
    @ObservedObject private var watch = CapawesomeWatch.shared

    var body: some View {
        Button("Send Message") { watch.sendMessage(["text": "Hello from the watch!"]) }
            .onAppear {
                watch.onMessageReceived = { data, reply in reply?(["text": "Hello back!"]) }
                watch.activate()
            }
    }
}
```

## Usage

### Check the connection to the watch

```typescript
import { Watch } from '@capawesome-team/capacitor-watch';

const { reachable, paired, watchAppInstalled } = await Watch.getConnectionInfo();
```

### Send a message with an optional reply

```typescript
import { Watch } from '@capawesome-team/capacitor-watch';

await Watch.sendMessage({ data: { text: 'Hello from the phone!' } });

const { reply } = await Watch.sendMessage({
  data: { text: 'Hello from the phone!' },
  expectsReply: true,
});
```

### Sync state and queue transfers

```typescript
import { Watch } from '@capawesome-team/capacitor-watch';

await Watch.updateState({ data: { counter: 42 } });
const { data } = await Watch.getReceivedState();

await Watch.transferUserInfo({ data: { sentAt: Date.now() } });
```

### Listen for data from the watch

```typescript
import { Watch } from '@capawesome-team/capacitor-watch';

await Watch.addListener('messageReceived', async (event) => {
  if (event.messageId) {
    await Watch.replyToMessage({ data: { text: 'Hello back!' }, messageId: event.messageId });
  }
});
await Watch.addListener('reachabilityChange', (event) => console.log(event.reachable));
await Watch.addListener('stateReceived', (event) => console.log(event.data));
await Watch.addListener('userInfoReceived', (event) => console.log(event.data));
```

## Notes

- Three channels with different guarantees: `sendMessage(...)` is live and requires reachability (rejects with `WATCH_NOT_REACHABLE` otherwise), `updateState(...)` delivers only the latest state and survives restarts, `transferUserInfo(...)` queues every transfer until delivered.
- All payloads must be JSON-serializable, `null` values are not supported, and the operating systems limit the payload to about 100 KB.
- Events: `messageReceived`, `reachabilityChange`, `stateReceived`, `userInfoReceived`. `replyToMessage(...)` is only possible if `messageReceived` provided a `messageId`.
- `getConnectionInfo()`: on Android `paired` is always `null` and `watchAppInstalled` is derived from the configured capability.
- Queued transfers are delivered in order on iOS; on Android there is no ordering guarantee and only the 100 most recent undelivered transfers are kept.
- Data received while the app is closed is replayed once listeners are registered. On Android it is persisted and survives an app restart; on iOS it is only retained in memory and lost if the app is terminated first.
- Identical state updates may not be redelivered on either platform if the data did not change.
- Cross-platform pairing is not supported: Wear OS works only with an Android phone, Apple Watch only with an iPhone.
- Unlike the official `@capacitor/watch` plugin (see `references/capacitor-watch.md`), which is iOS/watchOS-only and renders a SwiftUI-template UI defined in web code, this plugin is a data bridge for Apple Watch **and** Wear OS whose watch UI is built natively.
