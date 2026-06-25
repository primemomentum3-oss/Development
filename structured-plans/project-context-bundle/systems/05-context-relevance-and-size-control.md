# 05 - Context Relevance and Size Control

## Purpose

Context Relevance and Size Control keeps the context bundle focused.

More context is not always better. Too much context can confuse the worker.

## Relevance signals

Start with simple signals:

```text
same project
same task area
same file path or folder
same worker role
same failure type
same verification command
recentness
approved memory only
```

## Size rules

```text
prefer short summaries
link raw logs as artifacts
limit repeated facts
deduplicate similar memory
exclude stale or unrelated context
keep source links
```

## Bundle size levels

Suggested levels:

```text
small: direct task and required proof only
standard: task plus approved relevant memory
expanded: includes recent failures and reports
manual: Stig-selected context
```

## Acceptance criteria

- raw logs are linked, not pasted fully.
- repeated memory is deduplicated.
- irrelevant historical context is excluded.
- bundle shows what was excluded and why.