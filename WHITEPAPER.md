# Agent OS Managed Control White Paper

Date: 2026-06-25
Status: top-level white paper
Audience: Stig and future implementation agents

## Scope of this white paper

This white paper explains the structured plans we have created.

It is intentionally limited to the structured-plan layer.

It does not try to explain every detailed file inside the `systems/` folders.

Think of the documentation like this:

```text
WHITEPAPER.md
  = natural-language overview of the structured plans

structured-plans/*/README.md and main guide files
  = the structured implementation plans

structured-plans/*/systems/*
  = deeper subsystem detail for implementation agents
```

This white paper focuses on:

```text
what the structured plans are
why they matter
how they connect
what Agent OS has today
what Agent OS needs next
what the future Agent OS experience should feel like
```

## 1. Why this matters

Agent OS is becoming a system for coordinating AI coding work.

The question is not only:

```text
Can an agent write code?
```

The real question is:

```text
Can Agent OS keep control while agents write code?
```

The structured plans we have created are about giving Agent OS that control.

They are not random notes. They are a connected planning package.

## 2. The simple vision

The long-term vision is:

```text
Stig talks to Agent OS.
Agent OS prepares the work.
Agent OS gives the worker the right context.
Agent OS gives the worker the right role.
Agent OS starts the worker.
Agent OS watches the worker.
Agent OS checks the result.
Agent OS asks Stig only for real decisions.
```

Stig should not have to manage loose Codex or Claude terminals by hand.

The terminal can still exist, but it should become a managed worker room inside Agent OS.

## 3. What Agent OS already has

Agent OS already has many strong foundations:

```text
tasks
event timeline
Git worktrees
file leases
command evidence
verification runs
reliability gates
patch artifacts
review records
memory candidates
approved project memory
reports
agent launch foundations
local process launch foundations
agent run contract foundations
Codex adapter foundations
frontend terminal panes
```

This is a good foundation.

The structured plans do not replace these pieces.

They explain how to connect them into one controlled worker lifecycle.

## 4. The current gap

The weak version of agent work looks like this:

```text
Stig request
   ↓
loose terminal agent
   ↓
agent says it is done
   ↓
Stig tries to understand what happened
```

That creates problems:

```text
the agent may miss context
the agent may use the wrong command
the agent may change too much
the agent may skip verification
the agent may get stuck
the agent may say done without proof
Stig may need to read raw logs manually
```

Agent OS should remove that burden from Stig.

## 5. What the reference system teaches us

The reference system is useful because it shows a control pattern.

The important lesson is not to copy its exact implementation.

The useful lesson is:

```text
Do not trust the agent as final truth.
Put backend control around the agent.
```

The Agent OS version should adapt that pattern to local terminal workers.

## 6. The structured plans we now have

The structured planning package now has four main areas:

```text
1. Managed Worker Conversation Flow
2. Backend Quality Control
3. Project Context Bundle
4. Role Profiles and Worker Contracts
```

These are the structured plans this white paper is about.

The detailed `systems/` files are underneath those plans, but this paper stays at the plan level.

## 7. Visual map of the future system

The future managed Agent OS flow should look like this:

```text
Stig request
   ↓
Work brief
   ↓
Project context bundle
   ↓
Role profile / worker contract
   ↓
Managed worker run
   ↓
Backend quality control
   ↓
Review / repair / approval
```

The important idea is that each layer has a job:

```text
Work brief
  = what should be done

Project context bundle
  = what the worker should know

Role profile / worker contract
  = what the worker may do

Managed worker run
  = how Agent OS starts and watches the worker

Backend quality control
  = how Agent OS checks what actually happened

Review / repair / approval
  = how the result moves forward safely
```

## 8. Structured plan 1: Managed Worker Conversation Flow

This plan explains how Stig should interact with Agent OS.

The key idea:

```text
Stig talks to Agent OS, not directly to a loose terminal.
```

Agent OS should:

```text
understand the request
create or connect a task
prepare the work brief
start the worker in managed mode
show progress in a readable way
surface decisions when needed
```

This improves the experience because Stig no longer needs to manage:

```text
which terminal to use
which prompt to write
which worker is doing what
whether the worker is stuck
what happened during the run
what decision is needed next
```

## 9. Structured plan 2: Backend Quality Control

This plan explains the control layer around the worker.

The key idea:

```text
The worker can write code.
Agent OS must check the work.
```

Backend quality control should answer:

```text
Was the worker allowed to start?
Did it stay alive?
Did it produce real changes?
Were those changes inside scope?
Was required verification actually run?
Did verification pass?
Is review or Stig approval required?
Should this be repaired, blocked, failed, or accepted for review?
```

This plan makes the result trustworthy because the official state comes from Agent OS records, not from the worker saying it is done.

