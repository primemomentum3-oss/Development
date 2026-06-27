# 05 Generated Surface Sync

## Purpose

Generated Surface Sync keeps public or semi-public documentation aligned with the real code.

The source code should be the source of truth. Generated docs should be checked so they do not drift.

## Surfaces to consider

Agent OS may need generated or checked documentation for:

- API routes;
- CLI commands;
- database schema summaries;
- tool registry summaries;
- config schema summaries;
- worker role/profile summaries.

## Required behavior

Each generated surface should support two modes:

```text
write mode: regenerate the file
check mode: fail if the committed file is stale
```

Example commands:

```text
agent-os gen:docs
agent-os gen:docs --check
```

## Why this matters

Workers may change a route, command, schema, or config option and forget to update docs.

Generated sync prevents stale documentation from becoming the hidden source of future mistakes.

## Backend integration

The canonical quality gate should include generated-surface check mode.

A managed worker run should not be ready if it changed a generated source but did not update the generated output.

## Frontend behavior

Show a failure like:

```text
Generated docs are stale.
The CLI command list changed but docs/cli.md was not regenerated.
Run: agent-os gen:docs
```

## First implementation

Start with whichever Agent OS surface is most stable:

- CLI commands, if Agent OS has a central command registry;
- API routes, if Agent OS has route definitions;
- config schema, if Agent OS has a schema registry.

## Acceptance

- generated files include a warning that they are generated;
- check mode fails when generated output is stale;
- canonical quality gate includes the check.