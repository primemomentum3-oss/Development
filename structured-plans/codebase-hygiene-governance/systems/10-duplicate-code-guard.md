# 10 Duplicate Code Guard

## Purpose

The Duplicate Code Guard prevents copy-pasted production code from increasing.

## Problem it solves

Workers may solve a task by copying nearby code and changing a few names.

This can look correct in the short term but creates long-term maintenance problems.

## Required behavior

The guard should scan handwritten production code for duplication.

It should exclude:

- generated files;
- migrations where duplication is expected;
- test goldens and fixtures;
- build output;
- vendored code.

## Threshold behavior

Use a ratcheted threshold.

The threshold may begin at the current state of the repo, but should not increase without explicit reason.

## Failure example

```text
Duplicate-code guard failed.
New duplicate block detected between src/tasks/foo.ts and src/tasks/bar.ts.
Extract a shared helper or justify why duplication is required.
```

## Backend integration

Duplicate-code failures should block readiness unless explicitly overridden.

If duplication appears only in tests or fixtures, the guard may ignore it based on config.

## Frontend behavior

Show:

- duplicated files;
- approximate duplicated line ranges;
- suggested action.

## First implementation

Start with a standard duplicate-code scanner and a simple threshold.

Only scan stable production folders first.

## Acceptance

- new production duplication fails above threshold;
- generated/test/build files are excluded;
- threshold is visible and reviewable;
- failure output points to duplicated locations.