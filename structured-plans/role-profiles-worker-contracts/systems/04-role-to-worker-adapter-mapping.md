# 04 - Role To Worker Adapter Mapping

## Purpose

Role to Worker Adapter Mapping defines which terminal tools can perform each role.

## Why this matters

Not every tool should perform every role.

A simple command runner may be good for verification but not planning.

A coding agent may be good for building but should not be the final approval actor.

## Example mapping

```text
builder: codex, claude, shell-test
reviewer: codex, claude
fixer: codex, claude
verifier: shell-test, local command runner
closer: codex, claude
planner: codex, claude
investigator: codex, claude
```

## Adapter requirements

Each adapter should declare:

```text
supported roles
requires PTY
can edit files
can run commands
supports non-interactive launch
supports log capture
```

## Acceptance criteria

- Agent OS rejects unsupported role/adapter pairs.
- UI shows which adapter is being used for which role.
- shell-test adapter can prove the flow before Codex/Claude adapters.