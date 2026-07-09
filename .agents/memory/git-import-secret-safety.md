---
name: Git-based data import/restore secret safety
description: Why importing/restoring a data directory from a remote git repo needs explicit secret snapshot/restore, not just .gitignore; and why onboarding-vs-settings instances of the same import flow need separate UI state.
---

When a feature lets a user pull ("restore"/"import") a whole data directory from a remote git repo
(as opposed to just pushing/backing it up), destructive git operations used to sync the working tree
to the remote state — `git reset --hard origin/<branch>` and `git clean -fd` — can overwrite or delete
local secret files (`.env`, credential files, pid/sock files) regardless of what the *local* `.gitignore`
says, because:
- `reset --hard` will happily overwrite a tracked-in-remote path even if it's locally gitignored.
- `clean -fd` removes untracked files, and only respects ignore rules in effect *at the time it runs* —
  if the remote's `.gitignore` is missing/different, local ignored files can still be caught.
- Re-asserting the correct `.gitignore` *after* the reset/clean is too late — the damage already happened.

**Why:** A code review caught that "we exclude secrets via .gitignore" (true for the backup/push
direction) does not carry over to the import/restore direction — the threat model is inverted (untrusted
remote content flowing in, not trusted local content flowing out). A second review pass caught that the
protected-path check only matched exact/top-level names, missing nested paths like `sub/.env`.

**How to apply:** For any git-based import/restore of a directory containing secrets:
1. Before any destructive git op, reject the import outright if the remote tree contains a file whose
   *basename* (not just exact path — check nested paths too, e.g. via `PurePosixPath(f).name`) matches a
   protected name/glob.
2. Snapshot (copy) local protected files/dirs (matched by basename, recursively, skipping `.git/`) to a
   temp location before `reset --hard`/`clean -fd`, and restore them in a `finally` block so they survive
   even if the git commands fail partway.
3. Only after restore, re-assert the canonical `.gitignore` for future commits.

**Separate concern — reusing the same import flow in two UI contexts** (e.g. a first-run onboarding
wizard offering "restore from backup" vs. a settings-panel "restore" button for an already-running
instance): give each call site its own state object (token/repo/loading/error/success), not one shared
set of flags. Sharing state causes stale success/error messages and entered credentials to bleed from
one flow into the other. Also reset any onboarding-flow-only state (like a "which onboarding path did
the user pick" flag) when the app's config is reset, or a stale choice can silently skip the onboarding
gate on the next visit.
