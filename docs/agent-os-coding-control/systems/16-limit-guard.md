# 16 - Limit Guard

## Plain meaning

The Limit Guard prevents managed coding work from running too long, changing too much, or retrying forever.

It is a safety boundary around local terminal workers.

## What it does

It can enforce limits such as:

- maximum run time.
- maximum silent time.
- maximum command runtime.
- maximum repair attempts.
- maximum changed files.
- protected files that need approval.
- optional budget/cost limits if the provider reports cost.

## How it works

Agent OS watches counters and timers while work is running and during result inspection.

A simple flow:

```text
run starts with limits
Agent OS tracks time, output, commands, and changed files
limit is exceeded
Agent OS records limit event
Agent OS blocks, stops, or requests approval
```

## What it requires

- Limit settings.
- Run timers.
- Last-output tracking.
- Changed-file counts.
- Repair attempt counts.
- Approval path for exceptions.

## Who owns it

Agent OS owns and enforces limits.

Stig may approve exceptions.

The agent should not be trusted to self-limit reliably.

## Terminal-based feasibility

Mostly feasible with terminals.

Time, silence, file, command, and retry limits are fully feasible. Token/cost limits are only available when the terminal provider reports them.

## Why it matters

It prevents runaway work, endless repair loops, and oversized changes.