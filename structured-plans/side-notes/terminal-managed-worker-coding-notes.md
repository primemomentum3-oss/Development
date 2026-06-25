# Side Note - Coding Notes For Terminal-Based Managed Workers

Date: 2026-06-25
Status: technical side note for implementation agents
Owner: Stig

## 1. Purpose of this side note

This file is for the coding agent that will eventually implement the Managed Worker Conversation Flow inside Agent OS.

The previous structured plan explains the product and user flow. This side note explains the coding approach: what to build, where the hard parts are, what to avoid, and how to adapt the reference-system idea to a terminal-based Agent OS.

The most important implementation rule is:

```text
Do not build this as if Agent OS has direct model API control.
Build this as terminal process orchestration.
```

Agent OS should control Codex, Claude, shell, or other workers as local terminal processes.

## 2. Confirmed reference-system lesson

The reference system we reviewed uses API keys and internal backend APIs around its run system.

Important observations:

- It forwards provider keys such as `ANTHROPIC_API_KEY` into the sandbox/runtime environment when needed.
- It can also use other provider keys such as OpenAI, Gemini, Google, or Pi keys depending on the runtime.
- It uses a GitHub token for host-side Git operations.
- Its backend creates run records, freezes agent configuration, resolves worker placement, provisions a sandbox/workspace, injects run environment, and then starts the runtime.
- It uses internal HTTP/API calls to control the worker runtime and stream events.

The useful idea is not the exact API shape.

The useful idea is:

```text
The backend owns the lifecycle around the agent.
The agent does the work inside that lifecycle.
```

For Agent OS, the equivalent should be:

```text
Agent OS owns the terminal process lifecycle.
The terminal worker does the coding inside that lifecycle.
```

## 3. Main implementation difference

### Reference-system style

```text
backend creates run
backend talks to runtime API
runtime starts sandboxed agent
runtime streams events back
backend reaps result
```

### Agent OS terminal style

```text
backend creates managed run
backend writes locked work brief
backend starts local terminal process / PTY
backend injects instruction or points worker to brief file
backend captures stdout/stderr/PTY output
backend watches PID/process group
backend inspects worktree after exit
backend records result
```

This means the Agent OS code should focus on:

- local process management.
- PTY or child process handling.
- stdout/stderr capture.
- file-based prompt/brief handoff.
- worktree inspection.
- event recording.
- timeout and crash handling.

Do not start by designing provider API abstractions.

## 4. Recommended backend modules

Use these module boundaries or equivalent names inside Agent OS.

### 4.1 Request intake

Purpose:

Turn Stig's natural language request into a backend work request.

Responsibilities:

- store original request.
- attach to project/workspace when known.
- classify work as planning, investigation, implementation, etc.
- decide whether this should become a managed coding run.

Suggested service name:

```text
services/workIntake.ts
```

Suggested core function:

```text
createManagedWorkRequest(input)
```

### 4.2 Work brief builder

Purpose:

Create the structured brief that will be sent to the coding worker.

Responsibilities:

- gather task context.
- gather role profile.
- gather project commands/checks.
- include approved memory/context.
- produce a structured brief object.
- never directly launch the worker.

Suggested service name:

```text
services/workBriefs.ts
```

Suggested functions:

```text
createDraftWorkBrief(input)
validateWorkBrief(input)
lockWorkBrief(input)
```

### 4.3 Managed worker runner

Purpose:

Start and track the terminal worker process.

Responsibilities:

- create run record before launch.
- write brief file.
- build process command.
- start terminal process.
- track PID/process group.
- capture output.
- record start/end/failure events.
- expose stop/cancel.

Suggested service name:

```text
services/managedWorkers.ts
```

Suggested functions:

```text
startManagedWorkerRun(input)
stopManagedWorkerRun(input)
getManagedWorkerRun(input)
listManagedWorkerRuns(input)
```

### 4.4 Worker process adapter

Purpose:

Hide provider/terminal details behind one simple interface.

Do not make Codex, Claude, and shell logic leak everywhere.

Suggested service name:

```text
services/workerProcessAdapters.ts
```

