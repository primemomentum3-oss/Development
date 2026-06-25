# Agent OS Codebase Integration Guide For Backend Quality Control

Date: 2026-06-25
Status: implementation guide
Owner: Stig

## Purpose

This guide tells the implementing agent how to add backend quality control into the Agent OS codebase.

The implementing agent does not need to know the reference system. This guide describes the Agent OS version directly.

## Existing Agent OS foundations to reuse

Agent OS already has important pieces that should not be duplicated:

```text
TypeScript backend
Postgres database
agent-os CLI
Desktop app
Task records
Event timeline
Git worktrees
File leases
Command evidence
Verification runs
Reliability gates
Patch artifacts
Review records
Memory candidates and promotion
Reports
Coordinator/runtime foundations
Trusted runner foundation
Agent launcher foundation
Local process launch foundation
Agent run contract foundation
Evidence gate and verification foundation
Workflow harness and closer package foundation
Codex adapter and automatic execution foundation
```

Backend quality control should connect these pieces around managed terminal runs.

It should not create a parallel task system, parallel evidence system, or parallel review system.

## New feature boundary

Name this feature area:

```text
managed worker quality control
```

Its job:

```text
Control and inspect terminal-based coding runs from backend code.
```

It should sit between:

```text
request/task/work brief
    ↓
managed terminal worker
    ↓
backend result inspection
    ↓
verification/review/approval workflow
```

## Proposed backend folder structure

Use existing Agent OS conventions where possible. If existing folder names differ, adapt the names but keep these boundaries.

Preferred structure:

```text
src/managed-worker/
  index.ts
  types.ts
  states.ts
  state-machine.ts
  preflight.ts
  launch.ts
  process-adapters.ts
  process-registry.ts
  output-capture.ts
  watchdog.ts
  result-inspector.ts
  patch-scope-checker.ts
  verification-gate.ts
  failure-classifier.ts
  repair-loop.ts
  cleanup-doctor.ts
  events.ts
  artifacts.ts
```

If Agent OS already has runtime/launcher folders that are a better fit, use this structure inside the existing runtime area instead:

```text
src/runtime/managed-worker/
```

The important thing is not the exact folder name. The important thing is keeping responsibilities separated.

## File responsibilities

### `types.ts`

Defines shared types:

```text
ManagedWorkerRun
ManagedWorkerRunState
ManagedWorkerRole
ManagedWorkerAdapterName
ManagedWorkerOutcome
ManagedWorkerFailureLabel
ManagedWorkerInspectionReport
ManagedWorkerPreflightResult
ManagedWorkerVerificationGateResult
ManagedWorkerPatchScopeResult
```

### `states.ts`

Defines allowed state names:

```text
preparing
starting
running
stopping
stuck
crashed
failed_before_start
exited
inspecting
blocked
needs_repair
ready_for_review
ready_for_approval
done
cancelled
```

### `state-machine.ts`

Owns legal transitions. No other file should casually update managed run state.

Expected functions:

```text
transitionManagedWorkerRunState(input)
assertManagedWorkerTransitionAllowed(from, to)
getAllowedManagedWorkerTransitions(state)
```

### `preflight.ts`

Checks whether a run is safe to start.

Expected checks:

```text
task exists
work brief locked
role known
worktree available
worker adapter configured
required proof known
no duplicate active run
scope rules present for implementation work
```

### `launch.ts`

Coordinates the launch:

```text
create run record
run preflight
write work brief artifact
create or attach worktree
build command
start worker process
attach process id
transition to running
```

Expected functions:

```text
startManagedWorkerRun(input)
rollbackManagedWorkerLaunch(input)
```

### `process-adapters.ts`

Defines adapters for different local terminal workers.

First adapter should be a simple test/shell worker. Codex and Claude adapters come after the simple adapter works.

Expected adapter shape:

```text
name
buildCommand(context)
buildEnvironment(context)
buildInitialInput(context)
requiresPty
```

Adapters:

```text
shellTestAdapter
codexCliAdapter
claudeCliAdapter
```

### `process-registry.ts`

Tracks active local worker processes:

```text
store process id
store process group id if available
lookup active process
stop process
mark process exited
recover after app restart where possible
```

### `output-capture.ts`

Captures worker output and writes it to Agent OS evidence, artifacts, and events:

```text
append terminal output events
update last activity
write full log artifact
truncate UI preview safely
keep raw log available
```

### `watchdog.ts`

Checks live workers:

```text
detect silence
detect timeout
detect process gone
detect stop requested
transition to stuck/crashed/stopped
preserve logs
```

### `result-inspector.ts`

Runs after worker exit:

```text
load run/task/brief/worktree
check process outcome
call patch-scope checker
call verification gate
check review/approval needs
write inspection report
transition to blocked, needs_repair, ready_for_review, ready_for_approval, or failed
```

