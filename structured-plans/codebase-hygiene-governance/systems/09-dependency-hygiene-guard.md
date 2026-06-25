# 09 Dependency Hygiene Guard

## Purpose

The Dependency Hygiene Guard prevents dependencies from growing accidentally.

Dependencies are part of the long-term maintenance burden of Agent OS. Workers should not add them casually.

## Problem it solves

Workers may add a package to solve a small problem even when the existing stack can solve it.

This creates:

- unused dependencies;
- undeclared dependencies;
- bundle-size growth;
- security/review burden;
- inconsistent implementation style.

## Required behavior

The guard should check:

- unused dependencies;
- dependencies used but not declared;
- new dependencies added in a patch;
- dependency changes without a recorded reason.

## New dependency policy

Adding a dependency should require:

- reason;
- where it is used;
- why existing tools are not enough;
- whether it affects frontend bundle size;
- whether it is runtime or dev-only.

## Backend integration

Dependency hygiene should be part of the canonical quality gate.

If a worker changes package manifests or lockfiles, Agent OS should make that visible in the result inspector.

## Frontend behavior

Show:

```text
Dependency hygiene warning.
The worker added `some-package` to package.json.
Reason required before this task can be ready.
```

If unused:

```text
Dependency hygiene failed.
`some-package` is listed but not used.
```

## First implementation

Start with unused/undeclared dependency detection.

Then add explicit new-dependency reporting.

## Acceptance

- unused dependencies fail;
- undeclared dependencies fail;
- new dependencies are visible;
- dependency changes require explanation or approval.