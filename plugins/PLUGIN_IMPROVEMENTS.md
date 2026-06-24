# Managed Apps Plugin Improvements — Session Report

**Date:** 2026-06-23  
**Session Goal:** Build a Meeting Summary app with Office 365 + Work IQ connectors  
**Outcome:** ✅ Successfully built and deployed app locally with all connectors wired

---

## Summary of Issues Encountered

During the app creation process, we encountered 4 distinct issues that affected the user experience and efficiency. Below is a detailed analysis of each with recommended improvements.

---

## Issue #1: Missing Domain-Specific Connector Recommendations

**Severity:** 🟡 Medium  
**Category:** Skill Guidance  

### What Happened
When gathering requirements for the app, I identified that the user wanted "AI-generated summaries" but the initial `create-app` skill guidance did not suggest using **Work IQ + Office 365 Calendar** automatically. Instead, the user had to explicitly ask which AI service to use, requiring an additional back-and-forth conversation.

### Root Cause
The `create-app` skill context provides a generic methodology for identifying connectors ("What data does your app need?") but lacks **domain-specific patterns and recommendations**. The skill knows about Work IQ's existence but doesn't explicitly recommend it for common scenarios.

### Impact
- ❌ Suboptimal planning phase (required extra user clarification)
- ❌ Risk of users choosing inappropriate connectors
- ❌ Longer time-to-value for building apps

### Recommended Fix

**Location:** `create-app/references/domain-patterns.md` (new file)

Create a reference guide listing common app scenarios with their recommended connectors:

```markdown
# Common App Patterns & Recommended Connectors

## Pattern: Meeting Summarization & Intelligence
**Use Case:** Show meetings, generate summaries, extract action items  
**Recommended Connectors:**
- Office 365 Outlook (action mode) — fetch calendar events
- Work IQ Copilot MCP (action mode) — generate summaries & action items
**Why:** Office 365 provides calendar metadata; Work IQ understands M365 context and generates summaries

## Pattern: Team Collaboration Tracking
**Use Case:** Track team activity, messages, files across a project  
**Recommended Connectors:**
- Teams (action mode) — list channels, messages
- SharePoint (table mode) — manage documents and lists
- OneNote (action mode) — capture notes and summaries

## Pattern: Data-Driven Dashboard
**Use Case:** Display metrics, KPIs from structured data  
**Recommended Connectors:**
- Dataverse (table mode) — structured business data
- SQL (table mode) — legacy data sources
- Excel (table mode) — lightweight data

## Pattern: Knowledge Search & Grounding
**Use Case:** Search across M365 documents, extract answers  
**Recommended Connectors:**
- Work IQ Copilot MCP (action mode) — knowledge retrieval & summarization
- SharePoint (table/action mode) — search & crawl documents
```

**Integration Point:** Update `create-app` skill Step 4 (Plan) to check this guide when the user's description matches a pattern keyword (e.g., "meeting" → suggest Office 365 + Work IQ).

**Implementation Example:**

In `create-app` skill, after gathering requirements:
```
If user's description contains "meeting" OR "summary" OR "calendar":
  → Suggest: "I notice you want meeting summaries. I recommend Office 365 Outlook 
             (to fetch meetings) + Work IQ (to generate AI summaries). OK?"
```

---

## Issue #2: GCM First-Run Authentication Trap

**Severity:** 🔴 High  
**Category:** Tooling / First-Run Experience

### What Happened
When running `ms app create`, the command appeared to fail with:
```
fatal: Authentication failed for 'https://<env-id>.d.environment.api.powerplatform.com/...'
App '<name>' was created, but local setup failed: Command failed: git fetch origin
```

Despite the error, the app **was actually created** in the service, but the local folder was empty. This is the documented **Git Credential Manager (GCM) first-run trap** — GCM needs an interactive browser auth flow on first use per environment, which happens asynchronously.

### Root Cause
- Git Credential Manager (GCM) is configured but requires interactive browser authentication on first-ever use per environment
- The `ms app create` command performs a `git fetch origin` during initialization, which triggers GCM auth
- GCM auth is not synchronous — it needs user browser interaction
- The error message doesn't explain that a retry will work (GCM is now cached)
- The skill context documents recovery but requires destructive operations (delete app, delete folder, retry)

