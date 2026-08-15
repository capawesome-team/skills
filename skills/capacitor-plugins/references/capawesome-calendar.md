# Calendar

Capacitor plugin to manage calendars and events on the device. Create, read, update and delete calendars and events, work with recurring events, present the system event dialogs, and listen for calendar changes.

**Package:** `@capawesome-team/capacitor-calendar`
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
npm install @capawesome-team/capacitor-calendar
npx cap sync
```

## Configuration

### Android

#### Permissions

Add to `android/app/src/main/AndroidManifest.xml` before or after the `application` tag. Only declare the permissions the app actually needs:

```xml
<!-- Required to read calendars and events, e.g. `getCalendars()` or `getEvents(...)`. -->
<uses-permission android:name="android.permission.READ_CALENDAR" />
<!-- Required to create, update or delete calendars and events, e.g. `createEvent(...)`. -->
<uses-permission android:name="android.permission.WRITE_CALENDAR" />
```

`createEvent(...)`, `updateEventById(...)` and `deleteEventById(...)` require `READ_CALENDAR` in addition to `WRITE_CALENDAR`, because they look up the calendar or event first. Only `createCalendar(...)` and `deleteCalendarById(...)` work with `WRITE_CALENDAR` alone.

#### Proguard

If using Proguard, add to `android/app/proguard-rules.pro`:

```
-keep class io.capawesome.capacitorjs.plugins.** { *; }
```

### iOS

#### Privacy Descriptions

Add to `ios/App/App/Info.plist`. Add only the keys that match the requested access:

```xml
<!-- Required on iOS 17+ if the app reads or modifies calendars or events. -->
<key>NSCalendarsFullAccessUsageDescription</key>
<string>The app needs access to your calendars to display and manage your events.</string>
<!-- Required on iOS 17+ only if `requestPermissions(...)` is called with only the `writeCalendar` permission. -->
<key>NSCalendarsWriteOnlyAccessUsageDescription</key>
<string>The app needs access to your calendars to add events for your bookings.</string>
<!-- Required on iOS 16 and older. -->
<key>NSCalendarsUsageDescription</key>
<string>The app needs access to your calendars to display and manage your events.</string>
```

If a required key is missing, `requestPermissions(...)` rejects with an error message.

## Usage

### Request permissions and get the calendars

```typescript
import { Calendar } from '@capawesome-team/capacitor-calendar';

const { readCalendar, writeCalendar } = await Calendar.requestPermissions();
const { calendars } = await Calendar.getCalendars();
const writableCalendars = calendars.filter((calendar) => calendar.writable);
const { calendar } = await Calendar.getDefaultCalendar();
```

### Create an event

```typescript
import { Calendar, EventAvailability } from '@capawesome-team/capacitor-calendar';

const startDate = new Date('2026-09-01T10:00:00').getTime();
const { id } = await Calendar.createEvent({
  event: {
    title: 'Dentist appointment',
    startDate,
    endDate: startDate + 60 * 60 * 1000,
    location: 'Main Street 1, Springfield',
    availability: EventAvailability.Busy,
    alerts: [60, 15],
  },
});
```

### Create a recurring event

```typescript
import { Calendar, RecurrenceFrequency, Weekday } from '@capawesome-team/capacitor-calendar';

const { id } = await Calendar.createEvent({
  event: {
    title: 'Team stand-up',
    startDate: new Date('2026-09-01T09:00:00').getTime(),
    recurrence: {
      frequency: RecurrenceFrequency.Weekly,
      interval: 1,
      count: 10,
      daysOfWeek: [Weekday.Monday, Weekday.Wednesday],
    },
  },
});
```

### Get, update and delete events

```typescript
import { Calendar, EventSpan } from '@capawesome-team/capacitor-calendar';

const from = Date.now();
const { events } = await Calendar.getEvents({ from, to: from + 7 * 24 * 60 * 60 * 1000 });

await Calendar.updateEventById({
  id: events[0].id,
  event: { location: 'Main Street 2, Springfield' },
});

// Delete only a single occurrence of a recurring event.
await Calendar.deleteEventById({
  id: events[0].id,
  instanceStartDate: events[0].startDate,
  span: EventSpan.ThisEvent,
});
```

### Display the system dialog and listen for changes

```typescript
import { Calendar } from '@capawesome-team/capacitor-calendar';

const { id } = await Calendar.displayCreateEvent({
  event: { title: 'Lunch with Jane', startDate: Date.now() },
});

await Calendar.addListener('calendarChange', () => {
  console.log('The calendars or events on the device have changed.');
});
```

## Notes

- All dates are timestamps in milliseconds. Only `title` and `startDate` are required to create an event; without `calendarId` the event is created in the default calendar. `alerts` are offsets in minutes before the start (negative values: after the start).
- On iOS 17+, requesting only `writeCalendar` requests write-only access, which is not sufficient for the methods of this plugin; `readCalendar` requests full access and stays `prompt` while only write-only access is granted.
- `getEvents(...)` expands recurring events, so each occurrence is a separate entry with its own `startDate`, while `id` and `recurrence` are shared across the series. `getEventById(...)` returns the series with its original start date and resolves with `null` if the event does not exist.
- `updateEventById(...)` and `deleteEventById(...)` operate on the whole series unless `instanceStartDate` (the `startDate` of an occurrence) is passed; `span` is `EventSpan.ThisEvent` (default) or `EventSpan.ThisAndFutureEvents`.
- In `updateEventById(...)`, omitted properties keep their values and `null` (or `[]`) removes a property. `allDay`, `calendarId`, `endDate`, `startDate` and `timezone` must not be set to `null`.
- All-day events (`allDay: true`) use midnight UTC on both platforms with an exclusive `endDate`. Build timestamps with `Date.UTC(...)`, otherwise events appear on the wrong day.
- Enums: `EventAvailability` (`Busy`, `Free`, `Tentative`, `Unavailable` — mapped to `Busy` on Android), `RecurrenceFrequency` (`Daily`, `Weekly`, `Monthly`, `Yearly`), `Weekday`, `EventSpan`. Type aliases: `EventStatus` (`'canceled' | 'confirmed' | 'tentative'`), `EventEditAction` (`'canceled' | 'deleted' | 'saved'`).
- The system dialogs report a result only on iOS: `displayCreateEvent(...)` returns `id`, `displayUpdateEventById(...)` returns `action`. On Android only the properties supported by the system intent are prefilled, the rest are ignored.
- The `calendarChange` event carries no payload — reload the data instead of applying a delta. `getEvents(...)` rejects with the error code `CALENDAR_NOT_FOUND` if the given `calendarId` does not exist.
- Further methods: `checkPermissions()`, `createCalendar(...)`, `deleteCalendarById(...)`, `openCalendar({ date })`, `openSettings()`, `removeAllListeners()`.
