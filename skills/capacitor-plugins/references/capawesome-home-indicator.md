# Home Indicator

Hide and show the iOS home indicator at runtime.

**Package:** `@capawesome/capacitor-home-indicator`
**Platforms:** iOS

## Installation

```bash
npm install @capawesome/capacitor-home-indicator
npx cap sync
```

This plugin is iOS-only. On Android and Web, all methods reject as unimplemented. To control the Android navigation bar, use the Navigation Bar plugin instead.

No additional setup is required. The plugin injects the home indicator override into the Capacitor bridge view controller automatically, so you do not need to subclass `CAPBridgeViewController` or edit your `AppDelegate`.

## Usage

```typescript
import { HomeIndicator } from '@capawesome/capacitor-home-indicator';

await HomeIndicator.hide();
await HomeIndicator.show();

const { hidden } = await HomeIndicator.isHidden();
```

## Notes

- **Auto-hide only**: `hide()` sets the home indicator to auto-hide. iOS does not allow permanently removing it — the indicator only fades out while the screen is idle.
- **Reappears on touch**: as soon as the user touches the screen the indicator reappears, then fades out again once the screen becomes idle. This is enforced by iOS and cannot be changed.
- **In-memory state**: `isHidden()` reflects the plugin-managed state, not the actual on-screen visibility (which the system controls). The state is kept in memory only and resets to shown when the app restarts, so call `hide()` again after a relaunch if needed.
- No error codes are defined for this plugin.
