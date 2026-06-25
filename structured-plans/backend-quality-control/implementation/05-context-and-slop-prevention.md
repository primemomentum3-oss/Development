# 05 - Context and AI Slop Prevention

Date: 2026-06-25
Status: implementation guidance
Audience: Agent OS implementation agent

## Purpose

This file explains how Agent OS should reduce AI slop and maintain codebase context for managed terminal workers.

This must be implemented through both:

```text
1. Better context before the worker starts.
2. Backend checks after the worker finishes.
```

Prompts alone are not enough.

## What AI slop means here

AI slop in coding work means results such as:

```text
vague broad changes
wrong architectural assumptions
unrelated refactors
fake success claims
missing tests
changed files outside scope
dependency changes without approval
low-quality patches
repeated mistakes from previous runs
ignoring project conventions
```

Agent OS should reduce this with context, boundaries, proof, review, and memory.

## Context bundle before every managed run

Every managed worker should receive a Context Bundle.

This is not just a prompt. It is a structured artifact created by Agent OS.

Recommended file:

```text
.agent-os/managed-runs/<run-id>/context-bundle.md
```

Recommended JSON snapshot:

```text
.agent-os/managed-runs/<run-id>/context-bundle.json
```

## Context Bundle contents

The bundle should include:

```text
1. Original Stig request
2. Locked work brief
3. Task status and acceptance criteria
4. Role profile
5. Allowed files/paths
6. Forbidden/protected files/paths
7. Required verification commands
8. Relevant project conventions
9. Relevant approved memory
10. Recent related evidence or failed attempts
11. Known risks
12. What the worker must not decide
```

## Context sources

Use existing Agent OS systems where possible:

```text
task record
validation contract
worktree record
file leases/scope rules
command evidence history
review findings
approved project memory
operator decisions
previous reports
project/repo settings
```

Do not invent a second memory system unless existing memory is insufficient.

## Approved memory vs candidate memory

Agent OS should distinguish:

```text
approved durable memory
candidate memory
run-local notes
raw agent comments
```

Only approved durable memory should be automatically placed into future Context Bundles.

Agents may suggest memory, but Agent OS/Stig/review should approve before promotion.

## Memory candidate examples

Good memory candidate:

```text
In this repo, use `npm run check` as the main verification command. `npm test` alone misses type errors.
```

Bad memory candidate:

```text
The agent felt like the login code is messy.
```

Good memory candidate:

```text
Do not edit `packages/api/generated/*` directly. Regenerate it using `npm run generate`.
```

## Context freshness

Context should not blindly grow forever.

Agent OS should prefer:

```text
current task facts
approved memory
recent related failures
project conventions
explicit Stig decisions
```

Avoid stuffing every old log into the prompt.

Use summaries and artifact links.

## Anti-slop backend gates

Context reduces mistakes before work.

Backend gates catch mistakes after work.

Required gates:

```text
Work Brief Safety Gate
Patch/Scope Checker
Dirty Work Checker
Verification Evidence Gate
Result Inspector
Failure Reason Classifier
Review/Approval Gate
```

## Work Brief Safety Gate

Before launch, check the brief includes:

```text
goal
role
allowed work
not allowed
required proof
success definition
approval rule
```

Reject or revise briefs that say vague things like:

```text
Make the app better.
Refactor as needed.
Fix everything.
Use your judgment everywhere.
```

## Patch/Scope Checker

After work, compare actual changed files to allowed scope.

This catches:

```text
unrelated refactors
silent dependency changes
protected file changes
broad cleanup edits
accidental generated-file edits
```

## Verification Evidence Gate

Require actual command evidence.

This catches:

```text
agent says tests pass but did not run them
agent ran wrong command
agent ran command before final edit
agent ran command in wrong worktree
```

## Result Inspector

The Result Inspector combines all backend facts and decides official state.

It should block:

```text
no patch
missing verification
failed verification
out-of-scope changes
dirty worktree
review required but missing
approval required but missing
```

## Review and Stig approval

Even when backend gates pass, important work should normally move to:

```text
ready_for_review
```

Then:

```text
ready_for_approval
```

Then Stig can approve final done.

Do not let a worker self-approve.

## How to maintain codebase quality over time

Agent OS should create a learning loop:

```text
managed run happens
backend inspects result
review finds issues
Stig approves or rejects
useful learning becomes memory candidate
approved memory enters future context bundles
future workers avoid same mistake
```

## Files to add for context/slop prevention

Recommended backend files:

```text
src/managed-workers/context-bundle.ts
src/managed-workers/context-sources.ts
src/managed-workers/work-brief-safety-gate.ts
src/managed-workers/memory-candidate-suggester.ts
src/managed-workers/slop-risk-classifier.ts
```

Recommended checks:

```text
src/managed-workers/checks/broad-change-checker.ts
src/managed-workers/checks/generated-file-checker.ts
src/managed-workers/checks/dependency-change-checker.ts
```

Recommended frontend cards:

```text
ContextBundlePreview.tsx
SlopRiskCard.tsx
MemoryCandidateCard.tsx
```

## Slop risk classifier

This should be a backend helper that flags risky patterns.

Inputs:

```text
changed files
diff size
file types
dependency files changed
generated files changed
number of unrelated directories changed
verification coverage
review findings
```

Possible labels:

```text
low_risk
broad_change
dependency_risk
generated_file_risk
missing_test_risk
large_diff_risk
unrelated_refactor_risk
```

This classifier should not reject all broad changes automatically, but it should surface them clearly and require review/approval.

## What the worker should see

The worker should see:

```text
You are working inside Agent OS.
Use the context bundle.
Follow the locked work brief.
Do not broaden scope.
Do not mark final done.
Run required verification.
Agent OS will inspect your result.
```

## What Stig should see

Stig should see plain results:

```text
Agent OS found a broad-change risk.
The worker edited 12 files across 5 unrelated areas.
This needs review before approval.
```

Or:

```text
Agent OS found a dependency-change risk.
package.json and package-lock.json changed, but dependency changes were not in the original scope.
Choose whether to allow this broader scope.
```

## Acceptance criteria

- every managed run has a context bundle artifact.
- context bundle includes approved memory and task rules.
- work brief safety gate rejects vague/unsafe briefs.
- dependency/generated/broad changes are flagged.
- useful lessons can become memory candidates.
- only approved memory enters future bundles.
- workers cannot self-approve quality.