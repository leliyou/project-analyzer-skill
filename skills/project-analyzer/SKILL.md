---
name: project-analyzer
description: Use when analyzing a local code repository and producing architecture diagrams, code logic flow diagrams, and stage-by-stage mock data examples. Trigger when the user wants to understand an unfamiliar project, document system structure, explain execution flow, or generate markdown docs under the target project's docs directory.
---

# Project Analyzer

## Overview

Use this skill to inspect a local project, identify its entrypoints and major modules, trace one or more core execution paths, and generate markdown documentation that explains the project structure in a reusable format.

Default behavior:

- If the user does not provide a path, analyze the current working directory
- If the user provides a path, analyze that target project directory
- Create a `docs/` directory under the target project if it does not exist
- Write concrete markdown output files instead of replying only in chat
- If the user explicitly specifies an output language, generate the docs in that language
- If the user does not specify an output language, infer the preferred language from existing docs in the target repository when possible

Primary deliverables:

- `docs/project-overview.md`
- `docs/architecture-analysis.md`
- `docs/mock-data-stages.md`

If the project is very small, it is fine to produce only the latter two files, but the default should be all three.

Important style rule:

- If the target repository already contains analysis docs, architecture drafts, `.txt` diagram samples, or earlier generated files, inspect them before writing new docs
- Prefer inheriting the target repository's existing language, heading style, diagram density, indentation, connector characters, and section naming when those examples are clearly intentional
- Do not treat existing analysis files as ground truth for technical facts without verification, but do treat them as strong style guidance for presentation

Important language rule:

- If the user explicitly asks for Chinese, English, 中文, 英文, or similar wording, that explicit language choice overrides the repository default
- If the user says bilingual or asks for two languages, prefer asking or generating one language unless they clearly want duplicated docs
- If no language is specified, follow the target repository's dominant existing documentation language

## When To Use

Use this skill when the user asks for any of the following:

- Analyze the current project
- Analyze a repository at a given path
- Generate a program architecture diagram
- Generate a code logic diagram
- Explain module responsibilities and call chains
- Show what data looks like at each stage of a workflow
- Produce mock input/output structures for a project flow
- Write project analysis docs into the repository

Do not use this skill for:

- High-level product brainstorming without a codebase
- A narrow code review that only needs bug findings
- Editing a single function without broader analysis

## Workflow

### 1. Resolve The Target Project

Determine the target directory from the user request.

- If a path is provided, use that path
- Otherwise use the current working directory

Confirm the resolved target in your progress update.

Also resolve the requested output language from the user's instruction.

- Examples: `用中文生成`, `generate in English`, `文档输出为中文`
- If the user gives no explicit preference, infer from repository docs
- State the resolved language in your progress update

### 2. Inspect The Repository Shape

Start with lightweight repository inspection:

- List top-level files and directories
- Find likely entrypoints
- Identify config, dependency, and deployment files
- Identify framework and runtime

Prefer fast local inspection with shell tools:

- `rg --files`
- `rg -n`
- `sed -n`
- `find` when needed

Focus on files that explain structure:

- `README*`
- dependency manifests
- service entrypoints
- routing files
- worker/task files
- config files
- deployment files

Also look for project-local style references before generating docs:

- existing `docs/*.md` analysis files
- root-level `*architecture*.md`, `*mock*stage*.md`, `*.txt`, or diagram drafts
- hand-written onboarding notes that show preferred terminology or diagram formatting

If such files exist, summarize:

- preferred document language, such as Chinese vs English
- whether diagrams are compact or highly annotated
- whether function names are shown in the diagrams
- whether module grouping uses swimlanes, sections, or layered blocks

If the user explicitly requested a language, do not let repository style references override that language choice. Only inherit the formatting style.

### 3. Identify The Main Execution Paths

Infer one or more core flows the user would care about. Common examples:

- HTTP request to handler to service to storage
- CLI command to orchestrator to worker
- queue consumer to processor to downstream systems
- UI action to API to persistence

For each important flow, identify:

- Trigger or entrypoint
- Validation or preprocessing
- Core business logic
- Integration boundaries
- Output or side effects

Be explicit when something is inferred from code structure rather than directly proven.

When the repository already has older analysis docs:

- preserve the same primary execution path if it is still valid
- update file names, call chains, and responsibilities only where the code proves a change
- prefer continuity of wording and diagram structure over inventing a new format

### 4. Build Three Output Documents

Write the following files into the target project's `docs/` directory.

#### `project-overview.md`

Keep this concise and practical. Include:

- What the project appears to do
- Tech stack and runtime
- Main directories and files
- Entry points
- External dependencies or services

#### `architecture-analysis.md`

Include:

- Program architecture diagram in plain-text code block
- Code logic diagram in plain-text code block
- Main modules and responsibilities
- Key call chains

