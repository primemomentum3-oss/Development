# 18 - Safe Agent Feedback Channel

## Plain meaning

The Safe Agent Feedback Channel gives a managed worker a narrow way to report useful information back to Agent OS while it is working.

It is not broad backend control. It is a safe reporting path.

## What it does

A worker could use it to say:

- I found a blocker.
- I need approval to touch a protected file.
- I recorded evidence.
- I finished this step.
- I need more information.
- This check failed and here is the log reference.

## How it works

Agent OS exposes a limited local command or tool that the worker can call.

Allowed actions should be narrow:

```text
record evidence
report blocker
request approval
attach log
mark step attempted
```

Dangerous actions should not be allowed:

```text
mark task done
approve itself
promote memory permanently
push
merge
deploy
change task policy
```

## What it requires

- A safe local command/tool.
- Permission policy.
- Structured message format.
- Audit record for every call.
- Clear list of allowed and blocked actions.

## Who owns it

Agent OS owns and validates the channel.

The agent may use it, but the system decides what to accept.

## Terminal-based feasibility

Feasible with terminals.

The worker can call a local `agent-os` command or MCP tool if allowed. The important part is that the channel must be narrow and audited.

## Why it matters

It lets workers communicate useful structured facts without giving them dangerous control over final task state.