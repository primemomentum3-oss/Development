# 03 - Permission Policy and Tool Boundaries

## Purpose

Permission Policy maps each role to what it can and cannot do.

The contract tells the worker. The permission policy lets Agent OS enforce the important parts.

## Permission examples

```text
builder may edit allowed files
reviewer may read and review but not edit source
fixer may edit only failure-related files
verifier may run checks but not edit source
closer may create report but not approve final done
```

## Permission questions

For each role, backend code should answer:

```text
Can this role edit files?
Can this role run commands?
Can this role create a patch?
Can this role create a review?
Can this role request approval?
Can this role mark done?
Can this role touch protected files?
```

## Backend enforcement

Connect permission policy to:

```text
file leases
scope checker
command policy
result inspector
approval system
memory promotion system
```

## Hard rule

No worker role can approve itself or mark important work finally done.

## Acceptance criteria

- forbidden role actions are rejected or blocked.
- result inspector can detect role violations.
- UI can show allowed and forbidden actions.