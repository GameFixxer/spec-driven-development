# Sprint Template

Use this template when creating `specs/sprints/sprint-[N].md`.

```markdown
# Sprint [N]

started: [Date]
status: refinement | execution | done
team_config: default | custom

## Team Roster

### Refinement Team (short-lived)
| Role | Responsibility |
|------|---------------|
| Product Manager | Validates specs against user value, prioritizes features, resolves scope conflicts |
| Solution Architect | Designs cross-feature technical approach, identifies shared components, resolves integration points |
| Frontend Dev | Estimates UI tasks, flags UX concerns, identifies component reuse |
| Backend Dev | Estimates API/data tasks, flags performance concerns, identifies service boundaries |
| App Dev | Estimates mobile/platform tasks, flags platform constraints |
| UI Designer | Reviews user flows, flags accessibility issues, validates design consistency |

### Execution Team (selective — spawned based on task assignments)
| Role | Spawned | Task Count |
|------|---------|------------|
| [role] | yes/no | [N] |

## Features

### [feature-name-1]
spec: specs/[feature-name-1]/spec.md
priority: P0 | P1 | P2
status: refined | in-progress | done
tasks: specs/[feature-name-1]/tasks.md

### [feature-name-2]
spec: specs/[feature-name-2]/spec.md
priority: P0 | P1 | P2
status: refined | in-progress | done
tasks: specs/[feature-name-2]/tasks.md

## Refinement Log

### [feature-name-1] — [Date]
attendees: PM, SA, FE, BE, APP, UI
decisions: [one line per key decision]
task_count: [N] tasks, [X] parallel groups
role_assignments: FE:[N], BE:[N], APP:[N], UI:[N]

## Execution Progress

phase: [current phase number]
gate_status: [waiting | cleared]

### Phase [N] Gate
all_roles_done: yes | no
blocked_roles: [list or "none"]
checkpoint: [pass | fail]
```
