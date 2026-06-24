---
name: add-workiq
description: Adds Work IQ Copilot MCP by delegating to /add-connector with api-id=shared_a365copilotchatmcp and action mode. Use when users need Microsoft 365 knowledge-grounded Work IQ search/chat.
user-invocable: true
allowed-tools: Read, AskUserQuestion, Skill
model: sonnet
---

**📋 Shared Instructions: [shared-instructions.md](${CLAUDE_PLUGIN_ROOT}/shared/shared-instructions.md)** — Cross-cutting concerns.

# Add Work IQ Copilot MCP (Wrapper)

This skill is a thin wrapper. Use `/add-connector` as the single implementation path.

## Delegation contract

Invoke `/add-connector` with:

- `api-id`: `shared_a365copilotchatmcp`
- `mode`: `action`

## When to use

Use this skill when the user asks for Work IQ knowledge/search/chat over Microsoft 365 content and does not explicitly require workload-specific tools (mail/calendar/teams/word/user).
