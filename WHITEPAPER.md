# Agent OS Managed Control White Paper

Date: 2026-06-25
Status: top-level white paper
Audience: Stig and future implementation agents

## 1. Why this white paper exists

Agent OS is becoming a system for coordinating AI coding work.

The hard part is no longer only:

```text
Can an agent write code?
```

The harder question is:

```text
Can Agent OS keep control while agents write code?
```

This white paper explains the full picture in natural language.

It connects the structured plans already created in this repository and explains how they fit together as one system.

The goal is that Stig can understand:

```text
What Agent OS already has.
What is missing.
What the reference system teaches us.
What new Agent OS systems we need.
How the new systems work together.
What the result should feel like from Stig's point of view.
```

## 2. The simple vision

The long-term product vision is:

```text
Stig talks to Agent OS.
Agent OS prepares the work.
Agent OS starts the right worker.
Agent OS gives the worker the right context and contract.
Agent OS watches the worker.
Agent OS checks the result.
Agent OS asks Stig for decisions only when needed.
```

Stig should not need to manage loose Codex or Claude terminals by hand.

Terminals can still exist, but they should become managed worker rooms inside Agent OS.

## 3. Current Agent OS, in simple terms

Agent OS already has many strong foundations.

It already has concepts like:

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
```

That means Agent OS already has many of the building blocks needed for trustworthy agent work.

The issue is that these pieces need to be connected into one controlled runtime loop around terminal-based workers.

## 4. The current weakness

The current risk is not that Agent OS has no proof system.

The current risk is that the worker can still feel too much like a loose terminal.

The weak version looks like this:

```text
Stig request
   ↓
Terminal agent
   ↓
Agent says it is done
   ↓
Stig tries to understand what happened
```

The problem with that is:

```text
The agent may miss context.
The agent may change too much.
The agent may skip verification.
The agent may get stuck.
The agent may say done without proof.
Stig may have to read logs manually.
```

Agent OS should remove that burden from Stig.

## 5. What the reference system teaches us

The reference system is useful because it shows a strong control pattern.

The important lesson is not to copy its exact API or runtime design.

The useful lesson is:

```text
Do not trust the agent as final truth.
Put backend control around the agent.
```

That pattern looks like this:

```text
Create a run record before the agent starts.
Give the agent a clear workspace and contract.
Run the agent in a controlled place.
Capture what happens.
Run a backend closing pass after the agent exits.
Check whether real work was produced.
Detect fake success.
Use repair loops only from specific failures.
Clean up after results are extracted.
```

Agent OS should adapt this pattern to local terminal workers.

## 6. The Agent OS version

Agent OS should not become an API-only agent runner.

Agent OS should be a terminal-based control system.

That means:

```text
Agent OS starts local terminal workers.
Agent OS writes or injects the work brief and context.
Agent OS captures process output.
Agent OS watches process health.
Agent OS checks files, patches, verification, review, and approval.
```

The terminal worker can be Codex, Claude, a shell test worker, or another local tool.

But the backend control loop should be the same.

## 7. Current system vs new system

### Today: pieces exist, but control is still spread out

```text
Tasks         Worktrees       Evidence       Patches       Reviews
  │              │              │              │             │
  └────── useful foundations exist, but worker control can feel fragmented
```

### Target: one managed work loop

```text
Stig request
   ↓
Work brief
   ↓
Context bundle
   ↓
Role contract
   ↓
Managed worker run
   ↓
Backend quality control
   ↓
Review / repair / approval
```

The new system does not replace Agent OS foundations.

It connects them.

## 8. The four major new system groups

The current structured plans now describe four major system groups.

```text
1. Managed Worker Conversation Flow
2. Backend Quality Control
3. Project Context Bundle
4. Role Profiles and Worker Contracts
```

Together, they turn terminal agents into controlled Agent OS workers.

## 9. System group 1: Managed Worker Conversation Flow

This is the user-facing flow.

It means Stig talks to Agent OS, not directly to terminals.

```text
Stig says what he wants
   ↓
