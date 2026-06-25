# System Map

## Goal

This map defines the repo-hygiene layer Agent OS should add so the codebase stays clean as humans and workers change it.

A worker result can be functionally correct and still make the codebase worse. For that reason, Agent OS needs hygiene checks in addition to task proof checks.

## Main systems

### 1. Canonical Quality Gate

One trusted command runs every required repository health check.

It should be usable by:

- humans;
- managed workers;
- CI;
- backend result inspection.

Possible command names:

- `agent-os verify`
- `bun run check:all`

### 2. Ratchet Budget System

A set of budgets that can improve over time but should not get worse.

Examples:

- coverage floor;
- file-size ceiling;
- function-size ceiling;
- complexity ceiling;
- duplicate-code threshold;
- bundle-size threshold;
- technical-debt allowlist.

Rule: old exceptions may be grandfathered, but new work should not increase the exception list.

### 3. File And Function Size Guard

Prevents oversized files and giant functions.

This protects readability and makes future edits safer.

### 4. Tracked Debt Marker Guard

Prevents unowned TODO/FIXME/HACK/XXX comments.

Each marker must be removed or tied to a task, issue, URL, or tracked work item.

### 5. Generated Surface Sync

Ensures generated documentation stays aligned with source code.

Examples:

- API documentation from route definitions;
- command documentation from CLI definitions;
- schema documentation from schema definitions.

### 6. Local And CI Parity Guard

Ensures local verification and CI verification are not different systems.

The same required checks should run locally, in CI, and at managed-run result inspection.

### 7. Instruction Drift Guard

Validates that worker guidance files reference real commands, real files, and current system names.

This prevents stale guidance from steering future work incorrectly.

### 8. Code Style And File Structure Contract

Defines naming, directory placement, module boundaries, test placement, import style, and TypeScript style.

The contract should be enforced where possible.

### 9. Dependency Hygiene Guard

Detects unused, undeclared, or casually added dependencies.

New dependencies should require a clear reason.

### 10. Duplicate Code Guard

Detects copy-pasted code and prevents duplication from growing.

### 11. Quality Metrics Report

A report-only summary of repository health.

It should show:

- coverage;
- complexity exceptions;
- file-size budget status;
- tracked debt;
- dependency health;
- bundle-size status;
- duplicate-code status.

## How this fits a managed coding run

1. Worker finishes.
2. Backend result inspector checks patch, proof, and scope.
3. Canonical quality gate runs.
4. Hygiene checks run through that gate.
5. If anything fails, the run is not ready.
6. Agent OS shows clear reasons and next actions.

## First milestone

Agent OS has one canonical quality command that runs lint, typecheck, tests, file-size guard, debt-marker guard, generated-docs check, dependency check, duplicate-code check, and local/CI parity check.