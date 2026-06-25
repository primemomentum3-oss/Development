# Structured Plan - Agent OS Managed Worker Conversation Flow

Date: 2026-06-25
Status: structured implementation plan
Owner: Stig

## 1. Plain-language goal

Agent OS should become the place Stig talks to.

Stig should not normally need to type directly into Codex, Claude, or shell worker terminals. Those terminals should still exist, but they should become managed worker terminals controlled by Agent OS.

The simple target is:

```text
Stig talks to Agent OS.
Agent OS prepares the work.
Agent OS starts the right worker terminal.
Agent OS sends the worker a clear work brief.
The worker does the coding.
Agent OS watches, records, checks, and reports back.
```

This creates one clear control surface instead of many loose terminals.

## 2. Why this matters

The current problem is control and visibility.

When Stig has to talk directly into terminals, he has to remember:

- which terminal is doing what.
- which task the terminal belongs to.
- whether the terminal is managed or manual.
- whether the agent has the right instructions.
- whether output was captured.
- whether the work is verified.
- whether the agent is stuck.
- whether the work is actually ready.

That is too much for Stig to manage manually.

Agent OS should take that burden.

The coding worker should not be the system. The coding worker should be one worker inside the system.

## 3. The new mental model

Use these names inside Agent OS.

| Name | Plain meaning |
| --- | --- |
| Stig Request | What Stig says in normal language. |
| Work Brief Builder | A specialist step that turns Stig's request into a clear worker brief. |
| Safety Gate | Agent OS checks whether the brief is safe, scoped, and complete. |
| Managed Work Runner | Agent OS starts and controls the worker terminal. |
| Worker Terminal | The Codex, Claude, shell, or other terminal process doing the work. |
| Work History Stream | The live and saved timeline of what happened. |
| Result Inspector | Agent OS checks what the worker actually produced. |
| Decision Panel | Where Stig approves, rejects, asks for changes, or gives more information. |

The important split:

```text
The worker can suggest and code.
Agent OS owns the process and truth.
Stig owns final product decisions.
```

## 4. End-to-end user flow

### Step 1 - Stig talks to Agent OS

Stig writes in one main Agent OS input box:

```text
Fix the login bug where the user gets stuck after clicking sign in.
```

This should not go directly to the coding worker.

It first becomes a Stig Request inside Agent OS.

### Step 2 - Agent OS classifies the request

Agent OS decides what kind of work this is.

Example classification:

```text
Work class: implementation
Needs worktree: yes
Needs patch: yes
Needs verification: yes
Needs review: yes
Needs Stig approval: yes
```

This classification should be visible in simple language:

```text
This looks like code-change work.
Agent OS will create a managed coding task.
```

### Step 3 - Work Brief Builder prepares a worker brief

Agent OS starts a specialist step called Work Brief Builder.

This can be an LLM worker, but it is not the coding worker.

Its job is to write a clean work brief for the coding worker.

It should produce something like:

```text
Task goal:
Fix the login bug where users get stuck after clicking sign in.

Role:
Builder.

Allowed work:
Inspect and edit login-related source files and related tests.

Not allowed:
Do not change unrelated authentication architecture.
Do not add dependencies without approval.
Do not deploy or push.
Do not mark the task done.

Required proof:
Run the relevant test command.
Save command output.
Submit a patch artifact.
Request review.

Success means:
The login bug is fixed, relevant checks pass, and the work is ready for review.
```

### Step 4 - Agent OS checks the brief

The Work Brief Builder is helpful, but it is not the final authority.

Agent OS checks the brief before giving it to the coding worker.

The Safety Gate checks:

- Does the brief match Stig's request?
- Is the scope clear?
- Are required checks included?
- Are dangerous actions forbidden?
- Does it mention that final approval belongs to Stig?
- Does it avoid giving the worker too much freedom?
- Does it need Stig confirmation before launch?

If the brief is safe, Agent OS locks it as the final worker instruction.

