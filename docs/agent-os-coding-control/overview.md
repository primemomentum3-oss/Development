# Managed Coding Control Overview

Date: 2026-06-25
Status: planning overview
Owner: Stig

## Simple summary

Agent OS needs a stronger backend layer around terminal-based coding agents.

The missing idea is not one single feature. It is a group of systems that make sure coding work is started, watched, recorded, checked, repaired, explained, and cleaned up by Agent OS itself.

The agent should not be trusted to be the whole system.

```text
Agent = does the thinking and coding.
Agent OS = owns the process, evidence, state, limits, cleanup, and final truth.
```

## Main gap areas

| Area | Plain name | What it improves |
| --- | --- | --- |
| Starting work | Managed Work Runner | Agent OS starts coding agents as managed workers, not loose terminals. |
| Seeing work live | Work History Stream | Agent OS stores and shows what happened during the run. |
| Preventing stuck work | Stuck Work Guard | Agent OS detects silent, crashed, or runaway runs. |
| Closing work | Result Inspector | Agent OS checks what the agent actually produced. |
| Failed starts | Start Failure Handler | Agent OS records launch failures and cleans up partial state. |
| Instructions | Work Instruction Packet | Agent OS gives the agent a clear job contract. |
| Roles | Role Boundary Profiles | Builder, reviewer, fixer, verifier, and closer each get different boundaries. |
| File scope | Change Boundary Checker | Agent OS checks whether changed files stayed inside the task boundary. |
| Repair | Repair Pass Loop | Agent OS creates narrow repair runs when verification or review fails. |
| Cleanup | Cleanup Doctor | Agent OS finds stale worktrees, orphaned processes, and expired leases. |
| History accuracy | Run Snapshot | Agent OS saves exactly what the agent was given at launch time. |
| Duplicate prevention | Duplicate Action Guard | Agent OS avoids double-click/double-launch mess. |
| Maintenance | Background Caretaker | Agent OS periodically checks for work that needs attention. |
| Failure clarity | Clear Failure Labels | Agent OS explains failures in plain language. |
| App preview | Local Preview Steward | Agent OS can start, track, and stop local previews for app changes. |
| Limits | Limit Guard | Agent OS controls time, silence, changed files, and fix attempts. |
| Failed checks | Check Failure Fixer | Agent OS can create focused repair work from failed tests or CI logs. |
| Agent-to-system messages | Safe Agent Feedback Channel | Agents can report blockers or evidence without gaining dangerous powers. |

## What should move out of the agent

These should be owned by Agent OS backend code, not by the agent's memory or honesty:

- whether a run has started.
- whether a process is alive.
- whether output was captured.
- whether the run is stuck.
- whether files changed outside scope.
- whether a patch exists.
- whether checks actually ran.
- whether checks passed.
- whether a failed start was recorded.
- whether duplicate actions are ignored.
- whether cleanup is needed.
- whether the task is ready, blocked, failed, or waiting for approval.

## What should remain agent work

The agent should still do the work that requires judgment:

- understand the task.
- inspect files.
- write code.
- explain choices.
- fix failures.
- summarize what it attempted.
- report blockers.
- propose next steps.

## Terminal-based feasibility

This can be implemented without direct model API access.

Agent OS can manage terminal agents by controlling the process around them:

```text
start process
send launch instructions
capture output
watch process health
read changed files
run checks
save logs
stop runaway work
inspect results
create repair passes
```

The system may not see the model's hidden internal reasoning, but it can still verify the operational facts that matter.

## Recommended first build target

The first real target should be:

```text
Managed Work Runner + Work History Stream + Stuck Work Guard + Result Inspector
```

Together, these make a terminal agent behave like a managed worker inside Agent OS.