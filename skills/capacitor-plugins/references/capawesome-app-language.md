# App Language

Capacitor plugin to manage the app's own language override, independent of the device language.

**Package:** `@capawesome/capacitor-app-language`

**Platforms:** Android, iOS

## Installation

```bash
npm install @capawesome/capacitor-app-language
npx cap sync
```

This plugin only affects natively rendered strings (e.g. permission dialogs, notifications, plugin-presented sheets). The language of your web UI should be handled by your app's i18n library.

## Configuration

### Android

#### Variables

This project variable can be set in `android/app/variables.gradle`:

- `$androidxAppCompatVersion` version of `androidx.appcompat:appcompat` (default: `1.7.1`)

#### Locale Configuration

To integrate the app language into the system settings (Android 13+), declare the supported locales in `android/app/src/main/res/xml/locales_config.xml` and reference it from the `<application>` tag in `android/app/src/main/AndroidManifest.xml`:

```xml
<!-- res/xml/locales_config.xml -->
<?xml version="1.0" encoding="utf-8"?>
<locale-config xmlns:android="http://schemas.android.com/apk/res/android">
    <locale android:name="en" />
    <locale android:name="de" />
</locale-config>
```

```xml
<application
    ...
    android:localeConfig="@xml/locales_config">
```

To persist the selected language across app restarts on Android 12 and below, add the following `<service>` inside the `<application>` tag of `AndroidManifest.xml`:

```xml
<service
    android:name="androidx.appcompat.app.AppLocalesMetadataHolderService"
    android:enabled="false"
    android:exported="false">
    <meta-data
        android:name="autoStoreLocales"
        android:value="true" />
</service>
```

### iOS

On iOS the app language can only be changed by the user in the system settings, so `setLanguage(...)` and `resetLanguage()` are not available. Use `openSettings()` to deep-link the user to the app's settings page.

The per-app language row is only shown if the app bundle provides more than one localization. Declare the supported localizations in `ios/App/App/Info.plist`:

```xml
<key>CFBundleLocalizations</key>
<array>
    <string>en</string>
    <string>de</string>
</array>
```

## Usage

```typescript
import { AppLanguage } from '@capawesome/capacitor-app-language';

const getLanguage = async () => {
  const { languageTag } = await AppLanguage.getLanguage();
  return languageTag; // null if no override is set
};

// Android only.
const setLanguage = async () => {
  await AppLanguage.setLanguage({ languageTag: 'de-DE' });
};

// Android only.
const resetLanguage = async () => {
  await AppLanguage.resetLanguage();
};

const openSettings = async () => {
  await AppLanguage.openSettings();
};
```

## Notes

- Platform support: `getLanguage()` and `openSettings()` work on Android and iOS; `setLanguage(...)` and `resetLanguage()` are Android only (reject as unimplemented on iOS). All methods reject as unimplemented on Web.
- `languageTag` uses BCP 47 tags (e.g. `de-DE`) and is `null` when no override is set and the app follows the device language.
- On Android, setting or resetting the language recreates the current activity, which reloads the web view.
