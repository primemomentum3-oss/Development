# 01 - Context Sources and Precedence

## Purpose

This system defines where Agent OS may pull context from and which source wins when sources disagree.

## Context sources

Allowed sources:

```text
Stig request
locked work brief
task validation contract
task history
approved durable project memory
project commands/checks
recent verification failures
recent reports
patch/review history
role profile
project defaults
```

## Precedence order

Recommended order:

```text
1. Stig instruction and safety policy
2. locked work brief
3. task validation contract
4. approved durable project memory
5. current task evidence and reports
6. role profile
7. project defaults
8. older historical context
```

## Conflict behavior

If context conflicts, Agent OS should not hide it.

Examples:

```text
old memory says npm test
new project setting says pnpm test
```

Agent OS should use the newer trusted source and record a warning.

## What not to do

Do not treat unapproved memory candidates as trusted facts.

Do not let older project notes override the current work brief.

Do not merge conflicting context silently.

## Acceptance criteria

- every bundle item has a source.
- every bundle item has a trust level.
- conflicts are visible.
- precedence is deterministic and testable.