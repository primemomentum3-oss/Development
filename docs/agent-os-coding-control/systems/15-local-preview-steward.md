# 15 - Local Preview Steward

## Plain meaning

The Local Preview Steward starts and manages local app previews for coding tasks.

It is useful when a task changes a web app, desktop UI, or other project with a visible local preview.

## What it does

It can:

- start a local dev server inside the task worktree.
- choose or record the preview port.
- show an `open preview` action.
- capture preview logs.
- stop stale previews.
- explain preview startup failures.

## How it works

Agent OS uses the project's known preview command and runs it as a managed process.

A simple flow:

```text
patch is ready or user requests preview
Agent OS starts preview command in task worktree
Agent OS records process and port
Stig opens preview
Agent OS captures logs
Agent OS stops preview when stale or requested
```

## What it requires

- Project preview command.
- Port tracking.
- Process tracking.
- Preview log artifact.
- Stop/cleanup behavior.
- UI action to open preview.

## Who owns it

Agent OS owns the preview process.

Agents may request a preview or use its logs, but should not be the only thing managing it.

## Terminal-based feasibility

Fully feasible with terminals.

A preview is just another local process with logs, ports, and lifecycle state.

## Why it matters

It lets Stig see the result of app changes, not only read reports.