# Existing Agent OS Systems Map For Backend Quality Control

Date: 2026-06-25
Status: integration map
Owner: Stig

## Purpose

This file maps backend quality-control work to Agent OS systems that already exist or are already planned in the Agent OS baseline.

The implementing agent should use this map to avoid duplicating existing Agent OS concepts.

## Existing foundation

Agent OS already has these concepts in the baseline:

```text
task records
event timeline
Git worktrees
file leases
command evidence
verification runs
reliability gate results
patch artifacts
review records
memory candidates
durable memory promotion
human-readable reports
coordinator/runtime foundations
trusted runner foundation
agent launcher foundation
local process launch foundation
agent run contract foundation
evidence gate and verification foundation
workflow harness and closer package foundation
Codex adapter and automatic execution foundation
```

Backend quality control should connect these into one controlled managed-worker path.

## Mapping table

| Backend quality-control need | Reuse existing Agent OS concept | New/extended code needed |
| --- | --- | --- |
| Know what work is being done | Task records | Managed run must require task id. |
| Show what happened | Event timeline | Add managed worker event kinds. |
| Isolate code changes | Git worktrees | Managed run must use task worktree. |
| Enforce file boundaries | File leases / scope rules | Patch/scope checker compares changed files to lease/scope. |
| Prove commands ran | Command evidence | Verification gate queries command evidence. |
| Prove required checks passed | Verification runs / reliability gates | Verification gate checks exit code, worktree, freshness. |
| Package changes | Patch artifacts | Result inspector requires patch artifact for implementation work. |
| Human/agent review | Review records | Inspector moves to ready_for_review when required. |
| Final product decision | Approval / operator decision flow | Important work stops at ready_for_approval. |
| Explain outcome | Reports / event timeline | Inspection report artifact + plain failure label. |
| Improve future context | Memory candidates | Quality issues can create candidates, not durable memory directly. |
| Run local agent process | Agent launcher / local process launch | Managed worker runner wraps/extends launcher. |
| Contract with worker | Agent run contract | Work brief and locked run snapshot should extend this concept. |

## What should not be duplicated

Do not create new systems for:

```text
tasks
worktrees
file leases
command evidence
verification records
patch artifacts
review records
approval records
memory promotion
reports
```

If the existing system is incomplete, extend it.

Do not create a parallel copy.

## New concepts that are probably needed

These may need new tables/services because they are specific to managed terminal workers:

```text
managed worker run
managed worker process attachment
managed worker state machine
managed worker inspection report
managed worker cleanup finding
managed worker failure label
```

## How data should flow

```text
Task
  ↓
Locked work brief / agent run contract
  ↓
Managed worker run
  ↓
Task worktree + process launch
  ↓
Event timeline + output/log artifact
  ↓
Patch artifact + command evidence + verification run
  ↓
Result inspector
  ↓
Review / approval / repair / blocked state
```

## Example: successful implementation work

```text
Task exists.
Managed worker run starts in task worktree.
Worker changes files.
Worker runs required verification.
Command evidence records exit code 0.
Patch artifact is created.
Result inspector sees in-scope files and passing fresh verification.
Run moves to ready_for_review or ready_for_approval.
```

## Example: blocked work

```text
Task exists.
Managed worker run starts.
Worker changes files.
No command evidence exists for required verification.
Result inspector blocks ready state.
Failure classifier returns verification_missing.
Stig sees: Required verification was not run.
```

## Example: out-of-scope work

```text
Task scope allows src/login/**.
Worker changes package.json.
Patch/scope checker flags dependency_change_needs_approval.
Result inspector blocks run.
Stig must approve wider scope or request a narrower fix.
```

## Example: fake success

```text
Worker says done.
Process exits with code 0.
Worktree is dirty.
No patch artifact exists.
Result inspector blocks run.
Failure classifier returns patch_missing or dirty_work_left.
```

## Product rule

The official Agent OS state should be based on backend records, not worker summaries.

Worker summaries are helpful explanations.

Backend records are proof.