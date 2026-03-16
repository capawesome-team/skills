# Google Play Store Credentials

## Prerequisites

- A [Google Play Developer](https://play.google.com/console) account.
- An app already created in Google Play Console.
- The first version of the app must have been uploaded manually to Google Play Console. This is a Google Play requirement, not a Capawesome Cloud limitation.

## Required Information

- **Package Name**: The Android application ID (e.g., `com.example.app`), defined in `android/app/build.gradle` as `applicationId`.
- **Service Account JSON Key File**: Generated from Google Cloud Console and linked in Google Play Console.

## Obtaining the Service Account JSON Key File

If the Google Cloud Console or Google Play Console appears in a non-English language, append `?hl=en` to the URL to switch to English.

1. Sign in to the [Google Cloud Console](https://console.cloud.google.com/).
2. Navigate to **IAM & Admin** > **Service Accounts**.
3. Click **Create Service Account**.
   1. In step 1, fill in the service account details (e.g., name: "Capawesome Cloud") and click **Create and Continue**.
   2. In step 2, assign the role **Service Account User** and click **Continue**.
   3. In step 3, leave the fields empty and click **Done**.
4. In the list of service accounts, locate the newly created service account. Copy its **email address** (needed in step 8). Then click the menu in the **Actions** column and select **Manage keys**.
5. In the **Keys** tab, click **Add Key** > **Create new key**.
6. Select **JSON** as the key type and click **Create**. Save the downloaded JSON key file securely.
7. Sign in to the [Google Play Console](https://play.google.com/console).
8. Navigate to **Users and Permissions** and click **Invite new user**. Enter the service account email address copied in step 4.
9. Click on the invited user and go to the **App permissions** tab. Add the desired applications to grant access to.
10. In the permissions dialog, check at least all permissions in the **Release** section, then click **Apply**.
11. Click **Invite User** to complete the setup.

## Troubleshooting

- **Deployment fails with permission error** → ensure the service account has at least **Release** permissions in Google Play Console for the specific app.
- **"First version must be uploaded manually" error** → upload the first APK or AAB directly through the Google Play Console before using automated deployments.
- **Service account email not accepted** → ensure the email address is the one from Google Cloud Console (ends with `@*.iam.gserviceaccount.com`), not a personal email.
