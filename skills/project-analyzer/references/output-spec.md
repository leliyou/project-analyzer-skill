# Output Spec

This file defines the expected deliverables for `project-analyzer`.

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
