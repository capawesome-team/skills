# Testing In-App Purchases

## iOS Testing

### Option 1: Sandbox Testing

Sandbox testing uses test accounts to simulate purchases against Apple's sandbox environment on a physical device.

#### Create a Sandbox Test Account

1. In [App Store Connect](https://appstoreconnect.apple.com), navigate to **Users and Access** > **Sandbox** > **Testers**.
2. Click **+** to create a new sandbox tester.
3. Use a unique email address that is **not** associated with any real Apple ID.
4. Set a password and fill in the required fields.

#### Test on a Physical Device

1. On the iOS device, go to **Settings** > **App Store** > **Sandbox Account** (iOS 14+) and sign in with the sandbox tester credentials.
2. Build and run the app on the device:
   ```bash
   npx cap run ios
   ```
3. Trigger a purchase in the app. The system payment sheet will display "[Environment: Sandbox]".
4. Sandbox subscriptions renew at accelerated rates:
   - 1 week = 3 minutes
   - 1 month = 5 minutes
   - 2 months = 10 minutes
   - 3 months = 15 minutes
   - 6 months = 30 minutes
   - 1 year = 1 hour
5. Sandbox subscriptions auto-renew up to 12 times, then expire.

#### Important Notes

- Sandbox testing does **not** work on the iOS Simulator — use a physical device.
- Do not sign into the sandbox account via the main App Store sign-in (Settings > Apple ID). Use only the Sandbox Account section.
- If prompted for a password during purchase, enter the sandbox account password.

### Option 2: StoreKit Testing in Xcode

StoreKit Testing uses a local configuration file to simulate purchases without connecting to Apple's servers. Available on iOS 14+.

#### Set Up StoreKit Configuration

1. In Xcode, select **File** > **New** > **File** > **StoreKit Configuration File**.
2. Name the file (e.g., `StoreKitConfig.storekit`) and save it in the project.
3. Add products to the configuration file matching the product IDs created in App Store Connect.
4. In the Xcode scheme settings (**Product** > **Scheme** > **Edit Scheme** > **Run** > **Options**), set **StoreKit Configuration** to the created file.

#### Run and Test

1. Build and run the app in the Simulator or on a physical device.
2. Purchases complete instantly without authentication prompts.
3. Use the **Transaction Manager** in Xcode (Debug > StoreKit > Manage Transactions) to view, approve, refund, or modify transactions.

#### Advantages

- Works in the Simulator.
- No sandbox account required.
- Instant transactions.
- Full control over transaction states (refunds, renewals, failures).

#### Limitations

- Does not test the real payment flow or Apple server communication.
- Not suitable for testing server-side receipt validation.

## Android Testing

### Set Up Internal Testing

1. In [Google Play Console](https://play.google.com/console), navigate to the app > **Testing** > **Internal testing**.
2. Create a new release and upload a signed AAB or APK.
3. Add testers by email under the internal testing track's **Testers** tab.
4. Share the opt-in link with testers so they can install the app from Google Play.

### Configure License Testing

1. In Google Play Console, navigate to **Settings** > **License testing**.
2. Add the Gmail addresses of the testers.
3. Set the **License response** to `RESPOND_NORMALLY` for realistic testing.

### Test on a Physical Device

1. The tester must install the app **from Google Play** via the internal testing opt-in link. Direct APK installs will not work for purchase testing.
2. Launch the app and trigger a purchase. The Google Play payment sheet will display "This is a test order" for license testers.
3. Test cards: license testers can use any payment method. The purchase completes without actual charges.

### Important Notes

- The app must be installed from Google Play — sideloaded APKs cannot make purchases.
- There may be a delay of several hours between uploading a new build and it being available for testers.
- Subscription test purchases use real-time durations (no acceleration like iOS Sandbox). However, license testers can cancel and re-subscribe freely.
- If purchases fail with "Item not found" or "Item unavailable", verify:
  1. The product is in **Active** status in Google Play Console.
  2. The app's `applicationId` in `build.gradle` matches the package name in Google Play Console.
  3. The app was installed from Google Play (not sideloaded).
  4. The tester's Google account is added to License Testing.

## General Debugging Tips

- **Enable verbose logging** in the purchases plugin during development to see detailed purchase flow logs.
- **Check product IDs** — ensure the product IDs in code exactly match those in App Store Connect and Google Play Console (case-sensitive).
- **Wait after creating products** — new products may take up to a few hours to propagate on both platforms.
- **Test restore purchases** — sign in on a different device or after reinstalling the app and verify that `syncTransactions()` (Capawesome) or `restorePurchases()` (RevenueCat) recovers the purchases.
- **Test error states** — cancel a purchase mid-flow, test with an expired subscription, and test with no network connection to verify error handling.