### `patch-scope-checker.ts`

Uses Git/file-system facts:

```text
list changed files
list untracked files
list deleted files
check allowed paths
check forbidden/protected paths
check dirty state
check patch artifact exists
flag dependency or lockfile changes
```

### `verification-gate.ts`

Checks command evidence and verification runs:

```text
find required verification commands
find evidence from this run/task
check exit codes
check command ran in correct worktree
check evidence freshness after final file change
return missing, failed, stale, or passed
```

### `failure-classifier.ts`

Turns facts into plain reason labels:

```text
failed_before_start
worker_crashed
worker_timed_out
patch_missing
out_of_scope_changes
verification_missing
verification_failed
repair_limit_reached
approval_required
```

### `repair-loop.ts`

Controls repair passes:

```text
allow repair only for specific backend-detected failures
create narrow repair brief
attach failed logs/evidence
limit attempts
stop and ask Stig when needed
```

### `cleanup-doctor.ts`

Finds stale local state:

```text
orphaned process scan
stale worktree scan
expired lease scan
missing artifact scan
safe cleanup actions
approval-required cleanup actions
```

### `events.ts`

Bridges managed worker events into the existing Agent OS event timeline.

Do not create a separate timeline if the existing task/event timeline can be used.

### `artifacts.ts`

Stores and links:

```text
work brief artifact
worker log artifact
inspection report artifact
patch artifact references
verification logs
```

Prefer existing Agent OS artifact/evidence services.

## Proposed database additions

Reuse existing task/evidence/worktree tables where possible.

Add only what is missing.

Suggested new records:

```text
managed_worker_runs
managed_worker_run_events
managed_worker_run_processes
managed_worker_inspection_reports
managed_worker_cleanup_findings
```

If Agent OS already has runtime run tables that fit, extend them instead of creating duplicate tables.

## Key integration points

### Task system

Every managed worker run must attach to a task. Implementation work should not run as a loose terminal job.

### Worktree system

Implementation work should run inside a task worktree. The result inspector should inspect that worktree.

### File lease system

Use file leases as part of scope enforcement where possible. The patch/scope checker should compare changed files with lease/scope rules.

### Command evidence system

Verification proof must come from command evidence, not terminal summaries.

### Verification runs

Use existing verification run records if they already exist. Do not invent a second verification system.

### Patch artifacts

Implementation work should require patch artifacts when task rules require a patch.

### Review records

If review is required, the inspector should move to ready_for_review instead of ready_for_approval.

### Approval system

Important work should stop at ready_for_approval until Stig approves. The worker must never set final done.

### Memory system

Backend quality control should not automatically promote memory. It can create memory candidates from repeated failures or inspection reports, but durable memory promotion must follow existing approval rules.

## CLI commands to add or extend

Suggested CLI commands:

```text
agent-os managed-run start --task <id> --adapter shell-test
agent-os managed-run show <run-id>
agent-os managed-run events <run-id>
agent-os managed-run stop <run-id>
agent-os managed-run inspect <run-id>
agent-os managed-run doctor
```

If Agent OS already has runtime commands, add these under the existing runtime CLI group instead.

## Smoke scripts to add

Add scripts similar to existing smoke scripts:

```text
scripts/managed-worker-smoke.mjs
scripts/managed-worker-failed-launch-smoke.mjs
scripts/managed-worker-watchdog-smoke.mjs
scripts/managed-worker-result-inspector-smoke.mjs
scripts/managed-worker-out-of-scope-smoke.mjs
scripts/managed-worker-missing-verification-smoke.mjs
scripts/managed-worker-cleanup-doctor-smoke.mjs
```

Package scripts can later expose:

```text
"managed-worker:smoke": "npm run build && node scripts/managed-worker-smoke.mjs"
"managed-worker-quality:check": "npm run check && npm test && npm run db:reset && node scripts/managed-worker-smoke.mjs && node scripts/managed-worker-result-inspector-smoke.mjs"
```

## Frontend integration

The desktop app should show:

```text
managed run status
worker role
adapter/tool
last activity
plain failure reason
inspection report
changed files
verification status
next action
raw terminal/log link
```

Terminal panes should be secondary. The main status comes from backend run state and inspection report.

## First implementation milestone

Implement one small full path before Codex/Claude integration:

```text
1. Create managed run record.
2. Launch shell test adapter.
3. Capture log.
4. Exit process.
5. Inspect worktree.
6. Require patch/evidence according to task type.
7. Produce official state and plain reason.
8. Expose via CLI.
```

Only after that should the implementing agent add Codex and Claude adapters.

## Implementation warning

Do not implement this as a separate mini Agent OS.

This layer should reuse existing Agent OS systems:

```text
tasks
events
worktrees
leases
command evidence
verification
patches
reviews
approval
artifacts
reports
memory candidates
```

The new code should connect these systems around managed terminal workers.