# 09 - Repair Pass Loop

## Plain meaning

The Repair Pass Loop creates narrow follow-up work when a coding run fails verification or review.

It prevents Agent OS from starting over from scratch when only a focused fix is needed.

## What it does

It handles cases such as:

- tests failed.
- type check failed.
- review requested changes.
- required evidence is missing.
- patch exists but is not ready.

## How it works

Agent OS creates a repair run with a smaller job than the original builder run.

A simple flow:

```text
verification or review fails
Agent OS records failure reason
Agent OS creates repair pass
repair worker receives failed log and narrow scope
repair worker fixes only that issue
Agent OS reruns verification/review
```

## What it requires

- Failure detection.
- Failed log/evidence capture.
- Repair role profile.
- Maximum repair attempt limit.
- Narrow instruction packet.
- Re-run of relevant checks.

## Who owns it

Agent OS owns the loop and decides when to create repair passes.

A fixer agent performs the repair work.

Stig may approve exceptions or stop the loop.

## Terminal-based feasibility

Fully feasible with terminals.

Agent OS can start a new terminal run with the failed command/log and a narrow instruction packet.

## Why it matters

It gives Agent OS a controlled way to improve failed work without letting agents spiral into broad, unrelated changes.