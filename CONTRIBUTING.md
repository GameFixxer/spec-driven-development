# Contributing to Spec-Driven Development

Thanks for considering a contribution. This repo is small and the bar for changes is friendly: clear intent, small scope, and a short note explaining the *why*.

## What's useful

- **Bug reports** — something the `sdd` skill does wrong, gets stuck on, or fails to detect.
- **Skill improvements** — sharper phase prompts, better Graphify integration, clearer task templates, additional examples.
- **Docs** — README clarifications, install gotchas, working examples of the five-phase flow on real projects.
- **New phases or modes** — propose first via an issue (see below) so we can agree on scope before you write code.

## Filing an issue

Open one at <https://github.com/GameFixxer/spec-driven-development/issues>. A useful report includes:

1. What you ran (`/sdd ...` command, prompt, or scenario).
2. What you expected to happen.
3. What actually happened, verbatim if possible.
4. Claude Code / Cowork version and OS.
5. Whether Graphify was installed in the target repo.

Bugs that come with a reproducible scenario get fixed faster.

## Submitting a change

1. **Fork** the repo and create a branch off `main`. Use a descriptive prefix:
   - `feat/<short-topic>` for new capabilities.
   - `fix/<short-topic>` for bug fixes.
   - `docs/<short-topic>` for doc-only changes.
2. **Keep it small.** One concern per PR. Two small PRs review faster than one large one.
3. **Follow the SDD workflow if your change is non-trivial.** This repo practices what it preaches:
   - Substantial changes go through `docs/superpowers/specs/` (a design doc) and `docs/superpowers/plans/` (an implementation plan) first.
   - For small fixes (typo, single-line correction, obvious doc tweak) skip the ceremony — just open the PR.
4. **Open a pull request** against `main`. Reference any related issue. Describe the *why* in the description, not just the *what*.

## Local development

Clone your fork, then point Claude Code at your local checkout instead of the public marketplace:

```
/plugin marketplace add /absolute/path/to/your/clone
/plugin install spec-driven-development@spec-driven-development
```

Iterate on `skills/sdd/SKILL.md` and the files under `skills/sdd/references/`. Reload the skill in Claude Code to pick up changes.

When you're ready to submit, push your branch to your fork and open the PR against `GameFixxer/spec-driven-development:main`.

## Commit style

Match the existing history:

- **Subject** — imperative, under ~70 chars, no trailing period. Examples already in the repo: `Add CHANGELOG.md with 0.1.0 entry`, `Enrich plugin.json with distribution metadata`.
- **Blank line**, then a body that explains the *why*. Skip the body only for genuinely trivial commits (typo fixes, whitespace).
- One concern per commit. If you find yourself writing "and also …" in the subject, split it.

Run `git log --oneline` before pushing — your commits should read like a clean story of what changed and why.

## License

By submitting a contribution, you agree that your work is licensed under the [MIT License](LICENSE) that covers this repository.
