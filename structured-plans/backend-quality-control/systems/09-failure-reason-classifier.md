# 09 - Failure Reason Classifier

## Purpose

The Failure Reason Classifier converts backend facts into clear failure labels and next actions.

It helps Stig understand what happened without reading raw logs.

## Plain-language meaning

Instead of showing only:

```text
Failed
```

Agent OS should show:

```text
Failed before start: Codex command was not found.
```

Or:

```text
Blocked: Required verification was not run.
```

Or:

```text
Needs repair: Tests failed.
```

## Inputs

The classifier should use deterministic facts:

```text
run state
process exit code
signal/killed status
watchdog result
preflight error
changed files result
scope checker result
patch checker result
verification gate result
review result
approval requirement
cleanup findings
```

It should not rely only on agent text.

## Failure labels

Suggested labels:

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

## Label behavior

Each label should include:

```text
short title
plain explanation
technical detail reference
recommended next action
whether repair is possible
whether Stig decision is needed
```

Example:

```text
Label: verification_missing
Title: Required check was not run
Explanation: The worker finished, but Agent OS could not find command evidence for the required verification command.
Next action: Ask worker to run verification or start a repair/check pass.
Repair possible: yes
Stig decision needed: no, unless repeated.
```

## Priority order

If multiple problems exist, choose the clearest blocking reason.

Suggested priority:

```text
1. failed_before_start
2. worker_crashed / timed_out / stopped
3. protected_file_changed
4. out_of_scope_changes
5. dirty_work_left / patch_missing
6. verification_failed
7. verification_missing / stale
8. review_changes_requested
9. approval_required
10. unknown_failure
```

## Implementation shape

Suggested function:

```text
classifyManagedRunFailure({ runId, inspectionReport })
```

Return:

```text
{
  label,
  title,
  explanation,
  technicalDetails,
  nextAction,
  repairPossible,
  stigDecisionNeeded
}
```

## User-facing display

Show:

```text
Status: Blocked
Reason: Changed files outside approved scope
What happened: The worker edited package.json, but the work brief only allowed login-related files.
Next: Allow broader scope, request narrower fix, or stop the task.
```

## What not to do

Do not expose raw internal enum names as the only explanation.

Do not hide the technical details completely.

Do not classify based on agent summary alone.

Do not mark unknown failures as success.

## Acceptance criteria

- every blocked/failed run has a plain reason.
- each reason has a next action.
- labels are based on backend facts.
- UI can show a non-coder-readable explanation.
- technical detail remains available for debugging.