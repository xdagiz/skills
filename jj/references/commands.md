# jj Command Reference

Complete reference for all Jujutsu commands.

---

## Working Copy & Changes

### `jj new [REVSETS]`
Create a new, empty change and edit it in the working copy.

```bash
jj new                    # New change parented at @
jj new main               # New change with main as parent (merge)
jj new A B                # New change with two parents (merge commit)
jj new --no-edit          # Create but don't switch to it
jj new -m "message"       # Create with description
jj new -A X               # Insert after X
jj new -B X               # Insert before X's children
```

### `jj commit [FILESETS]`
Describe current change and create a new empty change on top.

```bash
jj ci                     # Commit all changes with editor
jj ci -m "msg"            # Commit with message
jj ci src/                # Commit only src/ changes
jj ci -i                  # Interactive: select which changes to keep
```

**Differences from `jj split`:**
- Not interactive by default (selects all changes)
- Always acts on working-copy commit (`@`)
- Does not move bookmarks forward
- Has no `-r` option

### `jj describe [REVSETS]` (alias: `desc`)
Update the description of a change without modifying content.

```bash
jj desc                   # Edit @ description in editor
jj desc -m "new message"  # Set description directly
jj desc -r X              # Describe specific revision
jj desc --stdin           # Read description from stdin
```

### `jj edit [REVSET]`
Set the specified revision as the working-copy revision.

```bash
jj edit X                 # Jump to change X as working copy
```

**Important:** This changes what `@` points to. Descendants auto-rebase.

### `jj abandon [REVSETS]`
Abandon a revision, rebasing descendants onto its parent(s).

```bash
jj abandon X              # Abandon X
jj abandon X Y            # Abandon multiple
jj abandon --retain-bookmarks  # Move bookmarks to parents
```

### `jj restore`
Restore paths from another revision.

```bash
jj restore -r X -- .      # Restore all files from X
jj restore -r X -- src/   # Restore only src/ from X
```

---

## Rewriting History

### `jj squash [FILESETS]` (alias: `sq`)
Move changes from a revision into another revision.

```bash
jj squash                  # Squash @ into its parent
jj squash -r X             # Squash X into its parent
jj squash --into @--       # Squash @ into grandparent
jj squash -i               # Interactive: select changes to squash
jj squash src/             # Squash only src/ changes
```

