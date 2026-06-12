---
name: add-teams
description: Adds Microsoft Teams by delegating to `/add-connector` with `api-id=shared_teams` and action mode. Use when integrating Teams messaging.
user-invocable: true
allowed-tools: Read, AskUserQuestion, Skill
model: sonnet
---

**📋 Shared Instructions: [shared-instructions.md](${CLAUDE_PLUGIN_ROOT}/shared/shared-instructions.md)** — Cross-cutting concerns.

# Add Teams (Wrapper)

This skill is a thin wrapper. Use `/add-connector` as the single implementation path.

## Delegation contract

Invoke `/add-connector` with:

- `api-id`: `shared_teams`
- `mode`: `action`
