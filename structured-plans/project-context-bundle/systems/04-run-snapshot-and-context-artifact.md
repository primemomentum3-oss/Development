# 04 - Run Snapshot and Context Artifact

## Purpose

The Run Snapshot and Context Artifact records exactly what context was given to the worker at launch time.

## Plain meaning

After a run, Stig and future agents should be able to see:

```text
This is what the worker knew when it started.
```

## Required artifacts

```text
context-bundle.json
context-bundle.md
source-list.json
warnings.json
```

## Required metadata

```text
bundle id
run id
task id
role
created at
source count
approved memory count
excluded memory count
warnings
hash/checksum
```

## Rules

The bundle must be frozen before launch.

The worker must not be able to rewrite the historical bundle.

If context changes later, create a new bundle version for a new run.

## Acceptance criteria

- every managed run can show its context bundle.
- bundle content is immutable after launch.
- bundle includes source list and warnings.
- repeated runs can compare different bundle versions.