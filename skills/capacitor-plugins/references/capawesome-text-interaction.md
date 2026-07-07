# Text Interaction

Enable and disable text interaction (selection, magnifier, callout menu) in the app's WebView.

**Package:** `@capawesome/capacitor-text-interaction`
**Platforms:** iOS

## Installation

```bash
npm install @capawesome/capacitor-text-interaction
npx cap sync
```

No additional native setup is required.

## Usage

```typescript
import { TextInteraction } from '@capawesome/capacitor-text-interaction';

// Turn off text selection, the selection magnifier and the callout (copy/paste) menu.
await TextInteraction.disable();

// Turn it back on (enabled by default).
await TextInteraction.enable();

const { enabled } = await TextInteraction.isEnabled();
```

## Notes

- **iOS-only.** On Android and Web all methods reject as unimplemented. On Web, use the CSS `user-select` property to prevent text selection instead.
- **Beyond CSS:** `user-select: none` only prevents selection of specific elements. This plugin disables the system text-interaction gestures as a whole — text selection, the selection magnifier, and the callout (copy/paste) menu — which is useful for app-like UIs where accidental text selection breaks the experience.
- Text interaction is **enabled by default**.
- Changes apply to the WebView's configuration. If a change does not take effect immediately on already-rendered content, reload the WebView content to apply it.
