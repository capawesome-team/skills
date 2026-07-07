# App Icon

Capacitor plugin to change the app icon at runtime by switching between alternate icons the app declares beforehand.

**Package:** `@capawesome/capacitor-app-icon`

**Platforms:** Android, iOS

## Installation

```bash
npm install @capawesome/capacitor-app-icon
npx cap sync
```

The plugin cannot add icons dynamically. Every icon you want to switch to must be declared by the app beforehand.

## Configuration

### Android

Each alternate icon is declared as an `<activity-alias>` that points at the launcher activity. The plugin enables the requested alias and disables the others.

In `android/app/src/main/AndroidManifest.xml`, remove the launcher `<intent-filter>` from `.MainActivity`, then add one `<activity-alias>` for the default icon (enabled) and one for each alternate icon (disabled). Exactly one alias must be `android:enabled="true"`.

```xml
<!-- Remove the <intent-filter> from the MainActivity. -->
<activity
    android:name=".MainActivity"
    android:exported="true"
    ... />

<!-- The default icon (enabled). -->
<activity-alias
    android:name=".AppIconDefault"
    android:enabled="true"
    android:exported="true"
    android:icon="@mipmap/ic_launcher"
    android:roundIcon="@mipmap/ic_launcher_round"
    android:targetActivity=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity-alias>

<!-- An alternate icon (disabled). -->
<activity-alias
    android:name=".AppIconChristmas"
    android:enabled="false"
    android:exported="true"
    android:icon="@mipmap/ic_launcher_christmas"
    android:roundIcon="@mipmap/ic_launcher_christmas_round"
    android:targetActivity=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity-alias>
```

The icon name passed to `setIcon(...)` is the alias name without the leading dot (e.g. `AppIconChristmas`). Add the referenced icon resources (e.g. `@mipmap/ic_launcher_christmas`) to your `res/mipmap-*` folders.

### iOS

Alternate icons are declared under the `CFBundleIcons` key in `ios/App/App/Info.plist`. The icon image files must be bundled with the app (e.g. `AppIconChristmas@2x.png`, `AppIconChristmas@3x.png`).

```xml
<key>CFBundleIcons</key>
<dict>
    <key>CFBundleAlternateIcons</key>
    <dict>
        <key>AppIconChristmas</key>
        <dict>
            <key>CFBundleIconFiles</key>
            <array>
                <string>AppIconChristmas</string>
            </array>
            <key>UIPrerenderedIcon</key>
            <false/>
        </dict>
    </dict>
</dict>
```

The icon name passed to `setIcon(...)` is the key inside `CFBundleAlternateIcons`. The default icon is restored with `resetIcon()`.

## Usage

```typescript
import { AppIcon } from '@capawesome/capacitor-app-icon';

const isAvailable = async () => {
  const { available } = await AppIcon.isAvailable();
  return available;
};

const getCurrentIcon = async () => {
  const { icon } = await AppIcon.getCurrentIcon();
  return icon; // null if the default icon is in use
};

const setIcon = async () => {
  await AppIcon.setIcon({ icon: 'AppIconChristmas' });
};

const resetIcon = async () => {
  await AppIcon.resetIcon();
};
```

## Notes

- `isAvailable()` always resolves to `true` on Android; on iOS it reflects `supportsAlternateIcons`.
- `getCurrentIcon()` returns `null` when the default icon is in use.
- On iOS the system shows a user-visible alert every time the icon changes, and the icon cannot be changed while the app is in the background.
- On Android the behavior after a change depends on the launcher: some apply the new icon only after the app's task is closed, and a few kill the app. Shortcuts pinned to a now-disabled alias may stop working.
- Error codes: `CHANGE_FAILED`, `ICON_NOT_FOUND`.
