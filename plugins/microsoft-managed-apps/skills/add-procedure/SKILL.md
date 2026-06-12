---
name: add-procedure
description: Adds SQL stored procedures by delegating to `/add-connector` with `api-id=shared_sql` and procedure mode. Use when binding SQL procedures.
user-invocable: true
allowed-tools: Read, AskUserQuestion, Skill
model: sonnet
---

**📋 Shared Instructions: [shared-instructions.md](${CLAUDE_PLUGIN_ROOT}/shared/shared-instructions.md)** — Cross-cutting concerns.

# Add SQL Stored Procedure (Wrapper)

This skill is a thin wrapper. Use `/add-connector` as the single implementation path.

## Delegation contract

Invoke `/add-connector` with:

- `api-id`: `shared_sql`
- `mode`: `procedure`
- required args: `connection-id`, `dataset`, `sql-stored-procedure`

Collect missing required values first, then delegate.
