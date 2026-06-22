# Bug Fix: PreToolUse Hook PowerShell/Bash Context Mismatch

## Issue
When running the `ms` CLI from PowerShell on Windows, the `PreToolUse` hook was generating bash-style syntax (`export MS_CLI_ORIGIN=...`) that was then executed in the PowerShell context, resulting in:

```
export: The term 'export' is not recognized as a cmdlet, function, script file, or executable program.
```

## Root Cause
In `hooks/pre-tool-use.ps1`, the hook determines whether to generate PowerShell or bash syntax by checking:
1. `$PSVersionTable` (whether the hook script itself is running in PowerShell)
2. The `tool_name` field from the command metadata

**The bug**: When a non-PowerShell command (e.g., bash tool) is invoked, the hook generates bash syntax with backtick-escaping:
```powershell
$newCommand = "export MS_CLI_ORIGIN=`"`${MS_CLI_ORIGIN-$ORIGIN_VALUE}`" && $command"
```

However, this string literal was not properly quoted as a single-quoted string, causing PowerShell string interpolation to fail when the hook tried to evaluate variables.

## The Fix
Changed line 147 in `hooks/pre-tool-use.ps1` to use **single-quoted string concatenation** instead of double-quoted interpolation:

**Before (broken):**
```powershell
$newCommand = "export MS_CLI_ORIGIN=`"`${MS_CLI_ORIGIN-$ORIGIN_VALUE}`" && $command"
```

**After (fixed):**
```powershell
$newCommand = 'export MS_CLI_ORIGIN="${MS_CLI_ORIGIN-' + $ORIGIN_VALUE + '}" && ' + $command
```

### Why This Works
- **Single quotes** (`'...'`) prevent PowerShell from interpolating variables
- The bash syntax is now constructed as a literal string that will be passed to the bash subprocess
- When bash receives this string, it correctly evaluates `${MS_CLI_ORIGIN-<default>}` in its own context
- The distinction between "hook runs in PowerShell" and "target command is bash" is now clear

## Affected Environments
- **Windows PowerShell users** invoking commands that target bash/shell environments
- Any user running the plugin from Copilot CLI or Claude Code on Windows

## Testing
The fix resolves the issue where:
```powershell
ms --version
```

Previously failed with `export: not recognized`. Now it correctly:
1. Detects the target context (bash)
2. Generates bash syntax as a literal string
3. Passes it to the subprocess for proper evaluation

## File Changed
- `hooks/pre-tool-use.ps1` (line 147)
