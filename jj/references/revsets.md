# jj Revsets — Deep Dive

Revsets are jj's functional language for selecting sets of revisions. They are used in most `jj` commands that accept `-r`, `--from`, `--to`, etc.

---

## Symbols

| Symbol | Meaning |
|--------|---------|
| `@` | Working-copy commit in current workspace |
| `workspace@` | Working-copy commit in named workspace |
| `name@remote` | Remote-tracking bookmark |
| `bookmark-name` | Bookmark named "bookmark-name" |
| `abc123` | Change ID (unique prefix) |
| `commit:abc123` | Commit ID (explicit) |
| `change:abc123` | Change ID (explicit) |
| `root()` | Root commit (parent of everything) |
| `none()` | Empty set (no revisions) |

### Resolution Priority

When you type a bare symbol like `main`, jj resolves it in this order:
1. Tag name
2. Bookmark name
3. Git ref
4. Commit ID or change ID

To force resolution, use explicit prefixes: `bookmark:main`, `tag:v1.0`, `commit_id:abc123`.

### String Literals

Use quotes to prevent interpretation:
```bash
jj log -r '"x-"'          # Literal symbol "x-", not parents of x
jj log -r '"main"'        # Literal "main", not the bookmark
```

---

## Operators

Operators are listed from strongest to weakest binding:

### 1. `f(x)` — Function Call
```bash
ancestors(X)
heads(X | Y)
```

### 2. `-x` — Negation
```bash
~X                        # All revisions not in X
```

### 3. `x::y` — Range (ancestors of x, descendants of y)
```bash
A::B                      # Ancestors of A that are also descendants of B
```

### 4. `x..y` — DAG Range
```bash
main..                    # All descendants of main (not including main)
A..B                      # Descendants of A that are also ancestors of B
```

### 5. `x:y` — Visible Range
```bash
A:B                       # Visible revisions from A to B
```

### 6. `x | y` — Union
```bash
A | B                     # Revisions in A or B
@ | @-                    # Working copy and its parent
```

### 7. `x & y` — Intersection
```bash
A & B                     # Revisions in both A and B
mutable() & ancestors(@)  # Mutable ancestors of working copy
```

### 8. `x ~ y` — Difference
```bash
A ~ B                     # Revisions in A but not B
all() ~ immutable()       # All mutable revisions
```

---

## Functions

### Set Operations

| Function | Description |
|----------|-------------|
| `none()` | Empty set |
| `all()` | All visible revisions |
| `mutable()` | All mutable revisions |
| `immutable()` | All immutable revisions |

### Ancestry

| Function | Description |
|----------|-------------|
| `ancestors(x)` | All ancestors of x (equivalent to `x::`) |
| `descendants(x)` | All descendants of x (equivalent to `x..`) |
| `parents(x)` | Immediate parents of x |
| `children(x)` | Immediate children of x |

### Heads and Roots

| Function | Description |
|----------|-------------|
| `heads(x)` | Revisions in x with no children in x |
| `roots(x)` | Revisions in x with no parents in x |
| `visible_heads()` | All visible heads (tips of branches) |

### Working Copy

| Function | Description |
|----------|-------------|
| `working_copies()` | All working-copy commits |
| `workspace_name(x)` | Workspace name for working-copy commit x |

### Bookmarks and Tags

| Function | Description |
|----------|-------------|
| `bookmarks()` | All bookmark targets |
| `tags()` | All tag targets |

### File Sets

| Function | Description |
|----------|-------------|
| `files(x)` | Files changed in revisions x |
| `file(x)` | Files matching pattern x |

### Author/Committer

| Function | Description |
|----------|-------------|
| `author(x)` | Revisions with author matching pattern x |
| `committer(x)` | Revisions with committer matching pattern x |
| `description(x)` | Revisions with description matching pattern x |

### Time

| Function | Description |
|----------|-------------|
| `timestamps(x)` | Revisions with timestamp matching x |
| `latest(x, n)` | n most recent revisions in x |

### Operation Log

| Function | Description |
|----------|-------------|
| `at_operation(op, x)` | Revisions visible at operation op |
| `operation_id()` | Current operation ID |

---

## Common Patterns

### Working Copy Context
```bash
@                         # Just the working copy
@-                        # Parent of working copy
@+                        # Child of working copy
@--                       # Grandparent
@ | @-                    # Working copy and parent
@::                       # All ancestors of working copy
```

### Branch/Bookmark Work
```bash
main                      # Bookmark named main
main..                    # Everything above main
..main                    # Everything below main (ancestors)
main::                    # Ancestors of main
bookmarks()               # All bookmark targets
```

### Filtering
```bash
mutable()                 # All editable changes
immutable()               # All protected history
all() ~ immutable()       # All mutable changes
heads(mutable())          # Tips of mutable branches
roots(immutable())        # Base of immutable history
```

### Searches
```bash
description("fix")        # Changes with "fix" in description
author("alice")           # Changes by alice
files("src/*.rs")         # Changes touching src/*.rs
```

### Complex Expressions
```bash
@ & mutable()             # Is @ mutable?
ancestors(@) & heads(all())  # Oldest ancestor that's a head
(all() ~ immutable()) :: @   # All mutable ancestors of @
```

---

## Revset Examples

### Show only mutable changes
```bash
jj log -r 'mutable()'
```

### Show changes on a branch since it diverged from main
```bash
jj log -r 'main..my-feature'
```

### Show the 5 most recent changes
```bash
jj log -r 'latest(all(), 5)'
```

### Show all changes touching Rust files
```bash
jj log -r 'files("*.rs")'
```

### Show changes by a specific author
```bash
jj log -r 'author("alice@example.com")'
```

### Show changes with "WIP" in description
```bash
jj log -r 'description("WIP")'
```

### Show working copy and all its ancestors
```bash
jj log -r '@::'
```

### Show all branches (heads of mutable history)
```bash
jj log -r 'heads(mutable())'
```

### Show merge commits
```bash
jj log -r 'children(parents(@)) & parents(children(@))'
```

---

## Tips

1. **Use quotes in shell:** Complex revsets need quoting: `jj log -r 'all() ~ immutable()'`
2. **Short IDs work:** `jj log -r abc123` works if prefix is unique
3. **Bookmarks are symbols:** `jj log -r main` shows the bookmark target
4. **Combine freely:** Operators and functions compose: `ancestors(@) & mutable()`
5. **Test with `jj log -r`:** Preview what a revset selects before using it in destructive commands

---

## Reference

Full documentation: `jj help -k revsets` or https://docs.jj-vcs.dev/latest/revsets/
