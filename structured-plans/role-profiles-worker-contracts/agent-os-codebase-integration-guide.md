# Agent OS Codebase Integration Guide For Role Profiles And Worker Contracts

Date: 2026-06-25
Status: implementation guide
Owner: Stig

## Purpose

This guide tells the implementing agent how to add Role Profiles and Worker Contracts into Agent OS.

The goal is to bind every managed worker run to a clear role and contract.

## Feature boundary

Name this feature area:

```text
worker role contracts
```

Its job:

```text
Define role-specific permissions, proof requirements, and blocked decisions for managed terminal workers.
```

## Proposed backend folder structure

Use existing Agent OS conventions where possible.

Suggested structure:

```text
src/worker-roles/
  index.ts
  types.ts
  roles.ts
  contracts.ts
  contract-builder.ts
  contract-validator.ts
  permission-policy.ts
  adapter-mapping.ts
  result-requirements.ts
  artifacts.ts
```

If this fits better inside the managed-worker area, use:

```text
src/managed-worker/roles/
```

## File responsibilities

### `types.ts`

Defines:

```text
WorkerRole
RoleProfile
WorkerContract
WorkerPermission
WorkerForbiddenAction
RoleResultRequirement
RoleContractValidationResult
```

### `roles.ts`

Defines supported roles:

```text
builder
reviewer
fixer
verifier
closer
planner
investigator
```

Start with:

```text
builder
reviewer
fixer
```

### `contracts.ts`

Stores reusable contract templates per role.

A contract should include:

```text
role name
goal
allowed actions
forbidden actions
required proof
output format
approval rules
```

### `contract-builder.ts`

Builds a run-specific contract from:

```text
role profile
task
work brief
context bundle
scope rules
verification requirements
```

### `contract-validator.ts`

Checks contract safety before launch.

Reject if contract:

```text
allows self approval
allows final done
omits required proof
omits scope boundaries
allows broad edits for narrow fixer/reviewer role
```

### `permission-policy.ts`

Maps role to backend-enforced permissions.

Examples:

```text
builder may edit allowed files
reviewer may not edit source
fixer may edit only failure-related files
verifier may run checks but not change source
closer may create report but not approve
```

### `adapter-mapping.ts`

Maps roles to allowed worker adapters/tools.

Example:

```text
builder: codex, claude, shell-test
reviewer: codex, claude
fixer: codex, claude
verifier: shell-test, local command runner
closer: codex, claude
```

### `result-requirements.ts`

Defines what each role must produce.

Example:

```text
builder -> patch + verification evidence
reviewer -> review record
fixer -> focused patch + rerun failed check
verifier -> command evidence
closer -> final report / approval package
```

### `artifacts.ts`

Stores locked worker contract artifacts:

```text
worker-contract.json
worker-contract.md
```

## Database additions

Add only if missing:

```text
worker_role_profiles
worker_contracts
worker_contract_artifacts
```

If Agent OS already has agent run contract records, extend those instead.

## Integration points

### Task system

Task classification should suggest a role.

Implementation work usually starts with builder.

Failed verification may start fixer.

Review stage starts reviewer.

### Work brief system

The worker contract should be derived from the locked work brief, not replace it.

### Managed worker runner

Every managed worker run must store role and contract id.

### File leases / scope

Permission policy should connect to file leases and scope checks.

### Result inspector

Result inspector should call role-specific requirements.

### Approval system

No worker role can grant final approval.

## CLI commands

Suggested commands:

```text
agent-os worker-role list
agent-os worker-role show builder
agent-os worker-contract build --task <id> --role builder
agent-os worker-contract validate <contract-id>
agent-os worker-contract show <contract-id>
```

## Smoke scripts

Suggested scripts:

```text
scripts/worker-contract-smoke.mjs
scripts/worker-contract-validator-smoke.mjs
scripts/worker-role-permission-smoke.mjs
scripts/worker-role-result-requirements-smoke.mjs
```

## First implementation milestone

Build:

```text
1. role enum/types
2. builder/reviewer/fixer role profiles
3. contract builder
4. contract validator
5. locked contract artifact
6. managed run stores role + contract id
7. result inspector checks role-specific requirements
```

## Implementation warning

Do not make this prompt-only.

The contract text guides the worker, but backend policy must enforce the important rules.