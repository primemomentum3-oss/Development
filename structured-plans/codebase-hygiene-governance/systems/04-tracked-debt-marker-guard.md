# 04 Tracked Debt Marker Guard

## Purpose

The Tracked Debt Marker Guard prevents silent technical debt from entering the codebase.

## Problem it solves

Workers may leave comments like:

```text
TODO fix later
FIXME temporary
HACK quick workaround
```

Those comments become invisible debt when they have no owner, task, or explanation.

## Required behavior

The guard should scan source files for debt markers:

- TODO;
- FIXME;
- HACK;
- XXX.

Each marker must have a reference on the same line.

Accepted references can include:

- task id;
- issue id;
- URL;
- approved follow-up id;
- explicit Agent OS tracked work id.

## Bad example

```text
// TODO clean this up later
```

## Good example

```text
// TODO(agent-os-123): remove fallback after managed worker adapter lands
```

## Allowlist behavior

If Agent OS already has untracked debt, it may be grandfathered into an allowlist.

The allowlist should only shrink over time.

Stale allowlist entries should fail.

## Backend integration

Debt marker failures should block readiness unless Stig explicitly approves a temporary exception.

## Frontend behavior

Show a clear message:

```text
Tracked debt guard failed.
The worker added an untracked TODO in src/runtime/foo.ts:42.
Add a task reference or remove the marker.
```

## Acceptance

- untracked debt markers fail;
- stale allowlist entries fail;
- tracked markers pass;
- failure output includes path and line.