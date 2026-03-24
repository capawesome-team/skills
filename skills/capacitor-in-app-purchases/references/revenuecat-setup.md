# RevenueCat — Installation and Configuration

## Prerequisites

- **Capacitor 8+** app.
- A **RevenueCat** account. Sign up at [revenuecat.com](https://www.revenuecat.com/).
- A RevenueCat project with API keys configured for the target platforms.

## Step 1: Create a RevenueCat Project

If the user does not already have a RevenueCat project:

1. Sign in to the [RevenueCat Dashboard](https://app.revenuecat.com/).
2. Create a new project.
3. Add an **App Store** app (provide the App Store Connect shared secret or App Store Connect API key).
4. Add a **Google Play** app (provide the Google Play service account JSON credentials).
5. Note the **public API key** for each platform (found under **API Keys** in the project settings).

## Step 2: Configure Products in RevenueCat

In the RevenueCat Dashboard:

1. Navigate to **Products** and add the product IDs that were created in App Store Connect and Google Play Console.
2. Navigate to **Entitlements** and create entitlements (e.g., "premium") that map to one or more products.
3. Navigate to **Offerings** and create an offering (e.g., "default") containing packages that reference the products.

This mapping allows RevenueCat to manage which products grant which access levels.

## Step 3: Install the Plugin

```bash
npm install @revenuecat/purchases-capacitor
npx cap sync
```

## Step 4: Configure iOS

Skip if `ios/` does not exist.

### Enable In-App Purchase Capability

Add the `In-App Purchase` capability to the iOS target. In the file `ios/App/App.xcodeproj/project.pbxproj`, verify that the `com.apple.InAppPurchase` entitlement is present. If not, the capability must be enabled.

The most reliable way to enable this capability:

1. Open `ios/App/App.xcworkspace` in Xcode.
2. Select the **App** target.
3. Go to **Signing & Capabilities**.
4. Click **+ Capability** and add **In-App Purchase**.

## Step 5: Configure Android

No additional native configuration is required for Android. The plugin handles Google Play Billing setup internally.

## Step 6: Sync

```bash
npx cap sync
```