Suggested interface:

```text
WorkerProcessAdapter {
  name
  buildCommand(runContext)
  buildEnvironment(runContext)
  buildInitialInput(runContext)
}
```

Adapters:

```text
codexCliAdapter
claudeCliAdapter
shellAdapter
```

### 4.5 Work history stream

Purpose:

Record and show what happened.

Responsibilities:

- append run events.
- record output chunks.
- keep sequence order.
- allow replay.
- support live UI subscription later.

Suggested service name:

```text
services/managedRunEvents.ts
```

Suggested functions:

```text
appendManagedRunEvent(input)
listManagedRunEvents(runId)
tailManagedRunEvents(runId, options)
```

### 4.6 Stuck work guard

Purpose:

Detect silent/stuck/crashed terminal workers.

Responsibilities:

- scan active managed runs.
- check last output/event time.
- check process existence.
- mark stuck or crashed.
- stop process when policy says to.
- preserve logs.

Suggested service name:

```text
services/managedRunWatchdog.ts
```

Suggested functions:

```text
runManagedWorkerWatchdogTick(input)
computeManagedRunIdleMs(input)
markManagedRunStuck(input)
```

### 4.7 Result inspector

Purpose:

Inspect what the worker produced after it exits.

Responsibilities:

- inspect changed files.
- check patch artifact.
- check verification evidence.
- check out-of-scope files.
- choose next official state.
- suggest repair/review/approval path.

Suggested service name:

```text
services/managedRunResultInspector.ts
```

Suggested functions:

```text
inspectManagedRunResult(runId)
classifyManagedRunOutcome(input)
```

## 5. Process control implementation notes

### 5.1 Prefer PTY when interactive behavior matters

Codex/Claude terminal tools may expect terminal-like behavior.

If the tool behaves poorly with normal `child_process.spawn`, use a PTY library.

Possible implementation options:

- Node `child_process.spawn` for simple non-interactive commands.
- `node-pty` or equivalent for real interactive terminal behavior.
- Electron-side terminal process management if current desktop architecture already owns terminal panes.

Early implementation can start with a simple spawn for deterministic shell workers, then add PTY for Codex/Claude.

### 5.2 Always use a process group

Managed worker runs should be stoppable as a group.

Reason:

A worker can start child commands. If Agent OS only kills the parent process, child processes may keep running.

Implementation expectation:

- start worker in a new process group where supported.
- stop by sending SIGTERM to the group.
- after grace period, send SIGKILL.
- record both attempts.

### 5.3 Capture output as stream and artifact

Do not only show terminal output in the UI.

Record output in two forms:

1. Event stream chunks for live/replay timeline.
2. Full log artifact for later inspection.

Use event stream for readability and artifact for completeness.

### 5.4 Do not rely on parsing model text for truth

The worker may say:

```text
Tests passed.
```

Agent OS should not trust that as proof.

Agent OS should verify from:

- command run records.
- exit codes.
- saved logs.
- changed files.
- patch artifacts.

The terminal output is useful context, not final truth.

## 6. Work brief handoff implementation

### 6.1 Preferred first version: file-based brief

Write the locked work brief to a file in a managed run artifact folder.

Example path:

```text
.agent-os/managed-runs/<run-id>/work-brief.md
```

Then start the worker with an instruction like:

```text
You are running as an Agent OS managed worker.
Read the work brief at .agent-os/managed-runs/<run-id>/work-brief.md and follow it.
Do not mark the task done. Agent OS will inspect your result.
```

This is easier to debug than injecting a long prompt directly into the terminal.

### 6.2 Optional later version: direct terminal injection

After file-based brief works, Agent OS can inject the brief directly into the worker terminal if needed.

But the file should still exist as the locked historical record.

### 6.3 Store the exact sent instruction

Store:

- work brief content.
- wrapper instruction/preamble.
- adapter name.
- command line.
- working directory.
- environment allowlist.

This becomes the Run Snapshot.

## 7. Database / record design details

Exact schema can be adapted to the current Agent OS database style, but the coding agent should preserve these concepts.

### 7.1 work_requests

