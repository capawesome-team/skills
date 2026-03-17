# iOS Signing Certificates

To build signed iOS apps in Capawesome Cloud, configure a signing certificate with a `.p12` file and one or more provisioning profiles (`.mobileprovision`).

## Obtain .p12 and Provisioning Profile

Ask the user if they already have both a `.p12` certificate file and a `.mobileprovision` provisioning profile file.

- **Both files exist** → Skip to [Upload via CLI](#upload-via-cli) or [Upload via Console](#upload-via-console).
- **Have `.p12` but no provisioning profile** → Skip to [Obtain Provisioning Profiles (Manual)](#obtain-provisioning-profiles-manual) or the provisioning profile steps in [Create with ASC CLI](#create-with-asc-cli) (starting at Step 2 and Step 6–8).
- **Have provisioning profile but no `.p12`** → Skip to [Create a .p12 Certificate (Manual)](#create-a-p12-certificate-manual) or the certificate steps in [Create with ASC CLI](#create-with-asc-cli) (Steps 3–5).
- **Neither file exists** → Ask the user which method they prefer:
  - **ASC CLI (Recommended)** → [Create with ASC CLI](#create-with-asc-cli)
  - **Manual** → [Create a .p12 Certificate (Manual)](#create-a-p12-certificate-manual) followed by [Obtain Provisioning Profiles (Manual)](#obtain-provisioning-profiles-manual)

## Create with ASC CLI

Use the [App Store Connect CLI (`asc`)](https://github.com/rudrankriyam/App-Store-Connect-CLI) to create signing certificates and provisioning profiles from the command line.

### Prerequisites

- `asc` CLI installed (see [installation instructions](https://github.com/rudrankriyam/App-Store-Connect-CLI))
- An App Store Connect API key with **Admin** or **Developer** role

### Step 1: Authenticate

First, check if an active session already exists:

```bash
asc auth status
```

If already authenticated, skip to Step 2.

Otherwise, ask the user for the following App Store Connect API key details:
- **Key ID** (e.g., `ABC123`)
- **Issuer ID** (e.g., `DEF456`)
- **Private key file path** (e.g., `/path/to/AuthKey.p8`)

Then run:

```bash
asc auth login \
  --name "MyKey" \
  --key-id "<KEY_ID>" \
  --issuer-id "<ISSUER_ID>" \
  --private-key <PRIVATE_KEY_PATH>
```

### Step 2: Register or Verify Bundle ID

Ask the user for their app's **bundle identifier** (e.g., `com.example.app`) and **app name**.

First, check if the bundle ID already exists:

```bash
asc bundle-ids list
```

If the bundle ID is not listed, create it:

```bash
asc bundle-ids create \
  --identifier <BUNDLE_IDENTIFIER> \
  --name "<APP_NAME>" \
  --platform IOS
```

Save the `id` field from the response (e.g., `52R8U94X54`) — it is needed later for provisioning profile creation.

If the bundle ID already exists, note its `id` field from the list output.

### Step 3: Generate CSR and Private Key

Generate a Certificate Signing Request (CSR) and private key:

```bash
asc certificates csr generate \
  --key-out "./cert.key" \
  --csr-out "./cert.csr"
```

This creates two files in the current directory:
- `cert.key` — private key (keep this safe)
- `cert.csr` — certificate signing request

### Step 4: Create Distribution Certificate

Check if a distribution certificate already exists:

```bash
asc certificates list --certificate-type DISTRIBUTION
```

If a certificate already exists, ask the user whether to reuse it or create a new one. Note: Apple limits the number of active distribution certificates. If the limit is reached, an existing certificate must be revoked before creating a new one.

To create a new distribution certificate:

```bash
asc certificates create \
  --certificate-type DISTRIBUTION \
  --csr "./cert.csr"
```

The response contains:
- `id` — certificate ID (needed for provisioning profile creation)
- `certificateContent` — base64-encoded DER certificate

Save both values for the following steps.

For development certificates, use `--certificate-type DEVELOPMENT` instead of `DISTRIBUTION`.

### Step 5: Export .p12

The `.p12` file requires combining the certificate with the private key using `openssl`.

**Important:** The following commands contain a `<PASSWORD>` placeholder. Print these commands for the user and ask them to replace `<PASSWORD>` with their desired certificate password, then run the commands manually. Do **not** ask the user for the password.

```bash
# Decode the base64 certificate content to a DER file
echo "<CERTIFICATE_CONTENT>" | base64 -d > cert.cer

# Convert DER to PEM format
openssl x509 -inform DER -in cert.cer -out cert.pem

# Combine certificate and private key into a .p12 file
openssl pkcs12 -export \
  -out cert.p12 \
  -inkey cert.key \
  -in cert.pem \
  -passout pass:<PASSWORD>
```

Replace `<CERTIFICATE_CONTENT>` with the `certificateContent` value from Step 4.

After the user confirms the `.p12` file has been created, the intermediate files (`cert.cer`, `cert.pem`, `cert.csr`) can be deleted. The `cert.key` file should be kept securely as a backup.

### Step 6: Select Profile Type

Ask the user which provisioning profile type to create:

| Type | Use Case |
|---|---|
| `IOS_APP_STORE` | App Store / TestFlight distribution (default) |
| `IOS_APP_ADHOC` | Ad Hoc distribution to registered devices |
| `IOS_APP_DEVELOPMENT` | Development builds on registered devices |
| `IOS_APP_INHOUSE` | Enterprise distribution (requires Apple Enterprise account) |

Default to `IOS_APP_STORE` if the user is unsure.

### Step 7: Create Provisioning Profile

Ask the user for a descriptive **profile name** (e.g., `"My App AppStore"`).

Create the provisioning profile using the **bundle ID** from Step 2 and the **certificate ID** from Step 4:

```bash
asc profiles create \
  --name "<PROFILE_NAME>" \
  --profile-type <PROFILE_TYPE> \
  --bundle <BUNDLE_ID> \
  --certificate <CERT_ID>
```

Save the `id` field from the response for the next step.

For apps with extensions, create a separate provisioning profile for each extension target (each with its own bundle ID).

### Step 8: Download Provisioning Profile

Download the provisioning profile using the **profile ID** from Step 7:

```bash
asc profiles download \
  --id <PROFILE_ID> \
  --output "./profile.mobileprovision"
```

After downloading, verify both files exist:
- `cert.p12` — signing certificate
- `profile.mobileprovision` — provisioning profile

Proceed to [Upload via CLI](#upload-via-cli) to upload the certificate to Capawesome Cloud.

## Create a .p12 Certificate (Manual)

### Export from Keychain Access

If a signing certificate is already installed on the user's Mac, print the following steps for the user and ask them to confirm when done:

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

If no signing certificate exists, print the following steps for the user and ask them to confirm when done:

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

## Obtain Provisioning Profiles (Manual)

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

If no provisioning profile exists, print the following steps for the user and ask them to confirm when done:

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
