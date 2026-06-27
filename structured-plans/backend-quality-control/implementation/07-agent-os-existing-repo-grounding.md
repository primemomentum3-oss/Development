# 07 - Agent OS Existing Repo Grounding

Date: 2026-06-25
Status: implementation grounding note
Audience: Agent OS implementation agent

## Purpose

This note grounds the backend quality-control plan in what Agent OS already appears to be.

Do not treat the plan as a separate product. It must be implemented inside the existing Agent OS TypeScript/Node codebase and reuse current foundations.

## Existing repo shape to respect

Agent OS is a local-first backend for coordinating AI coding agents.

The repo is TypeScript/Node and exposes CLI binaries:

```text
agent-os
agent-os-mcp
```

It also has a desktop app workspace under:

```text
apps/*
```

The package scripts already show that Agent OS has many backend and runtime smoke checks, including:

```text
runtime:smoke
runner:smoke
launcher:smoke
launch-process:smoke
agent-run:smoke
verification-gates:smoke
codex-adapter:smoke
codex-adapter-failure:smoke
codex-no-evidence:smoke
codex-process-boundary:smoke
auto-execute:smoke
auto-fix-pass:smoke
auto-blocked-path:smoke
codex-scope:smoke
real-execution:check
backend-system:test
local:clean-slate
```

This means the managed-worker quality-control implementation should fit the current pattern:

```text
TypeScript backend service
CLI command/debug path
smoke script
unit tests
real-execution check integration
desktop UI integration
```

## What not to duplicate

Agent OS already has names and systems for tasks, runtime, launcher, agent-run contract, Codex adapter, verification gates, auto execution, auto fix pass, blocked path, scope smoke, command evidence, and backend tests.

The implementation agent should inspect those existing systems before adding new ones.

If an existing system already owns something, extend it or bridge into it instead of creating a parallel copy.

Examples:

```text
Do not create a new task system.
Do not create a new worktree system.
Do not create a new command evidence system.
Do not create a new approval system.
Do not create a separate hidden event history.
```

The managed-worker layer should orchestrate these existing systems around a single terminal-worker lifecycle.

## Where the new code probably belongs

Use a grouped feature module such as:

```text
src/managed-workers/
```

But before creating that folder, check whether Agent OS already has a runtime or launcher folder that should own part of this.

A good rule:

```text
Existing launcher/process code starts tools.
New managed-worker code owns lifecycle quality control around those tools.
Existing evidence/check systems store proof.
New managed-worker code requires and interprets that proof.
```

## How to integrate with existing checks

The package scripts already include check/smoke patterns.

Add new scripts only after backend services exist:

```text
managed-worker:smoke
managed-worker:quality-smoke
managed-worker:check
```

Then consider adding them into a larger check such as:

```text
real-execution:check
backend-completion:check
```

But do not break existing checks by wiring too early.

## Implementation dependency order inside Agent OS

Before coding, inspect current implementations for:

```text
runtime run records
agent launcher/process launcher
Codex adapter
verification gates
command evidence
patch artifacts
worktree handling
file leases
operator decisions
review records
memory promotion
```

Then implement the managed-worker layer as a coordinator around those.

## How the code should feel when finished

A fake worker should be launchable by Agent OS.

Agent OS should record:

```text
run created
brief written
worker started
output captured
worker exited
inspection started
patch/scope checked
verification checked
failure/ready state chosen
```

The worker can lie or be sloppy.

Agent OS should still produce the correct official state.

## Important implementation warning

The goal is not to make the agent smarter first.

The goal is to make Agent OS stricter first.

Start with deterministic fake workers and negative tests. Add Codex/Claude terminal adapters only after the backend lifecycle catches bad behavior reliably.