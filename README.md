# Spec-Driven Development (SDD)

A Claude Code / Cowork plugin that guides you through a structured, specification-first workflow for building software features — so you never jump straight into code again.

## What it does

SDD enforces a five-phase workflow:

1. **Initialize** — sets up a `specs/` directory and project constitution
2. **Specify** — captures *what* to build and *why* (user stories, acceptance criteria, non-goals)
3. **Plan** — translates the spec into a technical approach, key decisions, and data model
4. **Task** — breaks the plan into small, test-first, parallelizable work items with checkpoints
5. **Execute** — works through the task list with test-first discipline, marking progress as it goes

It also supports **Sprint Mode** — a multi-feature, multi-agent team workflow for running parallel sprints across several features at once.

## Why spec-first?

Projects that skip specification accumulate drift between what was intended and what gets built. Specs catch misunderstandings when they're cheap to fix. Plans force technical decisions before you're knee-deep in implementation. Task lists give you and Claude a shared view of what's done and what's left.

## Graphify integration

If [Graphify](https://graphify.net) is installed in your repository, the skill automatically detects it and uses `graphify-out/GRAPH_REPORT.md` as the primary source of codebase structure — consulting god nodes and community clusters before falling back to file searches. This prevents conflicts with Graphify's `PreToolUse` hook and makes the technical planning phase significantly more accurate.

## Installation

Install via the Claude Code marketplace:

```
/plugin marketplace add GameFixxer/spec-driven-development
/plugin install spec-driven-development@spec-driven-development
```

That's it — the `sdd` skill becomes available immediately.

## Usage

Just describe what you want to do and the skill will route you to the right phase:

- "Set up SDD for this project" → Phase 1 (Initialize)
- "New feature: user authentication" → Phase 2 (Specify)
- "Plan the auth feature" → Phase 3 (Plan)
- "Break auth into tasks" → Phase 4 (Task)
- "Start working on auth" → Phase 5 (Execute)
- "Resume where we left off" → Phase 5 (Resume)
- "Start a sprint with features X, Y, Z" → Sprint Mode

## License

MIT
