# 02 - Context Bundle Builder

## Purpose

The Context Bundle Builder assembles the context package for a managed worker run.

## Plain meaning

Before the worker starts, Agent OS builds a clean packet of relevant project information.

It should be structured, source-labeled, and saved as part of the run record.

## Required sections

```text
Task goal
Work brief summary
Worker role
Allowed scope
Required proof
Approved memory
Known commands
Relevant prior failures
Risks and warnings
Artifact links
```

## Build flow

```text
load task
load locked work brief
load role profile
load approved memory
load project commands
load recent relevant evidence and reports
apply source precedence
apply relevance filter
apply size control
save bundle artifact
attach bundle to run
```

## Output formats

Store both:

```text
context-bundle.json
context-bundle.md
```

JSON is for backend code. Markdown is for human inspection and worker launch context.

## Acceptance criteria

- a task can produce a structured bundle.
- bundle includes sources and trust labels.
- bundle is saved before worker launch.
- managed worker run stores bundle artifact id.