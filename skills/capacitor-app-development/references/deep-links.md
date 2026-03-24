# Deep Links and Universal Links

Set up deep linking to open specific content in a Capacitor app from external URLs.

## Overview

- **iOS**: Universal Links — uses HTTPS domain verification via `apple-app-site-association`.
- **Android**: App Links — uses HTTPS domain verification via `assetlinks.json`.

Both mechanisms require a two-way association: the app declares it handles URLs for a domain, and the domain hosts a file that confirms the app is authorized.

## Prerequisites

- A domain with HTTPS hosting for the site association files.
- For iOS: Enrollment in the Apple Developer Program.
- For Android: A signing keystore with a known SHA256 fingerprint.

## Step 1: Listen for Deep Link Events

Add a listener for the `appUrlOpen` event using the `@capacitor/app` plugin. This should run early in the app initialization.

```typescript
import { App } from '@capacitor/app';

App.addListener('appUrlOpen', (event) => {
  const url = new URL(event.url);
  const path = url.pathname;
  // Route to the appropriate page based on the path
  // Example: router.push(path);
});
```

Adapt the routing logic to the framework in use (Angular Router, React Router, Vue Router, etc.).

## Step 2: Create Site Association Files

### iOS — `apple-app-site-association`

Create the file (no `.json` extension) and host it at `https://<domain>/.well-known/apple-app-site-association`:

```json
{
  "applinks": {
    "details": [
      {
        "appIDs": ["<TEAM_ID>.<BUNDLE_ID>"],
        "components": [
          {
            "/": "/products/*",
            "comment": "Match all product URLs"
          },
          {
            "/": "/settings",
            "comment": "Match settings page"
          }
        ]
      }
    ]
  }
}
```

- Replace `<TEAM_ID>` with the Apple Developer Team ID (found in the Apple Developer portal under Membership).
- Replace `<BUNDLE_ID>` with the app's bundle identifier (e.g., `com.example.myapp`).
- The `components` array defines which URL paths the app should handle.

The file must be served with `Content-Type: application/json`.

### Android — `assetlinks.json`

Create the file and host it at `https://<domain>/.well-known/assetlinks.json`:

```json
[
  {
    "relation": ["delegate_permission/common.handle_all_urls"],
    "target": {
      "namespace": "android_app",
      "package_name": "<APPLICATION_ID>",
      "sha256_cert_fingerprints": ["<SHA256_FINGERPRINT>"]
    }
  }
]
```

- Replace `<APPLICATION_ID>` with the Android application ID (e.g., `com.example.myapp`).
- Replace `<SHA256_FINGERPRINT>` with the SHA256 fingerprint of the signing certificate.

To obtain the SHA256 fingerprint from a keystore:

```bash
keytool -list -v -keystore /path/to/keystore.jks -alias <alias>
```

Copy the `SHA256` value from the output and format it as colon-separated uppercase hex (e.g., `14:6D:E9:...`).

## Step 3: Configure iOS

Add an Associated Domain capability to the Xcode project:

1. Open `ios/App/App.xcodeproj` in Xcode.
2. Select the **App** target > **Signing & Capabilities** tab.
3. Click **+ Capability** > **Associated Domains**.
4. Add an entry: `applinks:<domain>` (e.g., `applinks:example.com`).

Alternatively, add the associated domain directly in `ios/App/App/App.entitlements`:

```diff
 <?xml version="1.0" encoding="UTF-8"?>
 <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
 <plist version="1.0">
 <dict>
     <key>com.apple.developer.associated-domains</key>
     <array>
+        <string>applinks:example.com</string>
     </array>
 </dict>
 </plist>
```

## Step 4: Configure Android

Add an intent filter to `android/app/src/main/AndroidManifest.xml` inside the `<activity>` tag for the main activity:

```diff
 <activity
     android:name=".MainActivity"
     ...>
+    <intent-filter android:autoVerify="true">
+        <action android:name="android.intent.action.VIEW" />
+        <category android:name="android.intent.category.DEFAULT" />
+        <category android:name="android.intent.category.BROWSABLE" />
+        <data android:scheme="https" android:host="example.com" />
+    </intent-filter>
 </activity>
```

The `android:autoVerify="true"` attribute tells Android to verify the domain association automatically.

## Step 5: Deploy and Verify

### Host the Association Files

Deploy the site association files to the web server. They must be accessible at:

- `https://<domain>/.well-known/apple-app-site-association`
- `https://<domain>/.well-known/assetlinks.json`

For web frameworks, place the files in the correct directory:

- **Angular**: Add to `src/.well-known/` and configure `angular.json` assets array to include `.well-known`.
- **React (CRA / Vite)**: Place in `public/.well-known/`.
- **Vue**: Place in `public/.well-known/`.

### Test

1. Build and deploy the native app to a physical device.
2. Open a link to `https://<domain>/products/123` in the device browser or tap a link from an external app (e.g., Notes, Messages).
3. If configured correctly, the app opens and the `appUrlOpen` listener fires.

### Debugging

- **iOS**: Use Apple's [Search API Validation Tool](https://search.developer.apple.com/appsearch-validation-tool/) to verify the association file.
- **Android**: Use the Digital Asset Links tool at `https://developers.google.com/digital-asset-links/tools/generator` to verify the association file.
- **iOS not opening the app**: Ensure the app was installed from TestFlight or the App Store (development builds may not verify domains). Long-press a link and check if "Open in App" appears.
- **Android not opening the app**: Verify `android:autoVerify="true"` is set. Check that the SHA256 fingerprint matches the signing key used for the installed build.
