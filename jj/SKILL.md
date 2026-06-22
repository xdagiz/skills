---
name: jj
description: Expert guide for Jujutsu (jj) version control. Use when working in a jj repo, creating commits, managing bookmarks, rebasing, splitting changes, resolving conflicts, or translating Git commands to jj.
---

# jj (Jujutsu) Version Control

## Core Mental Model

jj tracks **changes**, not branches or snapshots. Internalize these:

- **No staging area.** File modifications auto-track in the current change (`@`). No `git add`.
- **No stash.** `jj new` pauses work; previous changes stay in their change.
- **No detached HEAD.** `jj edit` jumps to any change; descendants auto-rebase.
- **Bookmarks replace branches.** Only needed for pushing to remotes. They don't auto-move.
- **Working copy IS a change.** `@` is always the current change you're editing.
- **Immutable by default.** History is rewrite-protected. Use `--ignore-immutable` to override.
- **Every operation is undoable.** `jj undo` reverses any action. `jj redo` re-applies.

## Quick Reference

| Task | Command |
|------|---------|
| Status | `jj st` |
| Log | `jj log` |
| Diff | `jj diff` |
| Show | `jj show` |
| New change | `jj new` |
| Commit | `jj ci -m "msg"` |
| Describe | `jj desc -m "msg"` |
| Edit change | `jj edit X` |
| Abandon | `jj abandon X` |
| Squash | `jj squash` |
| Split | `jj split` |
| Rebase | `jj rebase -b X -d Y` |
| Undo | `jj undo` |

## Bookmarks (Git Branches)

| Task | Command |
|------|---------|
| List | `jj bookmark list` |
| Create | `jj bookmark create <name>` |
| Move | `jj bookmark move <name> --to <dest>` |
| Delete | `jj bookmark delete <name>` |

## Git Integration

| Task | Command |
|------|---------|
| Clone | `jj git clone URL` |
| Fetch | `jj git fetch` |
| Push | `jj git push -b BOOKMARK` |

## Revset Quick Reference

Revsets select revisions. Key symbols:

- `@` — working copy
- `X@remote` — remote bookmark
- `bookmark-name` — bookmark target
- `root()` — root commit

Operators: `::` (ancestors), `..` (descendants), `|` (union), `&` (intersection), `~` (negation).

Common patterns: `@ | @-` (current + parent), `mutable()` (all editable), `all() ~ immutable()` (all mutable).

## References

Load these with `read` when you need deeper detail:

- [commands.md](references/commands.md) — Full command reference with all flags and examples
- [revsets.md](references/revsets.md) — Revset language: operators, functions, patterns
- [workflows.md](references/workflows.md) — Step-by-step guides for common workflows
- [git-comparison.md](references/git-comparison.md) — Git ↔ jj command mapping (load when user mentions Git concepts)

**When to load:**
- User asks about a specific command → `commands.md`
- User needs complex revision selection → `revsets.md`
- User is migrating from Git or says "how do I do X in jj" → `git-comparison.md`
- User needs a multi-step workflow (rebase, conflict resolution, etc.) → `workflows.md`
