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
- Call graph identifies real caller -> callee edges for at least one important chain
- Mock data stages match the actual code structure
- Uncertainties clearly marked
- If existing docs exist, new output visibly matches their diagram style and terminology
- Old docs were used as style input, not blindly copied as factual truth
