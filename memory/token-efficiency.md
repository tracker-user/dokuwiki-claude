---
name: token-efficiency
description: "User wants minimal token spending — lean CLAUDE.md, on-demand source lookups, no full-tree indexing"
metadata: 
  node_type: memory
  type: feedback
---

Minimize token usage per session. Do not index the full DokuWiki source tree.

**Why:** Limited usage budget; plugin reviews are the primary task and rarely need full codebase context.
**How to apply:** Read DokuWiki source files from `F:\Projects\dokuwiki-stable` by absolute path only when needed to verify a specific API or hook. Don't glob/grep the full source tree speculatively. Keep CLAUDE.md lean (~100 lines). Use memory for reference material that's only needed occasionally. When reviewing a plugin, read just that plugin's files.
