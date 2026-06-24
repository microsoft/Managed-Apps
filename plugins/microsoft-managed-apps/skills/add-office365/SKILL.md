---
name: add-office365
description: Adds Office 365 Outlook by delegating to `/add-connector` with `api-id=shared_office365` and action mode. Use when integrating Outlook mail/calendar.
user-invocable: true
allowed-tools: Read, AskUserQuestion, Skill
model: sonnet
---

**📋 Shared Instructions: [shared-instructions.md](${CLAUDE_PLUGIN_ROOT}/shared/shared-instructions.md)** — Cross-cutting concerns.

# Add Office 365 Outlook (Wrapper)

This skill is a thin wrapper. Use `/add-connector` as the single implementation path.

## Delegation contract

Invoke `/add-connector` with:

- `api-id`: `shared_office365`
- `mode`: `action`

---

# Office 365 Connector: Method Selection Guide

After the connector is added, you'll have access to `Office365OutlookService` with 20+ methods. **Use this guide to select the right method for your use case.**

## Calendar Operations

### For Fetching Events in a Date Range: `GetEventsCalendarViewV2`

**Best for:** Meeting lists, calendar views, event queries between two dates (e.g., "past 7 days", "next 30 days").

**Key Parameters:**
- `calendarId` (string) — Always discover using `CalendarGetTables()` first
- `startDateTimeOffset` (ISO 8601 string) — Use `.toISOString()`
- `endDateTimeOffset` (ISO 8601 string) — Use `.toISOString()`
- `top`, `skip` (optional) — For pagination (200-1000 range typical)

**Pattern:**

```typescript
import { Office365OutlookService } from '../generated'

// Step 1: Discover calendar ID
const calendarsResult = await Office365OutlookService.CalendarGetTables()
const calendarId = calendarsResult.data?.value?.[0]?.Name ?? 'Calendar'

// Step 2: Query events
const eventsResult = await Office365OutlookService.GetEventsCalendarViewV2(
  calendarId,
  startDate.toISOString(),
  endDate.toISOString()
)

const meetings = eventsResult.data?.value ?? []
```

**Response Structure:**
- Array of `CalendarEventClientReceiveStringEnums` (access via `.data.value`)
- Contains: `Subject`, `Start`, `End`, `Id`, `Organizer`, `RequiredAttendees`, `OptionalAttendees`

### For Immediate Upcoming Events: `OnUpcomingEventsV3`

**Best for:** "Show me my next meeting" queries using a minutes-based time window.

**Key Parameters:**
- `table` (string) — Calendar ID
- `minutesWindow` (number) — Time window in minutes (e.g., 60 for next hour, 1440 for next 24 hours)

### For Creating Events: `V3CalendarPostItem`

**Parameters:**
- `table` — Calendar ID
- `item` — `CalendarEventHtmlClient` object with `Subject`, `Start`, `End` (required)

### For Updating Events: `CalendarPatchItem`

**Parameters:**
- `table` — Calendar ID
- `id` — Event ID
- `item` — Updated event properties

### For Deleting Events: `CalendarDeleteItem`

**Parameters:**
- `table` — Calendar ID
- `id` — Event ID

### Discovery: `CalendarGetTables`

**Purpose:** List available calendars to discover correct calendar IDs (required before using other calendar methods)

## Email Operations

### For Sending Emails: `SendEmailV2`

**Best for:** Sending notifications, alerts, summaries.

**Pattern:**

```typescript
const result = await Office365OutlookService.SendEmailV2({
  To: 'recipient@example.com',
  Subject: 'Meeting Summary',
  Body: '<p>Here is your meeting summary...</p>',
  Importance: 'Normal'
})
```

### For Fetching Inbox: `GetEmails`

**Parameters:**
- `folderPath` (string) — E.g., "Inbox", "Drafts"
- `fetchOnlyUnread` (boolean) — Filter for unread emails
- `top` (number) — Limit

### For Reading Single Email: `GetEmail`

**Parameters:**
- `messageId` (string) — Email ID

### For Marking as Read: `MarkAsRead`

**Parameters:**
- `messageId` (string) — Email ID

### For Replying to Email: `ReplyToV3`

**Parameters:**
- `messageId` (string) — Email ID to reply to
- `body` (string) — Reply text

## Response Pattern (All Methods)

All Office 365 methods return `IOperationResult<T>`:

```typescript
const result = await Office365OutlookService.SomeMethod(...)

if (!result.success) {
  throw new Error(result.error?.message ?? 'Operation failed')
}

const data = result.data  // Safe to access
```

---

## Key Patterns

**1. Always discover calendar ID first using `CalendarGetTables()`**

This ensures you have the correct calendar ID for the user (don't hardcode "Calendar").

**2. Use ISO date strings for date parameters**

Use `.toISOString()` when passing dates to Office 365 methods.

**3. Access array results via `.data.value`**

List operations return wrapped responses — the actual array is in `data.value`.

**4. Handle empty results as valid state**

Empty array (no emails/events) is a valid result, not an error. Show empty state to user.

**5. Surface errors by throwing**

When API fails, throw an error so it bubbles up to component error handling. Don't silently use mock data.
