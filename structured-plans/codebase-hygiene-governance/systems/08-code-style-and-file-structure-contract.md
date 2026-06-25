# 08 Code Style And File Structure Contract

## Purpose

The Code Style And File Structure Contract defines how Agent OS code should be organized.

It gives workers clear rules for where code belongs and what names should look like.

## Problem it solves

Without explicit structure rules, workers may create inconsistent files and folders.

Examples:

- new files in random folders;
- mixed casing styles;
- tests far away from implementation;
- helpers placed in unrelated modules;
- types scattered across the repo;
- booleans and functions named unclearly.

## Rules to define

Agent OS should document and enforce rules for:

- file naming;
- directory naming;
- test placement;
- module boundaries;
- type placement;
- import style;
- constant naming;
- boolean naming;
- frontend component naming;
- backend service naming.

## Suggested defaults

These are suggested starting rules, not final requirements:

- backend files: kebab-case;
- backend folders: kebab-case;
- tests: next to the unit under test;
- React components: PascalCase;
- helpers/hooks: kebab-case or existing repo convention;
- variables/functions: camelCase;
- types/classes/components: PascalCase;
- true constants: SCREAMING_SNAKE_CASE;
- booleans: predicate names such as `isOpen`, `hasEvidence`, `canRun`.

## Enforcement

Where possible, enforce through lint rules or custom checks.

Where enforcement is not easy, document the rule and add review checklist items.

## Backend integration

The canonical quality gate should include style and naming checks.

A managed worker run should be blocked if it introduces files that violate the contract.

## Frontend behavior

Show:

```text
File structure contract failed.
New backend file `ManagedWorkerRunner.ts` should use kebab-case.
Suggested name: `managed-worker-runner.ts`.
```

## Acceptance

- file naming rules are documented;
- folder naming rules are documented;
- tests placement is documented;
- violations fail where tooling supports it;
- workers receive these rules through role contracts and context bundles.