Agent OS understands and prepares the work
   ↓
Agent OS creates a work brief
   ↓
Agent OS starts the worker terminal
   ↓
Agent OS watches and reports back
```

### What it improves

It reduces the need for Stig to manually manage:

```text
which terminal to use
what prompt to write
which agent is doing what
whether the worker is stuck
what happened during the run
what decision is needed next
```

### What the frontend should feel like

Stig should see:

```text
Task: Fix login bug
Worker: Codex Builder
Mode: Managed
Status: Running
Last activity: 8 seconds ago
Next action: none yet
```

The terminal remains available, but it becomes secondary.

## 10. System group 2: Backend Quality Control

This is the non-agent control layer.

It is actual backend code, not a prompt.

It checks what happened before, during, and after the worker run.

```text
Before run: Is it safe to start?
During run: Is the worker alive and healthy?
After run: Did it produce valid work with proof?
```

### What it checks

```text
run state
process status
log artifact
changed files
scope boundaries
patch artifact
command evidence
verification result
review requirement
approval requirement
cleanup needs
```

### Why it matters

The worker can say:

```text
I am done.
```

Agent OS should answer:

```text
Maybe. I will inspect the facts.
```

The official state should come from backend records, not agent confidence.

## 11. System group 3: Project Context Bundle

This system gives every worker the right project knowledge before it starts.

It prevents agents from starting blind.

```text
Task goal
Approved memory
Project commands
Relevant reports
Known risks
Required checks
File boundaries
```

### What it improves

Without this, workers may:

```text
use the wrong test command
miss existing project conventions
repeat old mistakes
ignore important task history
change files that should not be touched
```

With this, each worker starts with a frozen context bundle.

### Important rule

Only approved memory should be treated as trusted context.

Memory candidates can be useful, but they should not silently become truth.

## 12. System group 4: Role Profiles and Worker Contracts

This system makes every worker narrow and understandable.

A worker should not just be a general agent.

It should have a role.

```text
Builder builds.
Reviewer reviews.
Fixer fixes one failure.
Verifier runs checks.
Closer prepares the final package.
Stig approves important work.
```

### What it improves

Without roles:

```text
fixers refactor too much
reviewers edit code
builders approve themselves
workers skip proof
agents broaden scope without permission
```

With roles:

```text
Agent OS knows what each worker is allowed to do.
Agent OS can block role violations.
Stig can see what the worker was allowed to do.
```

## 13. How the four systems work together

The complete flow should look like this:

```text
                ┌────────────────────┐
                │   Stig request      │
                └─────────┬──────────┘
                          ↓
                ┌────────────────────┐
                │   Work brief        │
                └─────────┬──────────┘
                          ↓
        ┌─────────────────┴─────────────────┐
        ↓                                   ↓
┌────────────────────┐            ┌────────────────────┐
│ Context bundle      │            │ Role contract       │
│ What worker knows   │            │ What worker may do  │
└─────────┬──────────┘            └─────────┬──────────┘
          └─────────────────┬─────────────────┘
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
          ┌─────────────────┴─────────────────┐
          ↓                                   ↓
