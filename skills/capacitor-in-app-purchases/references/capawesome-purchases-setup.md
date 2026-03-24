# Capawesome Purchases — Installation and Configuration

## Prerequisites

- **Capacitor 8+** app.
- A **Capawesome Insiders** license key. Obtain one at [capawesome.io](https://capawesome.io/).

## Step 1: Configure the Capawesome npm Registry

Check if the `@capawesome-team` npm registry is already configured:

```bash
npm config get @capawesome-team:registry
```

If the registry is **not** configured (returns `undefined`), set it up:

```bash
npm config set @capawesome-team:registry https://npm.registry.capawesome.io
npm config set //npm.registry.capawesome.io/:_authToken <YOUR_LICENSE_KEY>
```

Ask the user for their license key if needed. Wait for confirmation before continuing.

## Step 2: Install the Plugin

```bash
npm install @capawesome-team/capacitor-purchases
npx cap sync
```

## Step 3: Configure Android

Skip if `android/` does not exist.

### Google Play Billing Library Version

The plugin uses Google Play Billing Library 8.2.0 by default. To override, set the `$googlePlayBillingVersion` variable in `android/variables.gradle`:

```diff
 ext {
     // ...
+    googlePlayBillingVersion = '8.2.0'
 }
```

Only add this if a different version is needed. The default (8.2.0) is correct for most projects.

## Step 4: Configure iOS

Skip if `ios/` does not exist.

### Enable In-App Purchase Capability

Add the `In-App Purchase` capability to the iOS target. In the file `ios/App/App.xcodeproj/project.pbxproj`, verify that the `com.apple.InAppPurchase` entitlement is present. If not, the capability must be enabled.

The most reliable way to enable this capability:

1. Open `ios/App/App.xcworkspace` in Xcode.
2. Select the **App** target.
3. Go to **Signing & Capabilities**.
4. Click **+ Capability** and add **In-App Purchase**.

This adds the required entitlement to the project. Alternatively, create or update the file `ios/App/App/App.entitlements`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>com.apple.developer.in-app-payments</key>
    <array>
        <string>merchant.com.example.app</string>
    </array>
</dict>
</plist>
```

### iOS Minimum Deployment Target

The plugin requires **iOS 15.0** or later. Verify the deployment target in `ios/App/App.xcodeproj/project.pbxproj`:

```
IPHONEOS_DEPLOYMENT_TARGET = 15.0;
```

If the current target is lower than 15.0, update **all** occurrences in `project.pbxproj`.

## Step 5: Sync

```bash
npx cap sync
```
