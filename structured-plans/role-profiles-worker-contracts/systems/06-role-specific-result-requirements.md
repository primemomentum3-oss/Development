# 06 - Role-Specific Result Requirements

## Purpose

Role-Specific Result Requirements define what each role must produce before Agent OS can move the run forward.

## Requirements by role

### Builder

Must produce:

```text
patch artifact
changed files inside scope
required verification evidence
summary of changes
```

### Reviewer

Must produce:

```text
review record
finding list
approve or request changes
no source edits
```

### Fixer

Must produce:

```text
focused patch
link to original failure
rerun failed check
no unrelated changes
```

### Verifier

Must produce:

```text
command evidence
exit code
log artifact
no source edits
```

### Closer

Must produce:

```text
final report
approval package
open risks
no final approval by itself
```

## Result inspector integration

The Result Inspector should call role-specific checks after normal patch/scope/verification checks.

## Acceptance criteria

- builder without patch is blocked.
- reviewer that edits source is blocked.
- fixer that changes unrelated files is blocked.
- verifier without command evidence is blocked.
- closer cannot mark final done.