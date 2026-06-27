# 02 - Contract Builder and Locking

## Purpose

The Contract Builder creates a run-specific worker contract and locks it before launch.

## Inputs

```text
role profile
task
locked work brief
context bundle
scope rules
required verification
approval rules
```

## Output

A worker contract with:

```text
role
job goal
allowed actions
forbidden actions
required proof
expected output
approval limits
artifact links
```

## Locking rules

The contract must be frozen before launch.

If the task changes, create a new contract version for the next run.

The worker must not rewrite its own historical contract.

## Artifacts

Store:

```text
worker-contract.json
worker-contract.md
```

## Acceptance criteria

- every managed run has a contract id.
- contract is created before launch.
- contract is immutable after launch.
- Stig can inspect the contract later.