# Frontend And Backend Result View

## Purpose

This document describes how codebase hygiene should appear to Stig and how backend results should be stored.

The goal is visibility without log overload.

## Backend result model

Every canonical quality-gate run should produce a structured result.

Suggested fields:

```text
quality_run_id
managed_run_id
trigger: manual | managed_worker_result | ci | scheduled
status: pending | running | passed | failed | cancelled
started_at
finished_at
gates_total
gates_passed
gates_failed
summary_json
artifact_id
```

Each gate should have its own result:

```text
gate_name
status
started_at
finished_at
duration_ms
failure_summary
full_log_artifact_id
rerun_hint
```

## Frontend placement

Show quality status in three places.

### 1. Task detail page

Show a compact card:

```text
Repository Quality
Status: Failed
Failed gates: file-size, debt-marker
Last run: 2 minutes ago
```

Actions:

- open details;
- open logs;
- copy re-run command;
- start repair pass if safe.

### 2. Managed run detail page

Show whether the run result passed hygiene checks.

Example:

```text
Result Inspector: passed
Patch Scope: passed
Verification Evidence: passed
Codebase Hygiene: failed
```

This helps Stig understand that a worker can produce a patch and still not be ready.

### 3. Repository health page

Show longer-term repo health:

```text
Coverage floor: passing
File-size budgets: 1 failure
Tracked debt: passing
Duplicate code: passing
Dependencies: passing
Docs sync: passing
CI parity: passing
```

## Failure display

Show failures in plain language.

Bad:

```text
Exit code 1
```

Good:

```text
File-size guard failed.
The worker made src/runtime/session-manager.ts larger than the allowed budget.
Recommended next step: split helper logic into a smaller module.
```

## Re-run hints

Every failure should show a clear command.

Examples:

```text
agent-os verify --gate file-size
agent-os verify --gate debt-marker
agent-os verify --verbose
```

## Override behavior

A quality failure should block readiness by default.

If Agent OS allows override, it must require:

- Stig decision;
- reason;
- timestamp;
- link to the failed quality run;
- follow-up task if needed.

## What the frontend must not do

The frontend must not hide hygiene failures behind terminal logs.

The result card should tell Stig:

- what failed;
- why it matters;
- how to fix it;
- whether the task can proceed.

## First UI milestone

Show one quality card on the managed run page with:

- status;
- failed gates;
- concise failure summary;
- full log link;
- re-run hint.