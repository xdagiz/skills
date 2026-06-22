# jj Workflows — Step-by-Step Guides

Common development workflows in Jujutsu.

---

## 1. Daily Development Workflow

### Starting a New Feature
```bash
# Create a new change (automatically becomes working copy)
jj new -m "feat: implement user authentication"

# Work on files...
# Changes auto-track in the current change (@)

# When done, commit (creates new change on top)
jj ci -m "feat: implement user authentication"

# Or, if you want to keep working on the same change:
jj desc -m "feat: implement user authentication"
```

### Making Quick Fixes
```bash
# Just edit files — they auto-track in @
jj desc -m "fix: typo in readme"

# Or commit to create a new change
jj ci -m "fix: typo in readme"
```

### Working on Multiple Features
```bash
# Create first feature
jj new -m "feat: add login page"
# ... work ...

# Create second feature (first stays where it was)
jj new -m "feat: add logout button"
# ... work ...

# Switch back to first feature
jj edit "feat: add login page"
# ... continue work ...
```

---

## 2. Amending and Editing History

### Amend the Current Change
```bash
# Just edit files — they auto-track
# Then update description if needed:
jj desc -m "feat: improved implementation"
```

### Squash Changes into Parent
```bash
# Move all changes from @ into its parent
jj squash

# Move specific files only
jj squash src/main.rs

# Interactive: select which changes to move
jj squash -i

# Squash into a specific revision
jj squash --into X
```

### Split a Change
```bash
# Split @ into two changes interactively
jj split

# The diff editor opens — select which changes go where
# Selected → original commit
# Remaining → new child commit

# Split into parallel siblings
jj split -p
```

### Edit a Specific Change
```bash
# Jump to any change as working copy
jj edit X

# Make changes (they track in X)
# Then move back:
jj edit @-
```

### Remove a Change
```bash
# Abandon X (rebase descendants to X's parent)
jj abandon X

# Or undo if you made a mistake
jj undo
```

---

## 3. Rebase Workflows

### Rebase a Branch
```bash
# Rebase everything on the branch containing @ onto main
jj rebase -b @ -d main

# Rebase a specific branch
jj rebase -b my-feature -d main
```

### Rebase Specific Commits
```bash
# Rebase only X and its descendants
jj rebase -s X -d main

# Rebase only X (not descendants)
jj rebase -r X -d main
```

### Insert Changes in History
```bash
# Insert X after Y (rebase Y's children onto X)
jj rebase -r X -A Y

# Insert X before Y (rebase Y and descendants onto X)
jj rebase -r X -B Y
```

### Reorganize Commit Graph
```bash
# Make X a child of Y
jj rebase -s X -d Y

# Move X to be a sibling of Y
jj parallelize X Y

# Simplify redundant parents
jj simplify-parents
```

---

## 4. Conflict Resolution

### Identify Conflicts
```bash
jj st                     # Shows conflicted files
jj diff                   # See conflict markers
```

### Resolve Conflicts
```bash
# Interactive resolution with merge tool
jj resolve

# Resolve specific file
jj resolve path/to/file

# Or manually edit files, then:
jj new                    # Creates new change with conflicts resolved
```

### View Conflict History
```bash
jj show -r X              # See conflicts in specific revision
jj evolog -r X            # See how conflicts evolved
```

---

## 5. Git Integration Workflows

### Clone a Repository
```bash
jj git clone https://github.com/user/repo
cd repo
```

### Initialize in Existing Git Repo
```bash
cd existing-git-repo
jj git init
```

### Daily Sync with Remote
```bash
# Fetch remote changes
jj git fetch

# See what's new
jj log -r 'main@origin'

# See your changes to push
jj log -r 'main..@'

# Push your changes
jj git push -b my-feature
```

### Create a Pull Request
```bash
# Create bookmark for your changes
jj bookmark create my-feature

# Push to remote
jj git push -b my-feature

# Create PR on GitHub/GitLab (or use their CLI)
```

