---
name: using-git-worktrees
description: Use when starting feature work that needs isolation from the current workspace, or before executing an implementation plan in a separate git worktree
---

# Using Git Worktrees

## Overview

Git worktrees create isolated workspaces sharing the same repository, allowing work on multiple branches simultaneously without switching.

**Core principle:** Directory choice is not enough. Project-local worktrees are safe only when the path is both ignored by git and not tracked in the repository.

**Announce at start:** "I'm using the using-git-worktrees skill to set up an isolated workspace."

## Directory Selection Process

Follow this priority order:

### 1. Check Existing Directories

```bash
# Check in priority order
ls -d .worktrees 2>/dev/null     # Preferred (hidden)
ls -d worktrees 2>/dev/null      # Alternative
```

**If found:** Treat them as candidates, not automatic choices. Reuse only after safety verification. If both exist, check `.worktrees` first.

### 2. Check CLAUDE.md

```bash
grep -i "worktree" CLAUDE.md 2>/dev/null
```

**If preference specified:** Use it without asking, but still run safety verification for project-local directories.

### 3. Ask User

If no directory exists and no CLAUDE.md preference:

```
No worktree directory found. Where should I create worktrees?

1. .worktrees/ (project-local, hidden)
2. ~/.config/superpowers/worktrees/<project-name>/ (global location)

Which would you prefer?
```

## Safety Verification

### For Project-Local Directories (.worktrees or worktrees)

**MUST verify two things before creating a worktree:**

1. The directory is ignored by git
2. The directory is not already tracked in the repository

```bash
# Ignored?
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null

# Tracked already?
git ls-files --stage .worktrees worktrees
```

**Expected result:**
- `git check-ignore` succeeds for the directory you plan to use
- `git ls-files --stage ...` prints nothing

**If NOT ignored:**
1. Add the directory to `.gitignore`
2. Commit that change
3. Re-run safety verification
4. Only then create the worktree

**If tracked:**
1. Stop
2. Do not reuse that project-local directory yet
3. Fix the tracked path first, or fall back to the global directory

**Why this matters:** A tracked `.worktrees/<branch>` path can appear as a gitlink/submodule-like entry and leak into normal commits. Existing directory != safe directory.

### Post-Creation Verification

Immediately after `git worktree add`, verify the new path did not appear in the main worktree's tracked state:

```bash
git status --short .worktrees worktrees
git ls-files --stage .worktrees worktrees
```

**Expected result:** No new tracked entry for the worktree path.

**If you see `.worktrees/<branch>` or `worktrees/<branch>` in status or as a `160000` gitlink entry:**
1. Stop
2. Do not continue implementation yet
3. Remove/fix the tracked entry or move the worktree outside the repo

### For Global Directory (~/.config/superpowers/worktrees)

No `.gitignore` verification needed; the worktree lives outside the repository.

## Creation Steps

### 1. Detect Project Name

```bash
project=$(basename "$(git rev-parse --show-toplevel)")
```

### 2. Create Worktree

```bash
# Determine full path
case "$LOCATION" in
  .worktrees|worktrees)
    mkdir -p "$LOCATION"
    path="$LOCATION/$BRANCH_NAME"
    ;;
  "$HOME"/.config/superpowers/worktrees/*)
    mkdir -p "$HOME/.config/superpowers/worktrees/$project"
    path="$HOME/.config/superpowers/worktrees/$project/$BRANCH_NAME"
    ;;
esac

# Create worktree with new branch
git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

### 3. Run Project Setup

Auto-detect and run appropriate setup:

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

### 4. Verify Clean Baseline

Run tests to ensure the worktree starts clean:

```bash
# Examples - use project-appropriate command
npm test
cargo test
pytest
go test ./...
```

**If tests fail:** Report failures and ask whether to proceed or investigate.

**If tests pass:** Report ready.

### 5. Report Location

```
Worktree ready at <full-path>
Tests passing (<N> tests, 0 failures)
Ready to implement <feature-name>
```

## Quick Reference

| Situation | Action |
|-----------|--------|
| `.worktrees/` exists | Candidate only; verify ignored + untracked before reuse |
| `worktrees/` exists | Candidate only; verify ignored + untracked before reuse |
| Both exist | Check `.worktrees/` first |
| Neither exists | Check CLAUDE.md -> ask user |
| Directory not ignored | Add to `.gitignore`, commit, re-verify |
| Directory already tracked | Stop and fix tracked path, or use global location |
| Post-create status shows worktree path | Stop and fix before implementing |
| Tests fail during baseline | Report failures + ask |
| No package.json/Cargo.toml | Skip dependency install |

## Common Mistakes

### Treating an existing directory as automatically safe

- **Problem:** Reusing `.worktrees/` just because it exists can reintroduce a tracked path or gitlink mess
- **Fix:** Existing directory means candidate only; verify ignored and untracked every time

### Checking ignore status but not tracked state

- **Problem:** `git check-ignore` alone does not catch an already tracked `.worktrees/<branch>` entry
- **Fix:** Always pair it with `git ls-files --stage .worktrees worktrees`

### Missing the post-creation check

- **Problem:** The worktree gets created, but the main worktree now shows `.worktrees/<branch>` as modified or tracked
- **Fix:** Run `git status --short .worktrees worktrees` immediately after creation

### Proceeding with failing tests

- **Problem:** Can't distinguish new bugs from pre-existing issues
- **Fix:** Report failures, get explicit permission to proceed

### Hardcoding setup commands

- **Problem:** Breaks on projects using different tools
- **Fix:** Auto-detect from project files (package.json, etc.)

## Example Workflow

```
You: I'm using the using-git-worktrees skill to set up an isolated workspace.

[Check .worktrees/ - exists]
[Verify ignored - git check-ignore confirms .worktrees/ is ignored]
[Verify untracked - git ls-files --stage .worktrees returns nothing]
[Create worktree: git worktree add .worktrees/auth -b feature/auth]
[Verify post-create status - no .worktrees/auth entry appears in main worktree]
[Run npm install]
[Run npm test - 47 passing]

Worktree ready at /Users/jesse/myproject/.worktrees/auth
Tests passing (47 tests, 0 failures)
Ready to implement auth feature
```

## Red Flags

**Never:**
- Create a project-local worktree without verifying both ignore status and tracked state
- Reuse `.worktrees/` or `worktrees/` just because the directory exists
- Ignore a `160000` gitlink entry for `.worktrees/<branch>`
- Skip the post-creation status check
- Proceed with failing tests without asking
- Assume directory location when ambiguous
- Skip CLAUDE.md check

**Always:**
- Follow directory priority: existing > CLAUDE.md > ask
- Verify project-local directories are both ignored and untracked
- Run a post-creation git status check from the main worktree
- Auto-detect and run project setup
- Verify clean test baseline

## Integration

**Called by:**
- **brainstorming** (Phase 4) - REQUIRED when design is approved and implementation follows
- **subagent-driven-development** - REQUIRED before executing any tasks
- **executing-plans** - REQUIRED before executing any tasks
- Any skill needing isolated workspace

**Pairs with:**
- **finishing-a-development-branch** - REQUIRED for cleanup after work complete
