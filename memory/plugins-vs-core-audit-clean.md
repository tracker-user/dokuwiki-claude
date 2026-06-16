---
name: plugins-vs-core-audit-clean
description: "All custom/modernized plugins re-audited against the core-audit lenses on 2026-06-04 — clean, no new findings"
metadata: 
  node_type: memory
  type: project
---

On 2026-06-04 all 18 custom/modernized plugins ([[plugin-inventory]]) were re-swept against
the four lenses distilled from a core DokuWiki security audit. **Result: no new findings.**
Don't redo this sweep unless plugin code changes.

Lenses applied and outcome:
- **F4/F10 normalize→check→act on the *same* value** — annotations cleans `$id` before ACL+write;
  pagebuttons uses core-cleaned `$ID`; move is the near-miss: `op.php::checkPage` does
  case-sensitive `auth_quickaclcheck($dst)` while `wikiFN($dst)` internally `cleanID`s (a real F4
  shape) **but** `move/helper/plan.php::addMove` cleanIDs src+dst before `op.php` ever sees them,
  so admin's raw `$INPUT->str('dst')` is neutralized upstream of the sink. Safe.
- **F5 `$INPUT->str()` in arithmetic → PHP 8.3 TypeError** — zero instances across all plugins.
- **F1 tokenless-but-ACL'd state-change** — every state-changing AJAX/action carries
  `checkSecurityToken()`; read-only paths (annotations `load`, prosemirror resolve/switch) correctly omit it.
- **F8/F9 manager-accessible admin (`forAdminOnly()=false`)** — only move (admin+tree) and
  translation. move enforces per-page ACL in `op.php` on both endpoints (not F9); translation admin
  is a read-only report (no state change, no reflected XSS).

Takeaway: the per-plugin reviews had already closed these classes; the systemic lens just confirmed
consistency. See [[syntax-attr-injection-xss]], [[csrf-vs-acl-distinction]], [[jsinfo-embed-xss]],
[[js-dom-xss-text-not-html]].
