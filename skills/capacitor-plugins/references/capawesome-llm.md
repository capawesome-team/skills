# LLM

Capacitor plugin for running on-device large language models using the system-provided models Apple Intelligence (Foundation Models) on iOS and Gemini Nano (AICore) on Android. Supports chat sessions, token streaming, and cancellation.

**Package:** `@capawesome-team/capacitor-llm`
**Platforms:** Android, iOS
**Capawesome Insiders:** Yes (requires license key)

## Installation

Set up the Capawesome npm registry:

```bash
npm config set @capawesome-team:registry https://npm.registry.capawesome.io
npm config set //npm.registry.capawesome.io/:_authToken <YOUR_LICENSE_KEY>
```

Install the package:

```bash
npm install @capawesome-team/capacitor-llm
npx cap sync
```

## Configuration

### Android

#### Minimum SDK Version

The ML Kit GenAI Prompt SDK requires a minimum SDK version of `26`. In `android/variables.gradle`, set `minSdkVersion` to at least `26`:

```groovy
ext {
    minSdkVersion = 26
}
```

#### Variables

Optionally override the dependency versions in `android/variables.gradle` to resolve conflicts with other plugins:

- `mlkitGenaiPromptVersion` — version of `com.google.mlkit:genai-prompt` (default: `1.0.0-beta2`)
- `kotlinVersion` — version of `org.jetbrains.kotlin:kotlin-gradle-plugin` (default: `2.1.20`)
- `kotlinxCoroutinesVersion` — version of `org.jetbrains.kotlinx:kotlinx-coroutines-android` (default: `1.10.2`)

#### Proguard

If using Proguard, add to `android/app/proguard-rules.pro`:

```
-keep class io.capawesome.capacitorjs.plugins.** { *; }
```

### iOS

No configuration required. The plugin uses the Foundation Models framework, which is part of the operating system.

## Usage

### Check the model availability

```typescript
import { Llm } from '@capawesome-team/capacitor-llm';

const { status } = await Llm.getAvailability();

await Llm.addListener('availabilityChange', (event) => {
  console.log(`Availability changed: ${event.status}`);
});
```

### Download the model (Android only)

```typescript
import { Llm } from '@capawesome-team/capacitor-llm';

const { status } = await Llm.getAvailability();
if (status === 'downloadable') {
  await Llm.addListener('downloadProgress', (event) => {
    console.log(`Download progress: ${event.progress * 100}%`);
  });
  await Llm.downloadModel();
}
```

### Create and delete a chat

```typescript
import { Llm } from '@capawesome-team/capacitor-llm';

const { id } = await Llm.createChat({
  instructions: 'You are a helpful assistant that answers briefly.',
  maxOutputTokens: 256,
  temperature: 0.7,
});

await Llm.deleteChat({ id });
```

### Generate text

```typescript
import { Llm } from '@capawesome-team/capacitor-llm';

const { text } = await Llm.generateText({
  chatId: 'CHAT_ID',
  prompt: 'Why is the sky blue?',
  temperature: 0.2, // Overrides the chat's default value
});
```

### Stream the response

```typescript
import { Llm } from '@capawesome-team/capacitor-llm';

await Llm.addListener('textChunk', (event) => {
  if (event.chatId === 'CHAT_ID') {
    console.log(event.text); // Append the chunks to your UI
  }
});
const { text } = await Llm.streamText({
  chatId: 'CHAT_ID',
  prompt: 'Tell me a short story about a magical dog.',
});
```

### Cancel a generation

```typescript
import { Llm } from '@capawesome-team/capacitor-llm';

await Llm.cancelGeneration({ chatId: 'CHAT_ID' });
```

## Notes

- iOS requires **iOS 26 or later** on an Apple Intelligence-enabled device (iPhone 15 Pro or later) with Apple Intelligence turned on. Building the plugin requires **Xcode 26 or later**. On older iOS versions all methods except `getAvailability()` reject as unavailable.
- Android requires a Gemini Nano-capable device with AICore (for example Google Pixel 9 series or Samsung Galaxy S25 series). The device list is short, so always check `getAvailability()` at runtime and provide a fallback. The underlying ML Kit GenAI Prompt SDK is still in beta.
- On the web only `getAvailability()` is implemented and resolves with `unavailable`; all other methods reject with an unimplemented error.
- `getAvailability()` never rejects. `AvailabilityStatus` values: `available`, `device-not-eligible`, `downloadable`, `downloading`, `not-enabled`, `not-ready`, `unavailable`. Android reports only `available`, `downloadable`, `downloading`, `unavailable`; iOS reports only `available`, `device-not-eligible`, `not-enabled`, `not-ready`, `unavailable`.
- `downloadModel()` and the `downloadProgress` event are Android only. On iOS the model download is managed by the system.
- Chats live in memory until `deleteChat(...)` is called, so delete unused chats to free native resources. `createChat(...)` accepts an optional `id`; reusing an existing one rejects with `CHAT_ALREADY_EXISTS`.
- Only one generation can be in flight per chat. Starting another rejects with `GENERATION_IN_PROGRESS`. Canceling rejects the pending promise with `GENERATION_CANCELED` — on Android cancellation is best-effort, so a few more chunks may still be emitted.
- `maxOutputTokens` and `temperature` are set per chat in `createChat(...)` and can be overridden per request. On Android `maxOutputTokens` is capped at `4096` and `temperature` must be between `0.0` and `1.0`; iOS allows values greater than `1.0`.
- Context limit is roughly 4,000 tokens on Android and ~4,096 tokens per session on iOS. Exceeding it rejects with `GENERATION_FAILED` — create a new chat in that case. `GENERATION_FAILED` is also raised for blocked content and exceeded AICore quota.
- The plugin only watches the availability status while at least one `availabilityChange` listener is attached.
- Events: `availabilityChange`, `downloadProgress` (Android only), `textChunk`.
