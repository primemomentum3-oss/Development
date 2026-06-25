# 02 Ratchet Budget System

## Purpose

The Ratchet Budget System prevents the codebase from getting worse over time.

It accepts that a repo may already have some imperfect areas, but it does not allow new work to make those areas worse without explicit review.

## Plain meaning

A ratchet says:

> We can improve from here, but we should not slide backward.

## Budgets to support

Recommended budget files:

```text
hygiene/file-size-budgets.json
hygiene/coverage-budgets.json
hygiene/debt-marker-allowlist.json
hygiene/duplicate-code-budget.json
hygiene/bundle-size-budgets.json
hygiene/complexity-overrides.json
```

## Required behavior

Each budget should support:

- default threshold;
- explicit grandfathered exceptions;
- stale exception detection;
- clear failure message;
- rule that exceptions should shrink over time.

## Example

If a file is allowed to be 850 lines today, the worker may not make it 900 lines.

The correct response is:

- split the file;
- move logic into smaller modules;
- reduce the file below the budget;
- or request an explicit override with a recorded reason.

## Backend integration

Budget failures should be represented as structured gate failures.

A budget failure should include:

- budget type;
- path or metric;
- current value;
- allowed value;
- suggested fix.

## Frontend integration

Show budget failures as plain language:

```text
File-size budget failed.
The worker made src/runtime/session.ts 120 lines larger than allowed.
Recommended next step: split the new helper logic into a smaller module.
```

## First implementation

Start with file-size and debt-marker budgets. Add coverage, duplication, bundle size, and complexity after the canonical gate is stable.

## Acceptance

The system is ready when budget failures block managed-run readiness and stale budget entries also fail the gate.