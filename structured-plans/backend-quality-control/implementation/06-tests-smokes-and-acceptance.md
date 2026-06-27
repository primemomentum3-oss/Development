# 06 - Tests, Smokes, and Acceptance Criteria

Date: 2026-06-25
Status: implementation guidance
Audience: Agent OS implementation agent

## Purpose

This file explains how to prove the backend quality-control system works.

Do not consider the implementation complete because the happy path works.

The system exists to catch bad worker behavior.

## Test categories

Use four layers:

```text
1. Unit tests
2. Service tests with fake adapters
3. Smoke scripts
4. Desktop/manual acceptance checks
```

## Unit tests

Add tests for deterministic logic.

Recommended files:

```text
tests/managed-worker-state-machine.test.ts
tests/managed-worker-preflight.test.ts
tests/managed-worker-failure-classifier.test.ts
tests/managed-worker-scope-checker.test.ts
tests/managed-worker-verification-gate.test.ts
tests/managed-worker-dirty-work-checker.test.ts
tests/managed-worker-context-bundle.test.ts
```

## State machine tests

Must cover:

```text
preparing -> starting allowed
starting -> running allowed
running -> exited allowed
exited -> inspecting allowed
inspecting -> ready_for_review allowed
worker cannot set done
running -> ready_for_approval rejected
blocked -> done rejected
illegal transitions append no event
legal transitions append event
```

## Preflight tests

Must cover:

```text
missing task fails
unlocked work brief fails
unknown adapter fails
missing worktree fails or creates safely
implementation work without scope fails
implementation work without verification policy fails
active duplicate run fails unless explicitly allowed
```

## Scope checker tests

Must cover:

```text
allowed file passes
forbidden file blocks
protected file requires approval
package file change flagged
lockfile change flagged
untracked file reported
deleted file reported
generated file flagged when configured
```

## Verification gate tests

Must cover:

```text
required command missing blocks
required command failed needs repair
passing command allows verification
passing command from wrong worktree rejected
passing command from old run rejected
passing command before final change marked stale
manual verification required state works
```

## Failure classifier tests

Must cover:

```text
failed_before_start -> clear setup failure
worker timeout -> timeout explanation
out-of-scope changes outrank missing verification
protected file changed outranks repair suggestion
verification failed -> needs repair
verification missing -> blocked
repair limit reached -> ask Stig
unknown facts -> unknown_failure, never success
```

## Fake adapter service tests

Use a fake shell adapter before real Codex/Claude.

Fake workers should simulate:

```text
success text but no changes
success text with patch but no verification
success text with verification failure
out-of-scope file change
dependency file change
silent hang
timeout
crash exit code
valid patch and passing verification
```

## Smoke scripts

Recommended scripts:

```text
scripts/managed-worker-smoke.mjs
scripts/managed-worker-fake-success-blocked-smoke.mjs
scripts/managed-worker-timeout-smoke.mjs
scripts/managed-worker-scope-smoke.mjs
scripts/managed-worker-verification-gate-smoke.mjs
scripts/managed-worker-cleanup-doctor-smoke.mjs
scripts/managed-worker-context-bundle-smoke.mjs
```

## Package scripts

After implementation, add scripts similar to existing Agent OS smoke/check style:

```text
"managed-worker:smoke": "npm run build && node scripts/managed-worker-smoke.mjs"
"managed-worker:quality-smoke": "npm run build && node scripts/managed-worker-fake-success-blocked-smoke.mjs && node scripts/managed-worker-scope-smoke.mjs && node scripts/managed-worker-verification-gate-smoke.mjs"
"managed-worker:check": "npm run check && npm test && npm run db:reset && npm run managed-worker:quality-smoke"
```

## Required negative smokes

These are more important than happy path.

### Smoke 1 - fake success blocked

Worker:

```text
prints Done
exits 0
changes no files
runs no checks
```

Expected:

```text
status: blocked
failure: no_patch_produced or verification_missing
```

### Smoke 2 - out-of-scope blocked

Worker:

```text
changes a file outside allowed path
exits 0
```

Expected:

```text
status: blocked
failure: out_of_scope_changes
Stig decision required if scope expansion possible
```

### Smoke 3 - verification missing blocked

Worker:

```text
creates patch
exits 0
runs no command evidence
```

Expected:

```text
status: blocked
failure: verification_missing
```

### Smoke 4 - verification failed repairable

Worker:

```text
creates patch
runs required command
command exits non-zero
```

Expected:

```text
status: needs_repair
failure: verification_failed
repairAvailable: true
```

### Smoke 5 - timeout detected

Worker:

```text
starts
prints nothing
never exits
```

Expected:

```text
watchdog marks stuck/timed_out
log preserved
stop possible
```

### Smoke 6 - valid run reaches review

Worker:

```text
creates allowed patch
runs required verification
verification exits 0
```

Expected:

```text
status: ready_for_review or ready_for_approval depending on policy
inspection report saved
proof visible
```

## Desktop/manual checks

Verify that Stig can see:

```text
managed run status
last activity
plain failure reason
changed files summary
verification evidence summary
next action
open terminal/logs link
repair button when repair is available
approval button only when backend says ready
```

## Definition of done for implementation

The backend quality-control system is not done until:

```text
state machine exists and is tested
managed run records exist
fake worker adapter works
result inspector runs after exit
patch/scope/dirty checks work
verification evidence gate works
failure labels are plain
negative smokes block fake success
frontend can show blocked/ready states
```

## Most important acceptance statement

The implementation must prove this:

```text
A worker can lie, forget tests, change the wrong files, or exit cleanly without doing real work.
Agent OS still catches it and blocks the result.
```

That is the reason this system exists.