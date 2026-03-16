# iOS Signing Certificates

To build signed iOS apps in Capawesome Cloud, configure a signing certificate with a `.p12` file and one or more provisioning profiles (`.mobileprovision`).

## Upload via CLI

```bash
npx @capawesome/cli apps:certificates:create \
  --app-id <APP_ID> \
  --name "Production iOS Certificate" \
  --platform ios \
  --type production \
  --file /path/to/certificate.p12 \
  --password <CERTIFICATE_PASSWORD> \
  --provisioning-profile /path/to/profile.mobileprovision
```

For apps with extensions, specify multiple provisioning profiles:

```bash
npx @capawesome/cli apps:certificates:create \
  --app-id <APP_ID> \
  --name "Production iOS Certificate" \
  --platform ios \
  --type production \
  --file /path/to/certificate.p12 \
  --password <CERTIFICATE_PASSWORD> \
  --provisioning-profile /path/to/app.mobileprovision \
  --provisioning-profile /path/to/widget.mobileprovision
```

Options:
- `--type`: `development` or `production`
- `--file`: Path to `.p12` file
- `--password`: Password for the `.p12` file
- `--provisioning-profile`: Path to `.mobileprovision` file (repeatable for multiple targets)

## Upload via Console

Navigate to [Signing Certificates](https://console.cloud.capawesome.io/apps/_/certificates) and provide:

- **Name**: Descriptive name (e.g., "Production iOS Certificate")
- **Platform**: `iOS`
- **Type**: `Development` or `Production`
- **Certificate File**: The `.p12` file
- **Certificate Password**: Password for the `.p12` file
- **Provisioning Profiles**: One or more `.mobileprovision` files

## Create a .p12 Certificate

### Export from Keychain Access

If a signing certificate is already installed on the user's Mac:

1. Open **Keychain Access** (Applications > Utilities > Keychain Access).
2. Select **login** > **My Certificates** in the sidebar.
3. Find the certificate:
   - **Apple Distribution: [Name]** for App Store/Production builds
   - **Apple Development: [Name]** for development builds
4. Expand the certificate to reveal the private key (key icon).
5. Select **both** the certificate and its private key (Command+click).
6. Right-click > **Export 2 items...**.
7. Choose a filename (e.g., `certificate.p12`), format **Personal Information Exchange (.p12)**.
8. Click **Save** and set a password when prompted.

**Important:** Both the certificate and its private key must be exported together. If no private key appears under the certificate, it was installed without the key.

### Create in Apple Developer Portal

1. Sign in to [Apple Developer](https://developer.apple.com/account).
2. Navigate to **Certificates, Identifiers & Profiles > Certificates**.
3. Click **+** to create a new certificate.
4. Select type: **Apple Development** (dev) or **Apple Distribution** (production).
5. Create a Certificate Signing Request (CSR):
   1. Open **Keychain Access**.
   2. Menu: **Keychain Access > Certificate Assistant > Request a Certificate From a Certificate Authority**.
   3. Enter email and name, select **Saved to disk**, click **Continue**.
6. Upload the CSR in the Apple Developer portal.
7. Download the `.cer` file and double-click to install in Keychain Access.
8. Follow the **Export from Keychain Access** steps above to get a `.p12` file.

## Obtain Provisioning Profiles

### Find Locally

If previously built in Xcode, profiles may exist locally:

```bash
for f in ~/Library/MobileDevice/Provisioning\ Profiles/*.mobileprovision; do echo "=== $f ==="; security cms -D -i "$f" 2>/dev/null | grep -A 1 'application-identifier\|<key>Name</key>\|ExpirationDate' | grep -v '^--$'; echo; done
```

Copy the desired profile:

```bash
cp ~/Library/MobileDevice/Provisioning\ Profiles/<profile-uuid>.mobileprovision ~/Desktop/
```

### Create in Apple Developer Portal

1. Sign in to [Apple Developer](https://developer.apple.com/account).
2. Navigate to **Certificates, Identifiers & Profiles > Profiles**.
3. Click **+** to create a new profile.
4. Select type:
   - **iOS App Development** for development
   - **App Store** for App Store/TestFlight
   - **Ad Hoc** for enterprise/beta distribution
5. Select App ID, certificates, and (for dev profiles) devices.
6. Name the profile and click **Generate**.
7. Download the `.mobileprovision` file.

Repeat for each app extension (each requires its own provisioning profile matching its bundle identifier).

**Note:** Provisioning profiles expire (typically after one year). Regenerate and re-upload if builds fail due to expired profiles.

## Custom Provisioning Profile Mapping

For apps with multiple targets using non-standard bundle ID patterns, set the `IOS_PROVISIONING_PROFILE_MAP` environment variable to a JSON object mapping bundle IDs to target names:

```json
{"com.example.wireguard":"WireGuardExtension","com.example.tunnelprovider":"PacketTunnel"}
```

Set this variable in the build environment. Capawesome Cloud uses these mappings in addition to automatic detection for the main app target.

## List Certificates

```bash
npx @capawesome/cli apps:certificates:list --app-id <APP_ID> --platform ios --json
```

## Delete a Certificate

```bash
npx @capawesome/cli apps:certificates:delete --app-id <APP_ID> --name "Production iOS Certificate" --platform ios --yes
```
