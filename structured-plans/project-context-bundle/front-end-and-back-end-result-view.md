# Frontend And Backend Result View For Project Context Bundle

Date: 2026-06-25
Status: implementation guide
Owner: Stig

## Purpose

This file explains how the Project Context Bundle should appear to Stig in Agent OS.

The goal is not to show a huge prompt. The goal is to show what important context Agent OS gave the worker and why.

## Main UI idea

Every managed run should have a `Context` panel.

The panel should answer:

```text
What did this worker know before it started?
Where did that context come from?
Was the memory approved?
Were any context risks or conflicts found?
```

## Main UI areas

### 1. Context summary card

Shows:

```text
Bundle id
Task title
Worker role
Created at
Attached run
Number of context items
Warnings
```

Example:

```text
Context Bundle
Task: Fix login bug
Role: Builder
Approved memory used: 3 items
Warnings: 1 stale memory item excluded
```

### 2. Context sections

Show sections in this order:

```text
Task goal
Worker role
Required proof
Approved project memory
Known project commands
Relevant previous failures
Boundaries and risks
Artifact links
```

### 3. Source list

For every major context item, show source:

```text
Source: approved memory
Source: task validation contract
Source: previous report
Source: command evidence
Source: project defaults
```

### 4. Trust label

Each item should have a trust label:

```text
Trusted
Candidate
Stale
Conflicting
Excluded
```

Only trusted items should be sent as instructions/facts.

### 5. Conflict and warning panel

Show warnings like:

```text
One memory candidate matched this task but was not approved, so it was excluded.
One old command was replaced by a newer approved command.
```

## User actions

Stig should be able to:

```text
Open full context bundle
Open source artifact/report
Approve memory candidate later
Reject memory candidate
Ask why a context item was included
Ask why a context item was excluded
```

## Backend view model

Suggested output:

```text
ContextBundleView {
  bundleId
  taskId
  runId
  role
  createdAt
  sections
  trustedItemCount
  excludedItemCount
  warnings
  sources
  artifactIds
}
```

## Why this matters

If a worker makes a bad decision, Stig and future agents can inspect whether:

```text
the right context was missing
the worker ignored context
the context was stale
the memory system needs an update
```

This makes failures easier to learn from.