┌────────────────────┐            ┌────────────────────┐
│ Repair / review     │            │ Stig decision       │
│ if needed           │            │ if needed           │
└────────────────────┘            └────────────────────┘
```

## 14. What each system contributes

### Work brief

Turns Stig's request into a clear job.

```text
What should be done?
What is success?
What proof is required?
```

### Context bundle

Gives the worker project knowledge.

```text
What should the worker know before starting?
```

### Role contract

Defines the worker boundary.

```text
What is this worker allowed to do?
What must it not do?
```

### Managed worker run

Starts and tracks the terminal worker.

```text
What process is running?
Is it alive?
What output is captured?
```

### Backend quality control

Checks the real outcome.

```text
Was there a valid patch?
Were files inside scope?
Did verification pass?
Is review or approval needed?
```

## 15. The new frontend experience

The frontend should feel like a control room.

Stig should not see raw terminal output first.

Stig should first see:

```text
What Agent OS understood
What worker is assigned
What context was given
What role contract applies
What the worker is doing
What Agent OS checked
What decision is needed
```

### Main panels

```text
1. Request / task panel
2. Work brief preview
3. Context bundle panel
4. Role contract panel
5. Managed run status card
6. Backend inspection panel
7. Decision panel
8. Raw terminal/logs as secondary detail
```

## 16. Example: normal successful work

Stig says:

```text
Fix the login bug.
```

Agent OS creates:

```text
Task: Fix login bug
Role: Builder
Context: login conventions, approved memory, known test command
Contract: may edit login files, must run required checks, cannot approve itself
```

Worker runs.

Agent OS checks:

```text
Patch exists.
Files are inside scope.
Required verification passed.
Review is required.
```

Frontend shows:

```text
Status: Ready for review
Reason: Patch exists, scope is clean, verification passed.
Next action: Start review.
```

## 17. Example: blocked work

Worker says:

```text
Done.
```

Agent OS checks and finds:

```text
No verification evidence exists.
```

Frontend shows:

```text
Status: Blocked
Reason: Required verification was not run.
Next action: Run verification or start a repair/check pass.
```

This is the difference between trusting the worker and controlling the work.

## 18. Example: role violation

A reviewer changes source files.

Agent OS checks the role contract.

Frontend shows:

```text
Status: Blocked
Reason: Reviewer changed source files, which this role is not allowed to do.
Next action: Reject review run or start a builder/fixer run.
```

## 19. Example: bad context

A worker used the wrong test command.

Agent OS can show:

```text
The context bundle did not include the correct command.
A memory candidate was created:
Use npm test -- login for login-related tasks.
```

Stig or review can later approve that memory.

The next worker gets better context.

## 20. What Agent OS does well today

Agent OS already has strong foundations:

```text
task records
worktrees
file leases
command evidence
verification
patch artifacts
review records
approval concepts
memory promotion
reports
launch/process foundations
frontend terminal panes
```

This is a strong base.

The new system should not replace those.

It should connect them into one reliable worker lifecycle.

## 21. What Agent OS needs to get better at

Agent OS needs stronger runtime control around terminal workers:

```text
start workers as managed runs
build context before launch
lock role contracts before launch
watch workers during execution
inspect results after execution
block fake success
explain failures clearly
create narrow repair passes
show Stig the official state in the UI
```

## 22. The implementation order

The clean order is:

```text
1. Managed worker records
2. Managed worker state machine
3. Preflight and launch rollback
4. Simple shell/test worker launch
5. Project context bundle
6. Role profile and worker contract
7. Output capture and log artifacts
8. Result inspector
9. Patch/scope checker
10. Verification evidence gate
11. Failure classifier
12. Watchdog
13. Repair loop
14. Cleanup doctor
15. Frontend result views
16. Codex/Claude adapters
```

Do not start with Codex or Claude.

Start with a simple shell/test worker so the backend control loop can be proven without model behavior.

## 23. The most important product rule

The agent can help.

The backend must decide.

```text
Worker text is not proof.
Backend records are proof.
Stig approval is final for important work.
```

## 24. Final picture

The future Agent OS should feel like this:

```text
I tell Agent OS what I want.
Agent OS creates the job.
Agent OS gives the worker the right context.
Agent OS gives the worker the right role.
Agent OS runs the worker in a controlled way.
Agent OS checks the work.
Agent OS tells me what happened.
Agent OS asks me only for real decisions.
```

That is how Agent OS becomes less chaotic, more trustworthy, and easier for Stig to control.

## 25. Where to read next

Main structured plan folders:

```text
structured-plans/backend-quality-control/
structured-plans/project-context-bundle/
structured-plans/role-profiles-worker-contracts/
structured-plans/agent-os-managed-worker-conversation-plan.md
```

The detailed `systems/` folders are for implementation agents.

This white paper is the top-level explanation for Stig and for anyone who needs to understand the complete system before going into implementation details.