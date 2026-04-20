---
name: sdd
description: >
  Spec-Driven Development (SDD) — a structured, specification-first development workflow.
  Guides you through 5 phases: initialize project structure, write feature specs, create
  technical plans, break work into tasks, and execute with progress tracking. Use this skill
  whenever the user mentions spec-driven development, SDD, creating or initializing specs,
  planning a feature, breaking work into tasks, resuming from a task list, tracking development
  progress, or reviewing implementation against a spec. Also trigger when the user says things
  like "set up SDD", "new feature spec", "plan this feature", "create tasks", "start working",
  "resume where we left off", "review against spec", "update progress", "start a sprint",
  "refine features", or "execute sprint". Even if the user doesn't use the term "SDD"
  explicitly, trigger this skill when they ask to create a structured spec-then-plan-then-tasks
  workflow for a feature or project, or when they want to coordinate multiple features with a
  team-based approach.
---

# Spec-Driven Development (SDD)

You are guiding the user through a specification-first development workflow. The core idea: never jump straight into code. Instead, write structured markdown files that capture *what* needs to be built and *why*, then *how* to build it, then break the *how* into small executable tasks. Only then do you write code — and when you do, you follow the task list and mark progress as you go.

This approach exists because projects that skip specification tend to accumulate drift between what was intended and what gets built. Specs catch misunderstandings early (when they're cheap to fix), plans force you to think through technical decisions before you're knee-deep in implementation, and task lists give both you and the user a shared view of what's done and what's left.

## Communication Protocol

All output from this skill follows a token-efficient format. This matters because SDD sessions are long-running, multi-phase workflows — and in Sprint Mode, multiple agents communicate simultaneously. Every unnecessary token compounds.

### Response format rules

1. **No preamble.** Don't say "Great, I'll now create the spec" — just create it.
2. **Structured over prose.** Use `key: value` pairs, tables, and checklists instead of paragraphs when reporting status.
3. **File-first.** Write artifacts to files, then reference them. Don't echo file contents back to the user/chat unless asked.
4. **Delta updates only.** When reporting progress, state only what changed since the last update — not the full state.
5. **Confirmation = one line.** After creating a file, confirm with: `✓ Created [path] — [one-line summary]`. Nothing more unless the user asks.
6. **Checkpoint reports use structured format** (see Phase 5 checkpoint template below).
7. **Subagent prompts are minimal** — context file paths + task description + constraint list. No narrative.

### User-facing communication

When talking to the user directly:
- Phase transitions: `→ Phase N: [name]` on one line
- File creation: `✓ [path]` on one line
- Questions: ask directly, no lead-in
- Review requests: present the file link, then list only items that need decision

## Graphify Integration

Graphify is a knowledge-graph tool that can be installed per repository inside Claude Code. When active, it maintains `graphify-out/GRAPH_REPORT.md` — a one-page structural summary of the codebase covering "god nodes" (highly-connected files), community clusters, and surprising cross-cutting connections. It also installs a `PreToolUse` hook that fires before every Glob/Grep call, and a `CLAUDE.md` directive asking Claude to consult the graph before architecture questions.

**Detecting Graphify:** At the start of Phase 3, Phase 4, and Phase 5, check whether `graphify-out/GRAPH_REPORT.md` exists. If it does, Graphify is active.

**When Graphify is active, use the graph first:**
- Read `graphify-out/GRAPH_REPORT.md` *before* doing any Glob/Grep-based codebase exploration. The graph already knows which files are structurally central — searching blind wastes tokens and may conflict with the PreToolUse hook's intent.
- In Phase 3 (Plan), use god nodes and community clusters to inform the Technical Approach and Data Model sections — they reveal actual architectural boundaries better than directory structure alone.
- In Phase 4 (Task), use community membership to assign `OWN_FILES` scopes for `[P]` parallel tasks. Tasks within the same community are more likely to conflict; tasks in different communities are safe to parallelize.
- In Phase 5 (Execute), when navigating to understand existing code before implementing, start from the graph's god nodes rather than open-ended Glob searches.
- In Sprint Mode execution, include `graphify-out/GRAPH_REPORT.md` in each subagent prompt's `READ:` list so teammates share the same structural map.

**When Graphify is not active:** proceed exactly as before — nothing changes.

---

## How to determine which phase to run

Read the user's message and match it to a phase:

| User says something like... | Phase to run |
|---|---|
| "Set up SDD", "Initialize specs", "start SDD for this project" | **Phase 1** — Initialize |
| "New feature: [name]", "Create a spec for [feature]", "spec out [feature]" | **Phase 2** — Specify |
| "Plan this feature", "Create the plan", "how should we build this?" | **Phase 3** — Plan |
| "Break into tasks", "Create tasks", "what are the steps?" | **Phase 4** — Task |
| "Start working", "Execute tasks", "let's build this" | **Phase 5** — Execute |
| "Resume", "Continue where we left off", "pick up from last time" | **Phase 5** — Resume variant |
| "Review against spec", "are we on track?", "check acceptance criteria" | **Review** — Compare implementation to spec |
| "Update progress", "log what we did" | **Progress update** |
| "Sprint", "start a sprint", "sprint mode", "team mode" | **Sprint Mode** — Multi-feature team workflow |
| "Refine features", "refinement meeting", "sprint planning" | **Sprint Mode** — Refinement phase |
| "Execute sprint", "start sprint execution" | **Sprint Mode** — Execution phase |

If the user's intent is ambiguous, ask which phase they want rather than guessing.

If the user asks to do multiple phases at once (e.g., "spec and plan this feature"), run them in sequence, pausing between phases to confirm the output before moving on.

---

## Phase 1 — Initialize Project Structure

This phase sets up the SDD scaffolding for a project. Run it once per project.

### Steps

1. **Create the specs directory.** At the project root, create `specs/` if it doesn't exist.

2. **Create the constitution.** Create `specs/memory/constitution.md` with the default architectural principles below. Then ask the user: *"Here are the default development principles. Want to customize any of these, or add your own?"*

   Default constitution content — use the template in `references/constitution-template.md`.

3. **Set up CLAUDE.md.** Add a development methodology section to `.claude/CLAUDE.md` (create the file if needed). This section tells future Claude sessions how to work within the SDD framework. Use the content in `references/claude-md-template.md`.

4. **Confirm completion.** One-line per file created (`✓ [path]`), then: "Ready. Name a feature to start Phase 2."

### Important

- If `specs/` already exists, don't overwrite anything. Ask the user if they want to reinitialize.
- If `.claude/CLAUDE.md` already exists, append the SDD section rather than replacing the file.

---

## Phase 2 — Specify (spec.md)

This phase captures *what* a feature should do and *why* it matters. Specs deliberately avoid technical implementation details — those belong in the plan.

### Steps

1. **Create the feature directory.** Convert the feature name to kebab-case and create `specs/[feature-name]/`.

2. **Interview the user.** Ask clarifying questions. Don't ask them all at once — have a conversation. Key things to understand:
   - What problem does this feature solve? Who experiences that problem?
   - Who are the users? (Specific roles or personas, not just "users")
   - What does success look like? (These become acceptance criteria)
   - What is explicitly *out of scope*? (These become non-goals — they're just as important as goals because they prevent scope creep)
   - Are there dependencies on other features or systems?

   If the user has already provided a lot of detail in their initial message, don't re-ask things they've already answered. Extract what you can and only ask about gaps.

3. **Write the spec.** Generate `specs/[feature-name]/spec.md` following this structure:

   ```markdown
   # [Feature Name]

   ## Overview
   [1-2 paragraphs: what this feature does and why it matters. Focus on the
   problem being solved and the value delivered. No implementation details.]

   ## User Stories
   - As a [role], I want [capability] so that [benefit]
   - ...

   ## Acceptance Criteria
   - [ ] [Specific, testable criterion]
   - [ ] [Another criterion]
   - ...

   ## Non-Goals
   - [Thing that is explicitly out of scope]
   - ...

   ## Open Questions
   - [NEEDS CLARIFICATION: question about something unclear]
   - ...

   ## Dependencies
   - [External system, library, or feature this depends on]
   - ...
   ```

4. **Review with the user.** Say `✓ Created specs/[feature]/spec.md` then list only items needing decision (vague criteria, open questions). Don't echo the full spec.

### Guardrails

- If you catch yourself writing implementation details in the spec (database schemas, API endpoints, class names), stop. Move that thinking to a mental note for Phase 3.
- Mark anything uncertain with `[NEEDS CLARIFICATION: ...]` and ask the user about it. Never assume.
- Non-goals are hard boundaries. Once something is listed as a non-goal, it stays out of scope for the entire workflow unless the user explicitly changes the spec.

---

## Phase 3 — Plan (plan.md)

This phase is where technical decisions happen. The plan translates the *what* from the spec into a *how*.

### Steps

1. **Read the inputs.** Before writing anything:
   - Check whether `graphify-out/GRAPH_REPORT.md` exists. If it does, read it first — the god nodes and community clusters give you the real architectural map before you touch any Glob or Grep.
   - Read `specs/[feature-name]/spec.md`
   - Read `specs/memory/constitution.md`
   These documents together define what you're building, the principles you're building with, and (when Graphify is present) the structural landscape you're building into.

2. **Generate the plan.** Write `specs/[feature-name]/plan.md`:

   ```markdown
   # [Feature Name] — Technical Plan

   ## Technical Approach
   [High-level architecture: what components are involved, how they interact,
   what the data flow looks like. Keep it at the right altitude — detailed
   enough to guide implementation, abstract enough that it doesn't become
   pseudocode.]

   ## Key Decisions

   | Decision | Choice | Rationale |
   |----------|--------|-----------|
   | [What was decided] | [What was chosen] | [Why this over alternatives] |

   ## Data Model
   [Entities, their attributes, and relationships. Use whatever notation is
   clearest — ERD-style text, table definitions, TypeScript interfaces, etc.]

   ## API Contracts
   [Endpoints, methods, request/response shapes. If this feature doesn't
   have an API, replace this section with whatever interface is relevant
   (CLI commands, UI components, event schemas, etc.)]

   ## Implementation Phases
   1. **Foundation** — Core data structures, database setup, basic scaffolding
   2. **Business Logic** — Domain logic, validation, core algorithms
   3. **API/Interface Layer** — Endpoints, UI components, CLI commands
   4. **Testing** — Integration tests, edge cases, error scenarios
   5. **Polish** — Error handling, logging, documentation, cleanup

   ## Risks and Mitigations

   | Risk | Impact | Mitigation |
   |------|--------|------------|
   | [What could go wrong] | [How bad would it be] | [What to do about it] |
   ```

3. **For complex features**, optionally generate additional files:
   - `specs/[feature-name]/research.md` — Background research, links to relevant docs, analysis of similar implementations
   - `specs/[feature-name]/data-model.md` — Detailed data model if the one in plan.md would be too long

4. **Review with the user.** Say `✓ Created specs/[feature]/plan.md` then list only the Key Decisions that need user sign-off. Don't echo the full plan.

### Guardrails

- Every decision in the Key Decisions table should respect the constitutional principles. If there's a tension (e.g., simplicity vs. a user requirement that demands complexity), call it out explicitly.
- The implementation phases exist to guide task creation in Phase 4. They don't need to be followed rigidly, but they should reflect a sensible build order (foundations before features, features before polish).

---

## Phase 4 — Task (tasks.md)

This phase breaks the plan into small, actionable work items. Each task should be implementable and testable in isolation.

### Steps

1. **Read the inputs.** Read:
   - `specs/[feature-name]/plan.md`
   - `specs/[feature-name]/spec.md` (for acceptance criteria cross-reference)
   - `.claude/CLAUDE.md` (for build commands, test commands, and platform targets)

   CLAUDE.md is the source of truth for how to build and test this project. The build commands defined there become the verification steps in the task list.

2. **Generate the task list.** Write `specs/[feature-name]/tasks.md`:

   ```markdown
   # [Feature Name] — Tasks

   ## Status Legend
   - `[ ]` Not started
   - `[x]` Complete
   - `[~]` In progress
   - `[P]` Parallelizable — will be executed concurrently via subagents
   - `[C]` Checkpoint — stop and verify before continuing

   ## Phase 1: Foundation
   - [ ] [Test: describe what the foundation should do] — write failing tests first
   - [ ] [Implement: build the thing the tests describe]
   - [C] Checkpoint: run tests, verify all pass, update tasks.md

   ## Phase 2: Business Logic
   - [P] [Test + Implement: independent unit A] — test first, then implement
   - [P] [Test + Implement: independent unit B] — test first, then implement
   - [C] Checkpoint: run full test suite, verify no regressions, update tasks.md

   ## Phase 3: API/Interface Layer
   - [ ] [Test: integration tests for endpoints] (depends on Phase 2)
   - [ ] [Implement: wire up endpoints]
   - [C] Checkpoint: run full test suite, verify against acceptance criteria

   ## Phase 4: Polish
   - [ ] [Error handling improvements]
   - [ ] [Documentation]
   - [C] Checkpoint: run all tests, review against spec acceptance criteria

   ## Build Verification
   - [ ] Full build: [build command from CLAUDE.md, e.g. `npm run build`]
   - [ ] Full test suite: [test command from CLAUDE.md, e.g. `npm test`]
   - [ ] [One task per additional platform target, e.g. `npm run build:prod`, `docker build .`]
   - [C] Final checkpoint: all builds green, all tests pass, acceptance criteria verified
   ```

3. **Review with the user.** Say `✓ Created specs/[feature]/tasks.md — [N] tasks across [M] phases, [X] parallel groups, [Y] checkpoints`. Ask only: "Anything to add, remove, or reorder?"

### Test-first task ordering

The constitution says test-first, and the task list must structurally enforce this. For every piece of functionality:

1. The test task comes **before** the implementation task — always. Not in a separate "Testing" phase at the end, but immediately before the code it validates.
2. For `[P]` parallel groups, each parallel task is a self-contained "test + implement" pair. The test is written first within that task, then the implementation. Don't split tests and implementation into separate parallel tracks.
3. Never create a standalone "Phase 4: Testing" section that comes after all implementation. Integration tests and edge case tests should appear at the boundary of the phase they're testing — e.g., after Phase 2 tasks are done, write integration tests for Phase 2's behavior before moving to Phase 3.

If you catch yourself writing tasks where implementation comes before its tests, reorder them. The task list is the source of truth for execution order.

### What makes a good task

- **Small enough to complete in one focused session.** If a task would take more than a couple of hours, break it down further.
- **Testable in isolation.** After completing the task, you should be able to verify it works without finishing the rest of the feature.
- **Clear on what "done" means.** The person reading the task list should know what the expected outcome is.
- **Dependencies are explicit.** If task B can't start until task A is done, say so.
- **Each task includes its verification.** A task like "implement login" is incomplete — it should be "implement login (test: valid credentials return JWT, invalid credentials return 401)".

### Parallelization — real concurrent execution

Tasks marked `[P]` aren't just a label — during execution, they are launched as concurrent subagents using the Agent tool. This means:

- Only mark tasks `[P]` when they are genuinely independent: no shared state mutations, no file conflicts, no ordering dependencies.
- Each `[P]` task must be fully self-contained: it includes its own test-writing and implementation steps, targeting a specific module or file set that won't conflict with other `[P]` tasks.
- A `[P]` group is always followed by a `[C]` checkpoint that runs the full test suite and verifies no conflicts arose from concurrent work.

### Checkpoints — catching drift

Checkpoints (`[C]`) are verification gates that prevent the task list from drifting out of sync with reality. At each checkpoint:

1. Run the full test suite (not just the tests written in the current phase)
2. Re-read `tasks.md` and verify the completion marks are accurate
3. Compare completed work against the relevant acceptance criteria from `spec.md`
4. If anything is out of sync — a test fails, a task was marked done but isn't actually working, or a criterion that should be met isn't — stop and fix it before continuing
5. Write a brief checkpoint entry in `progress.md` recording what passed and what didn't

Think of checkpoints as save points. Without them, small errors compound across phases until you're debugging a tangled mess at the end. With them, you catch problems within one phase of where they were introduced.

### Build Verification — the final group

Every task list ends with a **Build Verification** group. This group exists because tests passing doesn't guarantee the project actually builds — type errors, missing imports, and broken configurations can all hide behind a green test suite.

To generate this group:

1. Read `.claude/CLAUDE.md` and look for build commands (e.g., `npm run build`, `cargo build`, `go build ./...`), test commands (e.g., `npm test`, `pytest`), and any platform-specific targets (e.g., `npm run build:prod`, `docker build .`, `make release`).
2. Create one task per build/platform target. Each task runs the command and verifies it exits cleanly.
3. Create one task for the full test suite command.
4. End with a final `[C]` checkpoint that confirms everything is green.

If CLAUDE.md doesn't have build commands (maybe the project is new), ask the user what build and test commands to use, or infer from the project structure (look for `package.json`, `Cargo.toml`, `Makefile`, `pyproject.toml`, etc.) and confirm with the user.

---

## Phase 5 — Execute and Track

This phase is where code gets written. Every task follows the same rhythm: write a failing test, make it pass, mark the task done, verify at checkpoints.

### Starting execution

1. Read all inputs:
   - Check whether `graphify-out/GRAPH_REPORT.md` exists. If it does, read it — use it to navigate the codebase structurally rather than with open-ended Glob/Grep searches.
   - `specs/[feature]/spec.md` — what to build and why
   - `specs/[feature]/plan.md` — how to build it
   - `specs/[feature]/tasks.md` — what to do next
   - `.claude/CLAUDE.md` — build commands, test commands, and project conventions

   CLAUDE.md is where you find the actual commands to run tests and build the project. Without it, you'd have to guess — and guessing leads to running `npm test` on a Python project or missing a required build flag.

2. Scan the task list and identify the next work to do — either the first `[ ]` task or a `[P]` group

### Executing a sequential task (`[ ]`)

For each `[ ]` task:

1. Mark it `[~]` in `tasks.md` (in progress)
2. Write the test first — a test that will fail right now but will pass when the task is done. Run it to confirm it fails. This isn't ceremonial; it forces you to define "done" concretely before you start coding.
3. Write the implementation until the test passes
4. Run the test suite (not just the new test — the full suite) to check for regressions
5. Mark the task `[x]` in `tasks.md`
6. Move to the next task

### Executing a parallel group (`[P]`)

When you hit a group of consecutive `[P]` tasks, launch them all concurrently using the **Agent tool**. This is actual parallel execution, not a suggestion — spawn one subagent per `[P]` task in a single message so they run simultaneously.

Each subagent gets a **minimal** prompt — no narrative, just structured fields:

```
TASK: [exact task line from tasks.md]
FEATURE: [feature-name]
READ: specs/[feature]/spec.md, specs/[feature]/plan.md, specs/memory/constitution.md, .claude/CLAUDE.md
OWN_FILES: [files/modules this task touches — nothing else]
RULES: test-first (write→fail→implement→pass), no shared config edits, no files outside OWN_FILES
REPORT_WHEN_DONE: STATUS: done | TASK: [id] | FILES: [changed] | RESULT: [one line]
```

This prompt is ~100 tokens vs ~200+ for a prose version. Over a sprint with 20+ parallel tasks, this halves the context overhead.

After all subagents complete:

1. Integrate their work (merge files, resolve any conflicts)
2. Run the full test suite
3. Mark all `[P]` tasks `[x]` in `tasks.md`
4. If any subagent's tests fail after integration, fix the conflicts before moving on

### Executing a checkpoint (`[C]`)

When you reach a `[C]` checkpoint:

1. **Run the full test suite.** Every test, not just recent ones. Use the test command from CLAUDE.md.
2. **Run the build.** Use the build command(s) from CLAUDE.md. A passing test suite with a broken build is still broken. This catches type errors, missing imports, and configuration problems that tests alone miss.
3. **Audit tasks.md.** Read the file and verify that every task marked `[x]` actually corresponds to working, tested code. If you find a task marked complete but its tests fail, unmark it (`[x]` → `[ ]`) and flag it.
4. **Check against spec.md.** Read the acceptance criteria and assess which ones are now satisfied by the completed work. Record this in the checkpoint entry.
5. **Write a checkpoint entry in progress.md** using this compact format:

   ```markdown
   ### CP: [Phase] — [Date]
   tests: X pass / Y fail / Z skip
   build: pass|fail [commands run]
   done: [task ids]
   rework: [task ids, if any]
   criteria_met: [AC ids from spec]
   issues: [one line per issue, or "none"]
   ```

6. **If issues were found, fix them before continuing.** Don't carry broken state into the next phase.
7. **Report to user** in one structured block — no narrative wrapping:

### Resuming

When the user says "resume" or "continue":

1. Read `specs/[feature-name]/progress.md` — look at the most recent checkpoint entry to understand the verified state of things
2. Read `tasks.md` to find the first incomplete task
3. Re-read `spec.md`, `plan.md`, and `.claude/CLAUDE.md` to refresh context and pick up build/test commands
4. **Run the test suite and build before writing any new code.** This confirms reality matches what progress.md claims. If tests fail or the build is broken at a point where the last checkpoint said everything was green, something changed — investigate before continuing.
5. Continue from the first `[ ]` task

If multiple features exist under `specs/`, ask the user which feature to resume unless it's obvious from context.

### Progress tracking

After each session, or when the user asks, write or update `specs/[feature-name]/progress.md`. Checkpoint entries accumulate in this file over time, giving a historical record:

```markdown
# [Feature Name] — Progress

updated: [Date]
status: [Phase N checkpoint passed | Phase N in progress]
blockers: [list or "none"]
next_session: [one line of context]

## Checkpoints

### CP: Phase 1 Foundation — [Date]
tests: 8/0/0
done: 1.1, 1.2, 1.3
criteria_met: AC-1, AC-2

### CP: Phase 2 Business Logic — [Date]
tests: 22/1/0
rework: 2.3 (race condition)
```

### Reviewing against the spec

When the user asks to review or check progress against the spec:

1. Read `spec.md`, specifically the acceptance criteria
2. Run the full test suite to get current reality (don't trust task marks alone)
3. For each criterion, check whether the current implementation satisfies it — both by reading the code and by checking test coverage
4. Report the results clearly: which criteria are met, which aren't, and what's needed to close the gaps

---

## Sprint Mode — Multi-Feature Team Workflow

Sprint Mode extends single-feature SDD into a multi-feature, multi-agent workflow using Claude Code **Agent Teams**. Instead of working on one feature at a time, you collect multiple features into a sprint, refine them with a cross-functional team, and execute with role-based task assignment and phase gates.

**When to use Sprint Mode:** The user has multiple features to build and wants them planned and executed together as a cohesive unit, or explicitly asks for "sprint mode" / "team mode".

**Prerequisite:** `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` must be enabled. If it isn't, tell the user: "Sprint Mode requires Agent Teams. Enable it with: `export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`" and fall back to standard single-feature SDD.

### How Agent Teams work (key concepts)

Agent Teams are fundamentally different from subagents. Understanding this is critical:

- **You (the lead session) create the team using natural language.** There is no programmatic API or tool call to spawn teammates. You literally tell Claude Code: "Create an agent team with these teammates: ..."
- **Teammates are independent Claude Code sessions.** Each has its own context window. They load `CLAUDE.md`, MCP servers, and skills from the project automatically — you do not need to tell them to read these files.
- **Teammates coordinate via a shared task list.** The lead creates tasks, teammates claim and complete them. Tasks can have dependencies that block until resolved.
- **Teammates message each other directly.** Unlike subagents (which only report back to the caller), teammates can communicate with each other by name.
- **The lead's conversation history does NOT carry over.** Teammates start fresh with only their spawn prompt and project context. Include task-specific details in the spawn prompt.
- **Reusable roles via subagent definitions.** You can define roles as `.md` files in `.claude/agents/` and reference them by name when spawning teammates: "Spawn a teammate using the backend-dev agent type."

### Agent Teams vs Subagents — when to use which

| Aspect | Subagents (`[P]` tasks) | Agent Teams (Sprint Mode) |
|--------|------------------------|--------------------------|
| Spawned via | `Agent` tool call | Natural language to lead |
| Communication | Report results back to caller only | Teammates message each other directly |
| Coordination | Main agent manages all work | Shared task list with self-coordination |
| Context | Inherit nothing, get a prompt | Load full project context (CLAUDE.md, skills, MCPs) |
| Best for | Focused, isolated parallel tasks | Complex work requiring discussion and collaboration |
| Token cost | Lower (results summarized back) | Higher (each teammate is a separate Claude instance) |

Standard Phase 5 execution uses **subagents** for `[P]` tasks. Sprint Mode uses **Agent Teams** for the entire sprint.

### Sprint artifacts

Sprint Mode introduces one new artifact:

- `specs/sprints/sprint-[N].md` — The sprint manifest. Lists features, team roster, refinement log, and execution progress.

All other artifacts (spec.md, plan.md, tasks.md, progress.md per feature) remain the same as standard SDD. Sprint Mode orchestrates them, it doesn't replace them.

### Optional: Define reusable teammate roles

Before starting a sprint, you can create subagent definitions so roles are reusable across sprints. Create `.md` files in `.claude/agents/`:

**`.claude/agents/backend-dev.md`**
```markdown
---
name: backend-dev
description: Backend developer for server logic, APIs, database, and infrastructure
model: sonnet
tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
---

You are a Backend Developer on an SDD sprint team. Your responsibilities:
- Implement server-side logic, APIs, database schemas, and services
- Follow test-first development: write failing test → implement → verify
- Only modify files assigned to you — do not touch files owned by other teammates
- When you complete a task, message the tech-lead with what you changed
- When blocked, message the tech-lead immediately with what's blocking you
- Read specs/memory/constitution.md for project principles
- Read .claude/CLAUDE.md for build and test commands
- Always run the test suite after completing a task
```

**`.claude/agents/solution-architect.md`**
```markdown
---
name: solution-architect
description: Solution architect for cross-feature design, shared components, and integration
model: sonnet
tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
---

You are a Solution Architect on an SDD sprint team. Your responsibilities:
- Review technical approaches and flag cross-feature integration issues
- Watch for shared components being modified by multiple features
- Identify API contract conflicts and duplicate code across features
- During refinement: review plans and provide complexity estimates
- During execution: own integration tasks and cross-cutting concerns
- Read specs/memory/constitution.md for project principles
```

Other common roles: `frontend-dev.md`, `ui-designer.md`, `app-dev.md`. The `tools` allowlist in the definition restricts what that teammate can do. Team coordination tools (SendMessage, task management) are always available regardless of the `tools` list.

If no subagent definitions exist, that's fine — you can still spawn teammates with inline role descriptions. Definitions just make it more consistent across sprints.

### Starting a sprint

When the user triggers Sprint Mode:

1. **Determine sprint number.** Check `specs/sprints/` for existing sprints. New sprint = max(N) + 1, or 1 if none exist.

2. **Collect features.** Ask the user: "Which features go in this sprint?" Accept a list. For each feature, check if a spec already exists under `specs/[feature-name]/`. Features without specs will need Phase 2 (Specify) during refinement.

3. **Present the default team roster.** Show:

   ```
   Refinement Team (all present during planning):
   - Product Manager — validates specs, prioritizes, resolves scope
   - Solution Architect — cross-feature tech design, shared components
   - Frontend Dev — UI estimates, component reuse, UX flags
   - Backend Dev — API/data estimates, perf concerns, service boundaries
   - App Dev — mobile/platform estimates, platform constraints
   - UI Designer — user flows, accessibility, design consistency
   ```

   Ask: "Use this default roster, or customize? (add/remove/rename roles)"

   For pure backend projects, suggest trimming to: Backend Dev + Solution Architect (+ PM if scope decisions are needed).

4. **Create the sprint file.** Write `specs/sprints/sprint-[N].md`. List all features with their current status (spec exists? plan exists? tasks exist?).

5. **Confirm.** `✓ Created specs/sprints/sprint-[N].md — [X] features, [Y] roles. Ready for refinement.`

### Sprint Refinement

Refinement is a short-lived team session where features get broken down into role-assignable tasks. Think of it as a planning meeting: everyone contributes their perspective, then the meeting ends.

#### Creating the refinement team

Tell Claude Code to create an agent team in natural language. This is the actual instruction you give:

```
Create an agent team for sprint refinement with these teammates:
- "pm" — Product Manager: validates specs, prioritizes features, resolves scope questions. 
  Use the product-manager agent type if it exists, otherwise create with this role description.
- "architect" — Solution Architect: reviews technical approach, flags cross-feature integration 
  issues, identifies shared components. Use the solution-architect agent type if it exists.
- "backend" — Backend Dev: estimates complexity for server/API/database tasks, flags performance 
  concerns. Use the backend-dev agent type if it exists.

Each teammate should read:
- specs/sprints/sprint-[N].md for the sprint overview
- specs/memory/constitution.md for project principles
- .claude/CLAUDE.md for build commands and project conventions

Require plan approval before any teammate makes file changes.
This is a refinement session — no code, only spec/plan/task files.
```

Adjust the teammate list based on the roster the user approved. If subagent definitions exist in `.claude/agents/`, reference them by name ("Use the backend-dev agent type"). If they don't, include the role description inline.

The main session acts as **Tech Lead** (facilitator). You are the lead — you create the team, assign work, and run checkpoints.

#### Refinement flow

For each feature in the sprint (in priority order):

1. **If no spec exists:** Tech Lead runs Phase 2 (Specify) to create `spec.md`. Message the PM teammate to review and validate the user stories and acceptance criteria:
   ```
   @pm Review specs/[feature]/spec.md — are the user stories complete? 
   Are acceptance criteria specific enough to test? Flag any scope concerns.
   ```

2. **If no plan exists:** Tech Lead runs Phase 3 (Plan) to create `plan.md`. Then message relevant teammates to review:
   ```
   @architect Review specs/[feature]/plan.md — flag cross-feature integration 
   issues or shared component conflicts with other sprint features.
   
   @backend Review specs/[feature]/plan.md — estimate complexity for the 
   backend tasks. Flag any performance concerns or missing technical decisions.
   ```
   Wait for teammates to respond. Incorporate their feedback into the plan.

3. **Task generation with role annotations:** Tech Lead runs Phase 4 (Task) to create `tasks.md`, but with one addition: every task gets a `@role` annotation indicating which role owns it.

   ```markdown
   ## Phase 1: Foundation
   - [ ] Test: database schema for user profiles @backend
   - [ ] Implement: database migration @backend
   - [C] Checkpoint @tech-lead

   ## Phase 2: Business Logic
   - [P] Test + Implement: auth service @backend
   - [P] Test + Implement: login form component @frontend
   - [P] Test + Implement: biometric auth module @app
   - [C] Checkpoint @tech-lead

   ## Phase 3: Integration
   - [ ] Test: API integration tests @backend
   - [ ] Implement: wire frontend to API @frontend
   - [ ] Implement: wire app to API @app
   - [C] Checkpoint @tech-lead

   ## Phase 4: Polish
   - [ ] Accessibility audit @ui-designer
   - [ ] Error states and empty states @frontend
   - [ ] API documentation @backend
   - [C] Checkpoint @tech-lead
   ```

   Tasks that span multiple roles get split into role-specific subtasks. A single task should never have two `@role` annotations — if it needs two roles, split it.

4. **Log the refinement.** After each feature is refined, write a refinement entry in `sprint-[N].md`:

   ```markdown
   ### [feature-name] — [Date]
   attendees: PM, SA, BE
   decisions: [key decisions from planning]
   task_count: [N] tasks, [X] parallel groups
   role_assignments: BE:[N], FE:[N], UI:[N]
   ```

5. **After all features are refined:** Ask the lead to clean up the refinement team:
   ```
   Ask all teammates to shut down, then clean up the team.
   ```
   Summarize to user:
   ```
   ✓ Refinement complete for sprint [N]
   Features: [count] refined
   Total tasks: [count] across all features
   Role distribution: BE:[N], FE:[N], UI:[N]
   Ready for execution.
   ```

#### Role annotation rules

- `@frontend` — UI components, client-side logic, CSS, browser APIs
- `@backend` — Server logic, APIs, database, infrastructure
- `@app` — Mobile/native/platform-specific code
- `@ui-designer` — Accessibility audits, design reviews, asset creation
- `@tech-lead` — Checkpoints, integration tasks, cross-cutting concerns
- Custom roles use the same `@kebab-case` convention

Every task MUST have exactly one `@role`. Checkpoints are always `@tech-lead`.

### Sprint Execution

Execution is a separate team session from refinement. After refinement cleanup, you create a new team for execution. Unlike refinement (which spawns all roles), execution only spawns roles that actually have tasks assigned.

#### Determining which roles to spawn

After refinement, scan all `tasks.md` files in the sprint:

1. Collect every unique `@role` annotation (excluding `@tech-lead` — that's you)
2. Count tasks per role
3. Only spawn teammates for roles with ≥1 assigned task
4. Update the Execution Team table in `sprint-[N].md`:

   ```markdown
   ### Execution Team (selective)
   | Role | Spawned | Task Count |
   |------|---------|------------|
   | Backend Dev | yes | 15 |
   | Frontend Dev | yes | 12 |
   | App Dev | no | 0 |
   | UI Designer | yes | 3 |
   ```

#### Creating the execution team

Tell Claude Code to create a new agent team. This is the actual instruction:

```
Create an agent team for sprint [N] execution with these teammates:

- "backend" — Backend Dev. Use the backend-dev agent type.
  Your tasks for this sprint (Phase 1 first, wait for phase gate before proceeding):
  [list all @backend tasks from all features' tasks.md files]
  
- "frontend" — Frontend Dev. Use the frontend-dev agent type.
  Your tasks for this sprint (Phase 1 first, wait for phase gate before proceeding):
  [list all @frontend tasks from all features' tasks.md files]

Rules for all teammates:
- Work through tasks in phase order (Phase 1, then Phase 2, etc.)
- DO NOT start the next phase until I (tech-lead) clear the phase gate
- Test-first: write failing test → implement → run full test suite
- Only modify files related to your assigned tasks
- When you finish all tasks in the current phase, message tech-lead
- When blocked, message tech-lead immediately
- Read .claude/CLAUDE.md for build/test commands

Wait for my signal to begin Phase 1.
```

Aim for 5-6 tasks per teammate. If one role has 20+ tasks, consider splitting into two teammates (e.g., "backend-1" and "backend-2") with non-overlapping file ownership.

#### Phase gate execution

Sprint execution proceeds phase by phase across all features. All roles must complete their Phase N tasks before anyone starts Phase N+1.

The Tech Lead (you, the main session) orchestrates this:

1. **Start Phase M.** Message all teammates:
   ```
   @backend @frontend → Phase [M]: [phase name]. Begin your Phase [M] tasks now.
   ```
   Or broadcast to all teammates simultaneously if appropriate.

2. **Teammates execute their Phase M tasks.** Each teammate:
   - Works through their `@role` tasks in the current phase
   - Uses test-first for every task
   - Messages tech-lead when all Phase M tasks are done
   - Messages tech-lead if blocked

3. **Tech Lead monitors.** Teammates notify the lead automatically when they go idle (finish their current work). When all teammates report done for Phase M:
   - Run the `[C]` checkpoint yourself (full test suite + build using commands from CLAUDE.md)
   - Record checkpoint in each feature's `progress.md`
   - Update `sprint-[N].md` gate status:
     ```
     ### Phase [M] Gate
     all_roles_done: yes
     blocked_roles: none
     checkpoint: pass
     ```
   - Message all teammates to advance: `Phase [M] gate cleared. → Phase [M+1]: [name]. Begin now.`

4. **If a teammate is blocked:**
   - Investigate the blocker by messaging the teammate directly
   - If it's a cross-role dependency, message the blocking teammate to coordinate
   - If it requires user input, escalate to the user
   - Gate does NOT clear until all blockers are resolved

5. **Phase gate rules:**
   - No teammate may start Phase N+1 tasks until you (Tech Lead) clear the Phase N gate
   - A gate clears only when: all roles done + checkpoint passes
   - If checkpoint fails, assign rework to the relevant teammate before clearing the gate
   - Checkpoints in Sprint Mode run the full test suite and build for ALL features in the sprint, not just the current feature — cross-feature regressions must be caught

#### Handling cross-feature concerns

During execution, watch for:

- Shared components being modified by multiple features → assign clear file ownership per teammate, coordinate via messaging
- API contract conflicts → resolve before gate clears
- Duplicate code across features → flag for refactoring in Polish phase

If a Solution Architect teammate is spawned, delegate cross-feature monitoring to them.

#### Sprint completion

After all phases complete across all features:

1. Run Build Verification yourself (the final group in each `tasks.md`) — full build, full test suite, fat JAR, Docker image
2. Update `sprint-[N].md`:
   ```
   status: done
   completed: [Date]
   features_delivered: [list]
   total_tasks: [N] completed, [M] reworked
   ```
3. Ask all teammates to shut down, then clean up the team:
   ```
   Ask all teammates to shut down, then clean up the team.
   ```
4. Report to user: `✓ Sprint [N] complete — [X] features delivered`

### Sprint Mode with single-feature fallback

If the user starts Sprint Mode with only one feature, it still works — refinement generates role-annotated tasks, execution spawns only needed roles. The overhead is minimal and the structure is consistent. Don't tell the user "you only have one feature, use standard mode" — just run it.

### Agent Teams troubleshooting

- **Teammates not appearing:** Press Shift+Down to cycle through active teammates (in-process mode). Check that `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` is enabled.
- **Lead doing work instead of delegating:** Tell the lead: "Wait for your teammates to complete their tasks before proceeding."
- **Too many permission prompts:** Pre-approve common operations in permission settings before spawning teammates, or use `--dangerously-skip-permissions` if appropriate for the project.
- **Teammate stopped on error:** Message the teammate directly with additional instructions, or ask the lead to spawn a replacement.
- **One team at a time:** Clean up the refinement team before creating the execution team. Only one team can exist per session.

---

## Behaviors that apply to all phases

- **Don't overwrite without asking.** If a spec file already exists, confirm with the user before replacing it. They may have made manual edits you'd lose.
- **Resolve open questions.** When you encounter `[NEEDS CLARIFICATION: ...]` markers, ask the user. Don't fill in answers based on assumptions.
- **Respect the constitution.** The principles in `specs/memory/constitution.md` apply to all decisions. If a user request conflicts with a constitutional principle, raise it — the user can override, but they should do so consciously.
- **Test-first is structural, not aspirational.** In the task list, the test task always precedes its implementation task. During execution, you write the test, run it to see it fail, then implement. If the project doesn't have a test framework set up, setting one up is the very first task. If you ever find yourself writing implementation code without a failing test already in place, stop and write the test first.
- **Non-goals are boundaries.** If something is listed in the spec's non-goals, do not build it, suggest building it, or plan for it — even if it would be "easy to add."
- **Specs say WHAT and WHY. Plans say HOW.** If you find yourself writing implementation details in a spec, move them to the plan. If you find yourself writing user stories in a plan, move them to the spec.
