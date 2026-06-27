# 07 - Role Boundary Profiles

## Plain meaning

Role Boundary Profiles define what each kind of worker is allowed to do.

A builder, reviewer, fixer, verifier, and closer should not all have the same freedom.

## What it does

It separates work responsibilities.

Examples:

```text
Builder: may edit allowed files and create a patch.
Reviewer: may inspect work and evidence, but should not edit code.
Fixer: may only repair a specific failed check or review finding.
Verifier: runs checks and records results.
Closer: prepares the final summary and approval package.
Stig: gives final approval.
```

## How it works

When Agent OS starts a managed run, it chooses a role profile.

The role profile influences:

- the instruction packet.
- what tools/actions are allowed.
- what evidence is required.
- what final state the worker can request.
- what the worker is not allowed to do.

## What it requires

- A small set of stable role names.
- Role-specific rules.
- UI labels showing which role a run has.
- Backend policy checks where possible.
- Clear distinction between provider and role.

## Who owns it

Agent OS owns the role definitions and applies them.

The agent acts inside the assigned role.

## Terminal-based feasibility

Fully feasible with terminals.

The role can be expressed in the launch packet and enforced by backend checks around files, commands, state changes, and evidence.

## Why it matters

This prevents a worker from acting like a free-roaming general agent.

It makes Agent OS safer and easier to reason about.