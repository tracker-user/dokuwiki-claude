---
name: csrf-vs-acl-distinction
description: "A CSRF token proves session origin, not authorization — state-changing actions need checkSecurityToken AND an ACL check"
metadata:
  node_type: memory
  type: reference
---

A CSRF token proves a request came from the user's own session — it does **not** prove
the user is authorized for the action. State-changing actions need **both**
`checkSecurityToken()` AND a permission check (`auth_quickaclcheck($id)` vs the required
level: `AUTH_EDIT` / `AUTH_DELETE` / `AUTH_ADMIN`).

Found in our reviews:
- **pagebuttons** (20ad6b2): `?do=deletepagebutton` checked only the CSRF token + page
  existence → anyone with a valid session could delete pages they had no edit rights
  for. Fix: require `auth_quickaclcheck($ID) >= AUTH_EDIT` (and `!hideDelete`) before
  deleting.
- **move** (2c78d92 / e5e0783): page-rename AJAX and plan-progress AJAX were missing
  `checkSecurityToken()` entirely (the media-manager path had it — asymmetry left by a
  prior pass). Also replaced `$_SERVER['REMOTE_USER']` with `$INPUT->server->str()` and
  null-guarded `$USERINFO['grps']` with `?? []`.

Related leak to flag in the same pass: **prosemirror** (02085f0) exposed
`getTraceAsString()`/`getFile()` server paths to all users when Sentry was absent — gate
stack traces behind `allowdebug` or admin role.

When reviewing any plugin AJAX handler, `action=`/`do=` branch, or admin POST, verify
**both** gates are present — token AND permission — and that the user id comes from
`$INPUT->server->str('REMOTE_USER')`, not `$_SERVER`.

**Refinement — the token only matters for a logged-in user.** `checkSecurityToken()` is an
**anon no-op** (`if (!REMOTE_USER) return true`) and `getSecurityToken()` returns `''` for
anonymous, so a CSRF token only exists for, and is only checked for, an authenticated
session. A purely anonymous state change (e.g. open self-registration) borrows no ambient
authority and needs no token — don't flag it as a CSRF gap. The lens: *a CSRF token is only
meaningful where the action exercises the victim's logged-in session authority* — which is
also why **login-CSRF** (a login form with no token, logging the victim into the attacker's
account) is real and is **not** SameSite-mitigated: it rides the response `Set-Cookie`, not
the victim's existing cookies. See [[syntax-attr-injection-xss]].
