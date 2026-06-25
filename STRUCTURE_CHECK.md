# Documentation Structure Check

Date: 2026-06-25
Status: structure verification note

## Purpose

This file records how the planning documents now fit together.

It also clarifies the scope of `WHITEPAPER.md`.

## Top-level narrative

Start here:

```text
WHITEPAPER.md
```

This is the natural-language explanation for Stig and future agents.

## White paper scope

The white paper is limited to the structured plan layer.

It covers:

```text
structured-plans/agent-os-managed-worker-conversation-plan.md
structured-plans/backend-quality-control/
structured-plans/project-context-bundle/
structured-plans/role-profiles-worker-contracts/
```

It does not attempt to explain every detailed file inside the `systems/` folders.

The `systems/` folders are implementation appendices for later coding work.

## Structured plan areas

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

The structured plans align around one lifecycle:

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

## Main rule shared by the structured plans

```text
Worker text is not proof.
Backend records are proof.
Stig approval is final for important work.
```

## Where detailed implementation agents should go

Stig should begin with `WHITEPAPER.md`.

Implementation agents can later use the detailed `systems/` folders when they need coding-level breakdowns.

Those detailed files are not the white paper's scope.