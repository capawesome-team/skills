# Testing Capacitor Apps

Strategies and tools for unit testing and end-to-end (E2E) testing Capacitor apps.

## Unit Testing

Unit tests for Capacitor apps test the web layer (JavaScript/TypeScript). The native bridge is mocked since unit tests run in Node.js, not in a WebView.

### Mocking Capacitor Plugins

Capacitor plugins can be mocked in unit tests using the testing framework's mocking capabilities.

**Jest example** (React, Vue, Angular):

```typescript
// __mocks__/@capacitor/camera.ts
export const Camera = {
  getPhoto: jest.fn().mockResolvedValue({
    webPath: 'https://example.com/photo.jpg',
    format: 'jpeg',
  }),
  checkPermissions: jest.fn().mockResolvedValue({ camera: 'granted', photos: 'granted' }),
  requestPermissions: jest.fn().mockResolvedValue({ camera: 'granted', photos: 'granted' }),
};
```

Place mocks in a `__mocks__/@capacitor/` directory at the project root (or configure via `moduleNameMapper` in Jest config).

**Vitest example**:

```typescript
import { vi } from 'vitest';

vi.mock('@capacitor/camera', () => ({
  Camera: {
    getPhoto: vi.fn().mockResolvedValue({
      webPath: 'https://example.com/photo.jpg',
      format: 'jpeg',
    }),
  },
}));
```

### Testing Platform-Specific Code

Mock `Capacitor.getPlatform()` and `Capacitor.isNativePlatform()` for platform-conditional logic:

```typescript
vi.mock('@capacitor/core', () => ({
  Capacitor: {
    getPlatform: vi.fn().mockReturnValue('android'),
    isNativePlatform: vi.fn().mockReturnValue(true),
    convertFileSrc: vi.fn((src) => src),
  },
}));
```

### Framework-Specific Test Runners

- **Angular**: `ng test` (Karma/Jasmine) or Jest via `@angular-builders/jest`.
- **React**: `vitest` or `jest` (included with Create React App).
- **Vue**: `vitest` (recommended by Vue CLI and Vite).

## End-to-End (E2E) Testing

E2E tests run the app in a real browser or on a device/emulator and simulate user interactions.

### Web E2E (Cypress, Playwright)

Test the web layer in a browser. Capacitor plugins must be mocked or stubbed since they are not available in a standard browser.

**Cypress** setup:

```bash
npm install cypress --save-dev
npx cypress open
```

**Playwright** setup:

```bash
npm install @playwright/test --save-dev
npx playwright install
npx playwright test
```

These tools test the web app as a PWA. They cannot test native functionality (camera, filesystem, push notifications, etc.).

### Native E2E (Appium, Detox)

For testing native functionality on real devices or emulators:

**Appium**:
- Cross-platform (Android + iOS).
- Tests interact with the native app via the WebDriver protocol.
- Install: `npm install appium --save-dev`

**Detox** (iOS-focused, some Android support):
- Developed by Wix for React Native, but works with any native app.
- Faster than Appium for iOS testing.

### Manual Testing on Devices

For native features, manual testing on physical devices is often the most reliable approach:

```bash
# Build and deploy to a connected device
npx cap run android --target <device-id>
npx cap run ios --target <device-id>

# List available devices
npx cap run android --list
npx cap run ios --list
```

Use `npx cap run --live-reload` to iterate quickly (see `references/live-reload.md`).

## Debugging

### Android WebView Debugging

1. Enable WebView debugging in `capacitor.config.ts`:
   ```typescript
   android: {
     webContentsDebuggingEnabled: true,
   }
   ```
2. Run the app on a device or emulator.
3. Open `chrome://inspect` in Chrome on the development machine.
4. The WebView appears under **Remote Target**. Click **inspect** to open DevTools.

### iOS WebView Debugging

1. Enable WebView debugging in `capacitor.config.ts`:
   ```typescript
   ios: {
     webContentsDebuggingEnabled: true,
   }
   ```
2. Run the app on a simulator or device.
3. Open Safari on macOS > **Develop** menu > select the device > select the WebView.
4. Safari Web Inspector opens with full debugging capabilities.

### Native Logging

- **Android**: View native logs with `adb logcat` or in Android Studio's Logcat panel. Filter by `Capacitor` tag.
- **iOS**: View native logs in Xcode's Console panel. Filter by `Capacitor`.
