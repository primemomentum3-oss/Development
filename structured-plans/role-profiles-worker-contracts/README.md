# Role Profiles And Worker Contracts

Date: 2026-06-25
Status: structured implementation specs
Owner: Stig

## Purpose

This folder defines the Role Profiles and Worker Contracts system Agent OS should add around managed terminal workers.

The goal is simple:

```text
Each worker should know exactly what role it has, what it may do, what it must prove, and what it cannot decide.
```

## Why this matters

Without role profiles, every worker behaves like a general-purpose agent.

That creates risk:

```text
fixers refactor too much
reviewers edit code
builders approve themselves
workers skip proof
agents broaden scope without permission
```

Role profiles make each worker narrower and easier to control.

## What this is not

This is not only prompt wording.

It is both:

```text
worker-facing contract
backend-enforced policy
```

The worker contract tells the agent what to do.

Backend policy checks what the role is allowed to do.

## Existing Agent OS systems to reuse

Reuse existing Agent OS foundations:

```text
tasks
worktrees
file leases
command evidence
verification runs
patch artifacts
review records
approval/operator decisions
agent launcher
agent run contract
workflow harness
```

## Main documents

Read in this order:

1. [System Map](./system-map.md)
2. [Agent OS Codebase Integration Guide](./agent-os-codebase-integration-guide.md)
3. [Implementation Sequence](./implementation-sequence.md)
4. [Frontend And Backend Result View](./front-end-and-back-end-result-view.md)
5. Individual system specs under `systems/`

## Individual system specs

1. [Role Taxonomy](./systems/01-role-taxonomy.md)
2. [Contract Builder and Locking](./systems/02-contract-builder-and-locking.md)
3. [Permission Policy and Tool Boundaries](./systems/03-permission-policy-and-tool-boundaries.md)
4. [Role to Worker Adapter Mapping](./systems/04-role-to-worker-adapter-mapping.md)
5. [Contract Validation and Safety Gate](./systems/05-contract-validation-and-safety-gate.md)
6. [Role-Specific Result Requirements](./systems/06-role-specific-result-requirements.md)

## First complete milestone

The first complete milestone should be:

```text
Agent OS can assign a role to a managed worker run, generate and lock a role-specific contract, enforce basic backend permissions, and show Stig what the worker is allowed to do.
```

## Product promise

When this is implemented, Stig should see:

```text
This worker is a Builder.
It may edit only approved task files.
It must produce a patch and verification evidence.
It cannot approve itself or mark the task done.
```

That gives clearer control and reduces broad agent behavior.