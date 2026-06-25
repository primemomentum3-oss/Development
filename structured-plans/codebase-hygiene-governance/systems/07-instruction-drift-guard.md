# 07 Instruction Drift Guard

## Purpose

The Instruction Drift Guard prevents worker guidance from becoming stale.

Worker-facing instructions are only useful if the commands, files, folders, and system names they mention still exist.

## Problem it solves

Agents can follow outdated repository instructions and do the wrong thing.

Examples:

- guidance says to run a command that no longer exists;
- guidance points to a file that was renamed;
- role docs mention an old workflow;
- setup docs reference removed folders;
- project instructions contradict current system names.

## Files to scan

Start with:

- AGENTS.md;
- CLAUDE.md if present;
- README files;
- role/profile docs;
- worker contract templates;
- structured plan docs that are used by workers.

## Required checks

The guard should detect:

- backtick command references;
- path references;
- package script references;
- references to required config files;
- references to generated docs.

It should confirm that referenced files exist and commands are present in the expected command registry or package scripts.

## Backend integration

Instruction drift should be part of the canonical quality command.

If guidance is stale, new worker runs may be built from bad information, so readiness should be blocked until fixed.

## Frontend behavior

Show:

```text
Instruction drift failed.
AGENTS.md references `agent-os old-command`, but that command does not exist.
```

## First implementation

Start by validating:

- package-script references;
- paths in backticks;
- known quality command names.

## Acceptance

- stale command references fail;
- stale path references fail;
- failure output includes file and line;
- guidance can be trusted by future workers.