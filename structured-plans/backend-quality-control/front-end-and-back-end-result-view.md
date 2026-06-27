# Frontend And Backend Result View

Date: 2026-06-25
Status: implementation guide
Owner: Stig

## Purpose

This file explains how backend quality control should appear to Stig in the Agent OS frontend.

The frontend should not ask Stig to read raw terminal logs first.

The frontend should show Agent OS's backend-checked result.

## Main frontend idea

The terminal is not the main truth surface.

The managed run result card is the main truth surface.

```text
Terminal output = raw detail.
Backend quality-control result = official status.
```

## Main UI areas

### 1. Managed run status card

Shows:

```text
Task title
Worker role
Worker adapter/tool
Current state
Last activity
Elapsed time
Plain status
Plain reason
Next action
```

Example:

```text
Task: Fix login bug
Worker: Codex Builder
State: Blocked
Reason: Required verification was not run
Next action: Start verification pass or ask worker to run checks
```

### 2. Backend inspection panel

Shows what Agent OS checked:

```text
Process result
Changed files
Scope result
Patch result
Verification result
Review requirement
Approval requirement
Failure label
```

This panel should make backend quality control visible and understandable.

### 3. Changed files panel

Shows:

```text
in-scope files
out-of-scope files
protected files
untracked files
deleted files
dependency/lockfile changes
```

Use simple labels:

```text
Allowed
Needs approval
Blocked
Warning
```

### 4. Verification panel

Shows:

```text
required check
was it run?
exit code
log link
freshness
status
```

Example:

```text
Required check: npm test
Status: Missing
Reason: No command evidence found for this run
```

Or:

```text
Required check: npm test
Status: Passed
Log: open artifact
Freshness: ran after final code change
```

### 5. Decision panel

Shows Stig what he can do next:

```text
Approve
Request changes
Start repair pass
Allow broader scope
Reject result
Stop task
Open logs
Open terminal
```

Only show actions that make sense for the current state.

## Status examples

### Ready for review

```text
Status: Ready for review
Patch exists.
Changed files are inside scope.
Required verification passed.
Review is required next.
```

### Blocked: missing verification

```text
Status: Blocked
Reason: Required verification was not run.
Next action: Run verification or start a repair/check pass.
```

### Blocked: out-of-scope changes

```text
Status: Blocked
Reason: The worker changed files outside the approved scope.
Next action: Allow broader scope or request a narrower fix.
```

### Needs repair

```text
Status: Needs repair
Reason: Required verification failed.
Next action: Start focused repair pass.
```

### Failed before start

```text
Status: Failed before start
Reason: Worker command was not available.
Next action: Configure the worker adapter.
```

## Frontend rule

Do not show raw logs as the first answer.

Show this first:

```text
What Agent OS checked
What Agent OS decided
Why it decided that
What Stig can do next
```

Then allow raw logs/terminal as secondary detail.

## Backend API shape for frontend

The frontend should receive a composed view model, not assemble everything itself.

Suggested endpoint/service output:

```text
ManagedRunResultView {
  runId
  taskId
  title
  state
  workerRole
  adapterName
  startedAt
  endedAt
  lastActivityAt
  plainStatus
  plainReason
  nextActions
  processSummary
  changedFilesSummary
  patchSummary
  verificationSummary
  reviewSummary
  approvalSummary
  inspectionReportId
  logArtifactId
  terminalPaneId
}
```

## Why this matters

Stig should be able to understand the state without being a coder.

The UI should answer:

```text
Is this safe?
Is this proven?
What failed?
What should I do next?
```

That is the product value of backend quality control.