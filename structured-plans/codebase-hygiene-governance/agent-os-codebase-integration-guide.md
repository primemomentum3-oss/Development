# Agent OS Codebase Integration Guide

## Purpose

This guide explains how the codebase hygiene systems should be integrated into Agent OS code.

The implementing agent should not create a separate product or parallel architecture. These checks should plug into the existing Agent OS backend, CLI, workflow harness, managed worker result inspection, and CI.

## Expected implementation shape

The implementation should be normal deterministic code, not language-model reasoning.

Recommended folder shape:

```text
src/codebase-hygiene/
  types.ts
  config.ts
  gate-manifest.ts
  run-gates.ts
  failure-signatures.ts
  file-size-guard.ts
  complexity-guard.ts
  debt-marker-guard.ts
  generated-surface-sync.ts
  ci-parity-guard.ts
  instruction-drift-guard.ts
  dependency-hygiene.ts
  duplicate-code-guard.ts
  quality-report.ts
```

If Agent OS already has a better home for repository checks, use that instead. The important rule is that the hygiene logic should be centralized and reusable.

## CLI integration

Add one canonical command.

Possible names:

```text
agent-os verify
```

or:

```text
bun run check:all
```

The command should run all mandatory gates in a fixed order.

Recommended order:

1. lint;
2. typecheck;
3. unit tests;
4. file-size guard;
5. debt-marker guard;
6. generated-surface sync;
7. dependency hygiene;
8. duplicate-code guard;
9. coverage/ratchet check;
10. CI parity guard.

The output should be concise:

```text
✓ lint (1.2s)
✓ typecheck (2.3s)
✗ file-size (0.4s)
  src/foo/bar.ts exceeds budget
  re-run: agent-os verify --gate file-size
```

## Backend integration

The backend result inspector should call the canonical quality gate after it has already confirmed:

- the worker produced a patch;
- files stayed inside scope;
- required verification evidence exists;
- command evidence is saved.

The hygiene gate result should be stored on the managed run.

Recommended managed-run fields:

```text
quality_gate_status: pending | passed | failed | skipped
quality_gate_started_at
quality_gate_finished_at
quality_gate_summary_json
quality_gate_artifact_id
```

## Artifact integration

Each full gate run should create an artifact.

The artifact should contain:

- gate list;
- pass/fail status per gate;
- concise failure signatures;
- full logs where needed;
- start and end timestamps.

The frontend should show the concise result first and keep full logs behind an expand/open action.

## CI integration

CI should call the same canonical gate command as local development and backend result inspection.

Do not allow CI to quietly run fewer checks than local verify.

Add a parity check that reads CI workflow definitions and confirms that the canonical gate is represented.

## Worker integration

Workers should be told to run the canonical quality command when required, but Agent OS must still check the result itself.

A worker claim like "checks passed" is not enough.

Agent OS should rely on stored gate results, command evidence, and artifacts.

## Configuration files

Use explicit budget/config files so rules are inspectable.

Possible files:

```text
hygiene/file-size-budgets.json
hygiene/coverage-budgets.json
hygiene/debt-marker-allowlist.json
hygiene/duplicate-code-budget.json
hygiene/bundle-size-budgets.json
hygiene/ci-parity-config.json
```

These files should be part of the repo and reviewed like code.

## Acceptance condition

The implementation is acceptable when a new coding run cannot be marked ready if the canonical quality gate fails.