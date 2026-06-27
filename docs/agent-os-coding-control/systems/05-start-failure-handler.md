# 05 - Start Failure Handler

## Plain meaning

The Start Failure Handler records and cleans up failures that happen before a coding run truly begins.

This covers situations where Agent OS tried to start a worker but something broke.

## What it does

It handles failures such as:

- terminal tool not installed.
- wrong command path.
- worktree creation failed.
- launch packet could not be written.
- process exited immediately.
- permission problem.
- missing project setup.

## How it works

Agent OS creates a run record before launch. If launch fails, the failure is attached to that run.

A simple flow:

```text
create run record
try to prepare worktree
try to start worker
launch fails
record clear failure reason
save setup log
clean up partial state
mark run failed-before-start
```

## What it requires

- Run record before process launch.
- Error capture around every launch step.
- Setup log artifact.
- Cleanup logic for partial worktrees/processes.
- Plain failure label.

## Who owns it

Agent OS owns this system.

The agent may never start, so this cannot depend on the agent.

## Terminal-based feasibility

Fully feasible with terminals.

Launch failures are normal process/setup errors and can be detected by backend code.

## Why it matters

Without this, failed starts create confusing half-created state.

With it, Stig sees exactly why the run never began and what to fix next.