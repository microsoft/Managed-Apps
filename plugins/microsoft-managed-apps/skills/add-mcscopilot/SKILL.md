---
name: add-mcscopilot
description: Adds Microsoft Copilot Studio by delegating to `/add-connector` with `api-id=microsoftcopilotstudio` and action mode. Use when integrating Copilot Studio agents.
user-invocable: true
allowed-tools: Read, AskUserQuestion, Skill
model: sonnet
---

**📋 Shared Instructions: [shared-instructions.md](${CLAUDE_PLUGIN_ROOT}/shared/shared-instructions.md)** — Cross-cutting concerns.

# Add Microsoft Copilot Studio (Wrapper)

This skill is a thin wrapper. Use `/add-connector` as the single implementation path.

## Delegation contract

Invoke `/add-connector` with:

- `api-id`: `microsoftcopilotstudio`
- `mode`: `action`
