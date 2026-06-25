# 02 - Data Model and Service Contracts

Date: 2026-06-25
Status: implementation guidance
Audience: Agent OS implementation agent

## Purpose

This file describes the records and service contracts needed to implement backend quality control inside Agent OS.

The exact SQL should follow Agent OS's current database style. The concepts below should remain stable.

## Main records

The managed-worker backend needs these conceptual records:

```text
managed_worker_runs
managed_worker_events
managed_worker_artifacts
managed_worker_inspection_reports
managed_worker_repair_links
```

It should also reuse existing Agent OS records:

```text
tasks
projects/repositories
worktrees
file leases
command evidence
patch artifacts
review records
approval/operator decisions
evidence/artifacts/traces
memory candidates
```

## `managed_worker_runs`

Purpose:

One terminal worker execution controlled by Agent OS.

Conceptual fields:

```text
id
 task_id
 work_brief_id
 role
 adapter_name
 mode
 status
 worktree_id
 cwd
 command
 argv_json
 env_summary_json
 pid
 process_group_id
 started_at
 ended_at
 last_activity_at
 exit_code
 exit_signal
 stopped_by
 failure_label
 failure_message
 inspection_report_id
 log_artifact_id
 created_at
 updated_at
```

Notes:

- `command` and `argv_json` should record what Agent OS launched.
- `env_summary_json` should not store secret values.
- `pid` and `process_group_id` are local runtime facts.
- `status` must be changed only through the state machine service.

## `managed_worker_events`

Purpose:

Ordered event timeline for the run.

Conceptual fields:

```text
id
 managed_run_id
 task_id
 seq
 ts
 kind
 stream
 summary
 payload_json
 artifact_id
 created_at
```

Example event kinds:

```text
run.created
preflight.started
preflight.failed
brief.written
worker.starting
worker.started
terminal.output
worker.no_activity_warning
worker.stop_requested
worker.stopped
worker.exited
inspection.started
inspection.completed
inspection.blocked
repair.available
repair.started
cleanup.warning
```

Important:

This should connect to Agent OS's existing task history / trace / artifact system where possible. Do not build a parallel world that the rest of Agent OS cannot see.

## `managed_worker_artifacts`

Purpose:

Store run-specific files.

Examples:

```text
work brief markdown
launch preamble
run snapshot JSON
full terminal log
inspection report JSON
patch/scope report
watchdog report
```

Conceptual fields:

```text
id
 managed_run_id
 task_id
 artifact_type
 path_or_uri
 summary
 size_bytes
 sha256
 created_at
```

## `managed_worker_inspection_reports`

Purpose:

Saved result of the backend closing pass.

Conceptual fields:

```text
id
 managed_run_id
 task_id
 process_result_json
 changed_files_json
 patch_result_json
 scope_result_json
 dirty_work_result_json
 verification_result_json
 review_readiness_json
 approval_readiness_json
 outcome
 failure_label
 next_action
 created_at
```

## `managed_worker_repair_links`

Purpose:

Connect repair runs to the run/failure they are repairing.

Conceptual fields:

```text
id
 source_run_id
 repair_run_id
 task_id
 failure_label
 evidence_artifact_ids_json
 attempt_number
 max_attempts
 status
 created_at
```

## Status values

Use exact or equivalent values:

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

## Failure labels

Use exact or equivalent labels:

```text
failed_before_start
worker_command_missing
worker_crashed
worker_timed_out
worker_stopped_by_stig
worker_stuck
no_patch_produced
patch_missing
dirty_work_left
out_of_scope_changes
protected_file_changed
dependency_change_needs_approval
verification_missing
verification_failed
verification_stale
review_changes_requested
approval_required
repair_limit_reached
cleanup_required
unknown_failure
```

## Core service contracts

### `startManagedWorkerRun(input)`

Starts a managed terminal worker.

Input:

```text
taskId
workBriefId
role
adapterName
requestedBy
mode = managed
```

Behavior:

```text
create managed run
run preflight
write brief artifacts
build adapter command
transition to starting
start process
record PID/process group
transition to running
append events
```

Failure behavior:

```text
if preflight fails -> failed_before_start
if process launch fails -> failed_before_start
rollback safe partial state
record failure label
```

### `stopManagedWorkerRun(input)`

Stops a running worker.

Input:

```text
runId
requestedBy
reason
```

Behavior:

```text
transition running/stuck -> stopping
send graceful stop to process group
wait grace period
force kill if needed
record stopped event
preserve logs
trigger inspection or final stopped state
```

### `appendManagedRunEvent(input)`

Records an ordered event.

Input:

```text
runId
kind
stream
summary
payload
artifactId optional
```

Behavior:

```text
assign sequence number
write event
publish to UI if live channel exists
bridge to task history/trace if supported
```

### `runManagedWorkerWatchdogTick(input)`

Checks active workers.

Behavior:

```text
load active runs
check process exists
check last activity
check max runtime
check stop requests
record warnings/failures
transition stuck/crashed/timed_out when needed
```

### `inspectManagedRunResult(runId)`

Runs backend closing pass.

Behavior:

```text
load run/task/brief/worktree
inspect process outcome
inspect changed files
check patch/scope/dirty work
check verification evidence
check review/approval readiness
classify failure/outcome
write inspection report
transition state
```

### `checkPatchScopeAndDirtyWork(input)`

Checks worktree changes.

Behavior:

```text
read git status
read changed/untracked/deleted files
compare to allowed/forbidden/protected paths
detect missing patch
classify dirty state
return blocking/non-blocking findings
```

### `checkVerificationEvidence(input)`

Checks proof.

Behavior:

```text
load required checks
load command evidence
verify command ran in correct worktree
verify command is fresh after final changes
verify exit code zero
return passed/missing/failed/stale
```

### `classifyManagedRunFailure(input)`

Creates plain failure label and next action.

Behavior:

```text
use inspection report facts
choose highest-priority blocking reason
write label/title/explanation/nextAction
```

### `createRepairPass(input)`

Creates a narrow repair pass from a specific backend failure.

Behavior:

```text
verify repair allowed
check max attempts
create repair brief
attach failed evidence/logs
start managed worker with fixer role
link repair run to source run
```

## Important invariants

### Invariant 1 - state changes through one service

No direct status updates outside the state machine.

### Invariant 2 - worker claims are not proof

Terminal text can be stored, but cannot by itself satisfy verification, scope, or patch checks.

### Invariant 3 - inspection is required after execution

Every run that starts a process must end with either inspection or a recorded reason why inspection was impossible.

### Invariant 4 - ready states need proof

`ready_for_review` and `ready_for_approval` require backend proof.

### Invariant 5 - final done is human/approval controlled

Important work cannot be marked final done by worker exit or worker text.

## First schema migration acceptance

A first migration is acceptable when it supports this vertical slice:

```text
create run
append events
store PID/exit status
store log artifact link
store inspection report
store failure label
link repair run later
```

Do not overbuild schema for future remote/cloud workers yet.