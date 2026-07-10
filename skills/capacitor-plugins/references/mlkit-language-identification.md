# MLKit Language Identification

Unofficial Capacitor plugin for ML Kit Language Identification. Determines the language of a string of text on the device.

**Platforms:** Android, iOS

## Installation

```bash
npm install @capacitor-mlkit/language-identification
npx cap sync
```

## Configuration

### Android

No configuration required.

Optional `variables.gradle` variable: `mlkitLanguageIdVersion` (default: `17.0.6`).

### iOS

Set minimum deployment target in `ios/App/Podfile`:

```ruby
platform :ios, '15.5'
```

CocoaPods only — SPM is not supported.

## Usage

```typescript
import { LanguageIdentification } from '@capacitor-mlkit/language-identification';

const { language } = await LanguageIdentification.identifyLanguage({
  text: 'Wie geht es dir?',
});
// `language` is a BCP-47 tag (e.g. `de`) or `und` if undetermined

const { identifiedLanguages } = await LanguageIdentification.identifyPossibleLanguages({
  text: 'Wie geht es dir?',
});
for (const item of identifiedLanguages) {
  console.log(item.language, item.confidence);
}
```

## Notes

- `identifyLanguage(...)` and `identifyPossibleLanguages(...)` are only available on Android and iOS; there is no web implementation.
- `identifyLanguage()` returns `und` when no language passes the `confidenceThreshold` (default `0.5`).
- `identifyPossibleLanguages()` returns all candidates above its `confidenceThreshold` (default `0.01`).
