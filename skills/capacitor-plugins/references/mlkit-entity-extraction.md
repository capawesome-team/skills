# MLKit Entity Extraction

Unofficial Capacitor plugin for ML Kit Entity Extraction. Detects structured data such as addresses, date-times, and phone numbers in raw text on the device.

**Platforms:** Android, iOS

## Installation

```bash
npm install @capacitor-mlkit/entity-extraction
npx cap sync
```

## Configuration

### Android

No configuration required.

Optional `variables.gradle` variable: `mlkitEntityExtractionVersion` (default: `16.0.0-beta6`).

### iOS

Set minimum deployment target in `ios/App/Podfile`:

```ruby
platform :ios, '15.5'
```

CocoaPods only — SPM is not supported.

## Usage

```typescript
import { EntityExtraction, EntityType, Language } from '@capacitor-mlkit/entity-extraction';

await EntityExtraction.downloadModel({ language: Language.English });

const { annotations } = await EntityExtraction.extractEntities({
  text: "Let's meet tomorrow at 5pm.",
  language: Language.English,
  referenceTime: Date.now(),
});
for (const annotation of annotations) {
  for (const entity of annotation.entities) {
    if (entity.type === EntityType.DateTime) {
      console.log(annotation.text, entity.timestamp);
    }
  }
}

const { languages } = await EntityExtraction.getDownloadedModels();
await EntityExtraction.deleteDownloadedModel({ language: Language.English });
```

## Notes

- The underlying ML Kit Entity Extraction API is in **beta**.
- A per-language model must be downloaded before extraction. `extractEntities()` throws `ErrorCode.ModelNotDownloaded` if the model is missing.
- Supported entity types include `Address`, `DateTime`, `Email`, `FlightNumber`, `Iban`, `Isbn`, `Money`, `PaymentCard`, `Phone`, `TrackingNumber`, and `Url`.
