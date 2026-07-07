# Localization

Read the user's localization preferences: preferred locales, regional formatting, time zone, and clock settings.

**Package:** `@capawesome/capacitor-localization`
**Platforms:** Android, iOS, Web

## Installation

```bash
npm install @capawesome/capacitor-localization
npx cap sync
```

No additional permissions or configuration are required on any platform.

## Usage

### Read the preferred locales

```typescript
import { Localization } from '@capawesome/capacitor-localization';

const { locales } = await Localization.getLocales();
// The first entry is the most preferred locale.
const primary = locales[0];
console.log(primary.languageTag); // e.g. 'de-DE'
console.log(primary.currencyCode); // e.g. 'EUR' (or null)
console.log(primary.textDirection); // 'ltr' | 'rtl'
console.log(primary.measurementSystem); // 'metric' | 'us' | 'uk' (or null)
```

### Read the regional settings

```typescript
import { Localization } from '@capawesome/capacitor-localization';

const settings = await Localization.getSettings();
console.log(settings.timeZone); // e.g. 'Europe/Berlin'
console.log(settings.uses24HourClock); // boolean
console.log(settings.firstDayOfWeek); // ISO 8601: 1 = Monday ... 7 = Sunday
```

## Notes

- Fields a platform cannot determine are returned as `null`.
- On Web, `currencyCode`, `currencySymbol`, and `measurementSystem` are always `null` because the browser does not expose this information.
- `firstDayOfWeek` follows the ISO 8601 convention (`1` = Monday, `7` = Sunday).
