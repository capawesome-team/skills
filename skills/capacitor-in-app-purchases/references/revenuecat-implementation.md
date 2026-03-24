# RevenueCat — Implementation

## Core Implementation

### Configure the SDK

Call `Purchases.configure()` once at app startup before any other RevenueCat methods. This must be the first RevenueCat call in the app.

```typescript
import { Purchases, LOG_LEVEL } from '@revenuecat/purchases-capacitor';

const configurePurchases = async () => {
  // Optional: enable verbose logging during development
  await Purchases.setLogLevel({ level: LOG_LEVEL.VERBOSE });

  await Purchases.configure({
    apiKey: 'your_revenuecat_api_key', // Platform-specific API key from RevenueCat dashboard
    appUserID: 'optional_user_id',     // Omit for anonymous users
  });
};
```

Ask the user for their RevenueCat API key. The API key is platform-specific — use the Apple API key for iOS and the Google API key for Android. For cross-platform apps, use `Capacitor.getPlatform()` to select the correct key:

```typescript
import { Capacitor } from '@capacitor/core';
import { Purchases, LOG_LEVEL } from '@revenuecat/purchases-capacitor';

const configurePurchases = async () => {
  await Purchases.setLogLevel({ level: LOG_LEVEL.VERBOSE });

  const apiKey = Capacitor.getPlatform() === 'ios'
    ? 'appl_your_apple_api_key'
    : 'goog_your_google_api_key';

  await Purchases.configure({ apiKey });
};
```

### Get Offerings

Fetch the available offerings and packages configured in the RevenueCat dashboard:

```typescript
import { Purchases } from '@revenuecat/purchases-capacitor';

const getOfferings = async () => {
  const offerings = await Purchases.getOfferings();
  const currentOffering = offerings.current;
  if (currentOffering) {
    const packages = currentOffering.availablePackages;
    for (const pkg of packages) {
      console.log('Product:', pkg.product.title);
      console.log('Price:', pkg.product.priceString);
      console.log('Identifier:', pkg.product.identifier);
    }
    return packages;
  }
  return [];
};
```

### Get Products Directly

Fetch specific products by their store product identifiers:

```typescript
import { Purchases } from '@revenuecat/purchases-capacitor';

const getProducts = async () => {
  const { products } = await Purchases.getProducts({
    productIdentifiers: ['monthly_sub', 'annual_sub', 'premium_lifetime'],
  });
  return products;
};
```

### Purchase a Package

The recommended approach — purchase a package from an offering:

```typescript
import { Purchases } from '@revenuecat/purchases-capacitor';

const purchasePackage = async (selectedPackage: any) => {
  try {
    const { customerInfo } = await Purchases.purchasePackage({
      aPackage: selectedPackage,
    });
    if (customerInfo.entitlements.active['premium']) {
      // Unlock premium content
    }
  } catch (error: any) {
    if (error.userCancelled) {
      // User cancelled — do nothing
    } else {
      // Handle other purchase errors
      console.error('Purchase failed:', error.message);
    }
  }
};
```

### Purchase a Store Product

Purchase a product directly without using offerings:

```typescript
import { Purchases } from '@revenuecat/purchases-capacitor';

const purchaseProduct = async (product: any) => {
  try {
    const { customerInfo } = await Purchases.purchaseStoreProduct({
      product,
    });
    if (customerInfo.entitlements.active['premium']) {
      // Unlock premium content
    }
  } catch (error: any) {
    if (error.userCancelled) {
      // User cancelled
    } else {
      console.error('Purchase failed:', error.message);
    }
  }
};
```

### Check Entitlements

Check the current user's active entitlements to determine access:

```typescript
import { Purchases } from '@revenuecat/purchases-capacitor';

const checkSubscriptionStatus = async (): Promise<boolean> => {
  const { customerInfo } = await Purchases.getCustomerInfo();
  return !!customerInfo.entitlements.active['premium'];
};
```

### Restore Purchases

Allow users to restore purchases on a new device:

```typescript
import { Purchases } from '@revenuecat/purchases-capacitor';

const restorePurchases = async () => {
  try {
    const { customerInfo } = await Purchases.restorePurchases();
    if (customerInfo.entitlements.active['premium']) {
      // Re-grant access
    }
  } catch (error) {
    console.error('Restore failed:', error);
  }
};
```

### Listen for Customer Info Updates

Subscribe to real-time entitlement changes (e.g., subscription renewed, expired, or cancelled):

```typescript
import { Purchases } from '@revenuecat/purchases-capacitor';

const listenForUpdates = async () => {
  await Purchases.addCustomerInfoUpdateListener((customerInfo) => {
    const isSubscribed = !!customerInfo.entitlements.active['premium'];
    // Update app state based on subscription status
  });
};
```

## User Management

### Identify Users

Associate purchases with a user ID when the user signs in:

```typescript
import { Purchases } from '@revenuecat/purchases-capacitor';

const loginUser = async (userId: string) => {
  const { customerInfo, created } = await Purchases.logIn({
    appUserID: userId,
  });
  console.log('New user created:', created);
  return customerInfo;
};

const logoutUser = async () => {
  const { customerInfo } = await Purchases.logOut();
  return customerInfo;
};
```

### Set Subscriber Attributes

Attach metadata to the subscriber for analytics and integrations:

```typescript
import { Purchases } from '@revenuecat/purchases-capacitor';

await Purchases.setEmail({ email: 'user@example.com' });
await Purchases.setDisplayName({ displayName: 'Jane Doe' });
await Purchases.setAttributes({
  '$email': 'user@example.com',
  'plan_type': 'team',
});
```

## Complete Setup Example

A typical initialization flow:

```typescript
import { Capacitor } from '@capacitor/core';
import { Purchases, LOG_LEVEL } from '@revenuecat/purchases-capacitor';

const initializePurchases = async () => {
  // Enable verbose logging in development
  await Purchases.setLogLevel({ level: LOG_LEVEL.VERBOSE });

  // Configure with platform-specific API key
  const apiKey = Capacitor.getPlatform() === 'ios'
    ? 'appl_your_apple_api_key'
    : 'goog_your_google_api_key';
  await Purchases.configure({ apiKey });

  // Listen for entitlement changes
  await Purchases.addCustomerInfoUpdateListener((customerInfo) => {
    const isSubscribed = !!customerInfo.entitlements.active['premium'];
    // TODO: Update app state
  });

  // Check current entitlements
  const { customerInfo } = await Purchases.getCustomerInfo();
  const isSubscribed = !!customerInfo.entitlements.active['premium'];
  console.log('User subscribed:', isSubscribed);

  // Load offerings for display
  const offerings = await Purchases.getOfferings();
  if (offerings.current) {
    console.log('Available packages:', offerings.current.availablePackages);
  }
};
```

## Notes

- RevenueCat handles receipt validation server-side. There is no need to call `finishTransaction()` manually.
- Use `Purchases.getOfferings()` to fetch products instead of querying the stores directly.
- Check entitlements via `customerInfo.entitlements.active` rather than individual transaction status.
- Amazon Appstore is supported by passing `useAmazon: true` in the `configure()` options.
