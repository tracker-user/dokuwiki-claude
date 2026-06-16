---
name: feedback-plugin-git-commit
description: "When committing plugin changes, use the plugin's own git repo, not the parent plugins/ folder"
metadata: 
  node_type: memory
  type: feedback
---

Each plugin directory under `storage/lib/plugins/` has its own `.git` repository. Commit plugin file changes from inside the plugin directory (e.g. `storage/lib/plugins/edittable/`), not from the parent `storage/lib/plugins/` folder.

**Why:** The parent repo's `.gitignore` has `/*` which ignores all plugin directories. Trying to `git add edittable/` from the parent triggers a "embedded git repository" warning and stages a submodule reference rather than the files. The plugin's own repo is the correct commit target.

**How to apply:** After editing plugin files, run `git add` and `git commit` with the working directory set to `storage/lib/plugins/<name>/`. Memory updates and CLAUDE.md changes still commit from the parent `storage/lib/plugins/` repo as before.
