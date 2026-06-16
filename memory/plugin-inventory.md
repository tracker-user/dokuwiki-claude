---
name: plugin-inventory
description: "Status of all custom/modernized DokuWiki plugins — which are done, WIP, or skipped"
metadata: 
  node_type: memory
  type: project
---

Custom plugins (built from scratch with Claude):
- **sitebackup** — shipped, reviewed 2026-05-31
- **hideip** — shipped, reviewed 2026-05-31
- **lastseen** — shipped, reviewed 2026-05-31
- **usersettings** — shipped, reviewed 2026-06-01 (per-user preference toggles; other plugins integrate via [[usersettings-api]])
- **annotations** — shipped, reviewed 2026-06-02

Modernized/improved for Librarian compatibility:
- **cellbg** — done (re-reviewed 2026-05-31)
- **color** — done (re-reviewed 2026-06-01)
- **note** — done (re-reviewed 2026-06-01)
- **sortablejs** — done (re-reviewed 2026-06-01)
- **searchtablejs** — done (re-reviewed 2026-06-01)
- **pagebuttons** — done (re-reviewed 2026-06-01)
- **move** — done (re-reviewed 2026-06-01)
- **typography** — done (re-reviewed 2026-06-01)
- **diffpreview** — done (re-reviewed 2026-06-02)
- **prosemirror** — done (re-reviewed 2026-06-02)
- **edittable** — done (reviewed 2026-06-01)
- **wrap** — done (reviewed 2026-06-01)
- **include** — done (reviewed 2026-06-01)
- **translation** — done (reviewed 2026-06-01)
- **nosecedit** — done (reviewed 2026-06-14; rewrote metadata flag to non-persistent current meta so it self-heals, see [[dokuwiki-metadata-flag-selfheal]])

Evaluated but not adopted:
- **codemirror** — reviewed/modernized but user decided against using it
- **moaieditor** — looked at but not adopted

**Why:** Tracks which plugins need review vs. already done. Prevents re-reviewing completed work.
**How to apply:** Always proceed with the review when asked. Use the inventory to determine whether it's custom or modernized (affects plugin.info.txt conventions).
