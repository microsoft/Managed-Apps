# Connector Reference

Applies to all `/add-*` skills.

## Connections — handled inline by the CLI

When you run `ms app add action --api-id <id>` or `ms app add table --api-id <id> ...`, the CLI:

1. Checks whether the active environment has a connection for that connector.
2. If it does, prompts you to pick which one (or accepts `--connection-id <id>` non-interactively).
3. If it doesn't, walks you through creating one inline (OAuth consent flow in a browser).

You do NOT need to pre-create connections via the maker portal before running an `/add-*` skill, and you do NOT need to pass `--connection-id` unless you want to bypass the interactive picker.

For non-interactive runs (`--non-interactive`), pass `--connection-id` explicitly. Discover connection IDs with `/list-connections`.

### Dataverse is different

Dataverse doesn't use the connection-id model — `ms app add table --api-id dataverse --table <name>` resolves the active environment's Dataverse automatically. No `--connection-id` is required (or accepted).

## Inspecting Large Generated Files

Generated service files (e.g., `Office365OutlookService.ts`) can be thousands of lines. **Do NOT read the entire file.** Instead:

1. **List available methods** with Grep:
   ```
   Grep pattern="async \w+" path="src/<service-folder>/<Connector>Service.ts"
   ```

2. **Find a specific method** and read just that section:
   ```
   Grep pattern="async GetEventsCalendarViewV2" path="src/<service-folder>/Office365OutlookService.ts" -A 20
   ```

3. **Find parameter types** in the models file:
   ```
   Grep pattern="interface CalendarEventHtmlClient" path="src/<service-folder>/Office365OutlookModel.ts" -A 30
   ```

This avoids context-window bloat. Exact subdirectory layout under `src/` is owned by `@microsoft/apps-actions` — codegen output lands directly under `src/` alongside handwritten code.

## Sub-Skill Invocation

When a connector skill is invoked from another skill (e.g., `/create-app` plans `/add-office365`):

- **Check `$ARGUMENTS`** — if provided, use it as the connector name or configuration.
- **Skip redundant questions** — don't re-ask things the caller already provided (project path, connection id, etc.).
- **Memory bank is still read** — but skip the summary if the caller just updated it.

## Build After, Don't Deploy

Every `/add-*` skill runs `npm run build` after the `ms app add ...` call to catch type errors in the generated services. **None of them push or deploy.** Deployment happens only via `/deploy`, which always requires explicit user confirmation.
