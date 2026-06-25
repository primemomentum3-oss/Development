# 01 - Lifecycle State Machine

## Purpose

The Lifecycle State Machine is the official backend truth for a managed worker run.

It prevents random code, terminal output, or agent claims from moving work into unsafe states.

## Plain-language meaning

Agent OS should always know exactly where a run is:

```text
Preparing
Starting
Running
Stuck
Crashed
Inspecting
Blocked
Needs repair
Ready for review
Ready for approval
Done
```

The worker cannot choose these states. Agent OS chooses them based on backend facts.

## Why this is needed

Without a strict state machine:

- a worker can say it is done without proof.
- a failed launch can look like an active run.
- a crashed process can stay marked running.
- review can be skipped by accident.
- approval can be confused with completion.

## Required states

Use exact or equivalent states:

```text
preparing
starting
running
stopping
stuck
crashed
failed_before_start
exited
inspecting
blocked
needs_repair
ready_for_review
ready_for_approval
done
cancelled
```

## Allowed transitions

Recommended legal transitions:

```text
preparing -> starting
starting -> running
starting -> failed_before_start
running -> stopping
running -> stuck
running -> crashed
running -> exited
stopping -> exited
stuck -> stopping
stuck -> running
exited -> inspecting
inspecting -> blocked
inspecting -> needs_repair
inspecting -> ready_for_review
inspecting -> ready_for_approval
ready_for_review -> ready_for_approval
ready_for_approval -> done
blocked -> needs_repair
blocked -> cancelled
needs_repair -> preparing
```

## Hard rules

```text
The worker cannot set done.
The worker cannot set ready_for_approval.
The worker cannot bypass inspecting.
The worker cannot erase a failed_before_start state.
The worker cannot convert blocked to ready.
```

## Implementation shape

Create one helper for state transitions.

Suggested function:

```text
transitionManagedRunState({ runId, from, to, reason, actor })
```

This function should:

- load current run state.
- verify the transition is allowed.
- write the new state.
- append a run event.
- reject illegal transitions.

## Actors

Track who or what caused the transition:

```text
system
worker_process
watchdog
result_inspector
stig
repair_loop
cleanup_doctor
```

## Acceptance criteria

- illegal state transitions fail loudly.
- every transition creates an event.
- done can only happen through the approved human/final approval path.
- crashed, stuck, failed, and blocked states cannot be silently overwritten.
- result inspection is required before ready states.