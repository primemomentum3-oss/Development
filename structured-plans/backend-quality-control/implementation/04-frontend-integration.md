# 04 - Frontend Integration

Date: 2026-06-25
Status: implementation guidance
Audience: Agent OS implementation agent

## Purpose

This file explains how backend quality control should appear in the Agent OS frontend.

The frontend should make backend facts understandable to Stig.

It should not make Stig read raw terminal output to know whether work is safe.

## Main frontend principle

```text
Agent OS is the control room.
The terminal is a worker room.
```

The frontend should show the managed run status, proof, failures, and decisions first.

The raw terminal should be available, but secondary.

## Recommended desktop feature folder

Use or adapt this structure:

```text
apps/desktop/src/features/managed-workers/
  api.ts
  types.ts
  hooks.ts
  ManagedRunPanel.tsx
  ManagedRunTimeline.tsx
  ManagedRunDecisionPanel.tsx
  ManagedRunTerminalLink.tsx
  WorkBriefPreview.tsx
  ResultInspectionCard.tsx
  FailureReasonCard.tsx
  VerificationEvidenceCard.tsx
  ScopeChangesCard.tsx
```

## What each frontend file does

### `api.ts`

Calls backend endpoints/IPC/CLI bridge for managed worker data.

Functions:

```text
startManagedRun
stopManagedRun
getManagedRun
listManagedRunEvents
getInspectionReport
createRepairPass
runCleanupDoctor
```

### `types.ts`

Frontend copies of backend response types.

Should mirror backend but remain UI-safe.

### `hooks.ts`

React hooks for loading and refreshing managed run data.

Examples:

```text
useManagedRun(runId)
useManagedRunEvents(runId)
useInspectionReport(runId)
useManagedRunActions(runId)
```

### `ManagedRunPanel.tsx`

Main status card.

Shows:

```text
task title
worker role
adapter/tool
mode
status
elapsed time
last activity
failure label if any
next action
```

Actions:

```text
stop
open terminal
open logs
inspect result
start repair
request review
approve if allowed
```

### `ManagedRunTimeline.tsx`

Readable event timeline.

Show simple event summaries:

```text
Brief written
Worker started
Output received
Worker exited
Inspection started
Verification missing
Blocked
```

Raw payload can be hidden behind details.

### `ManagedRunDecisionPanel.tsx`

Shows decisions Stig needs to make.

Examples:

```text
Allow dependency file change?
Start repair pass?
Stop stuck worker?
Approve final result?
```

### `ManagedRunTerminalLink.tsx`

Link/button to open raw worker terminal.

Must show warning:

```text
This is a managed worker terminal. Normal control should happen through Agent OS.
Typing directly here may mark this run as guided manual.
```

### `WorkBriefPreview.tsx`

Shows the locked work brief.

Sections:

```text
Goal
Allowed work
Not allowed
Required proof
Success means
Approval rules
```

### `ResultInspectionCard.tsx`

Shows backend closing-pass result.

Sections:

```text
Process result
Patch result
Scope result
Verification result
Review/approval readiness
Official outcome
```

### `FailureReasonCard.tsx`

Shows plain failure reason.

Example:

```text
Blocked: Required verification was not run.
The worker finished, but Agent OS could not find a command evidence record for the required test command.
Next: Run verification or start a repair/check pass.
```

### `VerificationEvidenceCard.tsx`

Shows required checks and whether they passed.

Example:

```text
Required check: npm test
Status: missing
Evidence: none found
```

### `ScopeChangesCard.tsx`

Shows changed files and whether they are allowed.

Example:

```text
Changed files: 3
Inside scope: 2
Needs approval: package.json
Blocked: yes
```

## Main user views

## 1. Task page integration

Every task page should show managed worker runs attached to the task.

Suggested sections:

```text
Current managed run
Past managed runs
Inspection reports
Repair attempts
Decisions needed
```

## 2. Workbench home

Show active managed work:

```text
Running workers
Stuck workers
Blocked results
Waiting for Stig approval
Repair available
```

## 3. Request preparation view

Before launching:

```text
Stig request
Generated work brief
Safety gate result
Start managed run button
```

## 4. Run detail view

During and after run:

```text
Managed run panel
Timeline
Proof cards
Terminal/log access
Decision panel
```

## Status language

Use plain statuses.

Do not expose raw technical states alone.

Recommended UI labels:

```text
Preparing work
Starting worker
Running
No recent activity
Possibly stuck
Stopping
Worker crashed
Inspecting result
Blocked
Needs repair
Ready for review
Ready for Stig approval
Done
```

## Backend-to-frontend response shape

Managed run response should include enough for a non-coder UI:

```text
id
taskTitle
role
adapterName
mode
status
statusLabel
lastActivityAt
elapsedSeconds
failureLabel
failureTitle
failureExplanation
nextAction
canStop
canRepair
canApprove
canOpenTerminal
```

Inspection report response should include:

```text
processSummary
changedFilesSummary
scopeSummary
patchSummary
verificationSummary
reviewSummary
approvalSummary
officialOutcome
plainReason
nextAction
```

## What Stig should experience

### Good result

```text
Worker finished.
Agent OS inspected the result.
Patch exists.
Changed files are inside scope.
Required check passed.
Next: Ready for review.
```

### Blocked result

```text
Worker finished, but Agent OS blocked the result.
Reason: Required verification was not run.
Next: Start repair/check pass or ask worker to run verification.
```

### Stuck result

```text
Worker may be stuck.
No output for 30 minutes.
Options: stop, keep waiting, open log.
```

## Important UI rules

### Rule 1 - terminal is not the main interface

The terminal is secondary. The managed run panel is primary.

### Rule 2 - show proof, not just summaries

If a run is ready, show what proof exists.

### Rule 3 - show next action

Every failed/blocked state should tell Stig what to do next.

### Rule 4 - keep raw details available

Advanced users/agents should be able to open raw logs and JSON details.

### Rule 5 - do not hide uncertainty

If Agent OS cannot verify something, say so clearly.

## First frontend milestone

A good first frontend milestone is:

```text
Task page shows one managed run.
Run panel shows status and failure reason.
Timeline shows events.
Inspection card shows patch/scope/verification result.
Stig can open terminal/logs but does not need them to understand the result.
```