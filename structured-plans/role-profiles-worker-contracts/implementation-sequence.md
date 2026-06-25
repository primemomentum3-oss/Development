# Role Profiles And Worker Contracts Implementation Sequence

Date: 2026-06-25
Status: implementation sequence
Owner: Stig

## Purpose

This file gives the implementing agent the order to build Role Profiles and Worker Contracts inside Agent OS.

## Phase 1 - Define role types

### Goal

Agent OS has stable role names and role profile types.

### Build

Create:

```text
src/worker-roles/types.ts
src/worker-roles/roles.ts
```

Start with:

```text
builder
reviewer
fixer
```

### Acceptance

Tests can load the role list and inspect each role's meaning.

## Phase 2 - Add contract templates

### Goal

Each role has a default contract template.

### Build

Create:

```text
contracts.ts
```

Each template includes:

```text
allowed actions
forbidden actions
required proof
output requirement
approval rule
```

### Acceptance

Builder, reviewer, and fixer each produce different contract content.

## Phase 3 - Build run-specific contracts

### Goal

Agent OS can turn a role template plus task/work brief into a locked worker contract.

### Build

Create:

```text
contract-builder.ts
```

Inputs:

```text
task
work brief
role
context bundle
scope rules
verification requirements
```

### Acceptance

A builder contract for one task includes specific allowed scope and required checks.

## Phase 4 - Add contract validation

### Goal

Unsafe contracts cannot launch.

### Build

Create:

```text
contract-validator.ts
```

Reject contracts that:

```text
allow self approval
allow final done
omit required proof
omit scope boundaries
allow reviewer to edit source
allow fixer to broaden scope
```

### Acceptance

Invalid contracts fail before launch with plain reasons.

## Phase 5 - Add backend permission policy

### Goal

Role rules are not only text.

### Build

Create:

```text
permission-policy.ts
```

Policy should answer:

```text
Can this role edit files?
Can this role run commands?
Can this role create patch?
Can this role create review?
Can this role request approval?
Can this role mark done?
```

### Acceptance

Backend rejects forbidden role actions.

## Phase 6 - Attach contracts to managed runs

### Goal

Every managed run records its role and locked contract.

### Build

Update managed worker launch:

```text
choose role
build contract
validate contract
save contract artifact
attach contract id to run
launch worker
```

### Acceptance

Managed run show displays role and contract artifact.

## Phase 7 - Add role-specific result requirements

### Goal

Result inspector checks role-specific outputs.

### Build

Create:

```text
result-requirements.ts
```

Examples:

```text
builder requires patch + verification evidence
reviewer requires review record
fixer requires focused patch + failed check rerun
verifier requires command evidence
closer requires report/approval package
```

### Acceptance

A reviewer run that changes source code is blocked.

A builder run without patch is blocked.

A fixer run that changes unrelated files is blocked.

## Phase 8 - Add frontend view

### Goal

Stig can see role and contract clearly.

### Build

UI shows:

```text
worker role
allowed actions
forbidden actions
required proof
approval limits
contract artifact
```

### Acceptance

Stig can inspect what a worker was allowed to do before and after the run.

## Phase 9 - Expand roles

After builder/reviewer/fixer work, add:

```text
verifier
closer
planner
investigator
```

## First complete milestone

```text
A managed worker run can launch with a locked builder/reviewer/fixer contract, and backend result inspection enforces the role-specific requirements.
```