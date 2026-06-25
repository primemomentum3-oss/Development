# Documentation Structure Check

Date: 2026-06-25
Status: structure verification note

## Purpose

This file records how the planning documents now fit together.

## Top-level narrative

Start here:

```text
WHITEPAPER.md
```

This is the natural-language explanation for Stig and future agents.

## Implementation plan folders

The white paper points to these implementation areas:

```text
structured-plans/backend-quality-control/
structured-plans/project-context-bundle/
structured-plans/role-profiles-worker-contracts/
structured-plans/agent-os-managed-worker-conversation-plan.md
```

## Relationship between the areas

```text
Managed Worker Conversation Flow
  = how Stig talks to Agent OS and how Agent OS starts workers.

Project Context Bundle
  = what the worker knows before launch.

Role Profiles And Worker Contracts
  = what the worker is allowed to do.

Backend Quality Control
  = how Agent OS checks what actually happened.
```

## Verification result

The documents now align around one lifecycle:

```text
Stig request
  ↓
work brief
  ↓
context bundle
  ↓
role contract
  ↓
managed worker run
  ↓
backend quality control
  ↓
review / repair / approval
```

## Main rule shared by all documents

```text
Worker text is not proof.
Backend records are proof.
Stig approval is final for important work.
```

## Where detailed implementation agents should go

Implementation agents should use the detailed `systems/` folders.

Stig should begin with `WHITEPAPER.md`.