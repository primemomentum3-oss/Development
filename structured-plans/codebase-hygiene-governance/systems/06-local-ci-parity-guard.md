# 06 Local And CI Parity Guard

## Purpose

The Local And CI Parity Guard ensures that local verification and CI verification do not drift apart.

The same required checks should be represented in both places.

## Problem it solves

Without parity, Agent OS may report a task as locally clean while CI later fails, or CI may pass while local/backend checks would have caught a problem.

## Required behavior

The guard should compare:

- canonical quality gate manifest;
- CI workflow commands;
- configured aliases for CI-specific variants;
- CI-only steps that are intentionally separate.

## Example

If the canonical gate includes:

```text
file-size
```

but CI does not run it directly or through the canonical quality command, the parity guard should fail.

## Configuration

Use a small config file for intentional differences.

Example:

```text
hygiene/ci-parity-config.json
```

It can define:

- aliases;
- CI-only commands;
- ignored workflow steps with reasons.

Every exception should have a comment or explanation.

## Backend integration

CI parity should be part of the canonical quality command.

It does not check the worker patch directly; it checks that the repo’s enforcement surface remains honest.

## Frontend behavior

Show:

```text
CI parity failed.
Local verify includes debt-marker, but CI does not appear to run it.
```

## Acceptance

- CI drift fails;
- intentional aliases are documented;
- CI-only steps are documented;
- local/CI differences cannot silently appear.