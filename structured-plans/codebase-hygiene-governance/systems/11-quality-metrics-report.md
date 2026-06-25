# 11 Quality Metrics Report

## Purpose

The Quality Metrics Report gives Stig and implementers a clear view of repository health.

It is a visibility layer. It should not replace blocking gates.

## What it reports

Recommended metrics:

- coverage percentage and floor;
- file-size budget status;
- number of oversized files;
- number of complexity exceptions;
- debt-marker status;
- duplicate-code status;
- dependency hygiene status;
- bundle-size status if relevant;
- generated-doc sync status;
- CI parity status.

## Blocking vs reporting

Some checks block readiness.

The report summarizes them.

Example:

```text
Codebase Health
Coverage: 82.4% / floor 80% passing
File-size: 1 failure
Debt markers: passing
Duplicate code: passing
Dependencies: passing
Docs sync: stale
```

## Backend integration

The report should read from the same gate outputs used by the canonical quality command.

Do not create a separate source of truth.

## Frontend integration

Show the report in:

- repository health page;
- managed run page after quality gate runs;
- task detail page when blocked by hygiene.

## Scheduled reporting

Later, Agent OS may run this as a background check and show drift over time.

This should be report-only unless a task/run is actively trying to become ready.

## Acceptance

- report uses gate data;
- report separates blocking failures from informational trends;
- report is readable by a non-coder;
- report links to detailed gate artifacts.