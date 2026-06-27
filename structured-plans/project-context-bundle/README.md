# Project Context Bundle

Date: 2026-06-25
Status: structured implementation specs
Owner: Stig

## Purpose

This folder defines the Project Context Bundle system Agent OS should add around managed terminal workers.

The goal is simple:

```text
Every worker should start with the right approved project knowledge.
```

The worker should not rediscover the same project rules, commands, conventions, architecture patterns, and previous failure lessons every time.

## Why this matters

Without a context bundle, workers often create AI slop because they:

```text
use the wrong test command
miss existing project conventions
repeat old mistakes
ignore important task history
change files that should not be touched
start coding without knowing the current proof requirements
```

The Project Context Bundle makes every run more grounded.

## What this is not

This is not unlimited memory.

This is not a pile of random notes.

This is not an LLM-only summarization feature.

It is a backend-controlled bundle that selects, orders, freezes, and records the context sent into a worker run.

## Existing Agent OS systems to reuse

Reuse existing Agent OS foundations:

```text
task records
event timeline
reports
memory candidates
durable project memory promotion
command evidence
verification runs
patch artifacts
review records
agent run contract
```

## Main documents

Read in this order:

1. [System Map](./system-map.md)
2. [Agent OS Codebase Integration Guide](./agent-os-codebase-integration-guide.md)
3. [Implementation Sequence](./implementation-sequence.md)
4. [Frontend And Backend Result View](./front-end-and-back-end-result-view.md)
5. Individual system specs under `systems/`

## Individual system specs

1. [Context Sources and Precedence](./systems/01-context-sources-and-precedence.md)
2. [Context Bundle Builder](./systems/02-context-bundle-builder.md)
3. [Approved Memory Gate](./systems/03-approved-memory-gate.md)
4. [Run Snapshot and Context Artifact](./systems/04-run-snapshot-and-context-artifact.md)
5. [Context Relevance and Size Control](./systems/05-context-relevance-and-size-control.md)
6. [Context Feedback and Memory Candidates](./systems/06-context-feedback-and-memory-candidates.md)

## First complete milestone

The first complete milestone should be:

```text
Agent OS can build a frozen context bundle for a managed worker run using task data, approved memory, project commands, recent relevant evidence, and role/work-brief requirements, then store that bundle as an artifact tied to the run.
```

## Product promise

When this is implemented, Stig should be able to trust that workers begin with:

```text
the current goal
the right project rules
the relevant approved memory
the correct checks
the known risks
the proof expectations
```

This should reduce repeated mistakes and make coding workers more consistent.