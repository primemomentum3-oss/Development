# 01 - Role Taxonomy

## Purpose

Role Taxonomy defines the worker roles Agent OS supports.

## Initial roles

Start with:

```text
builder
reviewer
fixer
```

Then add:

```text
verifier
closer
planner
investigator
```

## Role meanings

### Builder

Edits code inside approved scope and produces patch plus verification evidence.

### Reviewer

Reviews patch and evidence. Should not edit source code.

### Fixer

Repairs a specific failed check or review finding. Should not broaden the task.

### Verifier

Runs checks and records evidence. Should not edit source code.

### Closer

Creates final report/approval package. Should not approve final done.

### Planner

Breaks work into steps. Should not implement code.

### Investigator

Reads and reports. Should not edit source code unless promoted to builder.

## Acceptance criteria

- each role has clear purpose.
- each role has allowed and forbidden actions.
- each managed run has one primary role.
- role names are stable and visible in UI.