If the target repository already has a recognizable architecture-doc style, match it closely. In particular:

- keep the same document language when the repository clearly prefers one language
- if existing diagrams use swimlane-like grouping such as `HTTP API`, `Celery Worker`, `核心补丁引擎`, preserve that pattern
- if existing diagrams annotate functions such as `require_auth()`, `_get_origin()`, `write_data()`, include that level of detail when the code supports it
- if existing diagrams use wide separators, arrows, inline comments, or layered sections, prefer that richer style over a minimal generic tree
- if there is both an old `architecture_analysis.md` and a new `docs/architecture-analysis.md`, use the old file as style guidance and the current code as fact source

Language precedence for all generated docs:

1. Explicit user instruction about output language
2. Existing target-project documentation language
3. Skill repository examples and defaults

Prefer diagrams that look like:

```text
User
  │
  ├──── API
  │      ▼
  │    app.py
  │      ├── validate()
  │      └── service.py
  │             └── db.py
```

And:

```text
┌────────────┐
│ entrypoint │
└─────┬──────┘
      ▼
  business logic
      ▼
   downstream
```

But do not stop at these minimal examples when the repository already demonstrates a richer house style.

#### `mock-data-stages.md`

This file is required for this skill.

For at least one important project flow, provide:

- Stage name
- What code/component owns that stage
- Input shape
- Output shape
- Mock JSON or structured demo data
- Notes on fields and meaning

Good stages often include:

- User input or request body
- Internal normalized object
- Database query result
- Transformed object
- Write payload
- Final response

If the project does not have obvious JSON payloads, adapt the examples to the real structure:

- CLI args
- environment config
- queue messages
- ORM objects
- rendered templates

### 5. Favor Concrete Outputs Over Generic Commentary

Do not write vague summaries like "the project has several modules." Instead, name the files and functions.

Do not say "for example" when the actual structure is available from the code. Use real names whenever possible.

When mock data is partially inferred, label it clearly:

- `Code-confirmed structure`
- `Illustrative mock values`

For architecture and logic diagrams:

- prefer real function names over generic labels like "validate input" when the function is visible in code
- prefer real queue names, config files, and service names when they are statically discoverable
- include module layering or operational scripts if the repository's existing docs highlight them
- avoid flattening a richly structured backend into a simplistic four-box flow

### 6. Explain Uncertainty Honestly

If some behavior depends on runtime configuration, network systems, feature flags, or code that is not present, say so explicitly.

Examples:

- "Downstream aggregation targets are loaded dynamically from ZK, so this section is inferred from the static call path."
- "This response shape is confirmed from the serializer, but the exact field values below are mock examples."

## Output Template Guidance

### Minimal `project-overview.md`

Include these sections:

- `# Project Overview`
- `## What It Does`
- `## Tech Stack`
- `## Main Entry Points`
- `## Directory Highlights`
- `## External Systems`

### Minimal `architecture-analysis.md`

Include these sections:

- `# Architecture Analysis`
- `## Program Architecture Diagram`
- `## Code Logic Diagram`
- `## Module Responsibilities`
- `## Key Execution Paths`

### Minimal `mock-data-stages.md`

Include these sections:

- `# Mock Data Stages`
- `## Flow Chosen`
- `## Stage 1 ...`
- `## Stage 2 ...`
- `## Stage N ...`
- `## Summary Of Data Shapes`

## File Writing Rules

- Always create the target project's `docs/` directory if needed
- Prefer creating or updating only the three analysis files
- Do not overwrite unrelated docs
- If a same-named analysis file already exists, update it only if the user is clearly asking for regeneration; otherwise create a timestamped variant or ask if overwrite is important

When existing non-`docs/` analysis files are present, such as:

- `architecture_analysis.md`
- `mock_data_stages.md`
- `.text.txt`

use them as formatting references, not as output targets, unless the user explicitly asks to regenerate those exact files.

When regenerating docs in a user-requested language:

- keep filenames unchanged unless the user asks for separate bilingual variants
- translate headings, connective prose, and explanatory notes into the requested language
- keep source-code identifiers, file names, queue names, and config keys in their original form

## Investigation Tips

- Start from entrypoints and trace inward
- Use config files to map environments, ports, queues, and databases
- Use worker/task registration files to locate async flows
- Use helper libraries to identify storage and integration layers
- Use logs, example payloads, and inline comments when available
- Compare old generated docs against current code before replacing them
- If a sample file already contains a better-looking diagram style, imitate the style but refresh the factual content from source code

## References

Read the following only as needed:

- `references/output-spec.md` for the expected markdown file structure
- `references/analysis-checklist.md` for the repository analysis checklist
- `references/examples.md` for example phrasing and diagram patterns
