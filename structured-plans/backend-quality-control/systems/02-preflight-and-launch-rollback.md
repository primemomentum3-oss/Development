# 02 - Preflight and Launch Rollback

## Purpose

Preflight and Launch Rollback prevents Agent OS from starting unsafe or half-prepared worker runs.

It also makes failed launches understandable instead of leaving broken partial state.

## Plain-language meaning

Before Agent OS starts Codex, Claude, or another terminal worker, it should check:

```text
Is everything ready?
If not, do not start.
If start fails halfway, clean up and record why.
```

## Preflight checks

Before launch, Agent OS should verify:

```text
Task exists.
Task is not stopped or done.
Work brief exists.
Work brief is locked.
Role is known.
Worker adapter is configured.
Project/repo exists.
Worktree exists or can be created.
Allowed scope is known for implementation work.
Required proof/checks are known when needed.
Terminal command is available.
Environment is safe.
No duplicate active run exists for the same task/role unless allowed.
```

## Launch sequence

Recommended order:

```text
1. Create managed run record in preparing state.
2. Run preflight checks.
3. Create/confirm worktree.
4. Write locked work brief artifact.
5. Build worker command and safe environment.
6. Transition to starting.
7. Start terminal process.
8. Save PID/process group.
9. Transition to running.
10. Append worker.started event.
```

## Rollback behavior

If anything fails before the worker is running:

```text
mark run failed_before_start
record failure reason
save setup log
release any acquired lease
remove empty temporary worktree if safe
leave dirty/non-empty worktree alone and report it
append launch.rollback event
```

## Failure examples

Use plain labels such as:

```text
worker_command_missing
worktree_create_failed
brief_write_failed
environment_invalid
duplicate_active_run
permission_denied
unknown_project
```

## Implementation shape

Suggested service:

```text
startManagedWorkerRun(input)
```

Internally:

```text
const run = await createRun(preparing)
try {
  await preflight(run)
  await prepareWorktree(run)
  await writeBrief(run)
  await transition(starting)
  const process = await launchWorker(run)
  await attachProcess(run, process)
  await transition(running)
} catch (err) {
  await rollbackLaunch(run, err)
}
```

## What not to do

Do not start a worker before creating a run record.

Do not create a worktree and then forget to attach it to a run.

Do not hide launch errors inside terminal output only.

Do not treat failed launch as agent failure. It is a system/setup failure.

## Acceptance criteria

- a missing worker command produces `failed_before_start`.
- no process is started when preflight fails.
- launch failure is visible in Agent OS UI.
- partial resources are cleaned up or reported.
- every launch attempt has a run record and event history.