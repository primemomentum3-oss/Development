# 05 - Contract Validation and Safety Gate

## Purpose

Contract Validation and Safety Gate checks the worker contract before launch.

## What it validates

Reject contracts that:

```text
omit role
omit task goal
omit allowed scope for implementation work
omit required proof
allow self approval
allow final done
allow reviewer to edit source
allow fixer to broaden scope
allow protected-file edits without approval
```

## Validation result

Return:

```text
valid: yes/no
errors
warnings
required Stig decisions
safe to launch: yes/no
```

## Safety gate behavior

If contract is invalid:

```text
do not launch worker
record contract validation failure
show plain reason
allow regenerate/edit/ask Stig
```

## Acceptance criteria

- unsafe contracts cannot launch.
- validation errors are readable.
- launch path requires valid locked contract.
- validation is deterministic and testable.