### Impact
- ❌ Confusing "Authentication failed" message masking a recoverable state
- ❌ User must manually recover or know to retry
- ❌ Risk of data loss if user runs the suggested cleanup without understanding the state
- ⏱️ Adds 1-2 minutes to first-time app creation

### Recommended Fixes

#### Fix 2a: Proactive GCM Detection
**Location:** `create-app` skill, Step 7 (Scaffold)

Before running `ms app create`, add a GCM readiness check:

```bash
# Pseudo-code
GCM_TEST_URL="https://<tenant-env-id>.d.environment.api.powerplatform.com/test"
if ! git ls-remote "$GCM_TEST_URL" >/dev/null 2>&1; then
  if [[ $? == 1 ]]; then  # Auth failure
    echo "⚠️  Git Credential Manager needs one-time setup."
    echo "I'll open a browser for you to authenticate..."
    # Trigger git fetch to cache GCM auth
    git -c credential.helper=manager fetch "$GCM_TEST_URL" 2>/dev/null || true
  fi
fi
```

**Result:** Before `ms app create` even runs, ensure GCM is authenticated.

#### Fix 2b: Auto-Retry on Auth Failure
**Location:** `create-app` skill, Step 7 (Scaffold)

Wrap the `ms app create` call with automatic retry logic:

```bash
# Pseudo-code
MAX_RETRIES=2
RETRY=0

while [ $RETRY -lt $MAX_RETRIES ]; do
  if ms app create "$FOLDER" --display-name "$DISPLAY_NAME" --non-interactive; then
    echo "✅ App created successfully"
    break
  elif [[ $? == 1 ]] && grep -q "Authentication failed" <(ms app create ...); then
    RETRY=$((RETRY + 1))
    if [ $RETRY -lt $MAX_RETRIES ]; then
      echo "⏳ GCM auth may need a moment. Retrying in 3 seconds..."
      sleep 3
    fi
  else
    echo "❌ App creation failed (not a GCM issue)"
    exit 1
  fi
done
```

**Result:** Silently retry once if GCM auth is pending, dramatically improving UX.

#### Fix 2c: Clearer Error Messaging
**Location:** `create-app` skill, Step 7 recovery section

If first attempt fails, show:

```
⚠️  First-run setup: Git Credential Manager needs authentication
   • The app was created in the cloud, but local setup encountered an auth prompt
   • I'm retrying automatically (this usually takes 5-10 seconds)
   
   If this persists:
   1. Check your browser — you may see an auth prompt
   2. Sign in and return to this terminal
   3. I'll retry the operation
```

**Result:** User understands what's happening and why a retry is safe.

---

## Issue #3: GCM Retry Loop Has Confusing Intermediate States

**Severity:** 🟡 Medium  
**Category:** UX / Recovery

### What Happened
After the first `ms app create` failed, the recovery instructions required:
1. Run `ms app delete --app <id>` (to clean up the partially-created app in the cloud)
2. Delete the local folder
3. Retry `ms app create`

The problem: **The first delete command failed with a permission error**, leaving the app in a confusing limbo state (exists in cloud, empty locally).

### Root Cause
- The skill's recovery flow assumes users have permissions to delete apps
- In some environments / tenants, app deletion requires elevated permissions
- The destructive cleanup commands are presented without sufficient guardrails
- No validation that deletion succeeded before proceeding with folder cleanup

### Impact
- ❌ Unclear whether app still exists in cloud
- ❌ User may be stuck unable to recover without manual intervention
- ⚠️ Risk of permission-denied errors blocking retry

### Recommended Fix

**Location:** `create-app` skill, `troubleshooting.md` section on GCM recovery

Update recovery to be more robust:

