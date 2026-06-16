---
name: environment-paths
description: Local paths and Docker setup for DokuWiki development environment
metadata: 
  node_type: memory
  type: reference
---

- **DokuWiki source (read-only):** `F:\Projects\dokuwiki-stable` — Librarian 2025-05-14b git repo
- **Docker project:** `F:\Projects\dokuwiki-docker` — docker-compose, Dockerfile, storage volume
- **Plugins directory:** `F:\Projects\dokuwiki-docker\storage\lib\plugins\` — where all custom/installed plugins live
- **Wiki config:** `F:\Projects\dokuwiki-docker\storage\conf\local.php`
- **Wiki data:** `F:\Projects\dokuwiki-docker\storage\data\`
- **Container:** `dokuwiki-docker-dokuwiki-1`, image `dokuwiki/dokuwiki:stable`, port 80→8080
- **In-container paths:** DokuWiki core/install is `/var/www/html` (e.g. `/var/www/html/inc/init.php`, `/var/www/html/doku.php`). Host `storage/` is bind-mounted at `/storage`, so `/storage/lib/plugins/<name>/...` works for lint — but core files are NOT under `/storage` (`/storage/inc`, `/dokuwiki/` don't exist).
- **docker exec via PowerShell:** the Bash tool is Git Bash, which mangles `/storage...` → `C:/Program Files/Git/storage...`; run docker / container-path commands in PowerShell. See [[feedback-tool-usage]].
- **URL:** `http://dokuwiki.local/`
- **PHP inside Docker:** 8.3, Debian 13
- **CLAUDE.md for reviews:** `F:\Projects\dokuwiki-docker\storage\lib\plugins\CLAUDE.md`
- **PHPUnit tests:** `F:\Projects\dokuwiki-stable\_test\` — `composer install` done, PHP 8.3.31 on host
- **Plugin symlinks:** all 18 custom plugins symlinked from `storage/lib/plugins/` into `dokuwiki-stable/lib/plugins/`

**How to apply:** Use these paths when the user asks to review plugins, run tests, or check DokuWiki internals. Prefer opening CC from the plugins directory for review sessions.
