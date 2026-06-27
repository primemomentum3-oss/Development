# 08 - Cleanup and Orphan Control

## Purpose

Cleanup and Orphan Control keeps Agent OS healthy after managed worker runs.

It finds stale local state, orphaned processes, half-created resources, and old worktrees.

## Plain-language meaning

Agent OS should not slowly become messy.

It should know when something is left behind:

```text
old worktree
orphaned process
dirty stopped run
expired lease
missing artifact
run says active but worker is gone
preview server still running
```

## What it scans

### Process scan

Find:

```text
managed process still alive after run ended
run marked running but process missing
child processes left after stop
old preview/dev server
```

### Worktree scan

Find:

```text
worktree for completed task
worktree for cancelled task
dirty worktree after failed run
empty worktree after failed launch
worktree missing from disk but present in database
```

### Lease scan

Find:

```text
expired file leases
leases from dead runs
overlapping active leases that should not overlap
```

### Artifact scan

Find:

```text
run without log artifact
inspection without report artifact
verification record without command log
artifact path missing from disk
```

## Cleanup actions

Actions should be separated into safe and risky.

### Safe automatic cleanup

```text
mark dead process as gone
release expired lease from dead run
remove empty failed-launch temp folder
close stale live event subscription
```

### Needs Stig approval

```text
delete dirty worktree
remove uncommitted changes
delete old artifact bundle
kill unknown process
```

## Implementation shape

Suggested functions:

```text
runCleanupDoctorScan(input)
classifyCleanupFinding(finding)
applySafeCleanupAction(action)
requestCleanupApproval(action)
```

## Findings

Finding shape:

```text
id
kind
severity: info | warning | dangerous
resource_type
resource_id
summary
safe_auto_fix_available
requires_stig_approval
recommended_action
```

## Schedule

Run cleanup scan:

```text
on Agent OS startup
periodically while app is open
before starting a new managed run
when a run exits
```

## What not to do

Do not delete dirty work automatically.

Do not hide orphaned state.

Do not kill processes if Agent OS is not sure it owns them.

Do not let cleanup failures block unrelated work unless they create safety risk.

## Acceptance criteria

- stale worktrees are visible.
- orphaned processes are visible.
- expired leases are cleaned or reported.
- dirty worktree deletion requires Stig approval.
- cleanup actions are audited.