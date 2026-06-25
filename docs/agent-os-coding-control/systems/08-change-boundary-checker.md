# 08 - Change Boundary Checker

## Plain meaning

The Change Boundary Checker verifies whether a coding worker changed only what it was allowed to change.

It compares the actual file changes against the task boundary.

## What it does

It catches issues such as:

- unrelated files changed.
- protected files changed.
- package or dependency files changed without approval.
- changes made in the wrong checkout.
- work done without an approved worktree.
- no patch created even though files changed.

## How it works

After or during a run, Agent OS checks Git/worktree state.

A simple flow:

```text
read task allowed paths
read task forbidden paths
read changed files
compare actual changes to allowed boundary
record accepted or blocked result
ask Stig if an exception is needed
```

## What it requires

- Allowed path rules.
- Forbidden path rules.
- Worktree state access.
- Changed-file detection.
- Exception/approval path.
- Clear result shown to Stig.

## Who owns it

Agent OS owns this system.

The agent can request broader scope, but Agent OS should decide whether the change is allowed.

## Terminal-based feasibility

Fully feasible with terminals.

This is mostly Git/file-system inspection. It does not require model API access.

## Why it matters

It stops accidental broad changes and keeps coding work tied to the original task.