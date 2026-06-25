# Role Profiles And Worker Contracts System Map

## Plain-language summary

Role Profiles tell Agent OS what kind of worker is being launched.

Worker Contracts tell the worker what it may do, what proof it must produce, and what decisions it cannot make.

Together they answer:

```text
Who is this worker?
What is this worker allowed to do?
What is this worker not allowed to do?
What proof is required?
What backend rules enforce the role?
What still requires Stig?
```

## Core principle

A worker should not be a general agent by default.

Each worker should have a narrow role.

```text
Builder builds.
Reviewer reviews.
Fixer fixes a specific failure.
Verifier verifies.
Closer prepares the final package.
Stig approves important work.
```

## Role profile flow

```text
Task type
  ↓
Role selection
  ↓
Role profile
  ↓
Worker contract
  ↓
Backend permission policy
  ↓
Managed worker run
  ↓
Role-specific result inspection
```

## Backend systems and jobs

| System | Job |
| --- | --- |
| Role Taxonomy | Defines the role names and what each role means. |
| Contract Builder and Locking | Creates the worker contract and freezes it for the run. |
| Permission Policy and Tool Boundaries | Maps roles to what they may do. |
| Role to Worker Adapter Mapping | Chooses which tool/adapter can perform each role. |
| Contract Validation and Safety Gate | Blocks unsafe or incomplete contracts. |
| Role-Specific Result Requirements | Checks whether the role produced the right output. |

## Example role split

```text
Builder: may edit code inside scope and produce patch.
Reviewer: may inspect patch/evidence and produce review, but not edit code.
Fixer: may fix a specific failed check or review finding.
Verifier: may run checks and record evidence, but not change source code.
Closer: may prepare summary and approval package, but not approve.
Stig: final product approval.
```

## How it improves Agent OS

Current Agent OS already has launcher foundations, Codex adapter foundations, task records, verification, review, and approval concepts.

The improvement is to make each worker launch explicitly role-bound and backend-checked.

Instead of:

```text
Start Codex on this task.
```

Agent OS says:

```text
Start Codex as Builder with these allowed actions, required proof, and blocked decisions.
```

## First implementation target

Start with three roles:

```text
builder
reviewer
fixer
```

Add verifier and closer after the first path works.

## Success example

A fixer run receives:

```text
Role: Fixer
Allowed: fix only the failed test from this log
Not allowed: refactor unrelated code, add dependencies, approve task, mark done
Required proof: rerun failed test and save evidence
```

If it changes unrelated files, Agent OS blocks the result.