# Output Spec

This file defines the expected deliverables for `project-analyzer`.

## Language Selection

Default rule:

- If the user explicitly requests an output language, use that language for generated docs
- Otherwise infer the preferred language from the target repository's existing docs

Apply the chosen language to:

- document titles
- section headings
- explanatory prose
- notes on uncertainty

Do not translate:

- file paths
- function names
- class names
- queue names
- config keys
- JSON field names unless the source project itself uses translated names

## Required Outputs

Write analysis files under:

```text
<target-project>/docs/
```

Default files:

```text
docs/project-overview.md
docs/architecture-analysis.md
docs/mock-data-stages.md
```

## `project-overview.md`

Purpose:

- Give a fast, high-signal summary of the repository

Recommended outline:

```markdown
# Project Overview

## What It Does

## Tech Stack

## Main Entry Points

## Directory Highlights

## External Systems
```

Content guidance:

- Keep it concise
- Use real file names and paths
- Prefer "what exists" over speculation

## `architecture-analysis.md`

Purpose:

- Explain structure and execution flow in a human-friendly way

Recommended outline:

```markdown
# Architecture Analysis

## Program Architecture Diagram

```text
...
```

## Code Logic Diagram

```text
...
```

## Module Responsibilities

## Key Execution Paths
```

Diagram guidance:

- Keep diagrams monospace-friendly
- Prefer vertical flow
- Use real module or file names
- Avoid over-detailing every helper unless it matters to flow
- If the target repository already contains architecture or logic diagrams, inherit their style before falling back to a generic format
- Match the repository's preferred language when existing docs make it obvious
- If the repository uses grouped sections such as API / Worker / Core Engine / Ops, preserve that structure
- If the repository uses rich inline annotations such as queue names, helper functions, or config files, keep them when code confirms them
- Prefer continuity with existing house style over inventing a brand-new diagram layout

Style inheritance order:

1. Existing target-project analysis docs and diagram samples
2. Existing target-project README or onboarding docs
3. The examples in this skill repository
4. Minimal generic fallback

Language resolution order:

1. Explicit user request
2. Existing target-project docs
3. Skill default examples

## `mock-data-stages.md`

Purpose:

- Show how data changes shape across the project

Recommended outline:

```markdown
# Mock Data Stages

## Flow Chosen

## Stage 1: ...

### Code owner

### Input shape

### Output shape

### Mock data

## Stage 2: ...
...

## Summary Of Data Shapes
```

Mock guidance:

- Use real field names when code confirms them
- Mark mock values as illustrative
- Distinguish code-confirmed structure from inferred demo content

## Regeneration Behavior

When the user asks to regenerate docs:

- Update existing generated docs if they appear to be analysis outputs
- Avoid touching unrelated hand-written docs
- If unsure, create a suffix variant such as:

```text
docs/architecture-analysis-v2.md
docs/mock-data-stages-v2.md
```

When the target project contains both legacy analysis files and `docs/` outputs:

- use the legacy files as style references
- use the current codebase as the authority on technical content
- refresh the `docs/` outputs so they read like the legacy docs, not like unrelated generic templates
