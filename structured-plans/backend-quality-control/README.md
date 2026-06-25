# Backend Quality Control Systems

Date: 2026-06-25
Status: structured planning specs
Owner: Stig

## Purpose

This folder defines the backend-only systems Agent OS needs around managed terminal workers.

These systems should work without asking an agent to reason about whether the work is valid.

The main idea:

```text
The agent can write code.
Agent OS must check the work.
```

Agent OS should use deterministic backend checks for lifecycle, safety, verification, result inspection, cleanup, and failure classification.

## What this is inspired by

The reference system has useful backend control patterns:

- create a run record before starting the worker.
- keep an official run state machine.
- watch worker health.
- run a backend closing pass after the worker exits.
- detect fake success, such as dirty work with no committed result.
- push/open follow-up only after backend checks pass.
- use repair workers only after the system identifies a specific failure.
- clean up workspaces after extracting results.

Agent OS should adapt these ideas for local terminal workers.

## What this is not

This is not a prompt pack.

This is not another agent role.

This is not a replacement for Stig approval.

This is backend control code that should keep working even if the coding agent is wrong, confused, silent, or overconfident.

## Files

Start with:

- [System Map](./system-map.md)

Then read the individual backend system specs:

1. [Lifecycle State Machine](./systems/01-lifecycle-state-machine.md)
2. [Preflight and Launch Rollback](./systems/02-preflight-and-launch-rollback.md)
3. [Worker Health Watchdog](./systems/03-worker-health-watchdog.md)
4. [Result Inspector](./systems/04-result-inspector.md)
5. [Patch, Scope, and Dirty Work Checker](./systems/05-patch-scope-and-dirty-work-checker.md)
6. [Verification Evidence Gate](./systems/06-verification-evidence-gate.md)
7. [Repair Loop and Check Fixer](./systems/07-repair-loop-and-check-fixer.md)
8. [Cleanup and Orphan Control](./systems/08-cleanup-and-orphan-control.md)
9. [Failure Reason Classifier](./systems/09-failure-reason-classifier.md)

## Priority order

Build these in this order:

```text
1. Lifecycle State Machine
2. Preflight and Launch Rollback
3. Worker Health Watchdog
4. Result Inspector
5. Patch, Scope, and Dirty Work Checker
6. Verification Evidence Gate
7. Failure Reason Classifier
8. Repair Loop and Check Fixer
9. Cleanup and Orphan Control
```

The first complete milestone should be:

```text
A terminal worker can run, exit, and then Agent OS independently decides whether the run is valid, failed, blocked, needs repair, or ready for review.
```