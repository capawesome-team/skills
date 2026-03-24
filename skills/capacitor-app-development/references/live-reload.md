# Live Reload

Set up live reload to instantly see web code changes on a native device or emulator without rebuilding the native binary.

## Overview

Live reload connects the native app's WebView to a development server running on the developer's machine instead of loading bundled web assets from the device filesystem. When web code changes, the dev server triggers a reload in the WebView.

## Prerequisites

- The development machine and the target device must be on the **same Wi-Fi network** (for physical devices).
- The development server must bind to `0.0.0.0` to accept connections from the device.

## Option A: `npx cap run --live-reload` (Recommended)

The simplest approach. The Capacitor CLI handles configuration automatically:

```bash
npx cap run android --live-reload --port 8100
npx cap run ios --live-reload --port 8100
```

Options:
- `--port <port>` — Dev server port (default: framework default).
- `--host <host>` — Dev server host.
- `--https` — Use HTTPS instead of HTTP.
- `--forwardPorts <port1:port2>` — Run `adb reverse` for Android (useful when USB debugging).

This command starts the dev server, configures the native project to point to it, builds, and deploys to the device.

## Option B: Ionic CLI

If using the Ionic CLI:

```bash
ionic cap run android -l --external
ionic cap run ios -l --external
```

The `--external` flag binds the dev server to `0.0.0.0`.

## Option C: Manual Configuration

For cases where the CLI options do not work, configure live reload manually.

### Step 1: Find the Development Machine's LAN IP Address

**macOS**:

```bash
ifconfig en0 | grep 'inet '
```

**Windows**:

```bash
ipconfig
```

Look for the `IPv4 Address` on the active network adapter.

### Step 2: Start the Dev Server

Start the framework's dev server bound to `0.0.0.0`:

```bash
# Vite
npx vite --host 0.0.0.0

# Angular
ng serve --host 0.0.0.0

# React (Create React App)
HOST=0.0.0.0 npm run start

# Vue CLI
npx vue-cli-service serve --host 0.0.0.0
```

### Step 3: Update Capacitor Config

Add the server URL to `capacitor.config.ts`:

```typescript
const config: CapacitorConfig = {
  server: {
    url: 'http://<LAN_IP>:<PORT>',
    cleartext: true,
  },
};
```

Replace `<LAN_IP>` with the IP from Step 1 and `<PORT>` with the dev server port.

### Step 4: Copy and Run

```bash
npx cap copy
npx cap run android
# or
npx cap run ios
```

### Step 5: Clean Up

**Remove the `server` block from `capacitor.config.ts` before building a production release.** Do not commit the server URL to version control.

## Troubleshooting

- **Device cannot reach the dev server**: Verify both are on the same network. Check if a firewall is blocking the port. Try pinging the dev machine from the device.
- **Android emulator cannot connect**: The emulator has its own network. Use `10.0.2.2` to reach the host machine from the emulator, or use `--forwardPorts` with `npx cap run`.
- **iOS simulator works but physical device does not**: The simulator shares the host network. Physical devices need the LAN IP. Verify the device is on the same Wi-Fi.
- **Changes not reflecting**: Verify the dev server is running and the URL in `capacitor.config.ts` is correct. Check the WebView's network requests in the browser developer tools.
