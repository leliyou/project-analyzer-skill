# Output Spec

This file defines the expected deliverables for `project-analyzer`.

## Language Selection

Default rule:

- If the user explicitly requests an output language, use that language for generated docs
- Treat short aliases such as `zh`, `cn`, `zh-CN`, `en`, and `en-US` as explicit language requests
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
docs/code-call-graph.md
docs/mock-data-stages.md
```

## Metadata Header

Each generated markdown file should start with a short metadata block immediately below the title.

Recommended shape:

```markdown
> generated_by: project-analyzer
> verified_at: 2026-04-09 14:30 Asia/Shanghai
> provenance: static code inspection + config reading + target-repo style inheritance
> coverage: request flow, worker flow, RESET, TAG_RESET
> evidence_gap: runtime ZK payload is not present in the repository
```

Guidance:

- `generated_by` should be constant for the skill
- `verified_at` should be concrete, not relative wording like "today"
- `provenance` should say what evidence sources were actually used
- `coverage` should tell the reader what the document explicitly covers
- `evidence_gap` is optional, but should be present when important facts are inferred or partial

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
- Mention major covered systems and any obvious evidence gaps in the metadata block

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
- If the repository uses inline responsibility notes after function names, such as `← query source data` or `← 回写 InfluxDB`, keep that pattern in the generated diagram
- In the program architecture diagram, each important file line should preferably include a short role note, inline comment, or child-method explanation
- In the code logic diagram, each major box should show concrete methods or numbered internal steps instead of only a high-level title
- In the code logic diagram, prefer `path:line method()` when line numbers are easy to derive from static inspection
- If a method internally calls other important helpers, list those helper calls inside the box when they materially explain the flow
- Do not infer branches only from endpoint or method names; branches should follow the actual helper calls, returned values, and downstream dependencies visible in code
- If a helper call returns data that determines later behavior, show that helper as a branch or downstream node when it materially affects understanding
- Keep monospace box borders visually aligned; avoid diagrams whose right edges drift line by line
- Prefer continuity with existing house style over inventing a brand-new diagram layout
- Prefer detailed annotations on important files, methods, and edges; notes should explain inputs, outputs, decisions, side effects, or dependency usage when code makes them visible
- When a note contains opaque project jargon or infrastructure identifiers, preserve the identifier but add a plain-language explanation of what it means in the current flow
- Prefer wording that a new teammate can understand without already knowing the repository's jargon

Style inheritance order:

1. Existing target-project analysis docs and diagram samples
2. Existing target-project README or onboarding docs
3. The examples in this skill repository
4. Minimal generic fallback

Language resolution order:

1. Explicit user request
2. Existing target-project docs
3. Skill default examples

Alias mapping guidance:

- `zh`, `cn`, `zh-CN` => Chinese
- `en`, `en-US` => English

## `code-call-graph.md`

Purpose:

- Show caller/callee relationships as a dedicated document

Recommended outline:

```markdown
# Code Call Graph

## Overview Call Graph

```text
...
```

## Call Chain 1: ...

```text
...
```

## Call Chain 2: ...

```text
...
```

## Runtime Dependency Chain

```text
...
```

## Notes On Inference
```

Call graph guidance:

- Prefer dedicated caller -> callee relationships over broad architectural summaries
- Use real file/module/function names
- Include helper calls when they materially change control flow or returned values
- Prefer `path:line method()` when line numbers are statically available
- Separate independent entry chains when the project has API, worker, scheduler, or CLI entrypoints
- Add short inline notes for important lines when static inspection makes the role clear
- Prefer annotating each important line in the call graph, not only the first line of each chain
- Prefer annotating nearly every non-connector line in the call graph so the flow is readable without mentally filling in unlabeled nodes
- If helper-returned data determines later branching, show that helper and the resulting branch explicitly
- Prefer notes that explain what data is returned, what condition is checked, what side effect is triggered, or why the caller depends on that callee
- Prefer flow-oriented sections that a human can read top-down, such as request submission, status lookup, worker execution, and runtime dependency refresh
- Show branch points explicitly and label what causes the branch, such as request type, task state, helper return value, or strategy selection
- For important edges, explain both the action and the purpose instead of only naming the callee
- When a value produced by one call is consumed by the next step, mention that handoff in the annotation when it is statically visible
- Aim for enough annotation that a reader can follow the main path without reopening every referenced source file
- If a queue, lock, watcher, cache, or broker name appears, explain what role that primitive plays in the current path instead of assuming reader familiarity
- If static analysis cannot prove a call edge, label it as inferred
- Use explicit `unknown:` or `evidence gap:` wording when a reader might otherwise mistake an inference for a confirmed fact

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
- Prefer stage-by-stage payload snapshots over one large undifferentiated example
- For each meaningful stage, include code owner, what changed, and why that output shape exists
- Add field-structure notes when the payload shape mixes variants, nested keys, or runtime-enriched fields
- When runtime configuration, external services, or scheduler data fill part of the payload, label the resulting mock as inferred or illustrative
- Detect primary request types, task modes, or dispatch branches from code before selecting stages
- If two to four top-level types are clearly supported, give each major type its own end-to-end stage chain rather than documenting only the richest branch
- Shared early stages may be referenced briefly, but each major type should still show its own later-stage payload evolution and final output shape
- Only summarize a type instead of fully expanding it when there are more than four materially different types or when code shows it is just an alias of another path
- If any major type is summarized, explicitly explain why and still show its request shape, branch point, and final payload difference
- Use explicit `unknown:` or `evidence gap:` wording when runtime configuration or external systems determine part of the payload shape

## Regeneration Behavior

When the user asks to regenerate docs:

- Update existing generated docs if they appear to be analysis outputs
- Avoid touching unrelated hand-written docs
- When regenerating multiple files, prefer drafting to a temporary docs subdirectory and replacing outputs as a batch instead of leaving half-updated files behind
- If unsure, create a suffix variant such as:

```text
docs/architecture-analysis-v2.md
docs/code-call-graph-v2.md
docs/mock-data-stages-v2.md
```

When the target project contains both legacy analysis files and `docs/` outputs:

- use the legacy files as style references
- use the current codebase as the authority on technical content
- refresh the `docs/` outputs so they read like the legacy docs, not like unrelated generic templates