If the brief is unclear, Agent OS asks Stig a simple question or sends it back to the Work Brief Builder.

### Step 5 - Agent OS starts the managed worker terminal

Agent OS creates a managed run record and starts the coding worker terminal.

This worker terminal could be:

- Codex CLI.
- Claude CLI.
- shell worker.
- another local agent tool.

But Stig should not have to manage that terminal directly.

Agent OS sends the locked work brief into the terminal.

### Step 6 - Agent OS watches the worker

While the worker runs, Agent OS shows a live state:

```text
Task: Fix login bug
Worker: Codex Builder
Mode: Managed
Status: Running
Last activity: 8 seconds ago
Current phase: working
```

Agent OS captures:

- terminal output.
- command starts and finishes.
- process status.
- logs.
- file changes.
- evidence.
- failures.

### Step 7 - Worker finishes or gets stopped

The worker may finish normally, fail, crash, or get stuck.

Agent OS should not trust the worker's final words alone.

When the terminal exits, Agent OS runs the Result Inspector.

### Step 8 - Result Inspector checks the output

The Result Inspector checks:

- Did files change?
- Were changed files allowed?
- Was a patch created?
- Were required checks run?
- Did checks pass?
- Is review required?
- Is approval required?
- Is a repair pass needed?

Then Agent OS shows Stig a plain result:

```text
The builder made code changes.
Relevant tests passed.
Patch is ready.
Review is required next.
```

Or:

```text
The builder changed files outside the allowed area.
Agent OS blocked the run from being marked ready.
Stig approval is needed to continue with this wider scope.
```

### Step 9 - Stig uses Agent OS decisions, not terminal typing

Stig should use buttons or natural language inside Agent OS:

- approve.
- request changes.
- ask for explanation.
- start repair pass.
- stop task.
- allow broader scope.
- reject result.

The coding terminal remains inspectable, but not the main control surface.

## 5. Frontend plan

### 5.1 Main Agent OS input

The UI needs one main input area where Stig speaks to Agent OS.

This input should feel like:

```text
Tell Agent OS what you want done.
```

Examples:

- Fix the login bug.
- Review why onboarding is confusing.
- Improve the dashboard loading state.
- Run a controlled investigation of the test failures.

### 5.2 Request review card

After Stig submits a request, show a simple card before launching work.

Card fields:

```text
Request understood as: code-change work
Task title: Fix login bug after sign in
Work mode: Managed coding run
Worker role: Builder
Needs review: yes
Needs Stig approval before final done: yes
```

Possible actions:

- Start managed run.
- Edit brief.
- Ask Agent OS to refine brief.
- Cancel.

### 5.3 Work brief preview

Before launch, Stig should be able to open the generated brief.

The UI should show it in plain sections:

```text
Goal
Allowed work
Not allowed
Required proof
Success means
Approval rules
```

Stig should not need to read a giant raw prompt unless he opens advanced view.

### 5.4 Managed run panel

Once work starts, the main panel should show:

```text
Task name
Worker role
Provider/tool
Managed mode badge
Status
Last activity
Current step
Elapsed time
Stop button
Open worker terminal button
Open logs button
```

The terminal should be visible if needed, but secondary.

### 5.5 Work history timeline

Show a timeline beside or below the run:

```text
10:41 Task created
10:42 Work brief locked
10:42 Builder started
10:43 Output captured
10:45 Files changed
10:47 Test command started
10:48 Test command failed
10:51 Repair pass suggested
```

The timeline should be readable to Stig.

### 5.6 Worker terminal view

The worker terminal should still be available.

But it should be clearly labeled:

```text
Managed Worker Terminal
Typing directly here may switch this run into guided manual mode.
```

This prevents confusion between normal manual terminal work and managed Agent OS work.

### 5.7 Decision panel

When work needs Stig, show a simple decision panel:

