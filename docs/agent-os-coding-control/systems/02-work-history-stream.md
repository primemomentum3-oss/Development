# 02 - Work History Stream

## Plain meaning

The Work History Stream is the live and saved timeline of a managed coding run.

It is not just terminal text. It is the system's record of what happened.

## What it does

It shows and stores events such as:

- worker started.
- instruction packet sent.
- command started.
- command finished.
- output captured.
- file changes detected.
- patch submitted.
- verification failed.
- repair pass started.
- review requested.
- run blocked or completed.

## How it works

Agent OS captures terminal output and system events, then stores them as backend records.

The UI can replay old records and then continue showing new records live.

A simple flow:

```text
terminal output or system action happens
Agent OS records an event
Agent OS stores the event
Agent OS shows the event in the UI
later readers can replay the same history
```

## What it requires

- Event storage connected to each run.
- Sequence numbers or timestamps.
- Output capture from terminal processes.
- A live UI feed.
- A way to replay past events.

## Who owns it

Agent OS owns the stream.

The agent may produce output, but Agent OS decides what is recorded and how it is shown.

## Terminal-based feasibility

Fully feasible with terminals.

Agent OS can capture stdout, stderr, process status, command logs, and its own backend events. It may not see hidden model internals, but it can record the operational facts.

## Why it matters

It turns coding work from a black box into a readable timeline.

Stig and future agents can understand what happened without trusting only the agent's final summary.