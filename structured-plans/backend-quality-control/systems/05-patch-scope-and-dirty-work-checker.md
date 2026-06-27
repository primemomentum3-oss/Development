# 05 - Patch, Scope, and Dirty Work Checker

## Purpose

The Patch, Scope, and Dirty Work Checker verifies that the worker produced real, packageable work inside the approved task boundary.

This is one of the most important backend-only quality systems.

## Plain-language meaning

Agent OS should check:

```text
Did the worker actually change anything?
Were the changes allowed?
Were the changes packaged properly?
Did the worker leave messy unfinished work behind?
```

## What it checks

### 1. Changed files

Use Git/file-system inspection to list:

```text
modified files
added files
deleted files
renamed files
untracked files
staged files
unstaged files
```

### 2. Scope boundary

Compare changed files against the work brief/task rules:

```text
allowed paths
forbidden paths
protected paths
files needing Stig approval
files needing special review
```

### 3. Dirty work

Detect messy states:

```text
files changed but no patch artifact
staged changes but no final package
untracked files not included in patch
workspace dirty after supposedly completed run
no commits/patches despite claimed completion
```

### 4. Dependency/package risk

Treat these as special:

```text
package.json
package-lock.json
pnpm-lock.yaml
yarn.lock
bun.lock
requirements.txt
pyproject.toml
Cargo.toml
Dockerfile
CI/workflow files
```

These may be allowed, but should require explicit scope or approval.

## Result labels

Possible labels:

```text
patch_ok
no_changes
patch_missing
dirty_work_left
out_of_scope_changes
protected_file_changed
dependency_change_needs_approval
untracked_files_left
deleted_files_need_review
```

## Implementation shape

Suggested function:

```text
checkPatchScopeAndDirtyWork({ taskId, runId, worktreePath, briefId })
```

Return:

```text
{
  status: "ok" | "blocked" | "needs_approval",
  changedFiles: [...],
  outOfScopeFiles: [...],
  protectedFiles: [...],
  dirtyState: {...},
  reason,
  nextAction
}
```

## How this connects to Result Inspector

The Result Inspector should call this checker.

If this checker returns blocked, the run cannot become ready for review or approval.

## What not to do

Do not trust agent text like:

```text
I only changed login files.
```

Check Git/files directly.

Do not ignore untracked files.

Do not ignore lockfile/dependency changes.

Do not allow dirty work to masquerade as success.

## Acceptance criteria

- out-of-scope file changes block readiness.
- dirty work after completion blocks readiness.
- dependency/lockfile changes require explicit scope or approval.
- untracked files are reported.
- no patch/no change cases are clearly explained.