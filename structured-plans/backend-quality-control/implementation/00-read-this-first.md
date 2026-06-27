# 00 - Read This First: Backend Quality Control Implementation

Date: 2026-06-25
Status: implementation guidance
Audience: Agent OS implementation agent

## What you are building

You are not building a prompt pack.

You are not building a new autonomous agent.

You are building deterministic Agent OS backend code that checks and controls managed terminal workers.

The coding worker may be Codex CLI, Claude CLI, a shell worker, or another local process. The worker can write code, but Agent OS must decide whether the work is valid.

## Main rule

```text
Agents produce work.
Agent OS verifies work.
```

This means the backend must inspect real facts:

```text
process state
exit code
stdout/stderr logs
worktree changes
patch artifact
command evidence
verification exit codes
review state
approval state
cleanup state
```

Do not use the coding worker's own summary as the source of truth.

## Reference-system lesson without requiring the reference system

The implementation agent does not need to know or read Warren.

The lesson to copy is simple:

```text
Create a run record before launch.
Run the worker inside a controlled workspace.
Record events while it runs.
After it exits, run a backend closing pass.
If backend facts disagree with agent success, block or fail the run.
Only move forward when proof exists.
Clean up after result extraction.
```

Agent OS must adapt this to local terminal workers.

The Agent OS version should be:

```text
Stig request
→ locked work brief
→ managed terminal worker
→ backend events/logs
→ result inspector
→ patch/scope/verification gates
→ ready/blocked/repair/failure state
→ Stig decision when needed
```

## What already exists in Agent OS

Agent OS already has useful foundations such as tasks, worktrees, file leases, command evidence, verification gates, review records, approval flow, patches, evidence, artifacts, and smoke scripts.

Do not duplicate those systems.

This work should connect managed terminal runs into the existing Agent OS backend truth model.

## What must be new or strengthened

The implementation should add or strengthen:

```text
managed worker run records
run lifecycle state machine
terminal process runner
worker process adapters
preflight checks
launch rollback
worker watchdog
result inspector
patch/scope/dirty-work checker
verification evidence gate integration
failure reason classifier
repair loop control
cleanup doctor integration
frontend managed-run view
smoke tests and unit tests
```

## First vertical slice

Build the smallest complete version first:

```text
1. Create a managed run record.
2. Lock a work brief.
3. Launch a fake shell worker from the brief.
4. Capture output and exit code.
5. Inspect the worktree after exit.
6. Block success if patch/verification is missing.
7. Show the final state in CLI or frontend.
```

Do not start by implementing full Codex/Claude behavior.

Start with a fake shell adapter so the backend lifecycle is provably correct.

## Non-negotiable design requirements

### 1. Backend owns official state

The worker cannot mark itself done, ready, approved, or verified.

### 2. Every important transition must be recorded

No invisible state changes.

### 3. Every failure must have a plain reason

Stig should see why something is blocked or failed.

### 4. Verification must be evidence-based

A worker saying `tests passed` is not proof.

### 5. Scope must be checked from files, not from summaries

A worker saying `I only changed login files` is not proof.

### 6. Dirty work must block readiness

If files changed but no patch/evidence exists, the run is not ready.

### 7. Stig approval remains final for important work

Backend checks can say `ready for approval`; they should not auto-finalize important work.

## Implementation order

```text
Phase 1: Data model + state machine
Phase 2: Fake worker runner + output capture
Phase 3: Preflight + launch rollback
Phase 4: Result inspector + patch/scope checker
Phase 5: Verification evidence gate
Phase 6: Watchdog + stop/timeout handling
Phase 7: Failure reason classifier
Phase 8: Repair loop
Phase 9: Frontend managed-run view
Phase 10: Codex/Claude terminal adapters
```

## How to know the first version is correct

A test worker should be able to claim success while doing no real work.

Agent OS must block it.

Example expected result:

```text
Worker exit code: 0
Worker said: done
Patch: missing
Verification: missing
Agent OS status: blocked
Reason: no patch produced and required verification missing
```

That is the core of the system.