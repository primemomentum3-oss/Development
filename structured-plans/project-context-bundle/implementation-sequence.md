# Project Context Bundle Implementation Sequence

Date: 2026-06-25
Status: implementation sequence
Owner: Stig

## Purpose

This file gives the implementing agent the order to build Project Context Bundle support inside Agent OS.

## Phase 1 - Define context bundle types

### Goal

Agent OS has a structured shape for context bundles.

### Build

Create:

```text
src/context-bundle/types.ts
```

Include:

```text
ContextBundle
ContextBundleSection
ContextBundleItem
ContextBundleSource
ContextBundleBuildInput
ContextBundleBuildResult
```

### Acceptance

A test can create a bundle object with sections for task, work brief, memory, verification, risks, and artifacts.

## Phase 2 - Load deterministic sources

### Goal

Agent OS can load known context sources without an LLM.

### Build

Create:

```text
sources.ts
```

Load:

```text
task
locked work brief
role contract
approved memory
known project commands
recent reports
required proof
```

### Acceptance

A task id can produce a raw source list.

## Phase 3 - Add precedence rules

### Goal

Agent OS knows which context wins when sources conflict.

### Build

Create:

```text
precedence.ts
```

Use order:

```text
Stig/safety > locked brief > validation contract > approved memory > task evidence > role defaults > project defaults
```

### Acceptance

Conflicting instructions are ordered and flagged.

## Phase 4 - Build first context bundle

### Goal

Agent OS can create a readable context bundle.

### Build

Create:

```text
builder.ts
```

Generate sections:

```text
Task goal
Worker role
Important context
Approved memory
Required verification
Boundaries
Risks
Artifact links
```

### Acceptance

A CLI command can build and print a bundle for one task.

## Phase 5 - Save frozen bundle artifact

### Goal

Every worker run can show exactly what context it received.

### Build

Create:

```text
snapshot.ts
artifacts.ts
```

Save:

```text
context-bundle.json
context-bundle.md
```

### Acceptance

A managed run can link to a frozen context bundle artifact.

## Phase 6 - Add approved memory gate

### Goal

Only approved durable memory becomes trusted context.

### Build

Create:

```text
memory-gate.ts
```

Rules:

```text
approved durable memory = trusted
memory candidate = candidate only
rejected memory = excluded
stale memory = warning
```

### Acceptance

Unapproved memory never appears as trusted context.

## Phase 7 - Add relevance and size control

### Goal

Bundles stay useful and not bloated.

### Build

Create:

```text
relevance.ts
size-control.ts
```

Start simple:

```text
project match
path match
role match
failure type match
recentness
```

### Acceptance

Large raw logs are linked as artifacts, not dumped into the bundle.

## Phase 8 - Attach to managed worker launch

### Goal

Managed worker launch uses the frozen context bundle.

### Build

Update managed worker launch flow:

```text
build context bundle
save artifact
include bundle path in work brief / run snapshot
launch worker
```

### Acceptance

A worker receives the context bundle path and the run stores the bundle id/artifact id.

## Phase 9 - Add frontend view

### Goal

Stig can inspect what context the worker received.

### Build

Desktop view should show:

```text
Context Bundle
- Task goal
- Role
- Approved memory used
- Required checks
- Risks
- Source list
```

### Acceptance

Stig can open a managed run and inspect the exact context bundle.

## Phase 10 - Add feedback loop

### Goal

Useful lessons become memory candidates after runs.

### Build

Create:

```text
feedback.ts
```

Possible triggers:

```text
repeated verification failure
review correction
worker missed project convention
wrong command used
```

### Acceptance

Agent OS can create a memory candidate, but cannot promote it without approval.

## First complete milestone

```text
A task can produce a frozen context bundle using task data, locked work brief, approved memory, required verification, and role rules, and a managed worker run can link to that bundle.
```