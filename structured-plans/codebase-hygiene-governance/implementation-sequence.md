# Implementation Sequence

## Goal

Implement the codebase hygiene layer in a safe order, starting with the smallest useful gate and expanding toward full repository governance.

## Phase 1: Canonical quality command

Create one command that becomes the official local, CI, worker, and backend quality entry point.

Deliverables:

- command entry point;
- fixed gate order;
- concise pass/fail output;
- artifact/log output;
- ability to run one gate by name for debugging.

Acceptance:

- humans can run one command and know whether the repo is healthy;
- backend result inspection can call the same command.

## Phase 2: Lint, typecheck, and test wiring

Wire existing checks into the canonical command.

Deliverables:

- lint gate;
- typecheck gate;
- test gate;
- failure signature extraction;
- clear re-run hints.

Acceptance:

- common failures show short useful summaries instead of dumping giant logs first.

## Phase 3: File and function size guard

Add file-size limits and function-size/complexity limits.

Deliverables:

- file-size scanner;
- budget file;
- grandfathered budget support;
- stale budget entry detection;
- guidance to split files instead of raising budgets.

Acceptance:

- new oversized files fail;
- existing large files cannot grow past their budget.

## Phase 4: Tracked debt marker guard

Add a scanner for untracked TODO/FIXME/HACK/XXX markers.

Deliverables:

- marker scanner;
- tracker-reference detection;
- allowlist file;
- stale allowlist detection.

Acceptance:

- untracked debt markers fail the gate.

## Phase 5: Generated surface sync

Add checks that generated docs stay in sync with source code.

Deliverables:

- generated API docs check;
- generated CLI docs check if Agent OS has CLI commands;
- generated schema/tool docs check where applicable;
- check mode and write mode.

Acceptance:

- changing a route, command, or schema without refreshing generated docs fails.

## Phase 6: Dependency hygiene

Add unused/undeclared dependency detection.

Deliverables:

- dependency scanner;
- ignore configuration for legitimate runtime-only cases;
- summary output;
- dependency-addition warning.

Acceptance:

- unused dependencies fail;
- new dependency additions are visible and reviewable.

## Phase 7: Duplicate-code guard

Add duplicate-code detection with a ratcheted threshold.

Deliverables:

- duplicate-code scanner;
- threshold config;
- exclusions for generated files, migrations, fixtures, and tests where appropriate;
- concise duplicate summary.

Acceptance:

- new copy-pasted production code above threshold fails.

## Phase 8: Local and CI parity guard

Ensure CI runs the same required gates as local verification.

Deliverables:

- workflow parser;
- canonical gate manifest;
- alias config for CI-only variants;
- failure when CI drifts.

Acceptance:

- CI cannot accidentally stop running a required gate.

## Phase 9: Instruction drift guard

Validate repository guidance files.

Deliverables:

- scan documented commands;
- scan documented paths;
- confirm they exist;
- fail on stale references.

Acceptance:

- worker guidance cannot reference missing commands or files.

## Phase 10: Quality metrics report

Add a read-only summary for humans and the frontend.

Deliverables:

- coverage summary;
- file-size budget status;
- debt-marker status;
- complexity exception count;
- dependency status;
- duplicate-code status.

Acceptance:

- the frontend can show repository health without running every check manually.

## Phase 11: Frontend integration

Expose quality state on task/run pages.

Deliverables:

- quality gate panel;
- hygiene failure list;
- re-run command hints;
- open artifact/log action;
- clear ready/blocked status.

Acceptance:

- Stig can see why a task is blocked without reading raw logs.

## Final success condition

A managed worker run cannot become ready for approval if the canonical quality gate fails, unless Stig explicitly overrides the result with a recorded reason.