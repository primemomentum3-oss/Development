# 03 - Approved Memory Gate

## Purpose

The Approved Memory Gate ensures that only reviewed and approved project memory is treated as trusted context.

## Why this matters

Bad memory is worse than no memory.

If a worker receives unapproved or stale memory as fact, it can repeat old mistakes at scale.

## Memory trust levels

Use clear levels:

```text
approved
candidate
rejected
stale
conflicting
excluded
```

## Rules

```text
approved memory can be included as trusted context.
candidate memory can be mentioned only as untrusted if explicitly allowed.
rejected memory must be excluded.
stale memory must be excluded or shown with warning.
conflicting memory must be flagged and resolved by precedence.
```

## Integration

Use existing Agent OS memory candidates and durable memory promotion.

Do not create a parallel memory system.

## Acceptance criteria

- unapproved memory is not silently included as trusted context.
- every memory item shows trust level and source.
- stale/conflicting memory creates a warning.
- durable project memory remains approval-controlled.