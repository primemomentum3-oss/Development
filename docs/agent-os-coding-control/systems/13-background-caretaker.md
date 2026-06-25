# 13 - Background Caretaker

## Plain meaning

The Background Caretaker is a safe maintenance loop inside Agent OS.

It periodically checks whether any managed work needs attention.

## What it does

It can look for:

- runs that have been active too long.
- tasks waiting for approval.
- failed verification needing repair.
- stale worktrees.
- expired leases.
- missing reports.
- preview servers that should be stopped.
- cleanup doctor warnings.

## How it works

Agent OS runs a scheduled check every so often. It should avoid overlapping with itself, so a slow check does not start another copy of the same check.

A simple flow:

```text
timer fires
Agent OS checks managed state
records findings
starts safe follow-up only when allowed
shows warnings when human decision is needed
```

## What it requires

- Timer or scheduler.
- Single-flight behavior.
- Safe scan functions.
- Event records for findings.
- Rules for what can happen automatically.
- Rules for what needs Stig approval.

## Who owns it

Agent OS owns this system.

Agents should not be responsible for remembering maintenance work.

## Terminal-based feasibility

Fully feasible with terminals.

This is backend scheduling and state inspection.

## Why it matters

It makes Agent OS proactive instead of waiting for Stig or an agent to notice problems.