# Google Play Console Setup

Configure in-app purchase products and subscriptions in Google Play Console.

## Prerequisites

- The app must be created in Google Play Console.
- The app must have been uploaded to at least the **internal testing track** (a signed APK or AAB). Products cannot be created until a build has been uploaded.
- All compliance and metadata steps in Google Play Console must be completed.

## Step 1: Complete App Configuration

In [Google Play Console](https://play.google.com/console), select the app and ensure:

1. **App content** declarations are complete (privacy policy, ads declaration, content rating, target audience, etc.).
2. **Store listing** has all required fields filled.
3. At least one **signed build** (APK or AAB) has been uploaded to any track (internal testing is sufficient).

## Step 2: Create In-App Products

Navigate to **Monetize** > **In-app products**.

1. Click **Create product**.
2. Fill in all required fields:
   - **Product ID** — Unique identifier (e.g., `com.example.app.coins_100`). Cannot be changed after creation. Use the same product ID as the equivalent App Store Connect product for cross-platform consistency.
   - **Name** — Display name shown to users.
   - **Description** — Product description shown to users.
   - **Default price** — Set the price. Google auto-converts to other currencies based on the default.
   - **Tax and compliance settings** — Configure as required for the product type.
3. Click **Save** and then **Activate** the product. The product must be in **Active** status to be purchasable.

Google Play does not distinguish between consumable and non-consumable at the product level. Consumption is handled in code by calling `finishTransaction()` for consumables after delivering the content.

## Step 3: Create Subscriptions

Navigate to **Monetize** > **Subscriptions**.

1. Click **Create subscription**.
2. Fill in all required fields:
   - **Product ID** — Unique identifier (e.g., `com.example.app.premium_monthly`). Use the same product ID as the equivalent App Store Connect subscription.
   - **Name** — Display name.
   - **Description** — Subscription description.
3. Add one or more **Base plans**:
   - **Plan ID** — Unique identifier for this base plan (e.g., `monthly`, `annual`).
   - **Billing period** — Weekly, monthly, 3 months, 6 months, or annually.
   - **Price** — Set the price per territory.
   - **Renewal type** — Auto-renewing or prepaid.
4. Optionally add **Offers** within a base plan:
   - **Free trial** — A period before billing starts.
   - **Introductory price** — A discounted price for new subscribers.
   - **Developer-determined offer** — Custom offers targeting specific user segments.
5. **Activate** each base plan. The subscription and its base plans must be Active.

## Step 4: Set Up Real-Time Developer Notifications (Optional)

For subscription lifecycle events (renewals, cancellations, billing issues):

1. Navigate to **Monetize** > **Monetization setup**.
2. Under **Real-time developer notifications**, enter a **Topic name** (Google Cloud Pub/Sub topic).
3. Set up a Google Cloud Pub/Sub subscription to receive notifications.
4. The backend receives notifications for events like `SUBSCRIPTION_RENEWED`, `SUBSCRIPTION_CANCELED`, `SUBSCRIPTION_EXPIRED`, `SUBSCRIPTION_PAUSED`, etc.

## Important Notes

- Products and subscriptions must be in **Active** status to appear in the billing API.
- The app must be installed from Google Play (not sideloaded) for purchases to work, even in testing.
- Product IDs are permanent and cannot be reused after deletion.
- Changes to product pricing may take several hours to propagate.
- For testing, add tester email addresses under **License testing** in Google Play Console settings.
