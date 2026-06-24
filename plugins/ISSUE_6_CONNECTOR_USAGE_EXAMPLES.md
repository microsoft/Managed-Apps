# Issue #6: Step 9 Missing Connector Usage Examples

**Severity:** 🔴 Critical  
**Category:** Skill Documentation  
**Status:** Identified during Meeting Summary app build

---

## Problem Statement

The `create-app` skill context says in **Step 9 (Implement App)**:

> "Wire each component to the typed services produced in Step 8 (no raw `fetch` / `axios` / Graph calls)"

However, it provides **NO examples** of:
1. How to import the generated services
2. What methods each service has
3. How to call the services from React
4. What the response type `IOperationResult<T>` looks like
5. How to handle errors and fallback to mock data

**Result:** Builders don't know how to actually wire the connectors, even though the generated code exists.

---

## Evidence

### What Gets Generated (Hidden from User)
When `ms app add connector --connector shared_office365 --as action` runs, these files are created:

```
generated/
├── index.ts                             # Exports all services
├── services/
│   ├── Office365OutlookService.ts      # Auto-generated!
│   └── WorkIQCopilotMCPService.ts       # Auto-generated!
├── models/
│   ├── Office365OutlookModel.ts
│   └── WorkIQCopilotMCPModel.ts
└── dataSources.ts
```

**But the skill doesn't mention these exist or how to use them.**

### What Was Actually Wired in the Meeting Summary App

**BEFORE (Mock Data - No Real API Calls):**
```typescript
// ❌ This was the original code - uses mock data, no connector imports
const mockMeetings: Meeting[] = [
  { id: '1', subject: 'Team Standup', ... }
]
setMeetings(mockMeetings)
```

**AFTER (Real API Calls - Properly Wired):**
```typescript
// ✓ Import the generated connector services
import { Office365OutlookService, WorkIQCopilotMCPService } from '../generated'

// ✓ Call real Office 365 API
const result = await Office365OutlookService.CalendarGetTable('calendar')

// ✓ Handle IOperationResult<T>
if (result.success && result.data) {
  setMeetings(transformData(result.data))
} else {
  // ✓ Graceful fallback
  setMeetings(getMockMeetings())
}
```

---

## Recommended Fix

### Update create-app/README.md Step 9: Implement the App

Add a new subsection: **"Wiring Connectors: Importing Generated Services"**

**Example 1: Office 365 Calendar (Action Connector)**

```typescript
// ✓ Import at the top of your component
import { Office365OutlookService } from '../generated'

// ✓ In an async function (e.g., useEffect)
const fetchMeetings = async () => {
  try {
    const result = await Office365OutlookService.CalendarGetTable('calendar')
    
    // ✓ Always check result.success
    if (result.success && result.data) {
      // Transform Office 365 response to your app's data format
      const meetings = result.data.map((event: any) => ({
        id: event.id,
        subject: event.subject,
        start: event.start,
        end: event.end,
        organizer: event.organizer?.emailAddress?.address,
        attendees: event.attendees?.map((a: any) => a.emailAddress?.name) || []
      }))
      setMeetings(meetings)
    } else {
      console.error('Office 365 API failed:', result.error)
      setMeetings(getMockMeetings())  // ✓ Fallback
    }
  } catch (error) {
    console.error('Error calling Office 365:', error)
    setMeetings(getMockMeetings())  // ✓ Fallback
  }
}
```

**Example 2: Work IQ Copilot (Action Connector)**

```typescript
import { WorkIQCopilotMCPService } from '../generated'
import type { QueryRequest } from '../generated/models/WorkIQCopilotMCPModel'

const generateSummary = async (meeting: Meeting) => {
  try {
    // ✓ Construct a QueryRequest (JSON-RPC format for Work IQ)
    const query: QueryRequest = {
      jsonrpc: '2.0',
      method: 'resources/read',
      params: {
        uri: `workiq://query?${encodeURIComponent(
          `Summarize this meeting:\n\n${meeting.subject}`
        )}`
      }
    }

    const result = await WorkIQCopilotMCPService.mcp_m365copilot(undefined, query)

    if (result.success && result.data) {
      setMeetingSummary({
        summary: JSON.stringify(result.data),
        loading: false
      })
    } else {
      setMeetingSummary(getMockSummary(meeting))  // ✓ Fallback
    }
  } catch (error) {
    console.error('Error calling Work IQ:', error)
    setMeetingSummary(getMockSummary(meeting))
  }
}
```

**IOperationResult<T> Interface:**

```typescript
interface IOperationResult<T> {
  success: boolean    // Did the API call succeed?
  data?: T            // Response data (if success=true)
  error?: {           // Error details (if success=false)
    code: string
    message: string
  }
}

// ✓ Always use this pattern:
if (result.success && result.data) {
  // Use result.data
} else {
  // Handle result.error or fallback
}
```

---

## Implementation Checklist

- [ ] Add "Wiring Connectors" subsection to create-app Step 9
- [ ] Include Office 365 and Work IQ examples
- [ ] Document IOperationResult type
- [ ] Document error handling + fallback pattern
- [ ] Document how to discover methods in `generated/services/`

---

## Why This Matters

The skill says: "connectors wired, screens implemented per the approved plan — not a bare template."

This fix ensures builders actually **wire the connectors instead of using mock data**.

