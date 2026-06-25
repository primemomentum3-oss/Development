# Backend Quality Control Systems

Date: 2026-06-25
Status: structured implementation specs
Owner: Stig

## Purpose

This folder defines the backend-only systems Agent OS needs around managed terminal workers.

These systems should work without asking an agent to reason about whether the work is valid.

The main idea:

```text
The agent can write code.
Agent OS must check the work with backend code.
```

Agent OS should use deterministic backend checks for lifecycle, safety, verification, result inspection, cleanup, and failure classification.

## Read this first if you are the implementing agent

You do not need access to the reference system to implement this.

Read in this order:

1. [Backend Quality Control Is Code, Not Prompts](./actual-code-not-prompts.md)
2. [System Map](./system-map.md)
3. [Existing Agent OS Systems Map](./agent-os-existing-systems-map.md)
4. [Agent OS Codebase Integration Guide](./agent-os-codebase-integration-guide.md)
5. [Implementation Sequence](./implementation-sequence.md)
6. [Frontend And Backend Result View](./front-end-and-back-end-result-view.md)
7. The individual backend system specs under `systems/`

## What this is inspired by

The reference system has useful backend control patterns:

- create a run record before starting the worker.
- keep an official run state machine.
- watch worker health.
- run a backend closing pass after the worker exits.
- detect fake success, such as dirty work with no committed or packaged result.
- push/open follow-up only after backend checks pass.
- use repair workers only after the system identifies a specific failure.
- clean up workspaces after extracting results.

Agent OS should adapt these ideas for local terminal workers.

Do not copy the API/runtime design directly.

Agent OS should implement the same control pattern around local terminal processes.

## What this is not

This is not a prompt pack.

This is not another agent role.

This is not a replacement for Stig approval.

This is backend control code that should keep working even if the coding agent is wrong, confused, silent, or overconfident.

## Existing Agent OS systems this must reuse

The backend quality-control layer should reuse Agent OS foundations where possible:

```text
tasks
event timeline
Git worktrees
file leases
command evidence
verification runs
reliability gates
patch artifacts
review records
approval/operator decisions
memory candidates
reports
agent launcher/local process foundations
```

Do not create parallel systems for these unless the existing system cannot support the required behavior.

## Files

Main documents:

- [Backend Quality Control Is Code, Not Prompts](./actual-code-not-prompts.md)
- [System Map](./system-map.md)
- [Existing Agent OS Systems Map](./agent-os-existing-systems-map.md)
- [Agent OS Codebase Integration Guide](./agent-os-codebase-integration-guide.md)
- [Implementation Sequence](./implementation-sequence.md)
- [Frontend And Backend Result View](./front-end-and-back-end-result-view.md)

Individual backend system specs:

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
1. Managed run records
2. Lifecycle State Machine
3. Preflight and Launch Rollback
4. Simple shell/test worker launch
5. Result Inspector stub
6. Patch, Scope, and Dirty Work Checker
7. Verification Evidence Gate
8. Failure Reason Classifier
9. Worker Health Watchdog
10. Repair Loop and Check Fixer
11. Cleanup and Orphan Control
12. Desktop/UI integration
13. Codex/Claude terminal adapters
```

## First complete milestone

The first complete milestone should be:

```text
A terminal worker can run, exit, and then Agent OS independently decides whether the run is valid, failed, blocked, needs repair, or ready for review.
```

## Product promise

When this is implemented, Stig should not need to ask:

```text
Did the agent really do the work correctly?
```

Agent OS should answer:

```text
I checked the files, patch, command evidence, verification result, scope, and approval requirements. Here is the official state and next action.
```