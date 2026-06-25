# 06 - Context Feedback and Memory Candidates

## Purpose

This system turns useful lessons from managed runs into reviewable memory candidates.

## Plain meaning

When a run reveals something useful for future workers, Agent OS should capture it as a candidate, not immediately trust it forever.

## Candidate triggers

Create a candidate when:

```text
same mistake repeats
review identifies a missing project convention
worker used wrong command
verification failed because context was missing
Stig corrected project behavior
result inspector finds recurring scope issue
```

## Candidate content

Each candidate should include:

```text
lesson
source run
evidence link
why it matters
suggested future use
approval status
```

## Rules

```text
candidate memory is not trusted until approved.
rejected memory is excluded from bundles.
approved memory can enter future bundles.
```

## Acceptance criteria

- run findings can create memory candidates.
- candidates link back to evidence.
- no candidate is promoted without approval.
- future bundles can use approved memory.