# 03 File And Function Size Guard

## Purpose

The File And Function Size Guard keeps the codebase readable by preventing oversized files and giant functions.

This is one of the strongest defenses against agent-generated code mess.

## Problem it solves

Workers may solve tasks by adding large blocks of code to one file.

That creates:

- hard-to-review patches;
- hard-to-test functions;
- hidden coupling;
- future maintenance pain;
- repeated helper logic.

## Required checks

The guard should check:

- total lines per source file;
- total lines per function where tooling supports it;
- cognitive complexity where tooling supports it;
- stale budget entries for deleted files.

## Budget behavior

New files should stay below the default threshold.

Existing large files may be grandfathered, but they cannot grow past their frozen budget.

## Recommended thresholds

Initial values can be conservative and adjusted to Agent OS’s current state.

Example starting point:

```text
file default maximum: 500 lines
function maximum: 100 lines
cognitive complexity maximum: 15
```

## Failure example

```text
File-size guard failed.
src/managed-worker/result-inspector.ts has 642 lines.
Default budget is 500 lines.
Split the file instead of raising the budget.
```

## Integration with worker results

If a worker causes a file-size failure, Agent OS should not mark the run ready.

If safe, Agent OS may create a repair pass with instructions to split the file without changing behavior.

## Acceptance

- oversized new files fail;
- oversized existing files cannot grow;
- stale budget entries fail;
- failure messages tell the worker what to split.