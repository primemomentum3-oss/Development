# 04 - Result Inspector

## Plain meaning

The Result Inspector is the backend closing check that runs after a coding worker exits.

It answers one simple question:

```text
What actually happened, and what state should the work be in now?
```

## What it does

It checks whether the worker produced acceptable work.

It can inspect:

- changed files.
- patch artifact.
- command evidence.
- verification result.
- review status.
- closer package status.
- task approval requirement.
- out-of-scope changes.
- missing proof.

## How it works

When the terminal process exits, Agent OS should not trust the agent's final words. It should inspect backend facts.

A simple flow:

```text
worker exits
Agent OS reads changed files
Agent OS checks patch and evidence
Agent OS checks required verification
Agent OS checks review/approval requirements
Agent OS marks run ready, blocked, failed, or waiting
```

## What it requires

- Access to worktree state.
- Patch artifact checks.
- Required verification rules.
- Evidence records.
- Task state rules.
- Plain failure categories.

## Who owns it

Agent OS owns this system.

The agent may say what it thinks happened, but Agent OS decides the official state.

## Terminal-based feasibility

Fully feasible with terminals.

Agent OS can inspect Git status, files, patches, command logs, and task records directly.

## Why it matters

This prevents the agent from declaring success without proof.

It turns terminal work into controlled backend truth.