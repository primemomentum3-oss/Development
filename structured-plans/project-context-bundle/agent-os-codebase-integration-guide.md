# Agent OS Codebase Integration Guide For Project Context Bundle

Date: 2026-06-25
Status: implementation guide
Owner: Stig

## Purpose

This guide tells the implementing agent how to add Project Context Bundle support into Agent OS.

The goal is to create a backend-built, frozen context package for each managed worker run.

## Feature boundary

Name this feature area:

```text
project context bundle
```

Its job:

```text
Assemble the right approved task, project, memory, verification, and role context before the worker starts.
```

## Proposed backend folder structure

Use existing Agent OS conventions where possible.

Suggested structure:

```text
src/context-bundle/
  index.ts
  types.ts
  sources.ts
  precedence.ts
  builder.ts
  memory-gate.ts
  relevance.ts
  size-control.ts
  snapshot.ts
  artifacts.ts
  feedback.ts
```

If Agent OS already has a memory/context area, this can live there instead:

```text
src/memory/context-bundle/
```

## File responsibilities

### `types.ts`

Defines:

```text
ContextBundle
ContextBundleSource
ContextBundleItem
ContextBundleSection
ContextBundleBuildInput
ContextBundleBuildResult
ContextBundleRiskFlag
ContextBundleMemoryItem
```

### `sources.ts`

Loads possible context sources:

```text
task
work brief
role contract
project memory
project commands
recent reports
recent verification failures
current patch/evidence state
approval rules
```

### `precedence.ts`

Defines which source wins when context conflicts.

Recommended order:

```text
1. Stig instruction / safety rule
2. locked work brief
3. task validation contract
4. approved durable project memory
5. current task evidence/reports
6. role defaults
7. general project defaults
```

### `builder.ts`

Builds the context bundle.

Expected function:

```text
buildContextBundle(input)
```

It should return a structured bundle, not only markdown.

### `memory-gate.ts`

Ensures only approved memory is treated as trusted project context.

Unapproved memory may be shown as candidate/context reference only if explicitly allowed.

### `relevance.ts`

Chooses which context items are relevant to this task.

Start simple:

```text
same project
same area/path
same role
same failure type
same verification command
```

Do not start with a complicated ranking system.

### `size-control.ts`

Keeps the bundle short enough to be useful.

Rules:

```text
prefer summaries over raw logs
link artifacts instead of embedding huge logs
deduplicate repeated facts
show stale context warnings
```

### `snapshot.ts`

Freezes the bundle for the run.

Every managed run should store exactly which context was given to the worker.

### `artifacts.ts`

Stores the bundle as a run artifact.

Expected artifacts:

```text
context-bundle.json
context-bundle.md
```

### `feedback.ts`

Creates memory candidates from repeated lessons after runs.

Examples:

```text
wrong command used repeatedly
same failure fixed twice
new project convention discovered
review found missing context
```

## Database additions

Add only what is missing.

Suggested records:

```text
context_bundles
context_bundle_items
context_bundle_sources
```

If artifact storage is enough for early versions, start with artifacts and add tables later.

## Integration points

### Task system

Bundle must include task goal, status, validation contract, and proof requirements.

### Work brief system

Bundle must include the locked work brief.

### Role profiles

Bundle should be role-specific. A reviewer gets different context than a builder.

### Memory system

Only approved durable memory should be treated as trusted context.

### Evidence and reports

Use recent task reports, command evidence, and verification failures as context references.

### Managed worker runner

Managed worker launch should require a context bundle or explicitly record why it was skipped.

## CLI commands to add or extend

Suggested commands:

```text
agent-os context-bundle build --task <id> --role builder
agent-os context-bundle show <bundle-id>
agent-os context-bundle explain <bundle-id>
agent-os context-bundle sources <bundle-id>
```

## Smoke scripts

Suggested scripts:

```text
scripts/context-bundle-smoke.mjs
scripts/context-bundle-memory-gate-smoke.mjs
scripts/context-bundle-size-control-smoke.mjs
scripts/context-bundle-snapshot-smoke.mjs
```

## First implementation milestone

Build this first:

```text
1. Load task and locked work brief.
2. Load approved project memory.
3. Load known project verification command.
4. Build structured context bundle.
5. Save context bundle artifact.
6. Attach bundle to managed worker run.
7. Show bundle in CLI.
```

## Implementation warning

Do not let unapproved memory silently become trusted context.

Do not dump every historical log into the prompt.

Do not let the worker edit its own context bundle after launch.

Do not make context generation depend only on an LLM. Backend code should assemble sources and enforce precedence.