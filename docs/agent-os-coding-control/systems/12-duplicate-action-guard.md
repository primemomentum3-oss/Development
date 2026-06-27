# 12 - Duplicate Action Guard

## Plain meaning

The Duplicate Action Guard prevents the same important action from happening twice by accident.

It protects against double-clicks, retries, or repeated requests that should only create one result.

## What it does

It prevents duplicates such as:

- two tasks for the same request.
- two builder runs from one click.
- two review requests.
- two patch submissions.
- two memory candidates.
- two cleanup actions.

## How it works

Agent OS stores a unique key for important actions. If the same action arrives again, Agent OS returns the existing result instead of doing it again.

A simple flow:

```text
request arrives
Agent OS checks action key
same action already exists
return existing result
no duplicate side effect
```

## What it requires

- Action key or request key.
- Stored request/result record.
- Conflict handling when the same key has different content.
- UI behavior that can safely retry.

## Who owns it

Agent OS owns this system.

Users and agents do not need to manage it manually.

## Terminal-based feasibility

Fully feasible with terminals.

This is backend bookkeeping and does not depend on model API access.

## Why it matters

It prevents accidental mess when the UI, CLI, or an agent retries an operation.