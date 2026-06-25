# Agent OS Coding Control Notes

Date: 2026-06-25
Status: planning notes
Owner: Stig

## Purpose

These notes describe the backend control systems Agent OS needs so coding agents can work inside a managed, accountable structure.

The goal is not to make agents smarter by prompt alone. The goal is to move responsibility from the agent into Agent OS wherever normal backend code can do the job more reliably.

In plain language:

```text
The agent writes code.
Agent OS manages the work around the code.
```

These notes use Agent OS-native names. They intentionally avoid borrowing external system names so the concepts are easier to own, discuss, and implement inside Agent OS.

## Assumption

Agent OS is terminal-based. It can start Codex, Claude, shell, or other tools as local terminal/process workers. It does not need direct model API access to implement most of these systems.

Agent OS can reliably manage:

- process start and stop.
- terminal input and output.
- logs.
- timeouts.
- worktrees.
- changed files.
- patches.
- command results.
- evidence records.
- cleanup.
- backend state.

Agent OS cannot perfectly see the model's private internal reasoning unless the terminal tool exposes it. That is acceptable. Agent OS only needs to control and verify the operational facts.

## Two-depth structure

Start here:

- [Overview](./overview.md) - short summary of the systems and why they matter.

Then read individual system notes:

1. [Managed Work Runner](./systems/01-managed-work-runner.md)
2. [Work History Stream](./systems/02-work-history-stream.md)
3. [Stuck Work Guard](./systems/03-stuck-work-guard.md)
4. [Result Inspector](./systems/04-result-inspector.md)
5. [Start Failure Handler](./systems/05-start-failure-handler.md)
6. [Work Instruction Packet](./systems/06-work-instruction-packet.md)
7. [Role Boundary Profiles](./systems/07-role-boundary-profiles.md)
8. [Change Boundary Checker](./systems/08-change-boundary-checker.md)
9. [Repair Pass Loop](./systems/09-repair-pass-loop.md)
10. [Cleanup Doctor](./systems/10-cleanup-doctor.md)
11. [Run Snapshot](./systems/11-run-snapshot.md)
12. [Duplicate Action Guard](./systems/12-duplicate-action-guard.md)
13. [Background Caretaker](./systems/13-background-caretaker.md)
14. [Clear Failure Labels](./systems/14-clear-failure-labels.md)
15. [Local Preview Steward](./systems/15-local-preview-steward.md)
16. [Limit Guard](./systems/16-limit-guard.md)
17. [Check Failure Fixer](./systems/17-check-failure-fixer.md)
18. [Safe Agent Feedback Channel](./systems/18-safe-agent-feedback-channel.md)

## Core principle

Agent OS should not rely on the agent to manage the whole work process.

The agent may say:

```text
I am done.
```

Agent OS should check:

```text
Was a task created?
Was the work run managed?
Was output captured?
Were files changed inside scope?
Was a patch created?
Were required checks run?
Did checks pass?
Was review completed when needed?
Is approval still required?
```

Only backend facts should move important work forward.

## Best first implementation order

1. Managed Work Runner.
2. Work History Stream.
3. Stuck Work Guard.
4. Result Inspector.
5. Change Boundary Checker.
6. Work Instruction Packet.
7. Repair Pass Loop.
8. Cleanup Doctor.

These first systems create the biggest practical improvement because they turn terminal agents into managed workers instead of free-floating terminals.