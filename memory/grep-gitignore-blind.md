---
name: grep-gitignore-blind
description: Built-in Grep tool honors .gitignore and is blind to plugin dirs; use PowerShell Select-String
metadata: 
  node_type: memory
  type: feedback
---

The built-in Grep tool (ripgrep) respects `.gitignore`. The plugin directories under `storage/lib/plugins/` are git-ignored from the `dokuwiki-docker` repo's perspective, so Grep silently returns "No matches found" even when the pattern clearly exists in a plugin file (confirmed: a `var(--`/`rgba(` scan missed annotations/style.css entirely, matching only the memory markdown files).

**Why:** Missing matches here aren't "no matches" — they're invisible files. Trusting an empty Grep result over plugin code leads to wrong conclusions about which plugins need a patch.

**How to apply:** When scanning across plugin source, don't rely on the Grep tool. Use PowerShell `Select-String`, which ignores gitignore:
`Get-ChildItem -Path '...\plugins' -Recurse -Include *.css,*.less | Where-Object { $_.FullName -notmatch 'node_modules' } | Select-String -Pattern '...'`
Glob (the tool) also bypasses gitignore for locating files. See [[feedback-tool-usage]], [[css-var-rgba-lesserphp]].
