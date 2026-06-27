# 10 - Cleanup Doctor

## Plain meaning

The Cleanup Doctor checks whether Agent OS has messy or stale local state that needs attention.

It is the health check for the coding workspace.

## What it does

It finds problems such as:

- stale worktrees.
- dirty worktrees from closed tasks.
- expired file leases.
- orphaned terminal processes.
- runs marked active even though the process is gone.
- missing artifacts.
- old preview servers.
- task state that no longer matches local files.

## How it works

Agent OS scans its backend records and local environment, then reports anything that looks unsafe or stale.

A simple flow:

```text
scan tasks
scan runs
scan worktrees
scan processes
scan leases
scan artifacts
show clean / warning / needs attention
```

## What it requires

- Local worktree registry.
- Process registry.
- Lease records.
- Artifact records.
- Safe cleanup rules.
- Dry-run mode before destructive cleanup.

## Who owns it

Agent OS owns this system.

Stig should approve risky cleanup, especially anything that might delete dirty work.

## Terminal-based feasibility

Fully feasible with terminals.

This is file-system, process, and database inspection.

## Why it matters

Long-running local systems get messy. The Cleanup Doctor keeps Agent OS trustworthy and understandable over time.