```markdown
## Recovery: First-run GCM Authentication Trap

If `ms app create` fails with "Authentication failed for 'https://...'":

### Automatic Recovery (Recommended)
I will automatically retry the command. This usually succeeds because Git Credential Manager 
is now cached. No manual action needed — just wait 5-10 seconds.

### Manual Recovery (if auto-retry fails)
Only proceed if I tell you to. **Important: The app exists in the cloud but is empty locally.**

1. Verify the app state:
   ```
   ms app list --json | grep -i <app-name>
   ```
   If the app is listed, it was created but setup failed locally. This is OK.

2. **EITHER** use a fresh folder name:
   ```
   ms app create <new-folder-name> --display-name "<same name>" --non-interactive
   ```
   (The CLI detects the existing app and reuses it.)

3. **OR** delete and retry:
   ```
   ms app delete --app <APP_ID> --force --non-interactive
   # Wait for success message before proceeding
   
   rm -rf <folder-name>
   ms app create <folder-name> --display-name "<name>" --non-interactive
   ```

   ⚠️ If the delete command fails with "403 Forbidden", contact your tenant admin — 
      you may lack app deletion permissions. Use approach #2 (fresh folder) instead.
```

**Result:** Users understand they have options and what the intermediate state means.

---

## Issue #4: Dev Server Port Exhaustion

**Severity:** 🟢 Low  
**Category:** Development Experience

### What Happened
When running `ms app dev`, the output showed:
```
Port 5173 is in use, trying another one...
Port 5174 is in use, trying another one...
Port 5175 is in use, trying another one...
...
Port 5180 is in use, trying another one...
  ➜  Local:   http://localhost:5180/
```

The dev server had to try 8 ports before finding an available one. This adds startup latency and can confuse users about which port the app is actually on.

### Root Cause
- Default Vite port range (5173–5180) is fixed and exhausted by:
  - Multiple previous dev sessions still running
  - Other Node.js dev servers (multiple users, or accidental backgrounding)
  - Port conflict from previous failed attempts
- The CLI automatically retries but doesn't allow configuration

### Impact
- ⏱️ Adds 2-5 seconds to startup
- ❌ Confusing logs if user reads old port numbers
- 🔧 No way to prefer a specific port range

### Recommended Fix

**Location:** `ms app dev` CLI and `create-app` skill Step 10

#### Fix 4a: Add `--port` Flag to `ms app dev`

```bash
ms app dev --port 8000
```

Allows users to explicitly set their preferred port instead of relying on auto-detection.

#### Fix 4b: Document Port Conflicts in Troubleshooting

**Location:** `create-app/references/troubleshooting.md`

Add a new section:

```markdown
## Troubleshooting: Dev Server Startup Delays

### Symptom
`ms app dev` tries multiple ports (5173, 5174, 5175...) before starting.

### Cause
Other Node.js dev servers are running on the default Vite port range.

### Solutions

**Option 1: Kill old processes (recommended for most users)**
```bash
# macOS / Linux
lsof -ti:5173 | xargs kill

# Windows PowerShell
Stop-Process -Id (Get-NetTCPConnection -LocalPort 5173).OwningProcess

# Check what's running on a port
netstat -ano | findstr ":5173"
```

**Option 2: Use a specific port**
```bash
ms app dev --port 8000
```

**Option 3: Use a different port range**
Set environment variable before running:
```bash
# macOS / Linux
export VITE_PORT_START=9000
ms app dev

# Windows PowerShell
$env:VITE_PORT_START=9000
ms app dev
```

### Prevention
When finished developing, press Ctrl+C to stop the dev server. Don't just close the terminal.
```

#### Fix 4c: Improve Startup Messaging

When the CLI finds an available port after retries, show:

```
  ⚠️  Ports 5173–5179 are in use (old dev sessions?)
  ➜  Local Play:   https://play.preview.managedapps.cloud.microsoft/apps/dev?...
  ➜  Local:        http://localhost:5180/  ← Using port 5180 instead
  
  Tip: Next time, use 'ms app dev --port 8000' to choose your port explicitly.
```

**Result:** User understands why the port changed and knows how to control it in the future.

---

## Summary Table: Recommended Plugin Updates

| Issue | File/Skill | Recommendation | Priority | Est. Effort |
|-------|-----------|-----------------|----------|------------|
| #1: Domain patterns | `create-app/references/domain-patterns.md` | Create new reference with common app scenarios & recommended connectors | High | 1 day |
| #2: GCM auth trap (proactive) | `create-app/references/prerequisites-reference.md` | Add GCM readiness check before `ms app create` | High | 2 days |
| #2: GCM auto-retry | `create-app` skill Step 7 | Implement auto-retry on "Authentication failed" | High | 1 day |
| #2: GCM messaging | `troubleshooting.md` | Clarify GCM first-run behavior & recovery options | High | 0.5 day |
| #3: Recovery guardrails | `troubleshooting.md` | Update recovery flow to explain intermediate states & permission errors | Medium | 1 day |
| #4: Port documentation | `troubleshooting.md` | Add port conflict diagnosis & solutions section | Low | 0.5 day |
| #4: Port flag | `ms app dev` CLI | Add `--port` flag support | Medium | 0.5 day |
| #4: Port messaging | `create-app` Step 10 | Improve startup output when port fallback occurs | Low | 0.25 day |

