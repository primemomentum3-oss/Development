# 01 Canonical Quality Gate

## Purpose

The Canonical Quality Gate is the single trusted command that answers:

> Is the Agent OS repository healthy enough after this change?

It should be used by humans, managed workers, CI, and backend result inspection.

## Problem it solves

Without one canonical gate, different actors run different checks.

That creates confusion:

- a worker says tests passed but did not run lint;
- local checks pass but CI fails;
- CI runs fewer checks than the backend expects;
- Stig cannot tell which command is authoritative.

## Required behavior

The gate should:

- run a fixed ordered list of checks;
- fail if any mandatory check fails;
- show concise per-gate status lines;
- preserve full logs as artifacts;
- provide a re-run hint for each failed gate;
- support verbose mode;
- support running a single gate for debugging.

## Recommended gates

Minimum useful list:

1. lint;
2. typecheck;
3. tests;
4. file-size guard;
5. debt-marker guard;
6. generated-surface sync;
7. dependency hygiene;
8. duplicate-code guard;
9. coverage/ratchet check;
10. CI parity guard.

## Output contract

Default output should be short:

```text
✓ lint (1.0s)
✓ typecheck (2.1s)
✗ file-size (0.2s)
  src/example/large-file.ts exceeds the file budget
  re-run: agent-os verify --gate file-size
```

Full logs should be available with:

```text
agent-os verify --verbose
```

## Backend integration

The backend result inspector should call this gate after patch/proof/scope checks.

A failed gate should set managed run status to blocked or failed according to existing Agent OS state rules.

## Frontend integration

The frontend should show:

- overall status;
- failed gates;
- plain-language failure summary;
- full log artifact;
- re-run hint.

## First implementation

Start with lint, typecheck, and tests. Then add file-size and debt-marker guards as the first new hygiene-specific gates.