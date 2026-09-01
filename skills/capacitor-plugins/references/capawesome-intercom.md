# Intercom

Unofficial Capacitor plugin for the Intercom live chat and customer support platform.

**Package:** `@capawesome/capacitor-intercom`
**Platforms:** Android, iOS, Web
**Documentation:** https://capawesome.io/docs/sdks/capacitor/intercom/

## Installation

```bash
npm install @capawesome/capacitor-intercom
npx cap sync
```

Requires an active [Intercom](https://www.intercom.com/) account. The **App ID** and the platform-specific **API keys** are in the Intercom dashboard under **Settings → Installation**.

## Configuration

### Android

#### Variables

Defined in your app's `variables.gradle`:

- `$intercomSdkVersion` version of `io.intercom.android:intercom-sdk-base` (default: `18.4.0`)

The plugin depends on `intercom-sdk-base` on purpose: the full `intercom-sdk` artifact registers its own `FirebaseMessagingService` and conflicts with apps that manage push notifications themselves.

#### Push Notifications

Delivered through Firebase Cloud Messaging. Forward the token and incoming messages from JavaScript:

```typescript
import { FirebaseMessaging } from '@capacitor-firebase/messaging';
import { Intercom } from '@capawesome/capacitor-intercom';

FirebaseMessaging.addListener('tokenReceived', async ({ token }) => {
  await Intercom.sendPushTokenToIntercom({ token });
});

FirebaseMessaging.addListener('notificationReceived', async ({ notification }) => {
  const data = notification.data ?? {};
  const { intercom } = await Intercom.isIntercomPushNotification({ data });
  if (intercom) {
    await Intercom.handlePushNotification({ data });
  }
});
```

### iOS

#### Info.plist

To manage push notifications yourself (and coexist with other push plugins), disable Intercom's automatic push integration in `ios/App/App/Info.plist`:

```xml
<key>IntercomAutoIntegratePushNotifications</key>
<false/>
```

#### Push Notifications

1. Enable the **Push Notifications** capability for the app target in Xcode.
2. Register for remote notifications (e.g. via `@capacitor-firebase/messaging` or `@capacitor/push-notifications`).
3. Forward the APNs device token as a hexadecimal string:

```typescript
import { PushNotifications } from '@capacitor/push-notifications';
import { Intercom } from '@capawesome/capacitor-intercom';

PushNotifications.addListener('registration', async ({ value }) => {
  await Intercom.sendPushTokenToIntercom({ token: value });
});
```

### Web

The Messenger is loaded from Intercom's CDN at runtime. Allow `https://widget.intercom.io`, `https://js.intercomcdn.com`, and `wss://*.intercom.io` in your Content Security Policy.

## Usage

### Initialize

```typescript
import { Intercom } from '@capawesome/capacitor-intercom';

await Intercom.initialize({
  appId: 'YOUR_APP_ID',
  androidApiKey: 'YOUR_ANDROID_API_KEY',
  iosApiKey: 'YOUR_IOS_API_KEY',
});
```

### Log in and update a user

```typescript
import { Intercom } from '@capawesome/capacitor-intercom';

await Intercom.setUserHash({ userHash: 'YOUR_HMAC_HASH' });
await Intercom.loginUser({ userId: 'jane-doe', email: 'jane.doe@example.com' });

await Intercom.updateUser({
  name: 'Jane Doe',
  customAttributes: { plan: 'pro' },
  companies: [{ id: 'capawesome', name: 'Capawesome', plan: 'enterprise' }],
});

await Intercom.logout();
```

### Present the Messenger and content

```typescript
import { Intercom } from '@capawesome/capacitor-intercom';

await Intercom.present({ space: 'home' });
await Intercom.presentContent({ type: 'article', id: '123456' });
await Intercom.presentMessageComposer({ initialMessage: 'I need help with…' });
await Intercom.hide();
```

### Unread conversation count

```typescript
import { Intercom } from '@capawesome/capacitor-intercom';

await Intercom.addListener('unreadConversationCountChange', ({ count }) => {
  console.log('Unread conversations:', count);
});

const { count } = await Intercom.getUnreadConversationCount();
```

## Notes

- `initialize(...)` must be called before every other method.
- Identity verification: generate the user hash (HMAC) or JWT on your backend and call `setUserHash(...)` / `setUserJwt(...)` **before** logging in the user. Never ship the secret key in the app.
- Spaces (`present`): `home`, `messages`, `help-center`, `tickets`. Content types (`presentContent`): `article`, `carousel`, `conversation`, `help-center-collections`, `survey`. `help-center-collections` takes `ids`, the others take `id`.
- The deprecated `register*` login methods are intentionally not exposed — use `loginUser(...)` and `loginUnidentifiedUser(...)`.
- Android and iOS only: `handlePushNotification(...)`, `isIntercomPushNotification(...)`, `sendPushTokenToIntercom(...)`, `setBottomPadding(...)`, and the `carousel` and `help-center-collections` content types.
- `messengerShown` / `messengerHidden` are not emitted on Android (the SDK exposes no visibility hook).
- `setBottomPadding(...)` uses pixels on Android and points on iOS.
- On the web, the launcher is visible right after `initialize(...)` — call `setLauncherVisible({ visible: false })` to match the mobile behavior. `getUnreadConversationCount()` returns the last value received from the change event there.
- This project is not affiliated with Intercom Inc.
