# Backend Quality Control Is Code, Not Prompts

Date: 2026-06-25
Status: implementation clarification
Owner: Stig

## Purpose

This note clarifies one important point for the implementing agent:

```text
Backend quality control must be implemented as Agent OS backend code.
It is not only a prompt improvement.
```

A prompt can tell a worker how to behave. Backend code must check whether the worker actually behaved correctly.

## The split

| Layer | Job |
| --- | --- |
| Work brief | Tells the worker what to do. |
| Worker agent | Performs the coding work. |
| Backend quality-control code | Checks process state, files, patch, evidence, verification, review, and approval requirements. |
| Stig | Makes final product decisions when needed. |

## What prompts should do

Prompts and work briefs should tell the worker:

```text
Here is the task.
Here is the allowed scope.
Here is what not to touch.
Here is the required verification.
Do not mark yourself done.
Do not approve yourself.
Report blockers clearly.
```

## What backend code should do

Backend code should independently check:

```text
Was there a managed run record?
Was a locked work brief used?
Did the worker process start?
Did it exit, crash, timeout, or stop?
What files changed?
Were changed files inside allowed scope?
Is there a patch artifact?
Is the worktree dirty?
Were required checks actually run?
Did checks pass?
Is verification evidence fresh?
Is review required?
Is Stig approval required?
What official state should the run move to?
```

## Rule for official truth

Agent text can be useful context.

But official state should come from backend facts:

```text
run records
process records
Git/file-system state
command evidence
verification runs
patch artifacts
review records
approval records
```

## First vertical slice

The first implementation should prove this path:

```text
1. Create managed run record.
2. Launch a simple test worker.
3. Capture output/log.
4. Worker exits.
5. Backend inspects worktree.
6. Backend checks patch/evidence.
7. Backend sets official state.
8. UI or CLI shows plain status and reason.
```

## Success example

The system is working when Agent OS can say:

```text
The worker exited, but Agent OS blocked the result because required verification evidence was missing.
```

Or:

```text
The worker exited and Agent OS found an in-scope patch with fresh passing verification, so the task is ready for review.
```