### Handle Upstream Changes
```bash
# Fetch
jj git fetch

# Rebase your work onto updated main
jj rebase -b @ -d main@origin

# Push updated branch
jj git push -b my-feature
```

---

## 6. Undo and Recovery

### Undo Last Operation
```bash
jj undo                   # Undo last operation
jj undo                   # Undo the undo (go forward)
```

### Undo Specific Operation
```bash
jj op log                 # Find operation ID
jj undo -o OP_ID          # Undo specific operation
```

### Restore to Previous State
```bash
jj op log                 # Find state you want
jj op restore OP_ID       # Restore entire repo to that state
```

### View Operation History
```bash
jj op log                 # List all operations
jj op show                # Show current operation details
jj op diff -f OP1 -t OP2  # Compare two operations
```

---

## 7. Workspaces for Parallel Work

### Create a Workspace
```bash
# Create workspace for running tests
jj workspace add ../repo-test

# Switch to it
cd ../repo-test

# Work here (changes track separately)
```

### Manage Workspaces
```bash
jj workspace list         # List all workspaces
jj workspace root         # Show current workspace root
```

### Remove a Workspace
```bash
cd /path/to/main/workspace
jj workspace forget test-workspace
```

---

## 8. Absorb Workflow (Clean Up Fixups)

### Absorb Changes into Ancestors
```bash
# Move fixup changes into appropriate ancestor commits
jj absorb

# Absorb from specific revision
jj absorb -r X

# Only absorb into mutable changes
jj absorb -t mutable()
```

### Review Absorb Results
```bash
jj op show -p             # See what absorb did
```

---

## 9. Collaboration Workflows

### Working with Code Review (Gerrit)
```bash
# Push to Gerrit
jj gerrit push

# List reviews
jj gerrit list
```

### Working with GitHub/GitLab
```bash
# Standard workflow
jj git fetch
jj rebase -b @ -d main@origin
jj git push -b my-feature
```

### Sharing Changes
```bash
# Export a patch
jj diff -r X > changes.patch

# Apply on another machine
jj new -m "imported changes"
# ... paste changes ...
```

---

## 10. Advanced Workflows

### Bisect to Find Bugs
```bash
jj bisect start
jj bisect bad @           # Current commit is bad
jj bisect good main       # main is good
# jj runs tests and narrows down
```

### Sparse Checkout (Large Repos)
```bash
jj sparse set src/ tests/  # Only track src/ and tests/
jj sparse add docs/        # Add docs/
jj sparse remove tests/    # Remove tests/
jj sparse list             # Show current patterns
```

### Signing Commits
```bash
jj sign X                 # Sign a commit
jj unsign X               # Remove signature
```

### View Change Evolution
```bash
jj evolog -r X            # See how X changed over time
jj evolog -r X --no-graph # Flat list of changes
```

---

## 11. Migration from Git

### Daily Commands Comparison
| Git | jj |
|-----|-----|
| `git status` | `jj st` |
| `git log` | `jj log` |
| `git diff` | `jj diff` |
| `git add` | (not needed) |
| `git commit -m "msg"` | `jj ci -m "msg"` |
| `git branch name` | `jj b c name` |
| `git checkout name` | `jj edit name` |
| `git rebase -i` | `jj squash` / `jj split` |
| `git stash` | `jj new` |
| `git stash pop` | `jj edit @-` |
| `git reset --hard` | `jj restore -r X -- .` |
| `git revert X` | `jj revert -r X -d @` |

---

## Tips

1. **Trust auto-tracking.** Files always track in `@`. No staging needed.
2. **Use `jj new` liberally.** It's how you "pause" work and start fresh.
3. **`jj undo` is safe.** Every operation is reversible.
4. **Bookmarks are cheap.** Create them when you need to share; forget them when done.
5. **Workspaces for isolation.** Use them for long-running tasks that shouldn't block your main work.
