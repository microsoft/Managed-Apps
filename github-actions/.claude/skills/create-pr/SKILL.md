---
name: create-pr
description: Prepare and open a release PR for the github-actions component. Branches from the latest origin/main, bumps github-actions/package.json, updates github-actions/CHANGELOG.md, rebuilds dist/, commits, pushes, and opens a remote PR. Use when shipping a change to the actions under github-actions/.
user-invocable: true
allowed-tools: Bash, Read, Edit, Write, AskUserQuestion
---

# Create release PR for github-actions

Opens a PR that bumps the version, updates the changelog, and rebuilds the
esbuild bundles for the actions under `github-actions/`, branching from the
latest `origin/main`. The version bump is what the release workflow keys on, so
this skill keeps `package.json`, `CHANGELOG.md`, and `dist/` in lockstep.

## Arguments

`/create-pr <patch|minor|major> [short summary of the change]`

- **bump** (required): `patch`, `minor`, or `major`.
- **summary** (optional): one line describing the change; used for the changelog
  entry and PR title.

Do not guess the bump type — if it is missing, ask. Reminder of the contract:
patch = bug fix (no surface change), minor = new backward-compatible capability,
major = breaking change.

## Procedure

Run in order. If any step fails, stop and report exactly what failed — never
leave a half-prepared branch pushed.

### 1. Resolve inputs
- Parse `bump` and `summary` from the arguments.
- If `bump` is absent → ask with `AskUserQuestion` (patch / minor / major, with
  the semver meanings above).
- If `summary` is absent → ask the user for a one-line summary.

### 2. Preflight
- `gh auth status` — confirm authenticated; abort if not.
- `git status --porcelain` — note whether the working tree has changes. Any
  uncommitted changes are the code being released and will be carried onto the
  new branch. If the tree is clean, tell the user this will be a
  **version-bump-only** PR and confirm before continuing.

### 3. Compute the new version
- Read current: `node -p "require('./github-actions/package.json').version"`.
- Apply the bump → `<NEW>`:
  - patch → `x.y.(z+1)`
  - minor → `x.(y+1).0`
  - major → `(x+1).0.0`

### 4. Branch from latest origin/main
- `git fetch origin`
- `git checkout -b "release/github-actions-v<NEW>" origin/main`
  - This carries any uncommitted working-tree changes onto the new branch.
  - If the branch name already exists, append `-2`, `-3`, … or abort and ask.

### 5. Bump package.json
- Edit `github-actions/package.json` `"version"` → `<NEW>`.

### 6. Update CHANGELOG.md (`github-actions/CHANGELOG.md`)
- Get today's date: `date +%F`.
- Rename `## [Unreleased]` → `## [<NEW>] - <today>`.
- Insert a fresh, empty `## [Unreleased]` heading above it.
- Ensure the change is described under the new version using
  `### Added` / `### Changed` / `### Fixed` / `### Removed`, framed by what a
  consumer pinning `@v<major>` would notice. Fold in the provided summary and any
  bullets that were already accumulating under `[Unreleased]`.

### 7. Rebuild bundles
- `cd github-actions && npm ci && npm run build`
  (required so the release workflow's `dist/` drift-guard passes).
- Sanity check: `git status --porcelain github-actions/dist` — fine whether or
  not it changed; the point is dist now matches source.

### 8. Commit, push, open PR
- `git add -A`
- Commit. End the message with:
  `Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>`
- `git push -u origin "release/github-actions-v<NEW>"`
- Open the PR against `main`:
  - `gh pr create --base main --title "github-actions v<NEW>: <summary>" --body "<body>"`
  - Body: what changed, the version bump (`<old> → <NEW>`), and a note that the
    matching `CHANGELOG.md` entry is included.
  - If `gh pr create` fails on the Projects-classic GraphQL deprecation, retry
    via REST: `gh api -X POST repos/{owner}/{repo}/pulls -f title=... -f head=... -f base=main -F body=@<file>`.
- Report the PR URL back to the user.

## Notes
- Touch **only** `github-actions/` — do not bump versions or changelogs elsewhere
  in the repo.
- The new `package.json` version must not already have a `v<NEW>` tag (the release
  workflow fails otherwise). If it does, choose a higher bump.
- This skill prepares the PR; it does not merge. The release tag + `v<major>`
  alias are created by the release workflow when the PR merges to `main`.
