# Troubleshooting

## First-Run Git Credential Manager Trap

**The most common failure on the first `ms app create` for a new account.**

**Shell note:** if you are in PowerShell and see `export: command not found`, you are looking at a bash-only snippet. Rerun the command without the `export` prefix, or use the PowerShell equivalents from [prerequisites-reference.md](./prerequisites-reference.md).

### Symptom

```
fatal: Authentication failed for 'https://<env-id>.d.environment.api.powerplatform.com/appframework/git/repositories/<repo-guid>/'
App '<name>' was created, but local setup failed: Command failed: git fetch origin
```

### Cause

`ms app create` provisions the app in the service and writes a `[credential ...]` block to `.git/config` pointing at the right OAuth client + scopes. But Git Credential Manager has to run its interactive browser flow at least once to mint a token for the remote endpoint. On the very first run for that user/tenant/cluster combo, GCM hasn't done that flow, so `git fetch origin` 401s before the local template scaffold can complete.

The result: the app exists in the service catalog, but the local directory is empty (or has only an auto-init `README.md`).

### Recovery

Before running recovery, ask for explicit user confirmation because this sequence deletes files in the current project folder.

```bash
PROJECT_ROOT="$(pwd)"
cd "$PROJECT_ROOT"

# Trigger GCM's interactive browser flow against the remote.
git fetch origin                                                    # browser opens; approve.

# Delete the half-formed app from the service.
# Set APP_ID to the created app GUID from the create output (or `ms app list --json`) first.
$BIN app delete --app "$APP_ID" --force --non-interactive

# Clean slate.
[ -n "$APP_ID" ] && [ -n "$PROJECT_ROOT" ] || { echo "Missing APP_ID or PROJECT_ROOT; refusing cleanup."; exit 1; }
[ "$PROJECT_ROOT" != "$HOME" ] && [ "$PROJECT_ROOT" != "/" ] || { echo "Refusing cleanup at unsafe path: $PROJECT_ROOT"; exit 1; }
find "$PROJECT_ROOT" -mindepth 1 -maxdepth 1 \
	! -name '.git' \
	! -name '.DS_Store' \
	! -name 'Thumbs.db' \
	! -name '.vscode' \
	-exec rm -rf {} +
cd "$PROJECT_ROOT"

# Re-run create. Auth is now cached, so the second attempt completes end-to-end.
$BIN app create --display-name "$DISPLAY_NAME" --non-interactive
```

### Detection

Match the substring `Authentication failed for 'https://` followed by `.d.environment.api.` in the `ms app create` output. When detected, surface the recovery sequence and ask for explicit approval before running destructive cleanup.

---

## Common `ms app create` Failures

| Error                                                                             | Cause                                                                                  | Fix                                                                                                                       |
| --------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| `defau.lt.environment.api.powerplatform.com: no such host` (or similar DNS error) | A malformed `--environment-id` value was passed (this only happens when the user supplied one). | Surface the error. Drop the explicit `--environment-id` and let `ms app create` auto-route, or have the user supply a valid environment ID. |
| `Could not provision a Developer environment for your tenant (status 403)`        | Routing service rejected the account. Tenant Governance may block Developer envs.      | Surface the error to the user — provisioning is blocked at the tenant level and the plugin cannot work around it. The user (or their tenant admin) needs to resolve the governance/access issue. |
| `Directory not empty; pass --force`                                               | Current project folder has prior files.                                                 | Run `/create-app` from an empty folder, or confirm using `--force` only when you intend to overwrite.                    |
| Repo init fails (`fatal: not in a git repository`)                                | Git is missing or `git config user.email` / `user.name` are unset.                     | Install Git; run `git config --global user.email "<you>@microsoft.com"` and `... user.name "<You>"`.                       |

## Common Build / Dev Failures

| Problem                                                              | Solution                                                                                                 |
| -------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| `npm install` fails with `EACCES` or permission errors               | Don't `sudo`. Use `nvm` or fix the global prefix (`npm config get prefix`).                              |
| `npm run build` fails with TS6133 (unused import)                    | Remove the unused import and retry once.                                                                 |
| `npm run build` fails with module-not-found                          | Run `npm install` and retry.                                                                             |
| `ms app dev` exits immediately with port-in-use                      | Pass `--port <other>` or kill whatever's on 8080.                                                        |
| `ms app dev` connector calls return 401/403 in the browser           | Auth session expired. Re-authenticate with `ms auth login` (check with `ms auth status`), then refresh the App Player tab. |
| Binary "command not found" after `npm install -g`                    | `npm config get prefix` directory's `bin` isn't on PATH. On Windows, default is `%APPDATA%\npm` — usually added by the Node installer, but a manual install or PowerShell-profile override can break this. |
| `npx ms ...` resolves to an unrelated package                        | Don't use `npx ms`; use the globally-installed binary directly. The public-registry `ms` is a date-parser shim. |

## Auth Failures

| Symptom                                                                          | Fix                                                                                                          |
| -------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `ms auth status` reports the wrong UPN                                           | `ms auth login` (interactive). Don't rely on cached state across tenants — always check `auth status` first. |
| `ms auth login` browser flow times out                                           | Re-run; the underlying MSAL flow has a finite window. If a popup blocker is involved, allow it for `login.microsoftonline.com`. |
| Token works for `ms app list` but `ms app create` returns 403                    | The account lacks Maker permissions in the target tenant. Either pick a different account or escalate access. |

## Resources

- **Connectors reference**: https://learn.microsoft.com/en-us/connectors/connector-reference/
- **Dataverse docs**: https://learn.microsoft.com/en-us/power-platform/
