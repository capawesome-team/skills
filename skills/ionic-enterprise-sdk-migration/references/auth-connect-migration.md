# Auth Connect → Capawesome OAuth

Migrate from `@ionic-enterprise/auth` to `@capawesome-team/capacitor-oauth`.

## Feature Mapping

| Feature                             | Auth Connect               | Capawesome OAuth                          |
| ----------------------------------- | -------------------------- | ----------------------------------------- |
| OAuth 2.0 Authorization Code + PKCE | Yes                        | Yes                                       |
| OpenID Connect support              | Yes                        | Yes                                       |
| Automatic endpoint discovery        | Yes                        | Yes                                       |
| Token refresh                       | Yes                        | Yes                                       |
| ID token decoding                   | Yes                        | Yes                                       |
| Access token expiration check       | Yes                        | Yes                                       |
| Multi-provider support              | Yes                        | Yes                                       |
| Android, iOS, Web                   | Yes                        | Yes                                       |
| Secure token storage                | Separate plugin            | Works with `@capawesome-team/capacitor-secure-preferences` |

## Key Differences

- **No setup step**: Auth Connect requires `AuthConnect.setup(...)` before use. The Capawesome OAuth plugin needs no upfront configuration — pass options directly to each method call.
- **No provider classes**: Auth Connect uses provider-specific classes (e.g., `Auth0Provider`). The Capawesome plugin uses a single, universal API for all providers.
- **Issuer URL instead of discovery URL**: Auth Connect requires the full discovery URL. The Capawesome plugin only needs the issuer URL and resolves the discovery document automatically.
- **Scopes as array**: Auth Connect takes a space-delimited string, while the Capawesome plugin takes an array of strings.
- **Individual token parameters**: Auth Connect operates on a combined `AuthResult` object. The Capawesome plugin takes individual values (e.g., `refreshToken`, `accessToken`).

## Step-by-Step Migration

### 1. Remove Auth Connect and install the Capawesome OAuth plugin

```bash
npm uninstall @ionic-enterprise/auth
```

Install the Capawesome OAuth plugin using the `capacitor-plugins` skill or follow the installation instructions in the reference file `references/capawesome-oauth.md` in the `capacitor-plugins` skill.

### 2. Remove the setup call

Delete the `AuthConnect.setup(...)` call. The Capawesome OAuth plugin needs no setup.

```diff
-import { AuthConnect } from '@ionic-enterprise/auth';
-
-await AuthConnect.setup({
-  platform: 'capacitor',
-  logLevel: 'DEBUG',
-  ios: { webView: 'private' },
-  web: { uiMode: 'popup', authFlow: 'PKCE' },
-});
```

### 3. Replace login

**Before:**

```typescript
import { AuthConnect, Auth0Provider } from '@ionic-enterprise/auth';

const provider = new Auth0Provider();
const result = await AuthConnect.login(provider, {
  audience: 'https://api.example.com',
  clientId: 'YOUR_CLIENT_ID',
  discoveryUrl: 'https://example.auth0.com/.well-known/openid-configuration',
  redirectUri: 'com.example.app://callback',
  scope: 'openid profile email offline_access',
  logoutUrl: 'com.example.app://logout',
});
```

**After:**

```typescript
import { Oauth } from '@capawesome-team/capacitor-oauth';

const result = await Oauth.login({
  issuerUrl: 'https://example.auth0.com',
  clientId: 'YOUR_CLIENT_ID',
  redirectUrl: 'com.example.app://oauth/callback',
  scopes: ['openid', 'profile', 'email', 'offline_access'],
  additionalParameters: { audience: 'https://api.example.com' },
});
```

### 4. Replace redirect callback handling (Web only)

**Before:**

```typescript
import { AuthConnect } from '@ionic-enterprise/auth';

const result = await AuthConnect.handleLoginCallback(window.location.href);
```

**After:**

```typescript
import { Oauth } from '@capawesome-team/capacitor-oauth';

const result = await Oauth.handleRedirectCallback();
```

This is only required on the web. On Android and iOS, the redirect is handled natively.

### 5. Replace token refresh

**Before:**

```typescript
import { AuthConnect, Auth0Provider } from '@ionic-enterprise/auth';

const provider = new Auth0Provider();
const newResult = await AuthConnect.refreshSession(provider, authResult);
```

**After:**

```typescript
import { Oauth } from '@capawesome-team/capacitor-oauth';

const result = await Oauth.refreshToken({
  issuerUrl: 'https://example.auth0.com',
  clientId: 'YOUR_CLIENT_ID',
  refreshToken: 'YOUR_REFRESH_TOKEN',
});
```

### 6. Replace token state checks

**Before:**

```typescript
import { AuthConnect } from '@ionic-enterprise/auth';

const isExpired = await AuthConnect.isAccessTokenExpired(authResult);
const isAvailable = await AuthConnect.isAccessTokenAvailable(authResult);
const isRefreshAvailable = await AuthConnect.isRefreshTokenAvailable(authResult);
```

**After:**

```typescript
import { Oauth } from '@capawesome-team/capacitor-oauth';

const { isExpired } = await Oauth.isAccessTokenExpired({ accessTokenExpirationDate });
const { isAvailable } = await Oauth.isAccessTokenAvailable({ accessToken });
const { isAvailable: isRefreshAvailable } = await Oauth.isRefreshTokenAvailable({ refreshToken });
```

### 7. Replace ID token decoding

**Before:**

```typescript
import { AuthConnect } from '@ionic-enterprise/auth';

const decoded = await AuthConnect.decodeToken('id', authResult);
```

**After:**

```typescript
import { Oauth } from '@capawesome-team/capacitor-oauth';

const { header, payload } = await Oauth.decodeIdToken({ token: idToken });
```

### 8. Replace logout

**Before:**

```typescript
import { AuthConnect, Auth0Provider } from '@ionic-enterprise/auth';

const provider = new Auth0Provider();
await AuthConnect.logout(provider, authResult);
```

**After:**

```typescript
import { Oauth } from '@capawesome-team/capacitor-oauth';

await Oauth.logout({
  issuerUrl: 'https://example.auth0.com',
  idToken: 'YOUR_ID_TOKEN',
  postLogoutRedirectUrl: 'com.example.app://oauth/logout',
});
```
