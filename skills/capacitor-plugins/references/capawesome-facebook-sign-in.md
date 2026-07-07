# Facebook Sign-In

Unofficial Capacitor plugin to sign in with Facebook. Supports classic login (access token) and Limited Login (authentication token / JWT) on iOS.

**Package:** `@capawesome/capacitor-facebook-sign-in`

**Platforms:** Android, iOS, Web

## Installation

```bash
npm install @capawesome/capacitor-facebook-sign-in
npx cap sync
```

This plugin requires a Facebook app. If you don't have one, create it in the [Meta App Dashboard](https://developers.facebook.com/apps/) and add the Facebook Login product. Note your App ID and Client Token from the app settings.

## Configuration

### Android

#### Variables

This project variable can be set in `android/app/variables.gradle`:

- `$facebookLoginVersion` version of `com.facebook.android:facebook-login` (default: `18.3.0`)

#### Resources

Add the following string resources to `android/app/src/main/res/values/strings.xml`:

```xml
<string name="facebook_app_id">YOUR_APP_ID</string>
<string name="facebook_client_token">YOUR_CLIENT_TOKEN</string>
```

#### Metadata

Add the following inside the `<application>` tag of `android/app/src/main/AndroidManifest.xml`:

```xml
<meta-data android:name="com.facebook.sdk.ApplicationId" android:value="@string/facebook_app_id" />
<meta-data android:name="com.facebook.sdk.ClientToken" android:value="@string/facebook_client_token" />
```

### iOS

Add the following keys to `ios/App/App/Info.plist`:

```xml
<key>FacebookAppID</key>
<string>YOUR_APP_ID</string>
<key>FacebookClientToken</key>
<string>YOUR_CLIENT_TOKEN</string>
<key>FacebookDisplayName</key>
<string>YOUR_APP_NAME</string>
<key>LSApplicationQueriesSchemes</key>
<array>
  <string>fbapi</string>
  <string>fb-messenger-share-api</string>
</array>
```

Also add the URL scheme for your Facebook App ID (your App ID prefixed with `fb`, e.g. `fb1234567890123456`) to `ios/App/App/Info.plist`:

```xml
<key>CFBundleURLTypes</key>
<array>
  <dict>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>fbYOUR_APP_ID</string>
    </array>
  </dict>
</array>
```

## Usage

```typescript
import { FacebookSignIn } from '@capawesome/capacitor-facebook-sign-in';

// Call once before any other method. `appId` is required on Web.
const initialize = async () => {
  await FacebookSignIn.initialize({
    appId: '1234567890123456',
  });
};

const signIn = async () => {
  const result = await FacebookSignIn.signIn();
  console.log(result.accessToken?.token);
  console.log(result.profile.id, result.profile.name, result.profile.email);
};

// iOS only: Limited Login (no tracking, returns a JWT instead of an access token).
const signInWithLimitedLogin = async () => {
  const result = await FacebookSignIn.signIn({
    limitedLogin: true,
    nonce: 'YOUR_NONCE',
  });
  console.log(result.authenticationToken);
};

const getCurrentAccessToken = async () => {
  const { accessToken } = await FacebookSignIn.getCurrentAccessToken();
  return accessToken?.token ?? null;
};

const signOut = async () => {
  await FacebookSignIn.signOut();
};
```

## Notes

- `initialize(...)` must be called once before all other methods. The `appId` option is required on Web; on Android and iOS it overrides the native configuration and is usually not needed.
- `signIn(...)` permissions default to `["public_profile", "email"]`.
- Limited Login (`limitedLogin: true`) and the `nonce` option are iOS only. With Limited Login no access token is returned (`accessToken` is `null`); an `authenticationToken` (JWT) is returned instead.
- If the user has not granted App Tracking Transparency permission, the Facebook SDK may fall back to Limited Login even when classic login was requested, leaving `accessToken` as `null`.
- `authenticationToken` is Android and iOS only. Verify it server-side against [Facebook's public keys](https://www.facebook.com/.well-known/oauth/openid/jwks/); validate access tokens via the Facebook Graph API. Never make authorization decisions from client-side token data.
- Error code: `SIGN_IN_CANCELED`.
