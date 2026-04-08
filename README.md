# project-analyzer-skill

A GitHub-friendly skill repository for generating codebase analysis docs from a local project.

This repository currently provides one installable skill:

- `project-analyzer`

The skill is designed for a very practical workflow:

- analyze the current project or a specified local repository
- understand entrypoints, modules, and execution paths
- generate plain markdown documentation into the target repository
- include architecture diagrams, code logic diagrams, code call graphs, and stage-by-stage mock data examples

It is intentionally generic and not tied to any single business domain.

Even though the skill is generic, it should not ignore a target repository's existing documentation style. If the target project already has architecture notes, older analysis markdown, or plain-text diagram samples, the skill should reuse that style as presentation guidance while still verifying technical facts from code.

## What The Skill Produces

When invoked on a target project, the skill should generate a `docs/` directory under that project root and write one or more markdown files.

Default outputs:

```text
docs/project-overview.md
docs/architecture-analysis.md
docs/code-call-graph.md
docs/mock-data-stages.md
```

These files are meant to be useful to:

- developers onboarding to an unfamiliar repository
- engineers who need to explain a system to teammates
- AI-assisted workflows that benefit from stable, reusable docs
- anyone trying to understand how data changes across a project flow

## Skill Scope

`project-analyzer` is best at:

- explaining project structure
- mapping entrypoints and module responsibilities
- tracing a main runtime flow
- generating text-based architecture diagrams
- generating text-based code logic diagrams
- generating text-based code call graphs
- generating stage-by-stage data shape examples
- writing the output as project-local markdown docs

It is not intended to replace:

- formal architecture decision records
- bug-focused code review
- performance profiling
- automated schema extraction tools

## How It Works

The skill follows this rough process:

1. Resolve the target project
2. Inspect repository structure
3. Find likely entrypoints and core modules
4. Trace at least one important flow end-to-end
5. Infer key data transformations
6. Write markdown docs into the target project's `docs/` directory

The default target resolution rule is:

- if the user provides a path, analyze that path
- otherwise analyze the current working directory

Language rule:

- if the user explicitly asks for Chinese, English, or another language, the generated docs should use that language
- short aliases such as `zh`, `cn`, `zh-CN`, `en`, and `en-US` should also count as explicit language instructions
- if the user does not specify a language, infer it from the target repository's existing docs when possible
- language selection should affect headings and explanatory prose, but not code identifiers, file paths, function names, queue names, or config keys

## Repository Layout

```text
project-analyzer-skill/
├── README.md
└── skills/
    └── project-analyzer/
        ├── SKILL.md
        ├── agents/
        │   └── openai.yaml
        └── references/
            ├── analysis-checklist.md
            ├── examples.md
            └── output-spec.md
```

## Installation

This repository is structured so you can later push it to GitHub and install it with an `npx`-based skill installer.

The exact installer command depends on the tool you use, but the target repository layout is compatible with the common pattern:

```bash
npx skills add <your-github-user>/project-analyzer-skill
```

Or install just the single skill subdirectory if your installer supports it:

```bash
npx skills add <your-github-user>/project-analyzer-skill/skills/project-analyzer
```

Replace `<your-github-user>` with your actual GitHub username or organization.

## Local Development

After cloning the repository locally, the most important files to edit are:

- `skills/project-analyzer/SKILL.md`
- `skills/project-analyzer/references/output-spec.md`
- `skills/project-analyzer/references/analysis-checklist.md`
- `skills/project-analyzer/references/examples.md`

If you later want richer UI metadata, you can extend:

- `skills/project-analyzer/agents/openai.yaml`

## Usage Examples

The intended usage style is short and easy to type.

Analyze the current project:

```text
/project-analyzer
```

Analyze the current directory explicitly:

```text
/project-analyzer .
```

Analyze another repository:

```text
/project-analyzer /path/to/repo
```

Analyze the current project and write docs in Chinese:

```text
/project-analyzer . generate docs in Chinese
```

Use a short alias to force Chinese output:

```text
/project-analyzer zh .
```

Analyze the current project and force English output:

```text
/project-analyzer . generate docs in English
```

Use a short alias to force English output:

```text
/project-analyzer en .
```

## Expected Output Style

The generated docs should be concrete rather than vague.

Good output includes:

- real file names
- real functions or classes where practical
- actual entrypoints
- honest notes on uncertain parts
- mock data that mirrors the code-confirmed structure
- diagram style continuity with existing docs inside the target repository
- call graph sections that explicitly show caller -> callee relationships

If the target repository already contains files such as:

- `architecture_analysis.md`
- `mock_data_stages.md`
- `.txt` diagram drafts

those files should be treated as style references for:

- document language
- diagram density
- section naming
- connector characters and indentation
- whether functions and queue names are shown inline

But if the user explicitly requests an output language, that explicit language choice takes precedence over the repository's default documentation language.

The `mock-data-stages.md` output should distinguish between:

- code-confirmed structure
- illustrative mock values

## Suggested GitHub Publishing Steps

Once you are happy with the repository:

1. Initialize git if needed
2. Create a GitHub repository
3. Push this folder
4. Install it using your preferred `npx` skill installer

Example:

```bash
cd ~/Untitled/devspace/project-analyzer-skill
git init
git add .
git commit -m "Initial project-analyzer skill"
git branch -M main
git remote add origin git@github.com:<your-github-user>/project-analyzer-skill.git
git push -u origin main
```

## Recommended Future Enhancements

If you want to evolve this skill later, good next steps are:

- add a script that scaffolds the target `docs/` files automatically
- add optional timestamped output naming
- add framework-specific reference files for common stacks
- add examples for backend, frontend, and worker-style repos
- add richer output conventions for databases, queues, and third-party APIs

## Authoring Notes

This repository keeps the skill focused and small:

- the trigger and workflow live in `SKILL.md`
- longer guidance lives in `references/`
- the README is for humans maintaining and publishing the repository

That split keeps the installed skill lean while still making the repo easy to maintain.
