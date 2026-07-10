# MLKit Smart Reply

Unofficial Capacitor plugin for ML Kit Smart Reply. Generates suggested replies for a conversation on the device.

**Platforms:** Android, iOS

## Installation

```bash
npm install @capacitor-mlkit/smart-reply
npx cap sync
```

## Configuration

### Android

No configuration required.

Optional `variables.gradle` variable: `mlkitSmartReplyVersion` (default: `17.0.4`).

### iOS

Set minimum deployment target in `ios/App/Podfile`:

```ruby
platform :ios, '15.5'
```

CocoaPods only — SPM is not supported.

## Usage

```typescript
import { SmartReply, SmartReplySuggestionResultStatus } from '@capacitor-mlkit/smart-reply';

const { status, suggestions } = await SmartReply.suggestReplies({
  messages: [
    { text: 'Are you free for lunch today?', timestamp: Date.now() - 60_000, isLocalUser: false, userId: 'user-1' },
    { text: 'Sure, where do you want to go?', timestamp: Date.now() - 30_000, isLocalUser: true },
    { text: 'How about the new place downtown?', timestamp: Date.now(), isLocalUser: false, userId: 'user-1' },
  ],
});
if (status === SmartReplySuggestionResultStatus.Success) {
  console.log(suggestions); // up to 3 suggestions
}
```

## Notes

- `suggestReplies(...)` is only available on Android and iOS; there is no web implementation.
- English-only. `status` returns `NotSupportedLanguage` when the conversation is not in English and `NoReply` when no suggestion is available.
- Messages must be ordered chronologically (oldest first); only the last 10 are considered. Set `userId` on messages where `isLocalUser` is `false`.