```text
Agent OS needs your decision

Reason:
The worker needs to edit package.json, which is outside the original scope.

Options:
Allow this change
Ask worker to avoid package.json
Stop task
Ask for explanation
```

### 5.8 Status language

Use plain statuses:

- Drafting work brief.
- Waiting for Stig confirmation.
- Starting worker.
- Running.
- No recent activity.
- Stuck.
- Failed before start.
- Worker crashed.
- Inspecting result.
- Needs repair.
- Waiting for review.
- Ready for Stig approval.
- Done.

Avoid vague status like only `failed` or `running`.

## 6. Backend plan

### 6.1 Core backend services

The backend should be split into clear services.

#### Request Intake Service

Owns the first request from Stig.

Responsibilities:

- create or attach task.
- classify the work.
- decide whether this needs managed coding work.
- store original Stig request.

#### Work Brief Builder Service

Turns the Stig request into a worker-ready brief.

Responsibilities:

- gather task context.
- gather project context.
- gather role profile.
- gather approved memory if relevant.
- generate structured brief.
- return a draft brief, not final authority.

#### Brief Safety Gate Service

Checks whether the draft brief is acceptable.

Responsibilities:

- require goal.
- require role.
- require allowed/not-allowed boundaries.
- require proof expectations.
- block missing approval language.
- block dangerous permissions.
- produce clear error or approved result.

#### Managed Work Runner Service

Starts and owns the worker terminal.

Responsibilities:

- create managed run record.
- start worker process.
- send locked brief.
- capture output.
- track PID/process group.
- stop/cancel process.
- hand result to Result Inspector.

#### Work History Service

Records what happens.

Responsibilities:

- append timeline events.
- store terminal output references.
- support live UI updates.
- support replay after refresh/restart.

#### Stuck Work Guard Service

Watches active runs.

Responsibilities:

- detect no-output timeout.
- detect crashed process.
- detect process gone while run says active.
- save logs.
- mark stuck/failed.

#### Result Inspector Service

Checks what the worker produced.

Responsibilities:

- inspect changed files.
- verify patch exists.
- verify required command evidence.
- detect out-of-scope changes.
- choose official run result.
- suggest repair/review/approval next step.

### 6.2 Conceptual backend records

These are conceptual. Exact schema can be designed later.

#### Stig Request

```text
id
project_id
raw_text
created_at
source: desktop | cli | mcp
status: received | classified | converted_to_task | cancelled
```

#### Work Brief

```text
id
task_id
request_id
status: draft | needs_revision | locked | rejected
role
summary
goal
allowed_work
not_allowed
required_proof
success_definition
approval_rules
created_by: work_brief_builder | manual | system
locked_at
```

#### Managed Run

```text
id
task_id
work_brief_id
role
provider_tool: codex_cli | claude_cli | shell | other
mode: managed | guided_manual | clean
status: preparing | starting | running | stuck | failed | inspecting | ready_for_review | ready_for_approval | done
process_id
process_group_id
started_at
ended_at
last_activity_at
failure_label
failure_message
```

#### Run Event

```text
id
managed_run_id
sequence
kind
summary
payload
created_at
```

Examples of event kinds:

```text
request.received
brief.drafted
brief.locked
worker.starting
worker.started
terminal.output
command.detected
worker.no_activity_warning
worker.exited
result.inspection_started
result.patch_missing
result.out_of_scope_change
result.ready_for_review
```

#### Worker Log Artifact

```text
id
managed_run_id
path_or_storage_uri
summary
size
created_at
```

#### Decision Request

```text
id
task_id
managed_run_id
reason
options
status: waiting | answered | cancelled
answered_by
answered_at
```

### 6.3 Backend state transitions

A simple managed run state flow:

```text
preparing
→ starting
→ running
→ inspecting
→ ready_for_review
→ ready_for_approval
→ done
```

Failure and interruption states:

```text
starting → failed_before_start
running → stuck
running → crashed
running → stopped_by_stig
inspecting → needs_repair
inspecting → blocked
```

