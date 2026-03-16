# Apple App Store Credentials

Two authentication methods are available for Apple App Store destinations.

## Option 1: API Key (Recommended)

Requires: `.p8` API key file, Key ID, Issuer ID, and Team ID.

### Team ID

1. Sign in to the [Apple Developer account](https://developer.apple.com/account).
2. Scroll down to **Membership Details**.
3. Copy the **Team ID**.

### API Key File, Key ID, and Issuer ID

1. Sign in to [App Store Connect](https://appstoreconnect.apple.com).
2. Navigate to **Users and Access**.
3. Select the **Integrations** tab, then select **App Store Connect API**. If this tab is not visible, the account does not have the necessary permissions (Admin or App Manager role required).
4. Click the **+** button to create a new **Team Key**.
5. Enter a name for the key (e.g., "Capawesome Cloud") and select **App Manager** as the access level.
6. Click **Generate**.
7. Copy the **Issuer ID** displayed at the top of the page.
8. Copy the **Key ID** from the newly created key row.
9. Click **Download API Key** to download the `.p8` file. This file can only be downloaded once — store it securely.

## Option 2: Apple ID + Password

Requires: Apple ID email, Apple App ID, app-specific password, and Team ID.

### Team ID

Same as Option 1 above.

### Apple ID

The email address associated with the Apple Developer account.

### Apple App ID

1. Sign in to [App Store Connect](https://appstoreconnect.apple.com).
2. Navigate to **Apps** and select the app.
3. Go to **App Information**.
4. Copy the **Apple-ID** value (a numeric identifier).

### App-Specific Password

1. Sign in to the [Apple ID account page](https://appleid.apple.com).
2. Navigate to the **Security** section.
3. Under **App-Specific Passwords**, click **Generate Password**.
4. Enter a descriptive label (e.g., "Capawesome Cloud").
5. Copy the generated password and store it securely.

## Troubleshooting

- **"Integrations" tab not visible in App Store Connect** → the account does not have Admin or App Manager role. Request elevated permissions from the account holder.
- **API key download unavailable** → the `.p8` file can only be downloaded once. If lost, delete the key and create a new one.
- **App-specific password not working** → ensure two-factor authentication is enabled on the Apple ID account. Generate a new app-specific password if the existing one was revoked.
