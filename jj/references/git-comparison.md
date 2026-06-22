# jj vs Git — Command Comparison

Side-by-side comparison of Jujutsu and Git commands.

---

## Core Concepts

| Concept | Git | jj |
|---------|-----|-----|
| Commits | snapshots | changes (rebased automatically) |
| Staging area | `git add` | (none — auto-tracked) |
| Branches | `git branch` | bookmarks (`jj bookmark`) |
| Stash | `git stash` | `jj new` (just start fresh) |
| Detached HEAD | `git checkout --detach` | `jj edit` (always valid) |
| Undo | `git reflog` + `git reset` | `jj undo` |
| History rewrite | `git rebase -i` | `jj squash` / `jj split` / `jj rebase` |

---

## Status & Inspection

| Task | Git | jj |
|------|-----|-----|
| Show status | `git status` | `jj st` |
| Show log | `git log` | `jj log` |
| Show log (all branches) | `git log --all` | `jj log -r 'all()'` |
| Show diff | `git diff` | `jj diff` |
| Show diff (staged) | `git diff --cached` | `jj diff` (auto-tracked) |
| Show specific commit | `git show X` | `jj show -r X` |
| Show file content | `git show X:file` | `jj file show -r X file` |
| List files | `git ls-files` | `jj file list` |
| Show operation log | `git reflog` | `jj op log` |

---

## Creating Changes

| Task | Git | jj |
|------|-----|-----|
| Stage file | `git add file` | (auto-tracked) |
| Stage all | `git add .` | (auto-tracked) |
| Commit | `git commit -m "msg"` | `jj ci -m "msg"` |
| Amend commit | `git commit --amend` | `jj desc -m "msg"` |
| Create empty commit | `git commit --allow-empty` | `jj new` |
| Create branch | `git branch name` | `jj b c name` |

---

## Navigating History

| Task | Git | jj |
|------|-----|-----|
| Checkout commit | `git checkout X` | `jj edit X` |
| Checkout branch | `git checkout branch` | `jj edit branch` |
| Create + checkout | `git checkout -b name` | `jj b c name && jj edit name` |
| Show parent | `git show HEAD~1` | `jj show -r @-` |
| Show ancestor | `git show HEAD~5` | `jj show -r 'ancestors(@, 5) | heads(...)'` |
| Log since branch | `git log main..` | `jj log -r 'main..'` |
| Log file changes | `git log -- path` | `jj log path` |

---

## Rewriting History

| Task | Git | jj |
|------|-----|-----|
| Interactive rebase | `git rebase -i X` | `jj squash` / `jj split` / `jj rebase` |
| Squash commits | `git rebase -i` (squash) | `jj squash` |
| Split commit | (no direct equivalent) | `jj split` |
| Rebase branch | `git rebase X` | `jj rebase -b @ -d X` |
| Rebase onto | `git rebase --onto X Y Z` | `jj rebase -s Z -d X` |
| Cherry-pick | `git cherry-pick X` | `jj duplicate X -d @` |
| Revert commit | `git revert X` | `jj revert -r X -d @` |
| Reset to commit | `git reset --hard X` | `jj restore -r X -- .` |
| Reset file | `git checkout -- file` | `jj restore -r @ -- file` |
| Abandon commit | (no equivalent) | `jj abandon X` |
| Move commit | `git rebase` | `jj rebase -r X -d Y` |
| Make siblings | (no equivalent) | `jj parallelize X Y` |

---

## Branches vs Bookmarks

| Task | Git | jj |
|------|-----|-----|
| List branches | `git branch` | `jj b l` |
| Create branch | `git branch name` | `jj b c name` |
| Delete branch | `git branch -d name` | `jj b d name` |
| Force delete | `git branch -D name` | `jj b d name` (always works) |
| Rename branch | `git branch -m old new` | `jj b r old new` |
| Move branch | `git branch -f name X` | `jj b m name -t X` |
| Checkout branch | `git checkout name` | `jj edit name` |
| Track remote | `git checkout -b name origin/name` | `jj b t name@origin` |

**Key difference:** Bookmarks don't move automatically. You must explicitly move them (`jj b m`).

---

## Remote Operations

