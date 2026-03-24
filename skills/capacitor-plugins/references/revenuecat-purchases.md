# RevenueCat Purchases

Capacitor plugin for in-app purchases and subscriptions powered by RevenueCat. Provides server-side receipt validation, cross-platform entitlement management, analytics, and integrations. Wraps StoreKit 2 (iOS) and Google Play Billing (Android) with the RevenueCat backend.

**Package:** `@revenuecat/purchases-capacitor`
**Platforms:** Android, iOS

## Installation

```bash
npm install @revenuecat/purchases-capacitor
npx cap sync
```

## Configuration

### iOS

#### Capabilities

Enable the `In-App Purchase` capability in Xcode under Signing & Capabilities. See [Add a capability to a target](https://help.apple.com/xcode/mac/current/#/dev88ff319e7).

### Android

No additional configuration required.

## Usage

### Configure the SDK

```typescript
import { Purchases, LOG_LEVEL } from '@revenuecat/purchases-capacitor';

await Purchases.setLogLevel({ level: LOG_LEVEL.VERBOSE }); // Optional: enable verbose logging

await Purchases.configure({
  apiKey: 'your_revenuecat_api_key',
  appUserID: 'optional_user_id', // Omit for anonymous users
});
```

### Get offerings

```typescript
import { Purchases } from '@revenuecat/purchases-capacitor';

const offerings = await Purchases.getOfferings();
const currentOffering = offerings.current;
if (currentOffering) {
  const packages = currentOffering.availablePackages;
  for (const pkg of packages) {
    console.log('Product:', pkg.product.title, pkg.product.priceString);
  }
}
```

### Purchase a package

```typescript
import { Purchases } from '@revenuecat/purchases-capacitor';

try {
  const { customerInfo } = await Purchases.purchasePackage({ aPackage: selectedPackage });
  if (customerInfo.entitlements.active['premium']) {
    // Unlock premium content
  }
} catch (error) {
  // Handle purchase error (user cancelled, payment failed, etc.)
}
```

### Purchase a store product

```typescript
import { Purchases } from '@revenuecat/purchases-capacitor';

const { products } = await Purchases.getProducts({
  productIdentifiers: ['monthly_sub', 'annual_sub'],
});

const { customerInfo } = await Purchases.purchaseStoreProduct({
  product: products[0],
});
```

### Restore purchases

```typescript
import { Purchases } from '@revenuecat/purchases-capacitor';

const { customerInfo } = await Purchases.restorePurchases();
if (customerInfo.entitlements.active['premium']) {
  // Re-grant access
}
```

### Get customer info

```typescript
import { Purchases } from '@revenuecat/purchases-capacitor';

const { customerInfo } = await Purchases.getCustomerInfo();
const isSubscribed = !!customerInfo.entitlements.active['premium'];
```

### Listen for customer info updates

```typescript
import { Purchases, CustomerInfo } from '@revenuecat/purchases-capacitor';

await Purchases.addCustomerInfoUpdateListener((customerInfo: CustomerInfo) => {
  const isSubscribed = !!customerInfo.entitlements.active['premium'];
});
```

### Identify and log out users

```typescript
import { Purchases } from '@revenuecat/purchases-capacitor';

// Log in (associate purchases with a user ID)
const { customerInfo, created } = await Purchases.logIn({ appUserID: 'user_123' });

// Log out (switch back to anonymous)
const { customerInfo: anonInfo } = await Purchases.logOut();
```

### Set subscriber attributes

```typescript
import { Purchases } from '@revenuecat/purchases-capacitor';

await Purchases.setAttributes({ '$email': 'user@example.com', 'plan_type': 'team' });
await Purchases.setEmail({ email: 'user@example.com' });
await Purchases.setDisplayName({ displayName: 'Jane Doe' });
```

## Notes

- A RevenueCat account and API key are required. Sign up at [revenuecat.com](https://www.revenuecat.com/).
- Products and entitlements must be configured in the RevenueCat dashboard before use.
- `configure()` must be called once before all other methods, typically at app startup.
- RevenueCat handles receipt validation server-side; you do not need to finish transactions manually.
- Use `Purchases.getOfferings()` to fetch products instead of querying the stores directly.
- Check entitlements via `customerInfo.entitlements.active` rather than individual transaction status.
- For Android testing: upload the app to Google Play Console (internal testing track), add test accounts under License Testing, and install from Google Play.
- For iOS testing: use Sandbox test accounts or StoreKit Testing in Xcode.
- Amazon Appstore is supported by passing `useAmazon: true` in the configure options.
- The plugin requires Capacitor 8 or later.
