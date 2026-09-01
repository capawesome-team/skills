# Crisp

Unofficial Capacitor plugin for the Crisp live chat and customer support platform.

**Package:** `@capawesome/capacitor-crisp`
**Platforms:** Android, iOS, Web
**Documentation:** https://capawesome.io/docs/sdks/capacitor/crisp/

## Installation

```bash
npm install @capawesome/capacitor-crisp
npx cap sync
```

Requires an active [Crisp](https://crisp.chat/) account. The **Website ID** is in the Crisp dashboard under **Settings → Website Settings → Setup & Integrations**.

## Configuration

### Android

#### Variables

Defined in your app's `variables.gradle`:

- `$crispSdkVersion` version of `im.crisp:crisp-sdk` (default: `2.0.22`)
- `$firebaseMessagingVersion` version of `com.google.firebase:firebase-messaging` (default: `24.1.1`)

#### Minimum SDK Version

The Crisp Android SDK requires `minSdkVersion` `24` or higher in `variables.gradle`.

#### Multidex

**Required.** Enable multidex in `android/app/build.gradle`:

```groovy
android {
  defaultConfig {
    multiDexEnabled true
  }
}
```

#### Push Notifications

Delivered through Firebase Cloud Messaging. Either add Crisp's own service to the `application` tag of `android/app/src/main/AndroidManifest.xml` (only if the app has no `FirebaseMessagingService` of its own):

```xml
<service
  android:name="im.crisp.client.external.notification.CrispNotificationService"
  android:exported="false">
  <intent-filter>
    <action android:name="com.google.firebase.MESSAGING_EVENT" />
  </intent-filter>
</service>
```

Or forward messages from an existing messaging setup:

```typescript
import { FirebaseMessaging } from '@capacitor-firebase/messaging';
import { Crisp } from '@capawesome/capacitor-crisp';

FirebaseMessaging.addListener('notificationReceived', async ({ notification }) => {
  const data = notification.data ?? {};
  const { crisp } = await Crisp.isCrispPushNotification({ data });
  if (crisp) {
    await Crisp.handlePushNotification({ data });
  }
});
```

### iOS

#### Privacy Descriptions

Required for chat attachments. Add to `ios/App/App/Info.plist`:

```xml
<key>NSCameraUsageDescription</key>
<string>The app needs access to your camera to send photos in the chat.</string>
<key>NSPhotoLibraryAddUsageDescription</key>
<string>The app needs access to your photo library to send photos in the chat.</string>
```

#### Push Notifications

Enable the **Push Notifications** capability in Xcode and register for remote notifications (e.g. via `@capacitor-firebase/messaging` or `@capacitor/push-notifications`). The plugin forwards the APNs device token to Crisp automatically. Use `setShouldPromptForNotificationPermission(...)` to control the Crisp permission prompt.

### Web

The chatbox is loaded from Crisp's CDN at runtime. Allow `client.crisp.chat`, `*.crisp.chat`, and `wss://*.relay.crisp.chat` in your Content Security Policy.

## Usage

### Configure and open the chat

```typescript
import { Crisp } from '@capawesome/capacitor-crisp';

await Crisp.configure({ websiteId: 'YOUR_WEBSITE_ID' });
await Crisp.openChat();
```

### Set the user

```typescript
import { Crisp } from '@capawesome/capacitor-crisp';

await Crisp.setUser({
  email: 'jane.doe@example.com',
  emailSignature: 'YOUR_HMAC_SIGNATURE',
  nickname: 'Jane Doe',
});
await Crisp.setCompany({ name: 'Capawesome', url: 'https://capawesome.io' });
```

### Attach session data

```typescript
import { Crisp, SessionEventColor } from '@capawesome/capacitor-crisp';

await Crisp.setSessionString({ key: 'plan', value: 'pro' });
await Crisp.setSessionSegment({ segment: 'checkout' });
await Crisp.pushSessionEvent({ name: 'purchase', color: SessionEventColor.Green });
```

### Listen for events

```typescript
import { Crisp } from '@capawesome/capacitor-crisp';

await Crisp.addListener('sessionLoaded', ({ sessionId }) => {
  console.log('Session loaded:', sessionId);
});
await Crisp.addListener('messageReceived', ({ content }) => {
  console.log('Message received:', content);
});
```

## Notes

- `configure(...)` must be called before every other method.
- The session ID is only available via the `sessionLoaded` event; there is no synchronous getter.
- Identity verification: generate an HMAC-SHA256 signature of the user's email on your backend and pass it as `emailSignature` to `setUser(...)`. Never ship the secret key in the app.
- `setTokenId(...)` (or `tokenId` in `configure(...)`) restores a session across devices and logins. `resetSession()` starts a new session.
- Android only: `isCrispPushNotification(...)`, `handlePushNotification(...)`, `setNotificationsEnabled(...)`. iOS only: `setShouldPromptForNotificationPermission(...)` — the iOS SDK handles its own notifications once the APNs token is forwarded.
- Helpdesk: `searchHelpdesk()` opens the search view, `openHelpdeskArticle(...)` a specific article.
- Crisp stops publishing CocoaPods releases in September 2026; prefer Swift Package Manager on iOS.
- Unread message count, bot scenarios, runtime locale override, and audio/video calls are not part of the plugin.
- The Crisp Android and iOS SDKs are proprietary closed-source binaries; the plugin only declares them as dependencies. This project is not affiliated with Crisp IM SAS.
