# 03 - Worker Health Watchdog

## Purpose

The Worker Health Watchdog checks whether managed terminal workers are alive, active, stuck, crashed, or runaway.

It should run without asking an agent what happened.

## Plain-language meaning

Agent OS should notice when a worker is not healthy.

Examples:

```text
The worker went silent.
The process disappeared.
The command has run too long.
The worker ignored stop.
The run says running, but no process exists.
```

## What to monitor

For every active managed run, track:

```text
process id
process group id
started_at
last_activity_at
last_output_at
last_event_at
elapsed time
current status
stop requested or not
configured timeout
configured silence timeout
```

## Watchdog checks

### Check 1 - process exists

If run is `running` but process is gone:

```text
mark crashed or exited_unknown
record event
save log artifact
start result inspection if appropriate
```

### Check 2 - silence timeout

If no output/event for too long:

```text
mark no_recent_activity or stuck
show Stig decision
optionally stop after stricter timeout
```

### Check 3 - maximum runtime

If run exceeds max runtime:

```text
mark timeout
send stop signal
preserve logs
block ready state
```

### Check 4 - stop request

If Stig or system requested stop:

```text
send graceful stop
wait grace period
force kill if needed
record stopped_by_stig or stopped_by_system
```

## Suggested tick loop

Run periodically:

```text
for each active managed run:
  load process info
  compute health
  append warning/failure events
  transition state when needed
  stop process if policy requires
```

This should be single-flight: do not allow two watchdog ticks to overlap.

## Policy settings

Suggested defaults:

```text
silence_warning_after_minutes: 10
stuck_after_minutes: 30
max_runtime_minutes: 90
graceful_stop_seconds: 10
force_kill_after_seconds: 30
```

These should be configurable per project/task later.

## User-facing statuses

Show plain language:

```text
Running normally
No recent activity
Possibly stuck
Stopping
Stopped
Crashed
Timed out
```

## What not to do

Do not depend on the worker saying it is stuck.

Do not leave a dead process marked running.

Do not kill immediately without preserving logs.

Do not silently force-kill without showing Stig what happened.

## Acceptance criteria

- dead process is detected.
- silent worker is detected.
- timeout is enforced.
- stop action kills process group, not only parent process.
- log artifact survives stop/crash.
- UI can explain why the run stopped.