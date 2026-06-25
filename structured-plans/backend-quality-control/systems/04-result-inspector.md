# 04 - Result Inspector

## Purpose

The Result Inspector is the backend closing pass after a managed worker exits.

It decides what actually happened based on files, process result, verification, evidence, and task rules.

## Plain-language meaning

When the worker says:

```text
I am done.
```

Agent OS should say:

```text
I will check.
```

## When it runs

Run the Result Inspector whenever a managed worker leaves active execution:

```text
normal exit
crash
timeout
stop by Stig
stop by system
repair pass exit
```

## Inputs

The inspector should load:

```text
managed run record
work brief
task
worktree path
process exit code
log artifact
changed files
patch artifact if any
command evidence
verification requirements
review requirements
approval requirements
scope rules
```

## Inspection steps

### Step 1 - process outcome

Check:

```text
exit code
signal
killed or not
timed out or not
stopped by whom
```

### Step 2 - changed files

Check Git/worktree state:

```text
changed files
untracked files
deleted files
staged files
unstaged files
files outside allowed scope
protected files touched
```

### Step 3 - patch/package result

For implementation work, require a real patch or equivalent change artifact.

Block if:

```text
no patch exists
workspace dirty but changes not packaged
patch exists but no changed files
changed files exist but patch artifact missing
```

### Step 4 - verification evidence

Check whether required commands actually ran and passed.

Do not accept terminal text alone.

Required proof should come from command evidence records, exit codes, and logs.

### Step 5 - review and approval requirements

Decide next state:

```text
ready_for_review
ready_for_approval
blocked
needs_repair
failed
```

## Outcome rules

Examples:

```text
exit code non-zero -> needs_repair or failed
no changes for implementation task -> blocked:no_patch_produced
changed files outside scope -> blocked:out_of_scope_changes
protected file changed -> blocked:protected_file_changed
verification missing -> blocked:missing_verification
verification failed -> needs_repair:verification_failed
verification passed + review required -> ready_for_review
review approved + Stig approval required -> ready_for_approval
```

## Output

The Result Inspector should produce:

```text
official outcome
plain reason
next recommended action
inspection report artifact
run event
updated run state
updated task state if applicable
```

## Implementation shape

Suggested function:

```text
inspectManagedRunResult(runId)
```

Suggested internal structure:

```text
loadContext()
inspectProcessOutcome()
inspectChangedFiles()
inspectPatch()
inspectVerification()
inspectReviewApprovalNeeds()
classifyOutcome()
writeInspectionReport()
transitionState()
```

## What not to do

Do not let the worker set the result.

Do not skip inspection after crash/timeout.

Do not treat a successful process exit as successful work.

Do not mark ready if proof is missing.

## Acceptance criteria

- a successful process exit with no patch is blocked.
- a worker claim of done is ignored unless evidence supports it.
- failed verification leads to needs_repair.
- out-of-scope changes are blocked.
- inspection report is saved and visible to Stig.