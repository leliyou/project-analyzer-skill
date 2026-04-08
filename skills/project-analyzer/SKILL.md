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

Primary deliverables:

- `docs/project-overview.md`
- `docs/architecture-analysis.md`
- `docs/mock-data-stages.md`

If the project is very small, it is fine to produce only the latter two files, but the default should be all three.

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

## Investigation Tips

- Start from entrypoints and trace inward
- Use config files to map environments, ports, queues, and databases
- Use worker/task registration files to locate async flows
- Use helper libraries to identify storage and integration layers
- Use logs, example payloads, and inline comments when available

## References

Read the following only as needed:

- `references/output-spec.md` for the expected markdown file structure
- `references/analysis-checklist.md` for the repository analysis checklist
- `references/examples.md` for example phrasing and diagram patterns
