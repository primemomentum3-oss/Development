# 11 - Run Snapshot

## Plain meaning

The Run Snapshot saves exactly what Agent OS gave to a worker at the moment the run started.

It is the historical record of the launch context.

## What it does

It answers questions such as:

- What task did the worker receive?
- What role did it have?
- What provider/tool was used?
- What worktree was used?
- What checks were required?
- What files were allowed?
- What memory/context was included?
- What limits were active?

## How it works

When Agent OS starts a run, it freezes the launch packet and related settings.

A simple flow:

```text
prepare launch packet
resolve role profile
resolve project context
resolve required checks
resolve limits
save snapshot
start worker
```

## What it requires

- Launch packet storage.
- Role/profile versioning.
- Provider/tool setting record.
- Context/memory references.
- File scope and verification requirements.

## Who owns it

Agent OS owns and stores the snapshot.

The agent uses the instructions, but should not be able to rewrite the historical snapshot afterward.

## Terminal-based feasibility

Fully feasible with terminals.

The snapshot is created before process launch and stored by Agent OS.

## Why it matters

Later, if a run behaves strangely, Stig and future agents can inspect exactly what the worker was told at the time.