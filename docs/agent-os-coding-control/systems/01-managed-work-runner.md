# 01 - Managed Work Runner

## Plain meaning

The Managed Work Runner is the part of Agent OS that starts, watches, and owns a coding run.

It turns Codex, Claude, or a shell worker from a loose terminal into a managed worker connected to a task.

## What it does

When Stig asks for coding work, Agent OS should not only open a terminal. It should create a real backend run and then start the terminal worker inside that run.

It should know:

- which task the worker belongs to.
- which project and worktree it is using.
- which role it is performing.
- when it started.
- whether the process is still alive.
- when it exited.
- where its logs are.

## How it works

Agent OS creates backend records first, then starts the process.

A simple flow:

```text
create task if needed
create run record
create or attach worktree
prepare work instruction packet
start terminal process
capture output
track process status
hand off to result inspection when done
```

## What it requires

- A run table or run record.
- Process start and stop support.
- PID or process group tracking.
- Output capture.
- Connection to task, project, role, and worktree.
- A clear managed vs clean/unmanaged terminal distinction.

## Who owns it

Agent OS owns this system.

The agent works inside it. The agent should not decide whether the run exists, whether it is managed, or whether it is final.

## Terminal-based feasibility

Fully feasible with terminals.

Agent OS can start the Codex or Claude CLI as a child process, send instructions, capture output, and watch process state. Direct model API access is not required.

## Why it matters

This is the foundation for everything else.

Without a Managed Work Runner, Agent OS is mostly observing a terminal. With it, Agent OS owns the coding lifecycle.