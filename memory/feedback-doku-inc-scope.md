---
name: feedback-doku-inc-scope
description: "DOKU_INC guards go on plugin logic files, not conf/ or lang/ — third-party plugin convention, not core's"
metadata: 
  node_type: memory
  type: feedback
---

Put the DOKU_INC guard on plugin **logic** files (`action.php`, `helper.php`, `admin.php`, `syntax.php`, etc. — class/logic files). Do **not** add it to `conf/default.php`, `conf/metadata.php`, or `lang/**/*.php` (pure data files — they only assign array values, produce no output, execute no logic).

**Why keep the guard at all:** It's the third-party **plugin** convention (the official plugin skeleton/wizard emits it), and it's a harmless one-line defensive measure. Note: DokuWiki **core bundled** plugins (`acl`, `authplain`, `authad`) actually omit it even on logic files, because those are class-only and fatal harmlessly on direct HTTP access. So don't justify the rule as "matches core" — core dropped it; the plugin ecosystem (move, include, edittable, wrap, and our custom plugins) kept it. We follow the plugin convention, not core's. Don't mass-remove existing guards to match core — that's pure churn with no benefit.

**Why skip conf/lang:** Caught when guards were added to pagebuttons lang/conf files during review. Core omits the guard on these everywhere — checked `inc/lang/en/lang.php`, `lib/plugins/acl/lang/en/lang.php`, `lib/plugins/authad/conf/default.php`.

**How to apply:** Guard on logic files, skip `conf/` and `lang/`. CLAUDE.md checklist (item #2, Mandatory bullet) already reflects this and stays as-is.
