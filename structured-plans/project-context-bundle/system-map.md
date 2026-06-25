# Project Context Bundle System Map

## Plain-language summary

The Project Context Bundle is the prepared context Agent OS gives to a worker before the worker starts.

It should answer:

```text
What is the task?
What project is this in?
What does the worker need to know?
What rules must it follow?
What checks must it run?
What mistakes should it avoid?
What context is approved and trustworthy?
```

## Core principle

The worker should not start from memory, vibes, or a raw prompt.

It should start from a controlled bundle assembled by Agent OS.

```text
Stig request
  ↓
Task and work brief
  ↓
Approved project context
  ↓
Role-specific context
  ↓
Frozen context bundle
  ↓
Managed worker run
```

## What goes into the bundle

A good context bundle includes:

```text
original Stig request
task title and goal
locked work brief
role contract
approved durable project memory
relevant memory candidates if explicitly allowed
known project commands
required verification
recent related failures
current task history
relevant reports
changed-file boundaries
approval requirements
```

## What should not go into the bundle

Avoid:

```text
unapproved memory as fact
old stale notes without timestamps
huge raw logs unless linked as artifacts
irrelevant previous tasks
agent guesses
contradictory context without warning
```

## Backend systems and jobs

| System | Job |
| --- | --- |
| Context Sources and Precedence | Defines where context comes from and which source wins. |
| Context Bundle Builder | Builds the bundle for a run. |
| Approved Memory Gate | Ensures only approved memory becomes trusted context. |
| Run Snapshot and Context Artifact | Saves exactly what was sent to the worker. |
| Context Relevance and Size Control | Keeps the bundle useful instead of bloated. |
| Context Feedback and Memory Candidates | Turns repeated lessons into reviewable memory candidates. |

## How it improves Agent OS

Current Agent OS already has task records, memory candidates, durable memory promotion, reports, command evidence, verification, and patch artifacts.

The improvement is to assemble those into a clean launch-time context bundle automatically.

So instead of:

```text
Agent starts with whatever prompt it got.
```

Agent OS creates:

```text
This is the task.
These are the project rules.
These are the relevant approved memories.
These are the checks.
These are the boundaries.
These are the risks.
```

## First implementation target

Do not build a complex retrieval system first.

Start with a simple deterministic bundle:

```text
task
locked work brief
role contract
project verification command
approved project memory
recent task report links
required proof
```

Then improve relevance scoring later.

## Success example

Stig asks for a login fix.

Agent OS creates a context bundle that says:

```text
Goal: Fix login bug.
Project command: npm test -- login.
Relevant memory: login state is handled in auth/session-store.ts.
Known risk: do not edit package.json without approval.
Required proof: run login tests and save command evidence.
```

The worker starts with this instead of guessing.