Purpose:

Original Stig request before it becomes a worker run.

Fields:

```text
id
project_id
repository_id
workspace_ref
raw_text
source
status
created_at
updated_at
```

### 7.2 work_briefs

Purpose:

Structured brief created from the request.

Fields:

```text
id
work_request_id
task_id
status
role
title
goal
context_json
allowed_work_json
not_allowed_json
required_proof_json
success_definition
approval_rules_json
risk_flags_json
created_by
locked_at
created_at
updated_at
```

Important status values:

```text
draft
needs_revision
locked
rejected
```

### 7.3 managed_worker_runs

Purpose:

One actual terminal worker execution.

Fields:

```text
id
task_id
work_brief_id
role
adapter_name
mode
status
cwd
command
argv_json
env_summary_json
pid
process_group_id
started_at
ended_at
last_activity_at
exit_code
failure_label
failure_message
log_artifact_path
created_at
updated_at
```

Important status values:

```text
preparing
starting
running
stuck
stopping
crashed
failed_before_start
exited
inspecting
needs_repair
ready_for_review
ready_for_approval
blocked
done
```

### 7.4 managed_run_events

Purpose:

Timeline of what happened.

Fields:

```text
id
managed_run_id
seq
ts
kind
stream
summary
payload_json
```

Example kinds:

```text
run.created
brief.written
worker.starting
worker.started
terminal.output
worker.no_activity_warning
worker.stopped
worker.exited
result.inspection_started
result.patch_missing
result.out_of_scope_change
result.ready_for_review
```

### 7.5 managed_run_artifacts

Purpose:

Logs, brief files, raw outputs, inspection reports.

Fields:

```text
id
managed_run_id
task_id
artifact_type
path
summary
size_bytes
sha256
created_at
```

## 8. Backend state machine details

Do not let random code update run status freely.

Create a small state transition helper.

Example allowed flow:

```text
preparing -> starting
starting -> running
starting -> failed_before_start
running -> stopping
running -> stuck
running -> crashed
running -> exited
stopping -> exited
exited -> inspecting
inspecting -> needs_repair
inspecting -> ready_for_review
inspecting -> ready_for_approval
inspecting -> blocked
ready_for_approval -> done
```

Important rule:

```text
done must not be set by the worker.
```

Only Stig/human approval path should finalize important work.

## 9. Terminal adapter details

### 9.1 Shell adapter first

Build the first proof with a simple shell adapter.

Reason:

It is deterministic and easier to test.

Example:

```text
adapter: shell
command: node
argv: ["scripts/fake-worker.js", "--brief", briefPath]
```

Acceptance:

- Agent OS can launch it.
- output is captured.
- events are recorded.
- exit is recorded.
- result inspector runs.

### 9.2 Codex CLI adapter

After shell adapter works, add Codex terminal adapter.

Adapter should define:

- command path.
- arguments.
- working directory.
- how to send initial brief.
- environment allowlist.
- timeout default.

Do not hardcode local machine paths everywhere. Use settings/config.

### 9.3 Claude CLI adapter

Same pattern as Codex.

The backend should not care whether worker is Codex or Claude. It should only care about the adapter interface.

## 10. Environment and security details

### 10.1 Use environment allowlist

Do not pass the full parent process environment blindly.

Create a safe environment builder.

Allow only needed values:

- PATH.
- HOME if required.
- SHELL if required.
- tool-specific auth already configured locally.
- Agent OS run ID / task ID if needed.

Do not pass secrets unless a specific adapter requires them and the risk is accepted.

### 10.2 Prefer local tool auth over provider API keys in Agent OS

Because this design is terminal-based, Agent OS should assume Codex/Claude CLI already handles its own login/auth.

Agent OS should not require direct model API keys for the first version.

### 10.3 Avoid broad backend callback powers

If the worker gets a callback command later, keep it narrow.

Allowed:

- report blocker.
- record evidence.
- request permission.

Not allowed:

- mark task done.
- approve itself.
- change policy.
- deploy.
- push.

## 11. Result inspection coding details

The Result Inspector should run after worker exit, no matter whether the worker claims success.