Important rule:

```text
The worker may say it is done.
Only Agent OS can move the run to ready states.
Only Stig or an approved human decision can move important work to final done.
```

## 7. Terminal-worker plan

### 7.1 How Agent OS talks to terminals

Agent OS can launch terminal tools as managed child processes.

Possible methods:

- start CLI process with a prompt argument if supported.
- write a work brief file and tell the worker to read it.
- inject text into a PTY/terminal session.
- run a wrapper script that starts the worker with the correct context.

Preferred early approach:

```text
write work brief to a file
start worker process in task worktree
send instruction: read this brief and follow it
capture output
```

This is easier to inspect and debug.

### 7.2 What the worker sees

The worker should see a clear managed-work preamble:

```text
You are running inside Agent OS as a managed worker.
You are not talking directly to Stig.
Follow the work brief.
Do not mark the task done.
Report blockers clearly.
Agent OS will inspect files, logs, and verification results after you finish.
```

Then it receives the work brief.

### 7.3 What the worker should not control

The worker should not control:

- final approval.
- task done status.
- pushing or deployment.
- permanent memory promotion.
- broad scope expansion.
- deleting unrelated files.
- disabling safety rules.

### 7.4 Worker terminal modes

#### Managed mode

Default. Stig talks to Agent OS. Agent OS talks to worker.

#### Guided manual mode

Stig may type into the terminal, but Agent OS labels the run as partly manual.

Use when:

- Stig needs to intervene.
- the worker asks for interactive input.
- debugging requires direct terminal use.

#### Clean terminal mode

Normal terminal not treated as full Agent OS proof.

Use when:

- exploring manually.
- doing unrelated work.
- escape hatch.

## 8. Work Brief Builder details

### 8.1 Purpose

The Work Brief Builder exists because Stig should not need to write perfect coding prompts.

It transforms normal language into a clear worker brief.

### 8.2 Inputs

The Work Brief Builder receives:

- Stig's original request.
- current project context.
- current task if one exists.
- work classification.
- role profiles.
- relevant approved memory.
- known commands/checks.
- file scope hints.
- Agent OS prompt guidelines.

### 8.3 Outputs

The output should be structured, not just a free-form prompt.

Required fields:

```text
title
plain_summary
role
goal
context
allowed_work
not_allowed
required_proof
success_definition
approval_rules
open_questions
risk_flags
```

### 8.4 Safety limits

The Work Brief Builder may suggest.

It may not:

- approve risky changes.
- mark work as done.
- remove verification requirements.
- grant protected file access.
- decide that Stig approval is unnecessary for important work.

Agent OS backend validates its output.

## 9. Frontend screens in detail

### 9.1 Workbench Home

Purpose:

A single place where Stig talks to Agent OS.

Needs:

- main request input.
- active work list.
- waiting decisions.
- recent task status.
- clear indication that direct terminal management is no longer the default.

### 9.2 Request Preparation View

Purpose:

Show what Agent OS understood before work starts.

Should show:

- interpreted task.
- work class.
- planned role.
- whether it needs a worktree.
- whether it needs checks/review/approval.
- generated brief preview.

Actions:

- start managed run.
- revise brief.
- cancel.
- ask a clarifying question.

### 9.3 Managed Run View

Purpose:

Show the current worker and its state.

Should show:

- task title.
- worker role.
- provider/tool.
- mode badge.
- status.
- elapsed time.
- last activity.
- current event.
- event timeline.
- terminal output preview.
- buttons for stop, inspect, open terminal, open logs.

### 9.4 Terminal Detail View

Purpose:

Allow inspection and emergency control.

Should show warning:

```text
This is a managed worker terminal. Normal control should happen through Agent OS.
Typing here may mark the run as guided manual.
```

### 9.5 Result Review View

Purpose:

Show Stig what happened after the worker exits.

Should show:

