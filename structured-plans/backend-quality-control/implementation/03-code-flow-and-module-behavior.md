# 03 - Code Flow and Module Behavior

Date: 2026-06-25
Status: implementation guidance
Audience: Agent OS implementation agent

## Purpose

This file explains how the code should actually behave during a managed terminal worker run.

It is written so an implementation agent can build the system without needing the reference system.

## Complete code flow

### Flow A - Start managed worker

```text
User/Agent OS requests managed run
→ startManagedWorkerRun
→ create managed run record in preparing state
→ run preflight checks
→ write work brief artifact
→ select worker adapter
→ build command/env/cwd
→ transition preparing -> starting
→ spawn terminal worker process
→ attach PID/process group
→ transition starting -> running
→ capture output
→ append run events
```

### Flow B - Worker output

```text
process emits stdout/stderr/PTY data
→ output capture handler receives chunk
→ append short terminal.output event
→ append chunk to full log artifact
→ update last_activity_at
→ optionally bridge summary to task history/trace
```

### Flow C - Worker exits

```text
process exits
→ record exit code/signal
→ append worker.exited event
→ transition running -> exited
→ call inspectManagedRunResult(runId)
```

### Flow D - Result inspection

```text
inspectManagedRunResult
→ transition exited -> inspecting
→ inspect process outcome
→ inspect changed files
→ check patch/scope/dirty state
→ check verification evidence
→ check review/approval readiness
→ classify outcome
→ write inspection report
→ transition to blocked / needs_repair / ready_for_review / ready_for_approval / failed
```

### Flow E - Watchdog tick

```text
periodic tick
→ load active runs
→ check process exists
→ check last activity
→ check elapsed runtime
→ record warnings
→ stop/mark stuck/crashed/timed out when needed
```

### Flow F - Repair pass

```text
inspection finds repairable failure
→ createRepairPass
→ check max attempts
→ create narrow repair work brief
→ attach logs/evidence
→ start managed worker with fixer role
→ repeat normal lifecycle
```

## Module behavior details

## 1. `runner.ts`

`runner.ts` owns process launch orchestration.

It should not contain all checking logic. It should call other modules.

Pseudo-structure:

```text
export async function startManagedWorkerRun(input) {
  const run = await repository.createManagedRun(...)

  try {
    await preflightManagedRun(run)
    const briefArtifacts = await writeManagedRunBriefFiles(run)
    const adapter = resolveWorkerAdapter(input.adapterName)
    const command = adapter.buildCommand({ run, briefArtifacts })
    const env = buildSafeWorkerEnv(adapter, run)

    await transitionManagedRunState(run.id, 'starting')
    const proc = await spawnManagedWorkerProcess(command, env)
    await repository.attachProcess(run.id, proc)
    await transitionManagedRunState(run.id, 'running')

    attachOutputHandlers(proc, run)
    attachExitHandler(proc, run)

    return run
  } catch (err) {
    await rollbackFailedLaunch(run, err)
    return failedRun
  }
}
```

Important:

- Create run before launch.
- Attach output handler immediately after spawn.
- Record process group if possible.
- Never let thrown launch errors leave a run in `preparing` forever.

## 2. `preflight.ts`

Preflight should be strict.

Pseudo-structure:

```text
export async function preflightManagedRun(input) {
  requireTaskExists()
  requireTaskNotTerminal()
  requireLockedWorkBrief()
  requireKnownRole()
  requireAdapterConfigured()
  requireWorktreeReadyOrCreatable()
  requireScopeRulesForImplementation()
  requireVerificationPolicyKnown()
  requireNoDuplicateActiveRunUnlessAllowed()
}
```

Preflight failures should become `failed_before_start`, not `blocked`.

Reason:

The worker never ran. This is a setup/system issue.

## 3. `state-machine.ts`

All state transitions go here.

Pseudo-structure:

```text
const transitions = {
  preparing: ['starting', 'failed_before_start', 'cancelled'],
  starting: ['running', 'failed_before_start'],
  running: ['stopping', 'stuck', 'crashed', 'exited'],
  exited: ['inspecting'],
  inspecting: ['blocked', 'needs_repair', 'ready_for_review', 'ready_for_approval'],
  ready_for_approval: ['done'],
}
```

This module should append a state transition event every time.

## 4. `events.ts`

Events must be readable.

Bad event summary:

```text
state=ERR_14 payload={...}
```

Good event summary:

```text
Worker exited with code 0.
Result inspection started.
Verification evidence missing.
```

