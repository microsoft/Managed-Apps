# Plugin Implementation Checklist

Based on the Meeting Summary app session (2026-06-23), use this checklist to implement improvements.

## Phase 1: Critical (Do These First)

### [ ] Issue #1: Domain Patterns Reference
- [ ] Create `create-app/references/domain-patterns.md`
- [ ] List 3+ common app scenarios:
  - Meeting Summarization (Office 365 + Work IQ)
  - Team Collaboration (Teams + SharePoint)
  - Data Dashboard (Dataverse + Excel)
  - Knowledge Search (Work IQ + SharePoint)
- [ ] Update `create-app` Step 4 (Plan) to check patterns when user mentions keywords
- [ ] Test: Build an app with "meeting" in description → verify Work IQ suggestion appears

### [ ] Issue #2: GCM Auto-Retry
- [ ] Wrap `ms app create` call with retry logic (2 attempts max)
- [ ] Detect "Authentication failed" in output
- [ ] Sleep 3-5 seconds between retries
- [ ] Add logging: "GCM auth needs a moment, retrying..."
- [ ] Test: Fresh account, new environment → `ms app create` should succeed on 2nd try without user intervention

### [ ] Issue #2: GCM Proactive Check
- [ ] Add pre-flight GCM test before `ms app create`:
  ```bash
  git ls-remote "$TEST_URL" 2>/dev/null && echo "✓ GCM ready" || echo "⚠ GCM needs setup"
  ```
- [ ] If test fails, trigger `git fetch` to cache auth (silent fail is OK)
- [ ] Test: Fresh GCM cache → verify test triggers browser auth once

### [ ] Issue #2: Clearer Error Messaging
- [ ] Update `troubleshooting.md` with GCM first-run section
- [ ] Explain: "App created in cloud, local setup encountered auth"
- [ ] Explain: "Retry will work because GCM is now cached"
- [ ] Provide fallback: "If still failing, use a fresh folder name"

---

## Phase 2: Polish (Do These Next)

### [ ] Issue #3: Recovery Guardrails
- [ ] Update recovery section in `troubleshooting.md`
- [ ] Add: Check app deletion succeeded before deleting folder
- [ ] Add: Handle `403 Forbidden` → suggest fresh folder instead of destructive cleanup
- [ ] Add: Verify app list shows app before cleanup
- [ ] Test: Fresh folder recovery → verify safety checks work

### [ ] Issue #4: Port Documentation
- [ ] Add "Port Conflicts" section to `troubleshooting.md`
- [ ] Include OS-specific commands:
  - `lsof -ti:5173 | xargs kill` (macOS/Linux)
  - `Stop-Process -Id (Get-NetTCPConnection -LocalPort 5173).OwningProcess` (Windows)
  - `netstat -ano | findstr ":5173"` (Windows diagnosis)
- [ ] Add: "Tip: Use `ms app dev --port 8000` to avoid conflicts"

### [ ] Issue #4: Port Flag Support
- [ ] Add `--port` flag to `ms app dev` CLI
- [ ] Allow users to specify custom port
- [ ] Document: `ms app dev --port 8080`
- [ ] Test: `ms app dev --port 8080` → verify app runs on 8080

### [ ] Issue #4: Startup Messaging
- [ ] Improve port fallback message in dev server startup
- [ ] Show: "Ports 5173–5179 in use (old sessions?)" instead of repetitive trials
- [ ] Show: "Tip: Use --port 8000 next time"
- [ ] Test: Run with exhausted ports → verify message is helpful

---

## Phase 3: Validation (Testing)

### [ ] End-to-End Tests
- [ ] Test fresh account → `ms app create` + domain pattern suggestion → app runs locally
- [ ] Test GCM recovery → first attempt fails → auto-retry succeeds
- [ ] Test permission denied → cleanup fails → user offered fallback
- [ ] Test port conflict → dev server finds available port → startup message explains why

### [ ] Documentation Reviews
- [ ] Domain patterns: Is each example clear? Would a user understand the "why"?
- [ ] Error messages: Would a user know what to do next?
- [ ] Recovery flow: Does it include enough guardrails to be safe?

### [ ] User Feedback
- [ ] Share updated `troubleshooting.md` with beta users
- [ ] Ask: "Did the auto-retry fix the GCM issue?"
- [ ] Ask: "Was the error message helpful?"
- [ ] Iterate based on feedback

---

## Rollout Plan

### Sprint 1: Critical Fixes (Week 1–2)
1. Implement auto-retry (Issue #2)
2. Add GCM proactive check (Issue #2)
3. Create domain-patterns.md (Issue #1)

**Release as:** `@microsoft/managed-apps-cli@0.10.1`

### Sprint 2: Polish (Week 3–4)
1. Add port flag support (Issue #4)
2. Update troubleshooting guide (Issues #2, #3, #4)
3. Improve error messaging (Issue #2)

**Release as:** `@microsoft/managed-apps-cli@0.11.0`

### Sprint 3: Validation & Iteration (Week 5–6)
1. Collect user feedback
2. Iterate based on feedback
3. Document edge cases

**Release as:** `@microsoft/managed-apps-cli@0.11.1` (patch)

---

## Files to Modify

```
microsoft-managed-apps/
├── skills/
│   └── create-app/
│       ├── README.md                           # Update Step 7 to mention auto-retry
│       ├── references/
│       │   ├── prerequisites-reference.md       # Add GCM proactive check section
│       │   ├── troubleshooting.md              # Add Issues #2, #3, #4 sections
│       │   └── domain-patterns.md              # NEW FILE
│       └── shared/
│           └── shared-instructions.md          # Update pattern-matching logic
├── ms-cli/                                      # (if in this repo)
│   └── app-dev.ts                              # Add --port flag
└── PLUGIN_IMPROVEMENTS.md                       # Reference guide (this doc)
```

---

## Success Criteria

- [x] **Issue #1 (Domain Patterns):** User asks "build app" → Copilot suggests correct connectors without asking
- [x] **Issue #2 (GCM Auto-Retry):** Fresh account → `ms app create` succeeds on attempt 2 without user knowing
- [x] **Issue #3 (Recovery Guardrails):** Permission error during recovery → user offered safe fallback instead of stuck state
- [x] **Issue #4 (Port Conflicts):** User runs dev server with exhausted ports → startup explains why and how to avoid next time

---

## Notes

- **Keep domain patterns maintainable:** Don't let it grow beyond 5–8 patterns. Archive edge cases to troubleshooting.
- **Test GCM on multiple clouds:** Public Azure + GCC + GCC High if applicable.
- **Consider env variables:** Allow users to set `MANAGED_APPS_PORT_START` or `MANAGED_APPS_PORT_RANGE` if --port flag isn't enough.

