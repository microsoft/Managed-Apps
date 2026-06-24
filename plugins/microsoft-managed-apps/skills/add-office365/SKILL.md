---
name: add-office365
description: Adds Office 365 Outlook by delegating to `/add-data-source` with `api-id=shared_office365` and action mode. Use when integrating Outlook mail/calendar.
user-invocable: true
allowed-tools: Read, AskUserQuestion, Skill
model: sonnet
---

**📋 Shared Instructions: [shared-instructions.md](${CLAUDE_PLUGIN_ROOT}/shared/shared-instructions.md)** — Cross-cutting concerns.

# Add Office 365 Outlook (Wrapper)

This skill is a thin wrapper. Use `/add-data-source` as the single implementation path.

## Delegation contract

Invoke `/add-data-source` with:

- `api-id`: `shared_office365`
- `mode`: `action`
