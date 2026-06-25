# 14 - Clear Failure Labels

## Plain meaning

Clear Failure Labels translate technical failures into simple, understandable states.

Instead of saying only `failed`, Agent OS should explain what kind of failure happened.

## What it does

It gives plain labels such as:

- failed before start.
- worker crashed.
- worker got stuck.
- verification failed.
- review requested changes.
- patch missing.
- changed files outside scope.
- waiting for approval.
- blocked by missing information.
- stopped by Stig.

## How it works

Agent OS maps backend facts to failure labels.

A simple flow:

```text
run ends or blocks
Agent OS reads process state, logs, patch, evidence, and task state
Agent OS chooses a plain failure label
Agent OS shows reason and next action
```

## What it requires

- Stable failure categories.
- Mapping from technical errors to plain language.
- Next-action suggestions.
- UI display.
- Event/report integration.

## Who owns it

Agent OS owns the labels and classification.

Agents may add context, but should not be the only source of the failure state.

## Terminal-based feasibility

Fully feasible with terminals.

Most failure labels come from process exit codes, logs, missing artifacts, changed files, and verification records.

## Why it matters

Stig should not need to understand technical logs to know what went wrong or what should happen next.