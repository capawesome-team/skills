# App Store Connect Setup

Configure in-app purchase products and subscriptions in App Store Connect.

## Step 1: Accept Developer Agreements

Sign in to [Apple Developer](https://developer.apple.com/account/) and accept the following agreements under the **Agreements** section:

1. **Apple Developer Agreement**
2. **Apple Developer Program License Agreement**

Both must show status **Active**.

## Step 2: Verify Business Configuration

In [App Store Connect](https://appstoreconnect.apple.com), navigate to **Business** (or **Agreements, Tax, and Banking** in older UI). Verify the following are all **Active**:

1. **Free Apps Agreement** — Must be active.
2. **Paid Apps Agreement** — Must be active. If missing, click "Request" and complete the process.
3. **Bank account** — Must be configured and verified.
4. **Tax forms** — Must be submitted for applicable regions (varies by country).

If any of these are missing or inactive, in-app purchase products will not be available to users.

## Step 3: Create Products

In App Store Connect, select the app and navigate to **Monetization** > **In-App Purchases** or **Subscriptions**.

### In-App Purchases (Consumables and Non-Consumables)

1. Click the **+** button to create a new in-app purchase.
2. Select the type:
   - **Consumable** — Single-use items (e.g., coins, tokens). Can be purchased multiple times.
   - **Non-Consumable** — Permanent unlocks (e.g., remove ads, premium features). Purchased once.
3. Fill in all required fields:
   - **Reference Name** — Internal name (not shown to users).
   - **Product ID** — Unique identifier (e.g., `com.example.app.coins_100`). Cannot be changed or reused after creation. Use the same product ID as the equivalent Google Play product for cross-platform consistency.
   - **Pricing** — Select a price tier or set a custom price.
   - **Localizations** — Add display name and description for each supported language.
   - **Review Screenshot** — A screenshot for the App Review team.
4. Ensure the product status reaches **Ready to Submit** or **Waiting for Review**.

### Subscriptions

1. Navigate to **Monetization** > **Subscriptions**.
2. Create a **Subscription Group** first (e.g., "Premium Plans"). Subscription groups allow users to upgrade/downgrade between plans in the same group.
3. Within the group, click **+** to create a subscription.
4. Fill in all required fields:
   - **Reference Name** — Internal name.
   - **Product ID** — Unique identifier (e.g., `com.example.app.premium_monthly`). Use the same product ID as the equivalent Google Play subscription.
   - **Subscription Duration** — Weekly, monthly, bimonthly, quarterly, semiannual, or annual.
   - **Pricing** — Set the price for each territory.
   - **Localizations** — Display name and description per language.
   - **Review Screenshot** — Required for review.
5. Optionally configure:
   - **Introductory Offers** — Free trials, pay-as-you-go, or pay-up-front offers for new subscribers.
   - **Promotional Offers** — Discounts for existing or lapsed subscribers (requires server-side signature).
   - **Offer Codes** — Redeemable codes for subscriptions.

## Step 4: Set Up Server Notifications (Optional)

For subscription lifecycle events (renewals, cancellations, expirations, billing issues):

1. In App Store Connect, navigate to **App Information** > **App Store Server Notifications**.
2. Enter the **Production Server URL** and **Sandbox Server URL** pointing to the backend endpoint.
3. Select **Version 2** notifications (recommended).
4. The backend receives signed JWS notifications for events like `DID_RENEW`, `DID_FAIL_TO_RENEW`, `EXPIRED`, `REFUND`, etc.

## Important Notes

- New products may take **up to a few hours** to become available via the StoreKit API after creation.
- Products must have all required fields completed to be testable in Sandbox.
- Product IDs are permanent — they cannot be modified or reused once created.
- For Sandbox testing, products do not need to be in "Approved" status — "Ready to Submit" is sufficient.
