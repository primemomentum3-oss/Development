# 03 - Stuck Work Guard

## Plain meaning

The Stuck Work Guard is the safety monitor for managed coding runs.

It watches for a worker that has gone silent, crashed, frozen, or started running forever.

## What it does

It detects conditions such as:

- no output for too long.
- process exited unexpectedly.
- process still running but no progress is visible.
- command timeout reached.
- run stayed active after the app restarted.

## How it works

Agent OS tracks the latest heartbeat for a run. A heartbeat can be the latest terminal output, latest event, or process state update.

If the heartbeat is too old, Agent OS marks the run as suspicious or failed, saves logs, and can stop the process.

A simple flow:

```text
run is active
Agent OS checks last activity time
last activity is too old
Agent OS records stuck state
Agent OS saves log
Agent OS stops or asks whether to stop
```

## What it requires

- Last-output timestamp.
- Process tracking.
- Timeout settings.
- Safe stop and force-stop behavior.
- Failure event recording.
- Log preservation.

## Who owns it

Agent OS owns this system.

The agent should not be trusted to report that it is stuck. The system should detect it.

## Terminal-based feasibility

Fully feasible with terminals.

Agent OS can watch process status and output silence without needing model API access.

## Why it matters

Coding agents can hang on commands, wait forever, or silently crash. This guard prevents invisible failure and resource waste.