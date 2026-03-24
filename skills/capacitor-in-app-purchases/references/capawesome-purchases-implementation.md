# Capawesome Purchases — Implementation

## Core Implementation

### Check Device Availability

Before making any purchase calls, verify the device supports in-app purchases:

```typescript
import { Purchases } from '@capawesome-team/capacitor-purchases';

const checkAvailability = async (): Promise<boolean> => {
  const { isAvailable } = await Purchases.isAvailable();
  return isAvailable;
};
```

### Fetch Product Details

Retrieve product information to display to the user (name, price, description):

```typescript
import { Purchases } from '@capawesome-team/capacitor-purchases';

// Single product
const getProduct = async (productId: string) => {
  const { product } = await Purchases.getProductById({ productId });
  console.log('Name:', product.displayName);
  console.log('Price:', product.displayPrice);
  console.log('Type:', product.type); // 'consumable' | 'non_consumable' | 'auto_renewable_subscription' | 'non_renewable_subscription'
  return product;
};

// Multiple products
const getProducts = async (productIds: string[]) => {
  const { products } = await Purchases.getProductsByIds({ productIds });
  return products;
};
```

### Purchase a Product

```typescript
import { Purchases } from '@capawesome-team/capacitor-purchases';

const purchaseProduct = async (productId: string) => {
  const { transaction } = await Purchases.purchaseProduct({ productId });
  // Deliver the purchased content or enable the service
  // ...
  // Finish the transaction after delivering the content
  await Purchases.finishTransaction({ transactionId: transaction.id });
};
```

On Android, for subscription products with multiple base plans or offers, pass `basePlanId` and optionally `offerId`:

```typescript
const purchaseSubscription = async (productId: string, basePlanId: string, offerId?: string) => {
  const { transaction } = await Purchases.purchaseProduct({
    productId,
    basePlanId,
    offerId,
  });
  // Deliver content and finish transaction
  await Purchases.finishTransaction({ transactionId: transaction.id });
};
```

### Finish a Transaction

Always call `finishTransaction()` after delivering the purchased content. Unfinished transactions block future purchases on some platforms.

```typescript
await Purchases.finishTransaction({ transactionId: transaction.id });
```

### Handle Unfinished Transactions on App Startup

Check for unfinished transactions every time the app launches. This handles cases where the app crashed or was closed before `finishTransaction()` was called.

```typescript
import { Purchases } from '@capawesome-team/capacitor-purchases';

const processUnfinishedTransactions = async () => {
  const { transactions } = await Purchases.getUnfinishedTransactions();
  for (const transaction of transactions) {
    // Verify and deliver the purchased content if not already delivered
    // ...
    // Finish the transaction
    await Purchases.finishTransaction({ transactionId: transaction.id });
  }
};
```

Call this function early in the app initialization flow.

### Restore Purchases

Allow users to recover purchases on a new device. This must be triggered by an explicit user action (e.g., a "Restore Purchases" button), because on iOS it displays a system authentication dialog.

```typescript
import { Purchases } from '@capawesome-team/capacitor-purchases';

const restorePurchases = async () => {
  await Purchases.syncTransactions();
  const { transactions } = await Purchases.getCurrentTransactions();
  for (const transaction of transactions) {
    // Re-deliver the purchased content or re-enable the service
  }
};
```

Do **not** call `finishTransaction()` after `syncTransactions()` — these transactions have already been processed.

### Get Current Transactions

Retrieve the user's currently active purchases (active subscriptions and non-consumed purchases):

```typescript
import { Purchases } from '@capawesome-team/capacitor-purchases';

const getCurrentPurchases = async () => {
  const { transactions } = await Purchases.getCurrentTransactions();
  return transactions;
};
```

## Receipt Validation

Server-side receipt validation is the developer's responsibility when using Capawesome Purchases.

### iOS

Each transaction includes a `verificationResult` field containing a JWS (JSON Web Signature) token. Send this to the backend for server-side validation against Apple's servers.

```typescript
const transaction = /* from purchaseProduct() or getUnfinishedTransactions() */;
const jws = transaction.verificationResult;
// Send jws to backend for validation
```

### Android

Each transaction includes `token`, `originalJson`, and `signature` fields. Send these to the backend for validation against the Google Play Developer API.

```typescript
const transaction = /* from purchaseProduct() or getUnfinishedTransactions() */;
const purchaseToken = transaction.token;
const originalJson = transaction.originalJson;
const signature = transaction.signature;
// Send to backend for validation via Google Play Developer API
```

## Optional Features

### Check Introductory Offer Eligibility

Determine if a user qualifies for an introductory offer (free trial, introductory price) on a subscription:

```typescript
import { Purchases } from '@capawesome-team/capacitor-purchases';

const checkIntroOffer = async (productId: string) => {
  const { isIntroOfferAvailable } = await Purchases.isIntroOfferAvailableForProduct({
    productId,
  });
  if (isIntroOfferAvailable) {
    // Show introductory offer UI
  }
};
```

### Get All Transactions (iOS Only)

Retrieve the complete purchase history on iOS:

```typescript
import { Purchases } from '@capawesome-team/capacitor-purchases';

const getAllPurchaseHistory = async () => {
  const { transactions } = await Purchases.getAllTransactions();
  return transactions;
};
```

This method is only available on iOS.

## Complete Setup Example

A typical initialization flow combining the core methods:

```typescript
import { Purchases } from '@capawesome-team/capacitor-purchases';

const initializePurchases = async () => {
  // Check if purchases are available on this device
  const { isAvailable } = await Purchases.isAvailable();
  if (!isAvailable) {
    console.warn('In-app purchases are not available on this device');
    return;
  }

  // Process any unfinished transactions from previous sessions
  const { transactions: unfinished } = await Purchases.getUnfinishedTransactions();
  for (const transaction of unfinished) {
    // Verify and deliver content, then finish
    await Purchases.finishTransaction({ transactionId: transaction.id });
  }

  // Load products for display
  const { products } = await Purchases.getProductsByIds({
    productIds: ['product_1', 'product_2', 'subscription_monthly'],
  });
  console.log('Available products:', products);
};
```
