# 06 - Verification Evidence Gate

## Purpose

The Verification Evidence Gate ensures that required checks were actually run and passed.

It prevents a worker from claiming success without proof.

## Plain-language meaning

Agent OS should ask:

```text
What check was required?
Was it actually run?
What was the exit code?
Where is the log?
Did it pass?
```

The answer must come from backend evidence, not from agent text.

## Required inputs

The gate should know:

```text
task type
required verification command(s)
command evidence records
exit codes
logs/artifacts
when command was run
which worktree command ran in
which run generated the evidence
```

## Verification levels

Suggested levels:

```text
none_required
manual_check_required
local_command_required
review_required
ci_required_later
```

Implementation work should usually require at least local command evidence.

## Checks

### Check 1 - command existed

Block if required command was never run.

### Check 2 - command belonged to this run/task

Block if the only passing command came from another task or old run.

### Check 3 - command ran in correct worktree

Block if verification ran in the wrong checkout.

### Check 4 - command passed

Block or repair if exit code was non-zero.

### Check 5 - evidence is fresh

Block if verification happened before the final file changes.

## Result labels

```text
verification_passed
verification_missing
verification_failed
verification_stale
verification_wrong_worktree
verification_from_wrong_run
manual_verification_needed
```

## Implementation shape

Suggested function:

```text
checkVerificationEvidence({ taskId, runId, requiredChecks })
```

Return:

```text
{
  status: "passed" | "missing" | "failed" | "stale" | "manual_needed",
  requiredChecks: [...],
  evidenceFound: [...],
  missingChecks: [...],
  failedChecks: [...],
  reason,
  nextAction
}
```

## Freshness rule

This is important.

A test pass before the final code change should not count as proof for the final result.

Agent OS should compare:

```text
last relevant file change time
verification command completed time
```

If verification is older than the final change, mark it stale.

## What not to do

Do not accept:

```text
The worker said tests passed.
```

Do not accept a pasted summary without command evidence.

Do not accept old test evidence from before the final edit.

Do not allow verification to be skipped silently.

## Acceptance criteria

- missing required verification blocks readiness.
- failed verification leads to needs_repair.
- stale verification blocks readiness.
- verification from wrong worktree does not count.
- Stig can see which proof is missing.