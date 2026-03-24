---
name: capacitor-in-app-purchases
description: "Guides the agent through setting up in-app purchases in Capacitor apps. Covers App Store Connect and Google Play Console product configuration, plugin selection (Capawesome Purchases vs. RevenueCat), plugin installation and platform setup, purchase flows for subscriptions and consumables, receipt validation, restore purchases, handling unfinished transactions, and testing with sandbox and StoreKit environments. Do not use for non-Capacitor mobile frameworks, Stripe payment processing, or physical goods checkout."
metadata:
  author: capawesome-team
  source: https://github.com/capawesome-team/skills/tree/main/skills/capacitor-in-app-purchases
---

# Capacitor In-App Purchases

Set up in-app purchases and subscriptions in Capacitor apps, covering store configuration, plugin integration, purchase flows, receipt validation, and testing.

## Prerequisites

1. **Capacitor 6, 7, or 8** app (Capawesome Purchases requires Capacitor 8+; RevenueCat requires Capacitor 8+).
2. Node.js and npm installed.
3. For **iOS**: A paid [Apple Developer Program](https://developer.apple.com/programs/) membership, Xcode installed, and an app created in [App Store Connect](https://appstoreconnect.apple.com).
4. For **Android**: A [Google Play Developer](https://play.google.com/console) account, Android Studio installed, and an app created in Google Play Console.

## Agent Behavior

- **Guide step-by-step.** Walk the user through the process one step at a time. Never present multiple unrelated questions at once.
- **Auto-detect before asking.** Check the project for platforms (`android/`, `ios/`), `package.json` dependencies, and Capacitor version. Only ask the user when something cannot be detected.
- **One decision at a time.** When a step requires user input, ask that single question, wait for the answer, then continue.
- **Present clear options.** Provide concrete choices (e.g., "Which purchases plugin do you want to use? (1) Capawesome Purchases (2) RevenueCat") instead of open-ended questions.

## Procedures

### Step 1: Analyze the Project

Auto-detect the following by reading project files — do **not** ask the user for information that can be inferred:

1. **Platforms**: Check which directories exist (`android/`, `ios/`). These are the platforms to configure.
2. **Capacitor version**: Read `@capacitor/core` version from `package.json`.
3. **Existing purchases plugins**: Check if `@capawesome-team/capacitor-purchases`, `@revenuecat/purchases-capacitor`, or any other in-app purchase plugin is already in `package.json`. If found, inform the user and ask whether to continue with the existing plugin or switch.

### Step 2: Configure Products in App Store Connect (iOS)

Skip if `ios/` does not exist.

Read `references/app-store-connect-setup.md` and guide the user through creating in-app purchase products and subscriptions in App Store Connect. This covers:

- Accepting required developer agreements
- Verifying business configuration (Paid Apps Agreement, bank account, tax forms)
- Creating in-app purchase products (consumables, non-consumables, subscriptions)
- Subscription group setup
- Product status requirements

### Step 3: Configure Products in Google Play Console (Android)

Skip if `android/` does not exist.

Read `references/google-play-console-setup.md` and guide the user through creating in-app purchase products in Google Play Console. This covers:

- Completing compliance and metadata requirements
- Creating in-app products and subscriptions
- Configuring pricing and tax settings
- Activating products

### Step 4: Choose a Purchases Plugin

Ask the user which plugin to use:

1. **Capawesome Purchases** (`@capawesome-team/capacitor-purchases`) — Lightweight, no third-party backend dependency. Requires a Capawesome Insiders license. Server-side receipt validation is the developer's responsibility.
2. **RevenueCat** (`@revenuecat/purchases-capacitor`) — Full backend service with server-side receipt validation, entitlement management, analytics, and integrations. Requires a RevenueCat account.

**Default recommendation**: Capawesome Purchases for apps that already have a backend for receipt validation or want to avoid third-party dependencies. RevenueCat for apps that want managed receipt validation and subscription analytics out of the box.

### Step 5: Install and Configure the Plugin

Based on the user's choice in Step 4:

- **Capawesome Purchases**: Read `references/capawesome-purchases-setup.md` and follow the installation and platform configuration steps.
- **RevenueCat**: Read `references/revenuecat-setup.md` and follow the installation and platform configuration steps.

### Step 6: Implement Purchase Flows

Based on the user's plugin choice:

- **Capawesome Purchases**: Read `references/capawesome-purchases-implementation.md` and add the purchase code to the project.
- **RevenueCat**: Read `references/revenuecat-implementation.md` and add the purchase code to the project.

Adapt imports and code structure to match the user's framework (Angular, React, Vue, etc.).

The implementation covers:

1. Fetching product details
2. Purchasing a product
3. Finishing transactions (Capawesome) or checking entitlements (RevenueCat)
4. Restoring purchases
5. Handling unfinished transactions on app startup
6. Receipt validation strategy

### Step 7: Configure Optional Features

Ask the user which optional features to enable:

1. **Subscription management** — Display subscription status, handle renewals and expirations.
2. **Introductory offer eligibility** — Check if a user qualifies for introductory pricing (Capawesome Purchases only).
3. **Server notifications** — Set up App Store Server Notifications and/or Google Play Real-time Developer Notifications for subscription lifecycle events.

For each selected feature, read the corresponding implementation reference file and apply the relevant code.

### Step 8: Test Purchases

Read `references/testing.md` and guide the user through testing in-app purchases on both platforms. This covers:

- iOS Sandbox testing with test accounts
- iOS StoreKit Testing in Xcode
- Android testing with Google Play Console internal track and license testing accounts
- Common testing pitfalls and debugging tips

### Step 9: Sync and Verify

```bash
npx cap sync
```

Build and run on each platform to verify purchases work end-to-end:

```bash
npx cap run android
npx cap run ios
```

## Error Handling

- **Products not appearing**: On iOS, products may take up to a few hours to become available after creation in App Store Connect. Verify product status is "Ready to Submit" or "Approved". On Android, verify products are in "Active" status and the app has been uploaded to at least the internal testing track.
- **Purchase fails silently on Android**: The app must be installed from Google Play (not via direct APK install). Upload the app to the internal testing track, add test accounts under License Testing, and install from Google Play.
- **iOS sandbox purchase loops or fails**: Ensure the device is signed into a Sandbox test account (Settings > App Store > Sandbox Account on iOS 14+). Do not use a production Apple ID for testing.
- **`purchaseProduct()` returns error on Android**: Verify `google-services.json` is present and the Google Play Billing Library version is compatible. Check `$googlePlayBillingVersion` in `android/variables.gradle`.
- **Receipt validation fails**: On iOS, verify the JWS token via `transaction.verificationResult`. On Android, validate `transaction.token`, `transaction.originalJson`, and `transaction.signature` against the Google Play Developer API.
- **Restore purchases shows nothing**: On iOS, `syncTransactions()` displays a system authentication dialog — it must be called in response to an explicit user action (e.g., a "Restore Purchases" button tap). On Android, it runs silently.
- **`finishTransaction()` not called**: Unfinished transactions block future purchases on some platforms. Always call `finishTransaction()` after delivering content. Check for unfinished transactions at every app launch.
- **Plugin not found at runtime**: Ensure `npx cap sync` was run after installation. For Capawesome Purchases, verify the Capawesome npm registry is configured.
- **Capacitor version mismatch**: Both Capawesome Purchases and RevenueCat require Capacitor 8+. Check `@capacitor/core` version in `package.json`.

## Related Skills

- **`capacitor-plugins`** — For general Capacitor plugin installation and configuration, including the Capawesome Purchases and RevenueCat plugins.
- **`capawesome-cloud`** — For app store publishing after in-app purchases are configured.
