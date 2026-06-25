# 01 - Agent OS Target File Map

Date: 2026-06-25
Status: implementation guidance
Audience: Agent OS implementation agent

## Purpose

This file explains how the backend quality-control system should be integrated into the Agent OS codebase.

The exact final paths may be adjusted to match the current repo layout, but the module boundaries should stay clear.

Agent OS is a TypeScript/Node project with CLI, MCP, smoke scripts, desktop workspace, database migrations, and backend tests. The implementation should follow that style.

## Desired high-level code structure

Add a new backend feature area for managed terminal workers.

Recommended root:

```text
src/managed-workers/
```

This keeps the new system separate from generic tasks, worktrees, reviews, memory, and MCP tools while still allowing it to call those existing systems.

## Proposed backend files

```text
src/managed-workers/
  index.ts
  types.ts
  state-machine.ts
  repository.ts
  preflight.ts
  brief-files.ts
  runner.ts
  process-groups.ts
  events.ts
  watchdog.ts
  result-inspector.ts
  failure-classifier.ts
  repair-loop.ts
  cleanup-doctor.ts
  adapters/
    worker-adapter.ts
    fake-shell-adapter.ts
    codex-cli-adapter.ts
    claude-cli-adapter.ts
  checks/
    git-changes.ts
    patch-checker.ts
    scope-checker.ts
    dirty-work-checker.ts
    verification-evidence-gate.ts
    approval-readiness.ts
```

## What each backend file does

### `index.ts`

Public exports for the managed worker feature.

Should export only stable service functions and types.

Example exports:

```text
startManagedWorkerRun
stopManagedWorkerRun
inspectManagedWorkerRun
runManagedWorkerWatchdogTick
createManagedWorkerRepairPass
```

### `types.ts`

Shared types and enums.

Contains:

```text
ManagedWorkerRun
ManagedWorkerStatus
ManagedWorkerMode
ManagedWorkerRole
ManagedRunEvent
ManagedRunFailureLabel
ManagedRunInspectionReport
WorkerAdapterName
```

### `state-machine.ts`

The only place that defines legal run transitions.

Contains:

```text
MANAGED_RUN_TRANSITIONS
canTransitionManagedRun(from, to)
transitionManagedRunState(input)
```

Important:

No other code should update managed run state directly.

### `repository.ts`

Database access for managed worker records.

Contains functions such as:

```text
createManagedRun
getManagedRun
listManagedRuns
setManagedRunProcess
setManagedRunStatus
appendManagedRunEvent
createManagedRunArtifact
saveInspectionReport
```

This should wrap SQL/database calls so business logic does not scatter raw queries everywhere.

### `preflight.ts`

Checks whether a run is safe to start.

Checks:

```text
task exists
work brief locked
worktree exists or can be created
role known
adapter configured
required checks known
no duplicate active run
scope rules known for implementation work
```

### `brief-files.ts`

Writes the locked work brief and launch preamble to disk.

Recommended artifact path:

```text
.agent-os/managed-runs/<run-id>/work-brief.md
.agent-os/managed-runs/<run-id>/launch-preamble.md
.agent-os/managed-runs/<run-id>/run-snapshot.json
```

### `runner.ts`

Starts the terminal worker.

Responsibilities:

```text
create run record
run preflight
write brief files
select adapter
build command/env/cwd
spawn process or PTY
capture output
record exit
trigger result inspector
```

This is the core runtime service.

### `process-groups.ts`

Handles process groups and safe stopping.

Responsibilities:

```text
start process in stoppable group where possible
send graceful stop
send force kill after grace period
avoid leaving child processes behind
```

### `events.ts`

Managed run event helper.

Responsibilities:

```text
append ordered events
summarize output chunks
link events to artifacts
deduplicate repeated warnings
support live UI later
```

Do not create a separate history system if Agent OS already has traces/events. This file should bridge managed worker events into the existing task/trace/evidence system where possible.

### `watchdog.ts`

Periodic health check for active runs.

Responsibilities:

```text
check process still exists
check last activity
check max runtime
handle stop requests
mark stuck/crashed/timed out
preserve logs
```

### `result-inspector.ts`

Backend closing pass after worker exit.

Responsibilities:

```text
load run/task/brief/worktree
check process result
call patch/scope/dirty checkers
call verification evidence gate
classify outcome
write inspection report
transition official state
```

### `failure-classifier.ts`

Turns backend facts into plain failure labels and next actions.

Examples:

```text
verification_missing
out_of_scope_changes
dirty_work_left
worker_timed_out
patch_missing
protected_file_changed
```

### `repair-loop.ts`

Creates narrow repair passes after specific backend failures.

Responsibilities:

```text
check repair is allowed
count attempts
create repair brief
attach failed evidence/logs
start repair worker
stop after max attempts
```

### `cleanup-doctor.ts`

Finds stale/orphaned state.