- summary.
- changed files.
- patch status.
- verification status.
- review status.
- approval requirement.
- plain failure label if any.
- recommended next action.

Actions:

- start repair pass.
- request review.
- approve.
- request changes.
- stop task.
- ask for explanation.

## 10. Implementation phases

### Phase 0 - Planning and naming

Goal:

Agree on the product names and boundaries.

Deliverables:

- this plan.
- notes for Managed Work Runner.
- names for user-facing statuses.
- decision that Stig talks to Agent OS by default.

Acceptance:

- everyone can explain the flow without mentioning external system names.

### Phase 1 - Backend records only

Goal:

Create backend records for requests, briefs, and managed runs without launching real workers yet.

Deliverables:

- Stig Request record.
- Work Brief record.
- Managed Run record.
- Run Event record.
- basic CLI or service calls to create sample records.

Acceptance:

- a fake request can become a fake work brief and fake managed run.
- UI or CLI can list the records.

### Phase 2 - Work Brief Builder prototype

Goal:

Generate structured work briefs from Stig requests.

Deliverables:

- Work Brief Builder prompt/guidelines.
- structured output format.
- safety validation rules.
- locked brief state.

Acceptance:

- a normal request becomes a readable brief with goal, allowed work, not allowed, required proof, and success definition.
- unsafe or incomplete brief is rejected or sent for revision.

### Phase 3 - Managed terminal launch prototype

Goal:

Start a real terminal worker from Agent OS with a locked brief.

Deliverables:

- launch worker process.
- write brief to file.
- send prompt/instruction to worker.
- capture stdout/stderr.
- store run start/end status.

Acceptance:

- Agent OS can start a Codex/Claude/shell worker from a managed run.
- output is captured.
- process exit is recorded.

### Phase 4 - Work History Stream

Goal:

Make the run visible live and replayable later.

Deliverables:

- event append API/service.
- output events.
- status events.
- UI timeline.
- replay after refresh.

Acceptance:

- Stig can see what happened without opening the raw terminal.

### Phase 5 - Stuck Work Guard

Goal:

Detect silent or stuck workers.

Deliverables:

- last activity tracking.
- timeout settings.
- stuck warning event.
- stop/continue option.
- forced stop path.

Acceptance:

- a silent test worker is marked stuck after timeout.
- logs are preserved.
- UI explains the state.

### Phase 6 - Result Inspector

Goal:

Inspect what the worker actually produced.

Deliverables:

- changed-file detection.
- patch check.
- verification evidence check.
- out-of-scope change detection.
- result status mapping.

Acceptance:

- a run cannot become ready just because the worker says done.
- missing patch, failed check, or out-of-scope file is detected.

### Phase 7 - Frontend default control surface

Goal:

Make Stig talk to Agent OS first, not terminals.

Deliverables:

- main Agent OS request box.
- request preparation view.
- work brief preview.
- managed run panel.
- decision panel.
- terminal as secondary view.

Acceptance:

- Stig can start a managed coding run without manually typing into Codex/Claude terminal.

### Phase 8 - Repair pass integration

Goal:

When verification or review fails, create a focused repair run.

Deliverables:

- repair role brief.
- failed log handoff.
- maximum repair attempts.
- result inspector rerun.

Acceptance:

- failed verification can start a narrow repair pass.
- repair loop stops after max attempts or needs Stig decision.

### Phase 9 - Cleanup and doctor integration

Goal:

Keep local state clean.

Deliverables:

- orphaned process scan.
- stale worktree scan.
- expired lease scan.
- cleanup warnings.
- safe cleanup actions.

Acceptance:

- Agent OS can show messy state and suggest safe cleanup.

## 11. User-facing examples

### Example A - Normal successful task

Stig says:

```text
Fix the login bug.
```

Agent OS shows:

```text
I understand this as code-change work.
I will create a managed builder run.
It will need verification, review, and Stig approval before done.
```

