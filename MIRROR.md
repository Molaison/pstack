# Syncing this mirror

This repo mirrors [`cursor/plugins/pstack`](https://github.com/cursor/plugins/tree/main/pstack) and adds its own edits: the top of `README.md` and harness-neutral rewrites in some skills.

Two branches keep those edits safe:

- `upstream` holds Cursor's files exactly, with no local edits.
- `main` is `upstream` plus this mirror's edits.

Never copy Cursor's files onto `main` directly. That erases the edits.

## Remotes

- `origin` is this account's fork, `github.com/Molaison/pstack`. Both branches are pushed there.
- `source` is `github.com/backnotprop/pstack`, the mirror this account follows for reference. It is read-only here.
- `upstream` in the branch list is a branch, not a remote. `cursor/plugins` is cloned to `/tmp` per sync and never added as a remote.

## Steps

1. Copy Cursor's latest `pstack/` folder onto the `upstream` branch, in a temporary worktree rather than this checkout (see [Files that stay local](#files-that-stay-local)):

   ```bash
   git clone --depth 1 --filter=blob:none --sparse https://github.com/cursor/plugins.git /tmp/cursor-plugins
   git -C /tmp/cursor-plugins sparse-checkout set pstack
   git switch upstream
   rsync -a --delete --exclude .git --exclude MIRROR.md /tmp/cursor-plugins/pstack/ ./
   git add -A
   git commit -m "upstream: cursor/plugins/pstack @ $(git -C /tmp/cursor-plugins rev-parse --short HEAD)"
   ```

2. Merge it into `main`. Git applies only what changed upstream and keeps the mirror's edits:

   ```bash
   git switch main
   git merge upstream
   ```

   A conflict means Cursor changed a line this mirror also changed. Keep Cursor's new meaning and reapply the harness-neutral wording.

3. Refresh the bundled Comment Sicko prompt, then check that no new Cursor-only instructions arrived:

   ```bash
   cp agents/comment-sicko.md skills/no-comments/references/comment-sicko.md
   git diff upstream@{1} upstream -- skills | grep -nE '\.cursor/|agent-transcripts|cursor-team-kit|create-skill|Task'
   ```

   Rewrite any new hits the same way as the existing edits. The Harness section in `skills/poteto-mode/SKILL.md` lists the mappings.

4. Push both branches:

   ```bash
   git push origin main upstream
   ```

## Files that stay local

Every `skills/*/agents/openai.yaml` in this checkout is a Codex-side adaptation (`policy.allow_implicit_invocation: false`), and stays untracked on purpose. Two consequences:

- Do not run `git add -A` with `upstream` checked out here: the snapshot would swallow all 57 of them. Build the snapshot in a temporary worktree instead — `git worktree add /tmp/pstack-upstream -b upstream-new origin/upstream`, do the `rsync` and commit there, then `git branch -f upstream upstream-new`.
- Stage explicit paths on `main`. A bare `git add -A` commits the same 57 files into the mirror.
