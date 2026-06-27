# 07 - Repair Loop and Check Fixer

## Purpose

The Repair Loop and Check Fixer creates controlled follow-up work when backend checks fail.

It should not let an agent keep trying forever. It should create narrow repair passes based on specific evidence.

## Plain-language meaning

If Agent OS finds a problem, it should not just say:

```text
Failed.
```

It should say:

```text
This failed for a specific reason.
A narrow repair pass can try to fix only that reason.
```

## Triggers

A repair pass can be created when:

```text
verification failed
review requested changes
required evidence is missing but recoverable
patch exists but needs a small correction
CI/check logs identify a specific failure
```

A repair pass should not be created when:

```text
scope is unclear
Stig approval is required first
the worker changed protected files without permission
max attempts reached
the failure requires a product decision
```

## Backend responsibilities

Agent OS controls:

```text
whether repair is allowed
which failure the repair is for
which log/evidence is attached
which files are allowed
how many attempts are allowed
when to stop and ask Stig
```

The repair agent only performs the repair work.

## Repair brief

A repair pass brief should be narrow:

```text
Fix only the failing verification error.
Use this failed command log.
Do not refactor unrelated code.
Do not change dependencies without approval.
Do not broaden scope.
Run the failed check again.
```

## Retry limits

Track:

```text
repair_attempt_number
max_repair_attempts
failure_type
previous_repair_run_id
```

Suggested default:

```text
max_repair_attempts: 2
```

After max attempts, block and ask Stig.

## Implementation shape

Suggested functions:

```text
createRepairPass({ sourceRunId, failureReason, evidenceIds })
canCreateRepairPass({ taskId, failureReason })
recordRepairAttempt({ taskId, repairRunId })
```

## States

Possible repair states:

```text
repair_available
repair_started
repair_succeeded
repair_failed
repair_limit_reached
repair_needs_stig
```

## Check fixer variant

For failed local/CI checks, use a special Check Fixer role.

It receives:

```text
failed command
exit code
log artifact
changed files
current patch
allowed scope
required re-check command
```

It should not receive broad permission to redo the whole task.

## What not to do

Do not start repair automatically when Stig approval is required.

Do not let repair loop run forever.

Do not let fixer change unrelated files.

Do not hide previous failed attempts.

Do not treat repair success as final done.

## Acceptance criteria

- failed verification can create a focused repair pass.
- repair pass has narrow allowed scope.
- repair attempts are counted.
- max attempts are enforced.
- Stig sees why repair started and what it is allowed to fix.