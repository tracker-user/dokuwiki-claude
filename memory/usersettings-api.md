---
name: usersettings-api
description: API for integrating plugins with the usersettings per-user preference system
metadata: 
  node_type: memory
  type: reference
---

Feature plugins register toggles via the `PLUGIN_USERSETTINGS_REGISTER` event (hook BEFORE in action.php). First registration wins on duplicate keys.

Registration in action.php::register():
```php
$controller->register_hook('PLUGIN_USERSETTINGS_REGISTER', 'BEFORE', $this, 'registerToggle');
```

Handler adds to `$event->data[]`:
- `plugin` (string) — plugin name
- `key` (string) — unique toggle key, `[A-Za-z0-9_]+` only
- `label` (string) — display label
- `type` — `'checkbox'` (default) or `'select'`. Any other value is coerced to `'checkbox'` (so a stale `'onoff'` still renders as a checkbox).
- `default` — default value (0/1 for checkbox; one of the option keys for select)
- `options` (for select) — `['val' => 'Label', ...]`, required and non-empty

Read preference: `$helper->getPreference('my_toggle', $user)` via `helper_plugin_usersettings`.
Storage: per-user JSON files in `meta/`.

**How to apply:** Reference when reviewing or building plugins that integrate with usersettings for per-user preferences.
