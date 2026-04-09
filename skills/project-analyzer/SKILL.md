---
name: project-analyzer
description: Use when analyzing a local code repository and producing architecture diagrams, code logic flow diagrams, code call graphs, and stage-by-stage mock data examples. Trigger when the user wants to understand an unfamiliar project, document system structure, explain execution flow, or generate markdown docs under the target project's docs directory.
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
- `docs/code-call-graph.md`
- `docs/mock-data-stages.md`

If the project is very small, it is fine to produce only the most relevant files, but the default should include all four.

Important style rule:

- If the target repository already contains analysis docs, architecture drafts, `.txt` diagram samples, or earlier generated files, inspect them before writing new docs
- Prefer inheriting the target repository's existing language, heading style, diagram density, indentation, connector characters, and section naming when those examples are clearly intentional
- Do not treat existing analysis files as ground truth for technical facts without verification, but do treat them as strong style guidance for presentation

Important language rule:

- If the user explicitly asks for Chinese, English, 中文, 英文, or similar wording, that explicit language choice overrides the repository default
- Treat short aliases such as `zh`, `cn`, `zh-CN`, `en`, and `en-US` as explicit language instructions when they appear in the user's skill invocation or request
- If the user says bilingual or asks for two languages, prefer asking or generating one language unless they clearly want duplicated docs
- If no language is specified, follow the target repository's dominant existing documentation language

## When To Use

Use this skill when the user asks for any of the following:

- Analyze the current project
- Analyze a repository at a given path
- Generate a program architecture diagram
- Generate a code logic diagram
- Generate a code call graph
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
- Also accept concise aliases such as `zh`, `cn`, `zh-CN`, `en`, `en-US`
- If the user gives no explicit preference, infer from repository docs
- State the resolved language in your progress update

Recommended interpretation:

- `zh`, `cn`, `zh-CN` => Chinese
- `en`, `en-US` => English

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

### 4. Build Four Output Documents

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
- if existing diagrams add inline responsibility notes after functions, such as `← 查 InfluxDB 源数据` or `← 回写 InfluxDB`, preserve that style when the code meaning is clear
- in the program architecture diagram, add an explanatory note for each important file line whenever the code makes that role clear
- if existing diagrams use wide separators, arrows, inline comments, or layered sections, prefer that richer style over a minimal generic tree
- if there is both an old `architecture_analysis.md` and a new `docs/architecture-analysis.md`, use the old file as style guidance and the current code as fact source
- when a line contains project-specific jargon, infrastructure terms, or opaque identifiers such as queue names, watcher names, broker names, or internal service names, keep the real identifier but add a plain-language explanation beside it
- prefer explanations that a new teammate or non-author can understand on first read; do not assume the reader already knows what Celery, Sentinel, CQ, watcher, or a queue name means in this project

Language precedence for all generated docs:

1. Explicit user instruction about output language
2. Existing target-project documentation language
3. Skill repository examples and defaults

Short alias examples that should be honored:

- `/project-analyzer zh .`
- `/project-analyzer en .`
- `/project-analyzer . zh`
- `/project-analyzer /path/to/repo en`

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

For detailed code logic diagrams:

- treat the code logic diagram primarily as a call-relationship diagram, not just a sequence of boxes with labels
- each major box should contain the file or module name plus a role description for that layer
- each major box should list the main methods or numbered steps inside the box when code confirms them
- prefer `1. ... 2. ... 3. ...` descriptions inside the box over a single vague label when the implementation has multiple steps
- if a box represents a processor, worker, or orchestrator, show the internal sequence of what it does
- when line numbers are statically discoverable, prefer `path:line method()` format such as `api/app.py:42 submit()`
- when a method makes important internal calls, list the main called functions or helper methods under it instead of stopping at the top-level method name
- do not decide branching only from route names or top-level method names; branch according to the real call graph and data dependencies visible in code
- if a function calls other functions to fetch results, compute intermediate values, or decide the next step, show those calls as child branches or downstream boxes when they materially affect the flow
- prefer detailed inline notes over minimal labels; explain what the function reads, writes, returns, decides, or triggers when the code makes that visible
- in program architecture, code logic, and call-graph outputs, annotate important nodes with concrete meanings such as input source, output payload, decision condition, side effect, queue handoff, or external dependency usage
- when a module or function is shown because it sits on the critical path, the note should explain why it matters in that path, not only restate its name
- when a note includes a technical identifier, also explain the practical effect in plain language, for example "dispatch to Celery queue `Patcher:data_patcher`, meaning the API hands the work to a background worker instead of executing it inline"

#### `code-call-graph.md`

This file is strongly recommended whenever the repository has non-trivial orchestration, worker flows, helper-heavy business logic, or multiple entrypoints.