| Task | Git | jj |
|------|-----|-----|
| Clone | `git clone URL` | `jj git clone URL` |
| Fetch all | `git fetch --all` | `jj git fetch` |
| Fetch specific | `git fetch origin` | `jj git fetch --remote origin` |
| Push | `git push` | `jj git push` |
| Push branch | `git push origin branch` | `jj git push -b branch` |
| Push new branch | `git push -u origin branch` | `jj git push -b branch` |
| Push commit | (no equivalent) | `jj git push --change X` |
| Add remote | `git remote add name url` | `jj git remote add name url` |
| List remotes | `git remote -v` | `jj git remote list` |

**Key difference:** `jj git push` uses force-with-lease semantics by default.

---

## Stashing vs jj new

| Task | Git | jj |
|------|-----|-----|
| Stash changes | `git stash` | `jj new` |
| List stashes | `git stash list` | `jj log -r 'bookmarks()'` |
| Pop stash | `git stash pop` | `jj edit @-` |
| Apply stash | `git stash apply` | `jj edit @-` (same as pop) |
| Drop stash | `git stash drop` | `jj abandon @` |
| Stash message | `git stash push -m "msg"` | `jj new -m "msg"` |

**Key difference:** `jj new` creates a new change (like a stash) and you can return with `jj edit`.

---

## Diffing

| Task | Git | jj |
|------|-----|-----|
| Working diff | `git diff` | `jj diff` |
| Commit diff | `git diff X~1 X` | `jj diff -r X` |
| Between commits | `git diff X Y` | `jj diff -f X -t Y` |
| Diff stat | `git diff --stat` | `jj diff --stat` |
| Diff name only | `git diff --name-only` | `jj diff --name-only` |
| Diff specific file | `git diff -- file` | `jj diff file` |
| Compare branches | `git diff main feature` | `jj diff -f main -t feature` |

---

## Undo & Recovery

| Task | Git | jj |
|------|-----|-----|
| Undo last commit | `git reset --soft HEAD~1` | `jj undo` |
| Undo last change | `git checkout -- file` | `jj undo` |
| Undo N operations | `git reset --hard HEAD~N` | `jj undo` (repeat N times) |
| View history | `git reflog` | `jj op log` |
| Restore to operation | `git reset --hard HEAD@{5}` | `jj op restore OP_ID` |
| Redo | (manual reflog) | `jj redo` |

**Key difference:** jj's undo/redo is per-operation and always safe.

---

## Workspaces

| Task | Git | jj |
|------|-----|-----|
| Worktree add | `git worktree add ../path branch` | `jj workspace add ../path` |
| List worktrees | `git worktree list` | `jj workspace list` |
| Remove worktree | `git worktree remove path` | `jj workspace forget name` |

---

## Tags

| Task | Git | jj |
|------|-----|-----|
| List tags | `git tag` | `jj tag list` |
| Create tag | `git tag name` | (use bookmarks) |
| Delete tag | `git tag -d name` | (use bookmarks) |

**Note:** jj uses bookmarks instead of tags for most workflows.

---

## Configuration

| Task | Git | jj |
|------|-----|-----|
| Show config | `git config --list` | `jj config list` |
| Get config | `git config key` | `jj config get key` |
| Set config | `git config key value` | `jj config set key value` |

---

## File Operations

| Task | Git | jj |
|------|-----|-----|
| Show file at commit | `git show X:file` | `jj file show -r X file` |
| List tracked files | `git ls-files` | `jj file list` |
| Show file diff | `git diff -- file` | `jj diff file` |
| Blame | `git blame file` | (not yet implemented) |

---

## Utility

| Task | Git | jj |
|------|-----|-----|
| Show version | `git --version` | `jj version` |
| Show repo root | `git rev-parse --show-toplevel` | `jj root` |
| Generate completions | `git completion --bash` | `jj util completion --bash` |
| Bisect | `git bisect` | `jj bisect` |
| Resolve conflicts | (manual) | `jj resolve` |

---

## Key Mental Model Shifts

1. **No staging.** Everything auto-tracks. Just edit and describe.
2. **No stash.** `jj new` pauses work; `jj edit @-` resumes.
3. **No detached HEAD.** `jj edit` always creates a valid state.
4. **Undo is built-in.** `jj undo` reverses any operation safely.
5. **Bookmarks don't auto-move.** Explicit `jj bookmark move` required.
6. **History is immutable.** Use `--ignore-immutable` to rewrite.
7. **Conflicts are tracked.** jj keeps conflict state; resolve when ready.
8. **Operations are first-class.** Every action is logged and reversible.
