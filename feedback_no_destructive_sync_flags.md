---
name: feedback-no-destructive-sync-flags
description: "Never use destructive sync flags (rsync --delete, scp -rf onto existing trees, cp -rf into a dir, git clean, git reset --hard on someone else's branch) without explicit confirmation. Default to additive/merge operations."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 6e96ac03-ec71-4963-9caa-8cea57c680b1
---

When syncing or copying files between machines or paths, default to additive operations. Never use `--delete` or any flag that removes files on the target side without an explicit "yes, delete those" from jzellis in this turn.

**Why:** I ran `rsync -av --delete` from this host to jzenbook to share two memory files. The `--delete` flag wiped seven Zenbook-only memories that weren't on this host — `project_background_worker_todo`, `project_alpha_status`, `feedback_pm2_env`, `feedback_overflow_hidden_dropdowns`, `feedback_fediverse_compat`, `feedback_design_aesthetic`, `feedback_cors_local_dev`. ext4, no trash, no btrfs snapshots — unrecoverable. jzellis trusted the share-memories request and I added a destructive flag without thinking through the asymmetric blast radius.

**How to apply:**
- For rsync between machines or paths I don't fully understand the contents of: omit `--delete` (and `--delete-after`, `--delete-during`, `--delete-excluded`). `rsync -av source/ dest/` is additive and idempotent.
- If pruning is genuinely needed, list what would be deleted first (`rsync --dry-run --delete`), show jzellis the list, and ask before re-running for real.
- Same principle applies to `git clean -fd`, `git reset --hard`, `cp -rf` over an existing tree, `find … -delete`, `mv X Y` where Y exists, and `git checkout .`/`git restore .` on dirty trees. Each of these can erase user-side work that I haven't seen and don't know about.
- This applies even when jzellis says "do em both" or otherwise green-lights the higher-level task — the destructive subset of the implementation still needs its own check.

Lesson in one line: **the cost of one extra confirmation prompt is always smaller than the cost of one silently-deleted file.**
