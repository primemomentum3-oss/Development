# Backend Quality Control Implementation Sequence

Date: 2026-06-25
Status: implementation sequence
Owner: Stig

## Purpose

This file tells the implementing agent the exact order to build backend quality control inside Agent OS.

The goal is to avoid a broad rewrite and instead build one reliable vertical slice at a time.

## Rule

Do not start with Codex or Claude.

Start with a simple local test worker so the backend quality-control loop can be proven without model behavior.

## Phase 1 - Add managed run records

### Goal

Agent OS can store a managed worker run as a backend object.

### Build

Add or extend database records for:

```text
managed_worker_runs
managed_worker_run_processes
managed_worker_inspection_reports
managed_worker_cleanup_findings
```

Use existing task, event, worktree, evidence, patch, review, and approval records where possible.

### Required fields

```text
run id
task id
worktree id/path
worker role
adapter name
state
started at
ended at
last activity at
process id
process group id
exit code
failure label
inspection report id
```

### Acceptance

A CLI command or service call can create and show a managed run record without launching anything.

## Phase 2 - Add state machine

### Goal

Agent OS has one official way to change managed run state.

### Build

Create:

```text
state-machine.ts
states.ts
```

Implement legal transitions.

### Acceptance

Tests prove:

```text
running -> inspecting is allowed
running -> done is rejected
ready_for_approval -> done requires approved actor
failed_before_start cannot silently become ready_for_review
```

## Phase 3 - Add preflight

### Goal

Agent OS refuses unsafe launches before any worker starts.

### Build

Create:

```text
preflight.ts
```

Check:

```text
task exists
work brief locked
worktree ready
adapter exists
scope rules known
required proof known
no duplicate active run
```

### Acceptance

A missing work brief blocks launch with a clear reason.

## Phase 4 - Add simple process launch

### Goal

Agent OS can start a simple local test worker and record the process.

### Build

Create:

```text
process-adapters.ts
launch.ts
process-registry.ts
output-capture.ts
```

First adapter:

```text
shellTestAdapter
```

This adapter can run a small script that writes logs and exits.

### Acceptance

Agent OS can:

```text
start worker
capture output
record process id
record exit code
save log artifact
transition from starting to running to exited
```

## Phase 5 - Add result inspector stub

### Goal

Every worker exit triggers inspection.

### Build

Create:

```text
result-inspector.ts
```

First version can inspect process outcome and create a basic report.

### Acceptance

A worker exit automatically moves run to inspecting, writes an inspection report, and chooses a final non-done state.

## Phase 6 - Add patch and scope checker

### Goal

Agent OS checks actual changed files.

### Build

Create:

```text
patch-scope-checker.ts
```

Use Git/file-system inspection.

Check:

```text
changed files
untracked files
deleted files
allowed paths
protected paths
patch artifact exists
worktree dirty state
```

### Acceptance

A run with out-of-scope changes is blocked.

A run with dirty work and no patch is blocked.

## Phase 7 - Add verification evidence gate

### Goal

Agent OS requires actual command evidence.

### Build

Create:

```text
verification-gate.ts
```

Use existing command evidence and verification run records.

Check:

```text
required command ran
exit code was zero
command belongs to this task/run
command ran in correct worktree
command is fresh after final file change
```

### Acceptance

A worker saying tests passed does not count unless command evidence exists.

## Phase 8 - Add failure classifier

### Goal

Agent OS explains failures clearly.

### Build

Create:

```text
failure-classifier.ts
```

Labels:

```text
failed_before_start
worker_crashed
worker_timed_out
patch_missing
out_of_scope_changes
verification_missing
verification_failed
approval_required
unknown_failure
```

### Acceptance

Every blocked/failed run has a plain reason and next action.

## Phase 9 - Add watchdog

### Goal

Agent OS detects unhealthy active runs.

### Build

Create:

```text
watchdog.ts
```

Checks:

```text
process missing
silence timeout
maximum runtime
stop requested
```

### Acceptance

A silent or missing worker is marked stuck/crashed and logs are preserved.

## Phase 10 - Add repair loop

### Goal

Agent OS can create narrow repair passes for specific failures.

### Build

Create:

```text
repair-loop.ts
```

Rules:

```text
repair only when backend identified a repairable failure
attach failed evidence/log
limit attempts
ask Stig after max attempts
```

### Acceptance

A failed verification can create a repair run, but out-of-scope changes require Stig decision first.

## Phase 11 - Add cleanup doctor

### Goal

Agent OS can find and report stale backend/local state.

### Build

Create:

```text
cleanup-doctor.ts
```

Checks:

```text
orphan processes
stale worktrees
expired leases
missing artifacts
runs marked active without process
```

### Acceptance

Cleanup scan produces findings and separates safe cleanup from approval-required cleanup.

## Phase 12 - Add frontend integration

### Goal

Stig can see backend quality-control results without reading raw logs.

### Build

Desktop UI should show:

```text
managed run state
worker health
inspection result
changed files summary
verification status
failure reason
next action
open log / open terminal links
```

### Acceptance

A non-coder can understand whether work is ready, blocked, failed, or needs repair.

## Phase 13 - Add Codex/Claude adapters

### Goal

After backend control works with shellTestAdapter, add real terminal agents.

### Build

Add:

```text
codexCliAdapter
claudeCliAdapter
```

Do not let adapter-specific logic leak into result inspection.

### Acceptance

Codex/Claude run through the same backend quality-control path as shellTestAdapter.

## Final milestone

The feature is ready when this works:

```text
Stig starts managed work.
Agent OS launches a terminal worker.
Agent OS captures output.
Agent OS detects process health.
Worker exits.
Agent OS inspects files, patch, scope, and verification evidence.
Agent OS sets a clear official state.
Stig sees a plain result and next action.
```