## 10. Structured plan 3: Project Context Bundle

This plan explains what the worker should know before it starts.

The key idea:

```text
Every worker should start with the right approved project knowledge.
```

The context bundle should include things like:

```text
task goal
locked work brief
approved project memory
known project commands
relevant reports
known risks
required checks
file boundaries
```

This improves quality because workers are less likely to:

```text
use the wrong test command
miss existing project conventions
repeat old mistakes
ignore important task history
change files that should not be touched
```

Important rule:

```text
Only approved memory should be trusted context.
```

## 11. Structured plan 4: Role Profiles and Worker Contracts

This plan explains what kind of worker is being launched.

The key idea:

```text
A worker should not be a general agent by default.
A worker should have a role.
```

Examples:

```text
Builder builds.
Reviewer reviews.
Fixer fixes one specific failure.
Verifier runs checks.
Closer prepares the final package.
Stig approves important work.
```

This improves safety because Agent OS can show and enforce:

```text
what the worker may do
what the worker must not do
what proof the worker must produce
what decisions still belong to Stig
```

## 12. How the plans connect

The structured plans are not separate ideas.

They form one lifecycle:

```text
Managed Worker Conversation Flow
  creates the user-facing work path

Project Context Bundle
  prepares what the worker knows

Role Profiles and Worker Contracts
  define what the worker may do

Backend Quality Control
  checks what the worker actually did
```

A simple view:

```text
              ┌────────────────────┐
              │   Stig request      │
              └─────────┬──────────┘
                        ↓
              ┌────────────────────┐
              │   Work brief        │
              └─────────┬──────────┘
                        ↓
        ┌───────────────┴───────────────┐
        ↓                               ↓
┌────────────────────┐        ┌────────────────────┐
│ Context bundle      │        │ Role contract       │
│ what worker knows   │        │ what worker may do  │
└─────────┬──────────┘        └─────────┬──────────┘
          └───────────────┬───────────────┘
                          ↓
                ┌────────────────────┐
                │ Managed worker run  │
                └─────────┬──────────┘
                          ↓
                ┌────────────────────┐
                │ Backend quality     │
                │ control             │
                └─────────┬──────────┘
                          ↓
              Review / repair / approval
```

## 13. What the future frontend should show

The future Agent OS frontend should feel like a control room.

It should show:

```text
what Agent OS understood
what worker is assigned
what context was given
what role contract applies
what the worker is doing
what Agent OS checked
what decision is needed
```

The raw terminal should be available, but secondary.

The main screen should be the official Agent OS status.

## 14. Example: successful work

Stig says:

```text
Fix the login bug.
```

Agent OS prepares:

```text
Task: Fix login bug
Context: login conventions, approved memory, known test command
Role: Builder
Contract: may edit login files, must run required checks, cannot approve itself
```

Worker runs.

Agent OS checks the result.

Frontend shows:

```text
Status: Ready for review
Reason: Patch exists, scope is clean, verification passed.
Next action: Start review.
```

## 15. Example: blocked work

Worker says:

```text
Done.
```

Agent OS checks and finds:

```text
Required verification was not run.
```

Frontend shows:

```text
Status: Blocked
Reason: Required verification was not run.
Next action: Run verification or start a repair/check pass.
```

This is the difference between trusting the worker and controlling the work.

## 16. Example: role violation

A reviewer changes source files.

Agent OS checks the role contract.

Frontend shows:

```text
Status: Blocked
Reason: Reviewer changed files, which this role is not allowed to do.
Next action: Reject review run or start a builder/fixer run.
```

## 17. Example: missing context

A worker uses the wrong command.

Agent OS can show:

```text
The context bundle did not include the correct command.
A memory candidate should be created for review.
```

After approval, future workers can receive better context.

## 18. High-level implementation order

At the structured-plan level, the clean order is:

```text
1. Managed Worker Conversation Flow
2. Backend Quality Control foundation
3. Project Context Bundle
4. Role Profiles and Worker Contracts
5. Frontend control-room views
6. Real Codex/Claude terminal adapters
```

Do not start by depending on Codex or Claude behavior.

Start with a simple controlled worker path so Agent OS proves the lifecycle first.

## 19. The most important product rule

The agent can help.

The backend must decide.

```text
Worker text is not proof.
Backend records are proof.
Stig approval is final for important work.
```

## 20. Where to read next

Use this white paper first.

Then read the structured plans:

```text
structured-plans/agent-os-managed-worker-conversation-plan.md
structured-plans/backend-quality-control/
structured-plans/project-context-bundle/
structured-plans/role-profiles-worker-contracts/
```

The detailed `systems/` folders remain implementation detail.

They are not the scope of this white paper.

This white paper is the clean natural-language overview of the structured plans only.