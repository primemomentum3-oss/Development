# Backend Quality Control System Map

## Plain-language summary

Agent OS needs a backend control layer that checks the work before, during, and after a coding worker runs.

This layer should not depend on the coding agent being honest or correct.

It should answer:

```text
Was the work allowed to start?
Did the worker stay alive?
Did the worker produce real changes?
Were those changes inside scope?
Was required verification actually run?
Did verification pass?
Is review or Stig approval required?
Should this be repaired, blocked, failed, or accepted for review?
```

## No reference-system knowledge required

The implementing agent does not need the reference system.

The Agent OS implementation should be understood like this:

```text
Agent OS starts a local terminal worker.
Agent OS records the run.
Agent OS watches the process.
Agent OS captures output.
Agent OS inspects the worktree.
Agent OS checks patch, scope, verification, review, and approval.
Agent OS decides the official state.
```

The reference-system lesson is only the pattern:

```text
Do not trust the worker as final truth.
Use backend code to control and check the lifecycle.
```

## The backend-only control loop

```text
1. Preflight
   Check that the task, work brief, role, worktree, and rules exist.

2. Launch
   Start the terminal worker only after records and guards are ready.

3. Watch
   Track process health, silence, timeout, crash, and stop requests.

4. Close
   When the worker exits, inspect what actually happened.

5. Verify
   Check patch, scope, changed files, command evidence, and required proof.

6. Decide
   Move to failed, blocked, needs repair, ready for review, or ready for approval.

7. Repair or clean up
   Start a narrow repair pass if appropriate, or clean stale resources.
```

## Core principle

The worker can say:

```text
I am done.
```

Agent OS should respond internally:

```text
Maybe. I will inspect the facts.
```

Only backend facts should move work forward.

## Backend systems and jobs

| System | Job | Agent involved? |
| --- | --- | --- |
| Lifecycle State Machine | Controls legal run states and transitions. | No |
| Preflight and Launch Rollback | Prevents unsafe or half-created starts. | No |
| Worker Health Watchdog | Detects stuck, crashed, silent, or runaway workers. | No |
| Result Inspector | Runs the backend closing pass after worker exit. | No |
| Patch, Scope, and Dirty Work Checker | Checks changed files, patch existence, dirty work, and boundaries. | No |
| Verification Evidence Gate | Requires real command evidence and passing checks. | No |
| Repair Loop and Check Fixer | Starts narrow repair work after specific failures. | Agent only performs repair; backend controls loop. |
| Cleanup and Orphan Control | Cleans or reports stale processes, worktrees, and records. | No |
| Failure Reason Classifier | Converts backend facts into plain failure labels. | No |

## How this differs from a prompt-only approach

Prompt-only says:

```text
Tell the agent to be careful.
```

Backend quality control says:

```text
Check whether the agent was careful.
Block progress if not.
```

Agent OS should use both:

- prompt/brief instructions before work.
- backend checks after and around work.

But the official result must come from backend checks.

## How this should plug into Agent OS

Agent OS already has several foundations. The quality-control layer should sit on top of them:

```text
Task record
  ↓
Locked work brief / agent run contract
  ↓
Managed worker run record
  ↓
Task worktree + terminal process
  ↓
Event timeline + log artifact
  ↓
Patch artifact + command evidence + verification run
  ↓
Result inspector
  ↓
Review / approval / repair / blocked state
```

## Minimum viable backend control

The smallest useful version should do this:

```text
- create managed run record.
- start a simple terminal worker.
- capture process exit code.
- capture log artifact.
- inspect Git changed files.
- require patch for implementation work.
- require verification evidence when needed.
- block ready state if proof is missing.
- produce a plain failure/next-action label.
```

This is enough to stop fake success.

## Desired user-facing result

Stig should see:

```text
Worker finished.
Agent OS inspected the result.

Status: Blocked
Reason: Required verification was not run.
Next action: Start repair/check pass or ask worker to run verification.
```

Or:

```text
Worker finished.
Agent OS inspected the result.

Status: Ready for review
Patch exists.
Changed files are inside scope.
Required checks passed.
Review is required before Stig approval.
```

## What the codebase should look like after implementation

The implementation should create a clear backend feature area, either:

```text
src/managed-worker/
```

or, if it fits the existing runtime structure better:

```text
src/runtime/managed-worker/
```

That area should contain separate files for:

```text
state machine
preflight
launch
process adapters
watchdog
result inspector
patch/scope checker
verification gate
failure classifier
repair loop
cleanup doctor
events/artifacts bridge
```

The implementation guide has the detailed file map.