Keep raw payload for debugging, but provide plain summaries for Stig.

## 5. `watchdog.ts`

The watchdog should be safe and boring.

It should not infer code quality.

It only checks runtime health:

```text
alive?
silent?
too long?
stop requested?
process gone?
```

It should append warning events before destructive actions when possible.

## 6. `result-inspector.ts`

This is the main backend quality-control module.

It should be deterministic and compositional.

Pseudo-structure:

```text
export async function inspectManagedRunResult(runId) {
  const ctx = await loadInspectionContext(runId)

  await transitionManagedRunState(runId, 'inspecting')

  const processResult = inspectProcessOutcome(ctx)
  const gitChanges = await inspectGitChanges(ctx)
  const patchResult = await checkPatch(ctx, gitChanges)
  const scopeResult = await checkScope(ctx, gitChanges)
  const dirtyResult = await checkDirtyWork(ctx, gitChanges)
  const verificationResult = await checkVerificationEvidence(ctx)
  const readiness = classifyReadiness({ processResult, patchResult, scopeResult, dirtyResult, verificationResult })
  const failure = classifyManagedRunFailure(readiness)

  await saveInspectionReport(...)
  await transitionManagedRunState(runId, readiness.nextState, failure)
}
```

Do not place Codex/Claude-specific logic here.

The Result Inspector should work for every worker adapter.

## 7. `checks/git-changes.ts`

This module should run Git commands safely in the task worktree.

Suggested commands:

```text
git status --porcelain=v1
git diff --name-status
git diff --cached --name-status
git ls-files --others --exclude-standard
```

Return normalized changed file records.

Do not let callers parse raw Git output themselves.

## 8. `checks/scope-checker.ts`

This module compares changed files to work brief/task scope.

Inputs:

```text
changed files
allowed path patterns
forbidden path patterns
protected path patterns
```

Outputs:

```text
ok
outOfScopeFiles
protectedFiles
requiresApprovalFiles
```

## 9. `checks/verification-evidence-gate.ts`

This module must connect to existing Agent OS command evidence.

It should answer:

```text
Was the required command run?
Was it run in the correct worktree?
Was it run after final changes?
Did it pass?
Where is the log?
```

If existing Agent OS command evidence already has enough data, reuse it.

If not, add the missing fields there rather than making a duplicate verification system.

## 10. `failure-classifier.ts`

This module should be a pure decision function as much as possible.

Input:

```text
inspection facts
```

Output:

```text
label
title
plain explanation
technical details
next action
repair possible?
Stig decision needed?
```

Example:

```text
label: verification_missing
title: Required check was not run
explanation: The worker finished, but Agent OS could not find command evidence for the required verification command.
nextAction: Run verification or start a repair/check pass.
```

## 11. `repair-loop.ts`

Repair loop should be controlled by backend rules.

It should not automatically repair everything.

Repair is allowed when:

```text
failure is specific
scope is still clear
max attempts not reached
no Stig approval is required first
```

Repair is not allowed when:

```text
protected file changed
product decision needed
scope unclear
repair limit reached
```

## 12. `cleanup-doctor.ts`

Cleanup Doctor can run on startup, after run exit, and manually.

It should produce findings first, not immediately delete.

Finding examples:

```text
orphaned process
stale worktree
dirty completed worktree
missing log artifact
expired lease
```

Risky cleanup requires Stig approval.

## How modules interact

```text
runner
  uses preflight
  uses adapters
  uses events
  uses state-machine
  triggers result-inspector

watchdog
  uses repository
  uses process-groups
  uses state-machine
  uses events

result-inspector
  uses checks/*
  uses failure-classifier
  uses state-machine
  uses repository

repair-loop
  uses result-inspector output
  uses work brief builder
  uses runner

cleanup-doctor
  uses repository
  uses worktree/process/artifact checks
```

## Coding style expectations

- Keep modules small.
- Keep side effects isolated.
- Prefer pure functions for classification.
- Write tests for every checker.
- Do not mix frontend language with backend logic.
- Do not put Git parsing in UI code.
- Do not put process management inside React components.
- Do not let adapters decide official status.

## Minimal acceptance scenario

A fake worker does this:

```text
prints 'done'
exits 0
makes no file changes
runs no verification
```

Agent OS should produce:

```text
status: blocked
failure label: no_patch_produced or verification_missing
plain explanation visible to Stig
inspection report saved
```

If this scenario passes, the backend is starting to protect against AI slop.