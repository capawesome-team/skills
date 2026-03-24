# Security Best Practices

Security guidelines for Capacitor app development.

## Never Embed Secrets in App Code

Client-side code (JavaScript, HTML, CSS) bundled in the app can be extracted through basic analysis. Never embed:

- API keys with write or admin access
- Encryption keys
- Database credentials
- OAuth client secrets
- Service account keys

**Instead**: Move operations requiring secret keys to a server-side API or serverless function. The app should call the server API, and the server handles the secret.

Read-only API keys (e.g., Google Maps API key restricted by platform) are acceptable in client code but should still be restricted by platform and usage quotas.

## Secure Storage for Sensitive Client Data

Do not store authentication tokens, encryption keys, or credentials in `localStorage`, `IndexedDB`, or `@capacitor/preferences`. These are not encrypted.

**Use native secure storage**:

- **iOS**: Keychain Services (encrypted, hardware-backed).
- **Android**: Android Keystore (hardware-backed key management).

Plugins that wrap these APIs:

- `@capawesome-team/capacitor-secure-preferences` — Encrypted key-value storage.

For plugin installation and configuration, use the `capacitor-plugins` skill.

## HTTPS Only

Never make requests to unencrypted HTTP endpoints. All network requests should use `https://`.

In `capacitor.config.ts`, do not set `server.cleartext: true` in production builds. This setting allows HTTP traffic and should only be used during development (e.g., live reload).

## Content Security Policy (CSP)

Add a Content Security Policy meta tag to `index.html` to restrict which resources the WebView can load:

```html
<meta http-equiv="Content-Security-Policy"
      content="default-src 'self';
               script-src 'self';
               style-src 'self' 'unsafe-inline';
               img-src 'self' data: https:;
               connect-src 'self' https://api.example.com;">
```

Adjust the policy based on the app's requirements:

- `default-src 'self'` — Only allow resources from the app's origin by default.
- `script-src 'self'` — Only allow scripts from the app's origin.
- `connect-src` — Whitelist API endpoints the app communicates with.
- `img-src` — Allow image sources as needed.

## Deep Link Security

- Prefer **Universal Links** (iOS) and **App Links** (Android) over custom URL schemes. Universal Links and App Links are verified via HTTPS domain ownership and cannot be hijacked by other apps.
- Never send sensitive data (tokens, credentials) through URL scheme parameters.
- For OAuth2 flows, use **PKCE (Proof Key for Code Exchange)** to prevent token interception during the callback.

See `references/deep-links.md` for deep link setup.

## WebView Debugging

Disable WebView debugging in production builds:

```typescript
const config: CapacitorConfig = {
  android: {
    webContentsDebuggingEnabled: false,
  },
  ios: {
    webContentsDebuggingEnabled: false,
  },
};
```

When `webContentsDebuggingEnabled` is `true`, anyone with physical access to the device can inspect the WebView's content, network requests, and local storage using Chrome DevTools (Android) or Safari Web Inspector (iOS).

## Code Obfuscation

The web layer is inherently inspectable. To make reverse engineering harder:

- Minify and bundle JavaScript for production (standard build tooling handles this).
- Avoid including proprietary algorithms or business logic in the client code. Move sensitive logic to the server.
- Consider using JavaScript obfuscation tools for additional protection, though obfuscation is not a substitute for proper server-side security.

## Network Certificate Pinning

For apps that communicate with a specific API backend, consider certificate pinning to prevent man-in-the-middle attacks. Certificate pinning ensures the app only trusts a specific server certificate rather than any certificate signed by a trusted CA.

This requires native implementation on each platform and must be updated when server certificates rotate.

## Privacy Manifest (iOS)

Starting with iOS 17, Apple requires apps to declare the reasons for using specific APIs in a Privacy Manifest (`PrivacyInfo.xcprivacy`). This file is located at `ios/App/App/PrivacyInfo.xcprivacy`.

Capacitor plugins that access privacy-sensitive APIs should include their own privacy manifests. When adding plugins, verify they provide the required privacy declarations.