**Total Estimated Effort:** 5.75 days (prioritize issues #1, #2, #3)

---

## Implementation Roadmap

### Phase 1: Critical Improvements (Address user friction)
1. ✅ **Domain patterns reference** — Guide Copilot when planning connector selection
2. ✅ **GCM auto-retry** — Eliminate the need for destructive manual recovery
3. ✅ **Clearer error messaging** — Help users understand what happened

### Phase 2: Polish (Improve DX)
4. ✅ **Recovery guardrails** — Make destructive operations safer & more understandable
5. ✅ **Port conflict docs** — Give users actionable solutions
6. ✅ **Port flag support** — Let users avoid conflicts proactively

---

## Appendix: Full Issues List (JSON Format)

```json
{
  "session": "meeting-summary-app-2026-06-23",
  "issues": [
    {
      "id": "issue-1-domain-recommendations",
      "severity": "medium",
      "category": "Skill Guidance",
      "title": "Missing domain-specific connector recommendations",
      "symptom": "User had to ask which AI service to use instead of getting a recommendation",
      "rootCause": "create-app skill lacks domain patterns for common scenarios",
      "recommendation": "Add domain-patterns.md reference with connector suggestions for 'meeting summarization', 'team collaboration', 'dashboard', etc.",
      "files": ["create-app/references/domain-patterns.md"]
    },
    {
      "id": "issue-2-gcm-trap-documentation",
      "severity": "high",
      "category": "Tooling/First-Run",
      "title": "GCM first-run trap causes non-obvious failure",
      "symptom": "App creation fails with 'Authentication failed' error but app exists in cloud",
      "rootCause": "Git Credential Manager needs interactive auth on first use per environment",
      "recommendation": "Add proactive GCM check before 'ms app create'; implement auto-retry on 'Authentication failed'",
      "files": ["create-app/references/prerequisites-reference.md", "create-app/troubleshooting.md"]
    },
    {
      "id": "issue-3-gcm-retry-friction",
      "severity": "medium",
      "category": "UX/Recovery",
      "title": "GCM retry loop has confusing intermediate states",
      "symptom": "Recovery instructions fail due to permission errors; user stuck in limbo state",
      "rootCause": "Recovery flow assumes users have app deletion permissions and doesn't handle permission denied",
      "recommendation": "Update recovery instructions to explain intermediate state and offer fallback (fresh folder) if deletion fails",
      "files": ["troubleshooting.md"]
    },
    {
      "id": "issue-4-port-exhaustion",
      "severity": "low",
      "category": "Development Experience",
      "title": "Dev server port exhaustion",
      "symptom": "Dev server tries 8 ports before starting; users confused about actual port",
      "rootCause": "Port range 5173–5180 exhausted by old dev sessions; no configuration option",
      "recommendation": "Add --port flag to 'ms app dev'; document port conflict diagnosis and solutions",
      "files": ["create-app/references/troubleshooting.md", "ms app dev CLI"]
    }
  ]
}
```

---

## Notes for Plugin Maintainers

### Testing This Report
To validate the fixes:

1. **Test domain patterns:** Build a "Task Management" app and verify the skill recommends "Dataverse + Teams"
2. **Test GCM recovery:** Run `ms app create` on a fresh account in a new environment to trigger first-run auth
3. **Test port fallback:** Run multiple `ms app dev` sessions and verify the startup messaging is clear
4. **Test error messages:** Introduce a permission error in the delete flow and verify users understand the fallback

### Integration with CI/CD
When integrating these improvements, consider:
- Adding E2E tests for GCM recovery flow
- Documenting expected startup times (with/without port fallback)
- Collecting telemetry on which app patterns users create most frequently (to prioritize future domain patterns)

