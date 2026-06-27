# Codebase Hygiene And Engineering Governance

## Purpose

This plan pack defines the non-agent systems Agent OS should use to keep its codebase clean over time.

This is separate from managed worker control and backend quality control.

Managed worker control asks:

- did the worker start, run, stop, and produce a result correctly?

Backend quality control asks:

- did the worker produce a valid patch with proof, verification, and scope compliance?

Codebase hygiene governance asks:

- is the Agent OS codebase staying clean, understandable, documented, tested, and safe for future agents to modify?

## Core principle

Agent OS should not rely on agents to remember good engineering discipline.

Agents can be told to write clean code, but Agent OS needs deterministic checks that make sloppy code harder to merge.

The system should enforce:

- one canonical quality command;
- file and function size limits;
- complexity limits;
- tracked technical debt;
- generated documentation sync;
- local and CI parity;
- dependency hygiene;
- duplicate-code detection;
- clear file and naming conventions;
- agent-instruction drift checks.

## Why this matters

Coding agents often create avoidable mess when the repo does not enforce boundaries.

Common failure modes:

- one huge file instead of clean modules;
- one huge function instead of readable steps;
- duplicate copied helpers;
- random TODO comments with no owner;
- new dependencies added casually;
- API routes changed without docs;
- tests passing locally but CI running different checks;
- stale agent instructions pointing to commands or files that no longer exist.

This plan pack prevents those problems by making codebase cleanliness part of the system, not just part of the prompt.

## Relationship to existing Agent OS plans

This pack should work alongside:

- managed worker conversation plan;
- backend quality control;
- project context bundle;
- role profiles and worker contracts.

The managed worker can do coding work.

The backend quality-control layer can decide if a run result is valid.

This codebase hygiene layer decides whether the repository remains maintainable after changes land.

## Target outcome

A future implementing agent should be able to open this folder and understand:

- what repo hygiene systems are required;
- where they should live in the Agent OS codebase;
- which checks should be included in the canonical verification command;
- which checks block merge/readiness;
- which checks only report status;
- how the frontend should display hygiene failures;
- how agents should be guided to fix hygiene failures without bypassing them.

## Reading order

1. `system-map.md`
2. `agent-os-codebase-integration-guide.md`
3. `implementation-sequence.md`
4. `front-end-and-back-end-result-view.md`
5. `acceptance-checklist.md`
6. the files under `systems/`

## Important implementation note

This is not an LLM-only system.

The worker may receive guidance like "keep files small" and "do not add dependencies without reason," but the actual guardrails must be implemented as code.

The canonical quality command and its underlying checks should be deterministic backend/tooling code.