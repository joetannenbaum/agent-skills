---
name: Raycast Script
description: Create a Raycast script command from a description. Use when asked for a Raycast script, script command, or "make me a Raycast command for X".
---

Write the script I describe as a Raycast script command and save it in `/Users/joetannenbaum/Documents/raycast`.

## Rules

- One file per command, named in kebab-case after the title, ending in `.sh`. Bash unless the task genuinely needs another language.
- `chmod +x` the file after writing it. Raycast ignores anything that isn't executable.
- Shebang is `#!/bin/bash`. Use `#!/bin/bash -l` only when the script needs a full login shell.
- If the script calls anything installed through my shell (`steve`, `laravel`, `pnpm`, `trash`, custom aliases and functions), add `source "$HOME/.zshrc"` after the metadata block. Plain system binaries (`sed`, `jq`, `open`, `osascript`, `pbcopy`) don't need it.
- Keep the output line short and with a bit of personality — Raycast shows it as a toast.
- Don't add comments explaining what the code does; the metadata block already covers that.
- Read a couple of the existing scripts in the directory first and match their shape.

## Metadata block

Goes directly under the shebang, in this order and with these headings:

```bash
#!/bin/bash

# Required parameters:
# @raycast.schemaVersion 1
# @raycast.title Some Title
# @raycast.mode silent

# Optional parameters:
# @raycast.icon 🤖
# @raycast.description What it does
# @raycast.argument1 { "type": "text", "placeholder": "Placeholder" }

# Documentation:
# @raycast.author joetannenbaum
# @raycast.authorURL https://raycast.com/joetannenbaum
```

Always include the author and authorURL lines exactly as above.

### Fields

| Field | Notes |
| --- | --- |
| `title` | Title case, what I'd type to find it |
| `mode` | `silent` (toast only), `compact` (one-line result), `fullOutput` (full window), `inline` (menu bar, needs `refreshTime`) |
| `icon` | An emoji. `🤖` is the fallback when nothing better fits |
| `description` | One line, shown under the title in Raycast |
| `packageName` | Groups commands together — skip unless I ask |
| `currentDirectoryPath` | Absolute path the script runs in |
| `needsConfirmation` | `true` for anything destructive |
| `refreshTime` | `10s`, `1m`, `1h`, `1d` — required for `inline` mode |

### Arguments

Up to three, `argument1` through `argument3`, each a JSON object on one line. They arrive as `$1`, `$2`, `$3`.

```bash
# @raycast.argument1 { "type": "text", "placeholder": "What did you do" }
# @raycast.argument2 { "type": "text", "placeholder": "App Name", "optional": true }
# @raycast.argument3 { "type": "dropdown", "placeholder": "Kit", "data": [{"title": "React", "value": "react"}, {"title": "Vue", "value": "vue"}] }
```

Types are `text`, `password`, and `dropdown`. Optional arguments must come after required ones. Default optional values with `${2:-fallback}`.

## After writing

Tell me the file path and the title I'll type into Raycast to run it.
