# Analysis Checklist

Use this checklist while analyzing a repository.

## Repository Shape

- Top-level files identified
- Main directories identified
- Runtime and language identified
- Dependency manifests identified
- Config files identified
- Existing analysis docs or diagram samples identified
- Preferred documentation language identified

## Entrypoints

- HTTP server entrypoint
- CLI entrypoint
- Worker/task entrypoint
- Frontend entrypoint
- Scheduler/cron entrypoint

Not every project has all of these. Identify the ones that exist.

## Core Logic

- Validation layer
- Routing layer
- Business logic layer
- Data access layer
- External integration layer
- Caller/callee relationships for at least one important flow
- Helper functions that materially affect later control flow
- Top-level request types, modes, or dispatch branches identified
- Major branch differences traced where multiple primary types exist

## Infrastructure

- Database configuration
- Queue configuration
- Cache configuration
- Logging
- Deployment or supervisor files

## Output Readiness

- At least one main flow traced end-to-end
- Architecture diagram uses real project names
- Logic diagram uses real functions/files when available
- Diagram annotations explain why important nodes matter, not only what they are called
- Diagram annotations explain project-specific jargon and infrastructure names in plain language when they appear
- Call graph identifies real caller -> callee edges for at least one important chain
- Call graph annotations explain data flow, decisions, or side effects where statically visible
- Call graph sections are readable as flows, with explicit branch points and result handoffs for the main paths
- Call graph does not leave most node lines unlabeled; non-connector lines are generally annotated with purpose or result
- Mock data stages match the actual code structure
- Mock coverage includes all major supported types or explicitly explains why some were summarized
- When only a small number of major types exist, each one is expanded as its own stage chain instead of being reduced to a short difference note
- Uncertainties clearly marked
- If existing docs exist, new output visibly matches their diagram style and terminology
- Old docs were used as style input, not blindly copied as factual truth
