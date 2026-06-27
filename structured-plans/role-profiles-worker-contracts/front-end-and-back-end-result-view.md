# Frontend And Backend Result View For Role Profiles And Worker Contracts

Date: 2026-06-25
Status: implementation guide
Owner: Stig

## Purpose

This file explains how Role Profiles and Worker Contracts should appear in Agent OS.

The goal is for Stig to know what kind of worker is running and what it is allowed to do.

## Main UI idea

Every managed run should show a role badge and contract panel.

Example:

```text
Worker: Codex
Role: Builder
Mode: Managed
Contract: Locked
```

## Main UI areas

### 1. Role badge

Shows:

```text
Builder
Reviewer
Fixer
Verifier
Closer
```

### 2. Contract summary

Shows:

```text
Allowed actions
Forbidden actions
Required proof
Approval limits
Expected output
```

### 3. Permission panel

Shows what Agent OS will enforce:

```text
May edit files: yes/no
May run commands: yes/no
May create patch: yes/no
May create review: yes/no
May request approval: yes/no
May mark done: no
```

### 4. Result requirement panel

Shows what this role must produce.

Examples:

```text
Builder: patch + verification evidence
Reviewer: review record
Fixer: focused patch + rerun failed check
Verifier: command evidence
Closer: final report package
```

### 5. Contract artifact link

Stig can open the exact worker contract used for the run.

## Status examples

### Builder ready

```text
Role: Builder
Result: Ready for review
Patch exists.
Verification passed.
Worker cannot approve final result.
```

### Reviewer violation

```text
Role: Reviewer
Status: Blocked
Reason: Reviewer changed source files, which this role is not allowed to do.
```

### Fixer violation

```text
Role: Fixer
Status: Blocked
Reason: Fixer changed files unrelated to the failed check.
```

## Backend view model

Suggested output:

```text
WorkerContractView {
  runId
  role
  contractId
  contractStatus
  allowedActions
  forbiddenActions
  requiredProof
  approvalLimits
  resultRequirements
  violations
  artifactId
}
```

## Why this matters

Stig should not have to guess what a worker is doing.

Agent OS should show:

```text
who the worker is
what it may do
what it must prove
what it cannot decide
whether it stayed inside the contract
```