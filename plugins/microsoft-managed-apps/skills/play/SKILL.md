---
name: play
description: Opens a Microsoft App in the browser via `ms app play`. Use when opening the live deployed version or a cloud-hosted preview (main branch or specific commit SHA) without starting a local dev server.
user-invocable: true
allowed-tools: Read, Grep, Bash, AskUserQuestion
model: sonnet
---

**📋 Shared Instructions: [shared-instructions.md](${CLAUDE_PLUGIN_ROOT}/shared/shared-instructions.md)** — Cross-cutting concerns.

# Play

Opens a Microsoft App in the browser by fetching its cloud play URL and launching it. No local dev server required.

| Mode | Command | What opens |
| ---- | ------- | ---------- |
| `live` (default) | `ms app play` | The most recently deployed version. A snapshot — does not update until `ms app deploy` is run again. |
| `preview` (main) | `ms app play --mode preview` | Built from the `main` branch on demand — always reflects the latest pushed code, even without a deploy. |
| `preview` (commit) | `ms app play --mode preview --commit <sha>` | App as of a specific commit SHA — useful for code review or regression testing. |

> **Live vs. preview:** Live requires a prior `ms app deploy`. Preview builds from source on demand — it works without any deployment and always reflects unpublished code.

## Workflow

1. Memory Bank → 2. Verify Project + Env → 3. Resolve Mode → 4. Run → 5. Surface URL

---

### Step 1: Check Memory Bank

Read `memory-bank.md` per [shared-instructions.md](${CLAUDE_PLUGIN_ROOT}/shared/shared-instructions.md) for the app's environment/app identifiers, if recorded.

### Step 2: Verify Project + Env

```bash
test -f ms.config.json || { echo "Not in a Microsoft App workspace."; exit 1; }
BIN=ms
$BIN auth status                                         # must report the expected UPN
```

`--app` and `--environment-id` can be passed explicitly to run from outside a project directory — `ms.config.json` is only needed for the defaults.

### Step 3: Resolve Mode

If not already clear from the user's request, ask:

> "Do you want to open the **live** deployed version, a **preview** of the latest code on main, or a preview of a **specific commit**?"

Map intent to mode:

| User says | Mode |
| --------- | ---- |
| "live", "deployed", no qualifier | `live` (default — no flag needed) |
| "preview", "latest", "unpublished", "before deploy" | `--mode preview` |
| "preview at commit X" / specific SHA provided | `--mode preview --commit <sha>` |

### Step 4: Run

**Live (default):**

```bash
$BIN app play
```

**Preview — latest code on main:**

```bash
$BIN app play --mode preview
```

**Preview — specific commit:**

```bash
$BIN app play --mode preview --commit <sha>
```

Full flag reference:

| Flag | Alias | Default | Description |
| ---- | ----- | ------- | ----------- |
| `--mode` | `-m` | `live` | `live` or `preview`. |
| `--commit` | `-c` | HEAD on main | Preview only. The commit SHA to open. Ignored for live mode (CLI throws if combined). |
| `--app` | | `appId` from `ms.config.json` | App name override for running outside the project directory. |
| `--no-browser` | | false | Print the URL without opening a browser tab. |
| `--json` | | | Emit `{ success, appName, mode, url, commitHash }` instead of human-readable output. |

### Step 5: Surface URL

Present the URL the CLI printed. For live mode, surface the CLI's built-in warning:

```
Live URL for 'my-app':
  https://apps.powerapps.com/play/e/<env-id>/app/<guid>

⚠ Live does not auto-update. Run /deploy to publish new code to the live URL.
```

For preview:

```
Preview URL for 'my-app' (main):
  https://apps.powerapps.com/play/e/<env-id>/app/<guid>/branch/main
```

For commit-pinned preview:

```
Preview URL for 'my-app' (commit abc1234):
  https://apps.powerapps.com/play/e/<env-id>/app/<guid>/commit/abc1234
```

---

## Edge Cases

| Situation | Action |
| --------- | ------ |
| `No live build available — run 'ms app deploy' first` | The app has never been deployed. Tell the user to run `/deploy`, then retry `ms app play`. |
| `--commit` passed with `--mode live` | Hard `UsageError` from the CLI. Explain that `--commit` only works with `--mode preview`. |
| No `appPlayUri` for preview mode | App record exists but play URI is missing. Surface the error verbatim and stop. |
| User wants URL without opening a browser | Pass `--no-browser`; the CLI prints the URL and exits without launching a tab. |
| User is outside a project directory | Pass `--app <name>` and `--environment-id <id>` explicitly. |
