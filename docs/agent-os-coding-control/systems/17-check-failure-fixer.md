# 17 - Check Failure Fixer

## Plain meaning

The Check Failure Fixer is a focused repair worker for failed tests, checks, or CI results.

It is a special version of the repair pass idea, focused on known check failures.

## What it does

It takes a failed check and creates a narrow repair run.

It should receive:

- the failed command or CI check.
- the failure log.
- the relevant task and patch.
- allowed files.
- instruction to make the smallest safe fix.

## How it works

Agent OS detects a failed local verification or imported CI failure and launches a fixer role.

A simple flow:

```text
check fails
Agent OS saves log
Agent OS creates focused fix run
fixer edits only what is needed
Agent OS reruns the failed check
Agent OS updates task state
```

## What it requires

- Verification failure capture.
- Log artifact.
- Fixer role profile.
- Repair attempt limit.
- Optional CI/GitHub integration later.
- Same result inspection as normal coding runs.

## Who owns it

Agent OS starts and controls the fix flow.

A fixer agent performs the code change.

Stig approves the final result when required.

## Terminal-based feasibility

Fully feasible for local checks.

CI-based fixing is also feasible if Agent OS can read CI status/logs, but that can come later.

## Why it matters

It makes failed checks actionable without restarting the entire task or giving the fixer too much freedom.