Responsibilities:

```text
orphaned processes
stale worktrees
expired leases
missing artifacts
dirty completed worktrees
run says active but process gone
```

## Adapter files

### `adapters/worker-adapter.ts`

Defines the common adapter interface.

Suggested shape:

```text
WorkerProcessAdapter {
  name: WorkerAdapterName
  buildCommand(context): WorkerCommand
  buildEnvironment(context): Record<string, string>
  buildInitialInput(context): string | null
  supportsPty: boolean
}
```

### `adapters/fake-shell-adapter.ts`

First implementation adapter.

Use this before Codex/Claude.

It should run a deterministic fake worker script for smoke tests.

### `adapters/codex-cli-adapter.ts`

Codex CLI terminal adapter.

Should only translate Agent OS run context into command/env/initial input.

It should not own lifecycle decisions.

### `adapters/claude-cli-adapter.ts`

Claude CLI terminal adapter.

Same rule as Codex adapter: adapter starts a tool, but Agent OS owns state and result checks.

## Check files

### `checks/git-changes.ts`

Reads changed files from the task worktree.

Should report:

```text
modified
added
deleted
renamed
untracked
staged
unstaged
```

### `checks/patch-checker.ts`

Checks whether implementation work produced a patch artifact or equivalent packageable result.

### `checks/scope-checker.ts`

Compares changed files to allowed/forbidden/protected path rules.

### `checks/dirty-work-checker.ts`

Detects unfinished work:

```text
dirty tree after completion
staged but not packaged
untracked files left
no patch despite changed files
```

### `checks/verification-evidence-gate.ts`

Checks that required commands were actually run and passed.

Must use command evidence records, not agent text.

### `checks/approval-readiness.ts`

Decides whether the run can be ready for review, ready for approval, or still blocked.

## Database/migration files

Use the repo's existing migration style.

Conceptual migration names:

```text
migrations/<timestamp>_managed_worker_runs.sql
migrations/<timestamp>_managed_worker_events.sql
migrations/<timestamp>_managed_worker_artifacts.sql
migrations/<timestamp>_managed_worker_inspections.sql
```

If Agent OS already has a migration directory with a different naming style, follow the existing style.

## CLI files

Add CLI commands for smoke/debug use.

Recommended:

```text
src/cli/commands/managed-workers.ts
```

Commands:

```text
agent-os managed-worker start --task <id> --adapter fake-shell
agent-os managed-worker show --run <id>
agent-os managed-worker events --run <id>
agent-os managed-worker inspect --run <id>
agent-os managed-worker stop --run <id>
agent-os managed-worker doctor
```

Wire this into the existing CLI router.

## Scripts

Add smoke scripts under `scripts/`.

Recommended:

```text
scripts/managed-worker-smoke.mjs
scripts/managed-worker-fake-success-blocked-smoke.mjs
scripts/managed-worker-timeout-smoke.mjs
scripts/managed-worker-scope-smoke.mjs
scripts/managed-worker-verification-gate-smoke.mjs
scripts/managed-worker-cleanup-doctor-smoke.mjs
```

Add package scripts later:

```text
"managed-worker:smoke": "npm run build && node scripts/managed-worker-smoke.mjs"
"managed-worker:check": "npm run check && npm test && npm run db:reset && node scripts/managed-worker-smoke.mjs && node scripts/managed-worker-fake-success-blocked-smoke.mjs"
```

## Tests

Recommended tests:

```text
tests/managed-worker-state-machine.test.ts
tests/managed-worker-preflight.test.ts
tests/managed-worker-result-inspector.test.ts
tests/managed-worker-scope-checker.test.ts
tests/managed-worker-verification-gate.test.ts
tests/managed-worker-failure-classifier.test.ts
tests/managed-worker-repair-loop.test.ts
```

## Desktop frontend files

Use the current desktop app layout, but keep the feature grouped.

Recommended feature root:

```text
apps/desktop/src/features/managed-workers/
```

Suggested files:

```text
api.ts
hooks.ts
types.ts
ManagedRunPanel.tsx
ManagedRunTimeline.tsx
ManagedRunDecisionPanel.tsx
ManagedRunTerminalLink.tsx
WorkBriefPreview.tsx
ResultInspectionCard.tsx
FailureReasonCard.tsx
```

## Frontend route/page integration

Add or integrate into the existing task/workbench page:

```text
Task page
  → Managed worker runs section
  → Current run panel
  → Work brief preview
  → Inspection result
  → Stop/repair/review/approve actions
```

Do not make the raw terminal the primary UI.

## Important integration rule

Do not duplicate existing Agent OS systems.

Reuse existing:

```text
tasks
worktrees
leases
command evidence
verification gates
patch artifacts
review records
approval records
evidence/artifact/trace systems
operator decisions
memory candidates
```

The new managed-worker layer should orchestrate and verify these existing systems around terminal workers.