**Options:**
- `-f/--from` — source revision(s) (default: `@`)
- `-t/--into` — destination revision (default: `@`'s parent)
- `-r/--revision` — shorthand for squashing into parent
- `-i/--interactive` — select changes interactively
- `-o/--onto`, `-A/--insert-after`, `-B/--insert-before` — create new commit

### `jj split [FILESETS]`
Split a revision in two using a diff editor.

```bash
jj split                   # Split @ interactively
jj split -r X              # Split specific revision
jj split -p                 # Parallel: make siblings instead of parent-child
jj split -o main            # Split and place selected changes on main
```

**Behavior:**
- Opens diff editor on changes in the revision
- Selected changes stay in original commit
- Remaining changes go into new child (or sibling with `-p`)
- With `-o/-A/-B`: selected extracted to new commit at position

### `jj rebase`
Move revisions to different parent(s).

```bash
# Which revisions to rebase:
jj rebase -s X -d Y       # Rebase X and descendants
jj rebase -b X -d Y       # Rebase branch containing X
jj rebase -r X -d Y       # Rebase only X (not descendants)

# Where to rebase:
jj rebase -b X -d Y       # Onto Y
jj rebase -b X -A Y       # After Y
jj rebase -b X -B Y       # Before Y's children
```

**Rebase modes:**
- `-s/--source` — rebase revision + descendants
- `-b/--branch` — rebase whole branch (ancestors not shared with destination)
- `-r/--revisions` — rebase only specified revisions

**Destination modes:**
- `-d/--destination` — rebase onto target
- `-A/--insert-after` — insert after target, rebase target's descendants
- `-B/--insert-before` — insert before target's children

### `jj duplicate [REVSETS]`
Create new changes with same content as existing ones.

```bash
jj duplicate X             # Duplicate X onto its parents
jj duplicate X -o Y        # Duplicate X onto Y
```

### `jj revert [REVSETS]`
Apply the reverse of given revision(s).

```bash
jj revert -r X -d main    # Revert X onto main
```

### `jj absorb [FILESETS]`
Move changes from a revision into the stack of mutable revisions.

```bash
jj absorb                  # Absorb @ changes into ancestors
jj absorb -f X             # Absorb from X
jj absorb -t mutable()     # Only absorb into mutable changes
```

**Use case:** Clean up fixup commits by moving their changes into appropriate ancestors.

### `jj parallelize [REVSETS]`
Parallelize revisions by making them siblings.

```bash
jj parallelize X Y         # Make X and Y siblings
```

### `jj simplify-parents [REVSETS]`
Simplify parent edges for specified revision(s).

```bash
jj simplify-parents        # Simplify @'s parents
```

---

## Viewing History

### `jj log [FILESETS]`
Show revision history with graph.

```bash
jj log                     # Default view (mutable changes + context)
jj log -r 'all()'          # Show everything including immutable
jj log -r 'main..'         # Changes on main and above
jj log -r '@ | @-'         # Current and parent
jj log -n 10               # Limit to 10 revisions
jj log --reversed          # Older first
jj log -T 'commit_id.short() ++ " " ++ description.first_line()'  # Custom template
jj log src/                # Show revisions touching src/
```

**Symbols in graph:**
- `@` — working copy
- `◆` — immutable revision
- `○` — mutable revision

### `jj show [REVSET]`
Show commit description and changes in a revision.

```bash
jj show                    # Show @ with full diff
jj show -r X               # Show specific revision
jj show -s                 # Summary only (modified/added/deleted)
jj show --stat             # Histogram of changes
jj show --name-only        # Just file names
```

### `jj diff [FILESETS]`
Compare file contents between two revisions.

```bash
jj diff                    # Diff @ vs its parent
jj diff -r X               # Diff X vs its parent
jj diff -f main -t @       # Diff from main to working copy
jj diff src/               # Diff only src/ paths
jj diff --stat             # Histogram
```

### `jj evolog [REVSETS]` (alias: `evolution-log`)
Show how a change has evolved over time.

```bash
jj evolog                  # Show evolution of @
jj evolog -r X             # Show evolution of X
```

### `jj interdiff`
Show differences between the diffs of two revisions.

```bash
jj interdiff -r X -r Y    # Compare what changed between X and Y
```

---

## Bookmarks (Git Branches)

### `jj bookmark` (alias: `b`)
Manage bookmarks.

```bash
jj bookmark list                     # List all bookmarks
jj bookmark list -r 'all()'          # Include remote bookmarks
jj bookmark create NAME                # Create at @
jj bookmark create NAME -r X           # Create at X
jj bookmark set NAME                # Set/create at @
jj bookmark set NAME -r X           # Set/create at X
jj bookmark move NAME -t X           # Move bookmark to X
jj bookmark delete NAME                # Delete (propagates on push)
jj bookmark forget NAME                # Forget (no propagation)
jj bookmark rename OLD NEW             # Rename bookmark
jj bookmark track NAME@REMOTE         # Track remote bookmark
jj bookmark untrack NAME@REMOTE   # Stop tracking
```

---

## Git Integration

### `jj git clone URL [PATH]`
Clone a Git repository.

```bash
jj git clone https://github.com/user/repo
jj git clone URL --colocated  # Colocated mode
```

### `jj git init`
Create a new Git-backed jj repository.

```bash
jj git init                # In current directory
jj git init --colocated    # Colocated with Git
```

### `jj git fetch`
Fetch from Git remotes.

```bash
jj git fetch               # Fetch all remotes
jj git fetch --remote origin  # Fetch specific remote
```

### `jj git push`
Push to Git remote.

```bash
jj git push                # Push tracked bookmarks
jj git push -b NAME        # Push specific bookmark
jj git push --change X     # Push change X (creates bookmark)
jj git push --all          # Push all bookmarks
jj git push --tracked      # Push all tracked bookmarks
```

**Safety:** Uses force-with-lease semantics. Remote only updates if state matches last fetch.

### `jj git export`
Update underlying Git repo with jj changes.

```bash
jj git export              # Sync Git refs to jj state
```

### `jj git import`
Update jj repo with Git changes.

```bash
jj git import              # Import Git refs into jj
```

### `jj git remote`
Manage Git remotes.

```bash
jj git remote list         # List remotes
jj git remote add NAME URL # Add remote
jj git remote remove NAME  # Remove remote
```

### `jj git colocation`
Manage colocation with Git.

```bash
jj git colocation enable   # Enable colocated mode
jj git colocation disable  # Disable colocated mode
```

---

## Operation Log

### `jj operation` (alias: `op`)
Work with the operation log (jj's undo/redo system).

```bash
jj op log                  # Show operation history
jj op show                 # Show changes in current operation
jj op show -p              # Show patch for operation
jj op restore OP           # Restore repo to earlier state
jj op revert OP            # Revert an operation
jj op diff -f OP1 -t OP2   # Compare two operations
jj op abandon              # Abandon old operations
```

**Undo/Redo:**
```bash
jj undo                    # Undo last operation
jj redo                    # Redo last undone operation
```

---

## Workspaces

### `jj workspace`
Multiple working copies on the same repository.

```bash
jj workspace list          # List all workspaces
jj workspace add PATH      # Add new workspace at PATH
jj workspace root          # Show current workspace root
jj workspace rename NAME   # Rename current workspace
jj workspace forget NAME   # Stop tracking workspace
```

**Use case:** Run slow builds/tests in one workspace while coding in another.

---

## File Operations

### `jj file`
File-level operations.

```bash
jj file list               # List all tracked files
jj file show PATH          # Show file content at @
jj file show -r X PATH     # Show file at specific revision
jj file print PATH         # Print file content
```

---

## Other Commands

### `jj resolve`
Resolve conflicted files with an external merge tool.

```bash
jj resolve                 # Resolve all conflicts
jj resolve --list          # List conflicted files
```

### `jj fix`
Update files with formatting fixes.

```bash
jj fix                     # Fix formatting in working copy
```

### `jj sign` / `jj unsign`
Cryptographic signing of revisions.

```bash
jj sign X                  # Sign revision X
jj unsign X                # Remove signature
```

### `jj bisect`
Find a bad revision by bisection.

```bash
jj bisect start
jj bisect bad X            # Mark X as bad
jj bisect good Y           # Mark Y as good
```

### `jj arrange`
Interactively arrange the commit graph.

```bash
jj arrange                 # Interactive graph arrangement
```

### `jj sparse`
Manage sparse checkout patterns.

```bash
jj sparse list             # Show sparse patterns
jj sparse set PATHS        # Set sparse patterns
jj sparse add PATHS        # Add to sparse patterns
jj sparse remove PATHS     # Remove from sparse patterns
```

### `jj tag`
Manage tags.

```bash
jj tag list                # List tags
```

### `jj root`
Show workspace root directory.

```bash
jj root
```

### `jj metaedit`
Modify metadata without changing content.

```bash
jj metaedit --author "Name <email>" -r X  # Change author
```

---

## Global Options

| Flag | Description |
|------|-------------|
| `-R, --repository <PATH>` | Path to repository |
| `--ignore-working-copy` | Skip snapshot of working copy |
| `--ignore-immutable` | Allow rewriting immutable commits |
| `--at-operation <OP>` | Load repo at specific operation |
| `--debug` | Enable debug logging |
| `--color <WHEN>` | Colorize: always, never, debug, auto |
| `--quiet` | Silence non-primary output |
| `--no-pager` | Disable pager |
| `--config <NAME=VALUE>` | Additional config (repeatable) |
| `--config-file <PATH>` | Additional config files (repeatable) |
