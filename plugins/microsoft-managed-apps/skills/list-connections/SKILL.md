---
name: list-connections
description: Lists connectors and connection-bound data sources for a Microsoft App. Use when discovering existing connections before adding data sources.
user-invocable: true
allowed-tools: Read, Bash, AskUserQuestion
model: sonnet
---

**📋 Shared Instructions: [shared-instructions.md](${CLAUDE_PLUGIN_ROOT}/shared/shared-instructions.md)** — Cross-cutting concerns.

# List Connections

Two read-only views, depending on what the user is asking:

1. **What data sources are already bound to *this* app?** → `ms app show --json` (inspects `ms.config.json`).
2. **What connectors are reachable in the active environment?** → `ms connector list-actions --connector <id>` for a specific connector, or browse the public reference for available api-ids.

## Workflow

1. Memory Bank → 2. Verify Env → 3. Mode → 4. Run + Present

---

### Step 1: Check Memory Bank

Read `memory-bank.md` for the app's environment, if recorded. This is for context/verification only (confirming you're working against the expected app/environment) — `ms app show --json` and `ms connector list-actions` don't take environment targeting from it and the listing isn't scoped by it.

### Step 2: Verify Env

```bash
BIN=ms
$BIN auth status                                      # confirm the active UPN
```

### Step 3: Mode

Ask (or infer):

- **app**: enumerate data sources already wired into the current project (requires `ms.config.json` in cwd).
- **connector**: list the operations exposed by a specific connector — useful when planning an `/add-*` invocation.

### Step 4a: App-bound data sources

```bash
test -f ms.config.json || { echo "Not in a Microsoft App workspace."; exit 1; }
$BIN app show --json
```

Parse the response for the data-source list and print a summary:

| Connector       | api-id                            | Type     | Name                  |
| --------------- | --------------------------------- | -------- | --------------------- |
| Dataverse       | `dataverse`                       | table    | `contact`             |
| SharePoint Online | `shared_sharepointonline`        | table    | `Project Milestones`  |
| Teams           | `shared_teams`                    | action   | (entire connector)    |

Use this to spot drift (e.g., `ms.config.json` claims a data source that no longer exists in the env).

### Step 4b: Discover operations on a connector

The CLI prompts for / creates connections inline when an `/add-*` skill runs, so users rarely need to look up connection IDs ahead of time. The more useful query is "what can this connector do?":

```bash
$BIN connector list-actions --connector <api-id> [--search <term>]
```

Examples:

```bash
$BIN connector list-actions --connector shared_office365 --search Mail
$BIN connector list-actions --connector shared_sharepointonline --search GetItems
```

Output is a list of operation names and their summaries. Use it to confirm an api-id supports the user's intent before invoking the relevant `/add-*` skill.

---

## When the user wants a connection ID

The MAAF-CLI's add flows create or reuse connections on the fly. You don't need a connection ID to start `/add-*`. Two exceptions:

- **`--non-interactive`**: when scripting, the CLI can't open a browser to create a connection, so it expects `--connection-id`. Discover one via `ms app show --json` for connections already bound, or via the maker-portal Connections page.
- **`add procedure` for SQL**: this skill expects an explicit `--connection-id` because each SQL connection points at a different database.

When a connection ID is required and the user doesn't have one, have them mint it by running the relevant `/add-*` skill interactively (omit `--non-interactive`). The CLI opens a browser through the Microsoft Apps player to create and consent the connection, then reuses it — afterwards the connection ID is discoverable via `ms app show --json`.
