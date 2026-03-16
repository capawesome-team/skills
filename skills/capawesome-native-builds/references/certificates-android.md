# Android Signing Certificates

To build signed Android apps in Capawesome Cloud, configure a signing certificate with a Java Keystore (JKS) file.

## Upload via CLI

```bash
npx @capawesome/cli apps:certificates:create \
  --app-id <APP_ID> \
  --name "Production Android Key" \
  --platform android \
  --type production \
  --file /path/to/my-release-key.jks \
  --password <KEYSTORE_PASSWORD> \
  --key-alias my-key-alias \
  --key-password <KEY_PASSWORD>
```

Options:
- `--type`: `development` or `production`
- `--file`: Path to `.jks` or `.keystore` file
- `--password`: Keystore password
- `--key-alias`: Alias of the signing key within the keystore
- `--key-password`: Password for the key alias

## Upload via Console

Navigate to [Signing Certificates](https://console.cloud.capawesome.io/apps/_/certificates) and provide:

- **Name**: Descriptive name (e.g., "Production Android Key")
- **Platform**: `Android`
- **Type**: `Development` or `Production`
- **Keystore File**: The `.jks` or `.keystore` file
- **Keystore Password**: Password for the keystore
- **Key Alias**: Alias of the signing key
- **Key Password**: Password for the key alias

## Create a Keystore

If the user does not have a keystore, create one using `keytool`:

```bash
keytool -genkey -v -keystore my-release-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias my-key-alias
```

The command prompts for:
1. Keystore password
2. Name and organizational information
3. Key password (press Enter to reuse keystore password)

The file `my-release-key.jks` is created in the current directory.

Alternatively, create via Android Studio:

1. Open Android Studio.
2. Click **Build > Generate Signed Bundle / APK**.
3. Choose **Android App Bundle** or **APK**, click **Next**.
4. Click **Create new...** to generate a new keystore.
5. Fill in keystore path, password, key alias, key password, validity, and certificate info.
6. Click **OK**.

**Important:** Store the keystore securely. Never commit it to version control. If lost, the app cannot be updated on Google Play Store.

## List Certificates

```bash
npx @capawesome/cli apps:certificates:list --app-id <APP_ID> --platform android --json
```

## Delete a Certificate

```bash
npx @capawesome/cli apps:certificates:delete --app-id <APP_ID> --name "Production Android Key" --platform android --yes
```