Checks:

### 11.1 Process result

Record:

- exit code.
- timed out or not.
- killed or not.
- stderr summary.
- log artifact path.

### 11.2 Changed files

Use Git from the task worktree.

Check:

- any changed files.
- allowed vs forbidden paths.
- protected files.
- deleted files.
- untracked files.

### 11.3 Patch artifact

If implementation work requires a patch, require a patch artifact.

The run should not become ready if files changed but no patch exists.

### 11.4 Verification evidence

If task requires verification, require command evidence.

Do not accept agent text as verification proof.

### 11.5 Next state

Examples:

```text
no changes, no error -> blocked: no patch produced
changes outside scope -> blocked: out-of-scope changes
verification failed -> needs_repair
verification missing -> blocked: missing verification
verification passed + review required -> ready_for_review
review approved + approval required -> ready_for_approval
```

## 12. Frontend coding details

### 12.1 Keep terminal secondary

The UI should present the managed run card first, not the terminal first.

Primary UI:

- task status.
- worker role.
- activity.
- timeline.
- decisions.

Secondary UI:

- raw terminal output.

### 12.2 Mark terminal mode clearly

Every terminal pane should show mode:

```text
Managed
Guided manual
Clean
```

If Stig types directly into a managed terminal, switch or mark the run as guided manual.

### 12.3 Use simple state cards

A non-coder should see:

```text
Running normally
No recent activity
Needs your decision
Tests failed
Patch missing
Ready for review
Ready for approval
```

### 12.4 Raw detail should be available but not primary

Keep logs, event payloads, command output, and raw JSON accessible, but not front-and-center.

## 13. Testing strategy

### 13.1 Unit tests

Test:

- state transitions.
- brief validation.
- path boundary checks.
- result outcome classification.
- failure label mapping.

### 13.2 Service tests

Test with fake process adapter:

- start run.
- capture output.
- exit successfully.
- exit failed.
- fail before start.
- timeout.

### 13.3 Integration smoke tests

Create smoke scripts:

```text
managed-worker-shell-smoke
managed-worker-timeout-smoke
managed-worker-result-inspection-smoke
managed-worker-out-of-scope-smoke
```

### 13.4 Desktop/manual checks

Check that:

- Stig request appears in UI.
- generated brief appears.
- run can start.
- terminal output appears in timeline.
- terminal can be opened.
- result appears after exit.

## 14. First implementation milestone

Build a small vertical slice first.

### Goal

A fake shell worker proves the full path.

### Required flow

```text
Stig request created
work brief created and locked
managed run created
fake terminal worker started
output captured as events
worker exits
result inspector runs
UI/CLI shows final state
```

### Do not include yet

- full Codex support.
- full Claude support.
- complex prompt specialist behavior.
- full repair loop.
- preview manager.
- CI fixer.

Get the basic managed-worker lifecycle correct first.

## 15. Implementation order for coding agent

1. Add records for work request, work brief, managed worker run, run events.
2. Add service functions for creating/locking work briefs.
3. Add fake shell worker adapter.
4. Add managed worker runner that can start fake worker.
5. Add output capture and event append.
6. Add process exit handling.
7. Add result inspector stub.
8. Add simple CLI smoke command.
9. Add UI surface after backend flow works.
10. Add real Codex/Claude adapters.

## 16. What to avoid

Avoid these mistakes:

- Do not make the coding worker responsible for updating official state.
- Do not trust terminal text as proof.
- Do not pass all environment variables blindly.
- Do not implement Codex/Claude first before a fake adapter works.
- Do not make direct terminal typing the default user path.
- Do not let the Work Brief Builder bypass backend validation.
- Do not create a broad callback channel that lets workers approve themselves.
- Do not hide raw logs completely; they must remain inspectable.

## 17. Simple final coding target

The first correct version should prove this:

```text
Agent OS can turn Stig's request into a locked work brief, start a terminal worker from that brief, capture the worker's output, record the run lifecycle, inspect the result, and show a plain final status without trusting the worker's own claim of completion.
```

That is the coding foundation. Everything else should build on top of it.