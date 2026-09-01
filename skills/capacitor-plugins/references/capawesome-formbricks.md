# Formbricks

Unofficial Capacitor plugin for Formbricks to run in-app surveys, identify users, and track actions.

**Package:** `@capawesome/capacitor-formbricks`
**Platforms:** Android, iOS, Web
**Documentation:** https://capawesome.io/docs/sdks/capacitor/formbricks/

## Installation

```bash
npm install @capawesome/capacitor-formbricks @formbricks/js
npx cap sync
```

`@formbricks/js` is required for the web implementation.

Requires a [Formbricks](https://formbricks.com/) project (cloud or self-hosted). You need the **app URL** of the instance and the **environment ID** of the project.

## Configuration

### Android

#### Variables

Optionally defined in your app's `variables.gradle` to resolve dependency conflicts:

- `$formbricksAndroidVersion` version of `com.formbricks:android` (default: `1.2.0`)

#### Data Binding

**Required.** The Formbricks Android SDK needs Data Binding enabled in `android/app/build.gradle`:

```groovy
android {
    buildFeatures {
        dataBinding true
    }
}
```

Android builds fail without this.

### iOS

#### Minimum Deployment Target

**Required.** The Formbricks iOS SDK requires iOS `16.6` or higher.

With Swift Package Manager, replace every `IPHONEOS_DEPLOYMENT_TARGET` entry in `ios/App/App.xcodeproj/project.pbxproj`:

```diff
-IPHONEOS_DEPLOYMENT_TARGET = 15.0;
+IPHONEOS_DEPLOYMENT_TARGET = 16.6;
```

With CocoaPods, update `ios/App/Podfile`:

```ruby
platform :ios, '16.6'
```

## Usage

### Set up the SDK

```typescript
import { Formbricks } from '@capawesome/capacitor-formbricks';

await Formbricks.setup({
  appUrl: 'https://app.formbricks.com',
  environmentId: 'YOUR_ENVIRONMENT_ID',
});
```

### Identify the user and set attributes

```typescript
import { Formbricks } from '@capawesome/capacitor-formbricks';

await Formbricks.setUserId({ userId: 'user-123' });

await Formbricks.setAttribute({ key: 'plan', value: 'pro' });
await Formbricks.setAttributes({
  attributes: { plan: 'pro', tier: 'gold' },
});
```

### Track an action

```typescript
import { Formbricks } from '@capawesome/capacitor-formbricks';

await Formbricks.track({ action: 'button_pressed' });
```

### Set the survey language and log out

```typescript
import { Formbricks } from '@capawesome/capacitor-formbricks';

await Formbricks.setLanguage({ language: 'de' });
await Formbricks.logout();
```

## Notes

- `setup(...)` must be called before every other method.
- `track(...)` only reports an action that *may* trigger a survey. Whether a survey is actually shown depends on the survey configuration in the Formbricks project.
- `appUrl` also accepts the URL of a self-hosted Formbricks instance.
- Attribute values are strings only; use `setAttributes(...)` to set several at once.
- `logout()` logs out the current user and clears all attributes.
- `setLanguage(...)` takes a language code such as `de`.
- The plugin has no listeners and no Capacitor config options.
- This project is not affiliated with Formbricks GmbH.