Include:

- a top-level call graph overview
- one or more focused call-chain sections for the most important flows
- caller -> callee relationships across files, modules, and infrastructure boundaries
- internal helper calls where they materially affect control flow or returned results

Good sections often include:

- overall call graph
- API request call chain
- worker / queue call chain
- core engine internal function call chain
- configuration and runtime dependency call chain

For call graph output:

- treat this as a dedicated "who calls whom" document, separate from the broader architecture summary
- prefer real caller/callee names over generic prose
- prefer expanding important helper calls when they determine later behavior
- include line numbers when they are easy to derive statically
- when useful, group by entrypoint, worker, scheduler, or core module
- add short inline notes for important caller, callee, helper, and dependency lines when the role is clear from static inspection
- when possible, annotate each important line in the call graph rather than only the top-level nodes
- in practice, every non-connector node line in the call graph should preferably carry an inline explanation unless the line is only a visual branch label
- if a branch comes from a helper return value or downstream dependency, show that branch explicitly instead of compressing it into one summary line
- prefer notes that explain data returned, conditions checked, downstream side effects, or why the edge exists
- organize the call graph as readable flow sections, not just one flat tree; separate submission flow, status flow, worker flow, and runtime dependency flow when they exist
- when a request path branches, show the branch point and label what causes the split, such as request method, task state, helper return value, or strategy choice
- for important edges, explain both the action and the purpose, such as "enqueue task for async execution", "read result backend state", or "build filters used by downstream query"
- when a function returns a value consumed by the next call, make that handoff visible in the note instead of leaving the relationship implicit
- prefer enough annotation that a reader can understand the flow without opening the source for every edge
- if the edge uses a queue, cache, watcher, lock, or other infrastructure primitive, explain what that primitive does in the current flow rather than naming it alone
- be explicit about where static inspection ends and inference begins

#### `mock-data-stages.md`

This file is required for this skill.

For at least one important project flow, provide:

- Stage name
- What code/component owns that stage
- Input shape
- Output shape
- Mock JSON or structured demo data
- Notes on fields and meaning
- A brief explanation of what changed at that stage and why it matters
- A clear distinction between code-confirmed structure and illustrative mock values when both appear

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

Preferred mock-document style:

- start with a short note that the examples are demo data shaped from the current code, not production values
- use stage-by-stage sections with concrete headings instead of a single large dump
- include short field-structure notes when a payload shape is easier to understand in text than in JSON alone
- when the code adds execution context, show the "before" and "after" payloads as separate stages
- when runtime configuration fills parts of the data, say that explicitly and label the downstream examples as inferred or illustrative
- detect the main top-level operation types, request modes, or dispatch branches from code before writing the mock document
- if the project clearly supports multiple primary types, do not document only one rich branch and omit the others
- when there are two to four major types, expand each major type as its own stage chain with request, normalized input, execution-context enrichment, source/intermediate data, patch or transform result, and final write or response stages when those stages exist in code
- do not treat a second major type as a mere appendix or short divergence note when the repository only has a small number of primary types
- if multiple major types share some early stages, you may state that the first one reuses the same validation or routing logic, but still show the later stages for each type separately with concrete payload examples
- only summarize variants instead of fully expanding them when there are more than four materially different types or when the code proves they are thin aliases of one another
- if any type is summarized rather than fully expanded, explicitly explain why and still provide its request shape, branch point, and final payload difference

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
- when functions in the diagram are important, add brief inline notes after them using the repository's preferred style, such as `← 查询源数据`, `← 触发下游任务`, `← 写回数据库`
- when those inline notes contain internal jargon, extend them so a reader unfamiliar with the codebase can still understand the operational meaning
- in the program architecture diagram, avoid bare file names with no explanation; add a role note or child-method explanation for each important file line
- in the code logic diagram, avoid boxes that only contain a title; include concrete methods or numbered substeps inside each meaningful box
- in the code logic diagram, prefer `path:line method()` over `method()` alone when line numbers are easy to obtain
- if internal calls are visible, show the real helper names, such as `_get_sql()`, `apply_async()`, `require_auth()`, rather than generic phrases
- if a function depends on helper-returned results, split that dependency into branches or downstream boxes rather than compressing it into one summary line
- keep box borders visually aligned when drawing text boxes; prefer consistent widths over jagged right edges
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

### Minimal `code-call-graph.md`

Include these sections:

- `# Code Call Graph`
- `## Overview Call Graph`
- `## Call Chain 1 ...`
- `## Call Chain 2 ...`
- `## Runtime Dependency Chain`
- `## Notes On Inference`

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
- `code_call_graph.md`
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