Agent OS generates a work brief.

Stig clicks Start.

Agent OS starts worker.

Worker finishes.

Agent OS inspects result and shows:

```text
Patch created.
Changed files are inside scope.
Relevant tests passed.
Ready for review.
```

### Example B - Worker gets stuck

Worker has no output for 30 minutes.

Agent OS shows:

```text
This worker may be stuck.
Last activity was 30 minutes ago.
You can stop it, keep waiting, or inspect the log.
```

### Example C - Worker changes too much

Worker edits package.json unexpectedly.

Agent OS shows:

```text
The worker changed a file outside the original scope.
This needs Stig approval before it can continue as accepted work.
```

### Example D - Brief needs clarification

Stig says:

```text
Make it better.
```

Agent OS asks:

```text
What should be better: speed, design, reliability, or something else?
```

Or it creates a planning/investigation task instead of implementation.

## 12. Key product decisions

### Decision 1 - Agent OS is the default conversation surface

Default behavior should be:

```text
Stig talks to Agent OS.
Agent OS talks to workers.
```

Direct terminal use is still available but secondary.

### Decision 2 - Work Brief Builder drafts, Agent OS decides

The Work Brief Builder is not the authority.

It creates a draft. Agent OS validates and locks it.

### Decision 3 - Workers cannot decide final truth

A worker can say it is finished.

Agent OS must inspect evidence before moving state forward.

### Decision 4 - Managed mode is different from manual mode

Agent OS must visibly label whether a terminal is:

- managed.
- guided manual.
- clean/unmanaged.

### Decision 5 - Stig approval remains final for important work

Even if the worker succeeds, important implementation work should stop at ready for Stig approval unless the product later defines a safe auto-approval mode.

## 13. Risks and protections

### Risk: Work Brief Builder creates an unsafe brief

Protection:

- backend safety gate.
- required fields.
- forbidden permission patterns.
- Stig confirmation for risky scope.

### Risk: terminal worker ignores instructions

Protection:

- file boundary checks.
- result inspection.
- patch/evidence checks.
- not allowing worker text to become official truth.

### Risk: Stig loses ability to intervene

Protection:

- terminal remains inspectable.
- guided manual mode exists.
- stop button exists.
- decision panel exists.

### Risk: too many screens become confusing

Protection:

- one main Agent OS input.
- simple status cards.
- terminal secondary.
- advanced raw detail hidden unless opened.

### Risk: system over-controls normal work

Protection:

- clean terminal escape hatch.
- planning/read-only modes.
- adjustable strictness later.

## 14. What this plan does not include yet

This plan does not require:

- direct model API access.
- cloud deployment.
- multi-user identity.
- remote worker fleet.
- automatic deployment.
- full CI integration.

Those can come later.

The first goal is local terminal-based managed work.

## 15. Success criteria

This system is successful when Stig can say:

```text
I no longer need to manage Codex and Claude terminals myself.
I tell Agent OS what I want.
Agent OS turns that into controlled work.
Agent OS shows me what is happening.
Agent OS tells me what needs my decision.
Agent OS does not treat the agent's words as proof.
```

Technical success should include:

- request becomes work brief.
- work brief is locked.
- worker terminal starts from Agent OS.
- output is captured.
- run state is visible.
- stuck run can be detected.
- result is inspected.
- missing proof blocks readiness.
- terminal remains available for inspection.

## 16. Short implementation summary

Build this in this order:

1. Store Stig request, work brief, and managed run records.
2. Generate and validate a structured work brief.
3. Start a managed terminal worker with the locked brief.
4. Capture and display run events.
5. Add stuck detection.
6. Add result inspection.
7. Make Agent OS the default UI control surface.
8. Add repair pass and cleanup systems.

The first real product milestone should be:

```text
Stig can submit a request in Agent OS, see the generated work brief, start a managed worker terminal, watch its run timeline, and receive a backend-inspected result.
```