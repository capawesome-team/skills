# Mail Composer

Open the native email composer prefilled with recipients, subject, body, and attachments.

**Package:** `@capawesome/capacitor-mail-composer`
**Platforms:** Android, iOS, Web (Web via `mailto:`, without attachments)

## Installation

```bash
npm install @capawesome/capacitor-mail-composer
npx cap sync
```

## Configuration

### Android

#### Attachments

To attach files, the plugin uses the `FileProvider` that Capacitor already registers for your app (authority `${applicationId}.fileprovider`). Ensure the directories your attachments are stored in are covered by the `android/app/src/main/res/xml/file_paths.xml` resource of your app. Otherwise, `composeMail(...)` rejects with an error.

## Usage

```typescript
import { MailComposer } from '@capawesome/capacitor-mail-composer';

// Check whether the device can compose emails.
const { canCompose } = await MailComposer.canComposeMail();

// Open the composer. The user decides whether to send; the plugin never sends.
const { status } = await MailComposer.composeMail({
  to: ['jane@example.com'],
  cc: ['john@example.com'],
  bcc: ['secret@example.com'],
  subject: 'Hello World',
  body: 'This is the body of the email.',
  attachments: ['/path/to/file.pdf'], // Not supported on Web.
});
// status: 'sent' | 'saved' | 'canceled' | 'unknown'
```

## Notes

- The plugin never sends the email itself; the user reviews it and decides whether to send.
- `canComposeMail()`: on iOS returns `true` only if a mail account is configured; on Android `true` if a mail app is installed; on Web always `true` (best effort, not verifiable).
- `status`: on Android always `'unknown'` (the mail intent returns no reliable result); on iOS the real status is returned; on Web always `'unknown'`.
- On iOS, `composeMail(...)` rejects as unavailable if no mail account is configured. Use `canComposeMail()` to check upfront.
- Web limitations (`mailto:`): attachments are not supported and cause `composeMail(...)` to reject; HTML bodies (`isHtml`) are not supported and the body is always plain text; long bodies may be truncated due to URL length limits.
- On Android, `isHtml` is best-effort because many mail apps ignore HTML.
- `ErrorCode` values: `ATTACHMENT_NOT_FOUND`, `COMPOSE_FAILED`.
