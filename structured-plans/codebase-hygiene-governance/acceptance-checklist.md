# Acceptance Checklist

This checklist defines when the Codebase Hygiene And Engineering Governance system is ready enough to implement or review.

## Global acceptance

- [ ] There is one canonical repository quality command.
- [ ] The same command is usable locally, in CI, and from managed worker result inspection.
- [ ] The command produces concise pass/fail output.
- [ ] Full logs are stored as artifacts when needed.
- [ ] A managed worker run cannot be marked ready when required hygiene gates fail.
- [ ] Each failure has a plain-language explanation.
- [ ] Each failure gives a clear re-run hint.

## Gate coverage

- [ ] Lint gate exists.
- [ ] Typecheck gate exists.
- [ ] Test gate exists.
- [ ] File-size guard exists.
- [ ] Function-size or complexity guard exists.
- [ ] Tracked debt marker guard exists.
- [ ] Generated surface sync check exists.
- [ ] Dependency hygiene check exists.
- [ ] Duplicate-code guard exists.
- [ ] Local/CI parity guard exists.
- [ ] Instruction drift guard exists.

## Ratchet behavior

- [ ] Existing exceptions can be grandfathered.
- [ ] New work cannot silently grow the exception list.
- [ ] Budget files are committed and reviewable.
- [ ] Stale budget entries fail.
- [ ] Lowering budgets is easy when the code improves.
- [ ] Raising budgets requires a recorded reason.

## Frontend visibility

- [ ] Task page shows codebase hygiene status.
- [ ] Managed run page shows quality gate status.
- [ ] Failed gates are listed by name.
- [ ] Each failed gate has a short explanation.
- [ ] Full logs are reachable but not shown first.
- [ ] Stig can see whether a task is blocked by hygiene, verification, review, or approval.

## Worker behavior

- [ ] Worker contracts tell workers to run the canonical quality gate when required.
- [ ] Worker claims are not trusted without backend evidence.
- [ ] If a hygiene gate fails, Agent OS blocks readiness.
- [ ] If safe, Agent OS can create a narrow repair pass for the specific failure.
- [ ] Workers are not allowed to bypass the gate by changing budgets without explicit approval.

## Documentation quality

- [ ] Agent-facing guidance names the canonical quality command.
- [ ] Developer docs explain each gate.
- [ ] Budget files explain how to update them.
- [ ] Generated docs clearly state they are generated and should not be hand-edited.
- [ ] CI parity docs explain any intentional CI-only aliases.

## First milestone acceptance

The first useful version is complete when:

- [ ] one quality command exists;
- [ ] it runs lint, typecheck, tests, file-size guard, and debt-marker guard;
- [ ] backend result inspection can call it;
- [ ] frontend shows pass/fail status;
- [ ] failed runs cannot be marked ready without explicit override.