# Marketplace Distribution for `spec-driven-development`

**Date:** 2026-04-24
**Status:** Approved (design)
**Author:** René Berndt
**Spec type:** Packaging / distribution

## Context

The `spec-driven-development` plugin is already "Published" via `claude.ai/settings/plugins/submissions`, but that state only makes it visible in the submitting user's **Personal** tab. It does not appear in the `Anthropic & Partners` tab (curated by Anthropic), and no other user can discover or install it through the claude.ai UI today.

The repo lives at `github.com/GameFixxer/spec-driven-development` and is public. The lowest-friction way to let other Claude Code users install the plugin — without waiting on curation — is to turn the existing GitHub repo into a self-hosted **Claude Code marketplace**. Users then install with:

```
/plugin marketplace add GameFixxer/spec-driven-development
/plugin install spec-driven-development@spec-driven-development
```

Applying to the Anthropic Partner program for listing in the curated tab is a separate, independent track and is out of scope for this spec.

## Goals

- Any Claude Code user can add and install the plugin with two commands, no drag-and-drop, no claude.ai account required.
- The repo contains all metadata a review pipeline or curious user typically looks for (homepage, repository, license, icon, changelog).
- The change is minimal, YAGNI-respecting, and does not preempt a future move to a multi-plugin marketplace.

## Non-goals

- Multi-plugin marketplace scaffolding.
- Screenshots or a gallery directory.
- GitHub Actions / release automation.
- Version bump to 1.0.0 (0.1.0 remains honest for a first public cut).
- Partner-program application materials.
- Breaking changes to the existing `skills/sdd/SKILL.md` content.

## Design

### Repository layout after this change

```
spec-driven-development/
├── .claude-plugin/
│   ├── plugin.json           # enriched (see §Plugin manifest)
│   └── marketplace.json      # NEW — marketplace manifest
├── skills/sdd/               # unchanged
├── docs/superpowers/specs/   # this spec lives here
├── icon.png                  # NEW — 256×256 plugin icon
├── CHANGELOG.md              # NEW
├── README.md                 # install instructions rewritten
├── LICENSE                   # unchanged
└── .gitignore                # unchanged
```

### Marketplace manifest — `.claude-plugin/marketplace.json`

A single-plugin marketplace. `source: "./"` tells Claude Code that the plugin lives at the root of this same repository, so one repo serves as both marketplace and plugin.

```json
{
  "name": "spec-driven-development",
  "owner": {
    "name": "René Berndt @ GameFixxer",
    "url": "https://github.com/GameFixxer"
  },
  "plugins": [
    {
      "name": "spec-driven-development",
      "source": "./",
      "description": "Spec-Driven Development (SDD) — a structured, specification-first workflow for planning and executing software features with Claude. Covers five phases: initialize, specify, plan, task, and execute. Includes Graphify integration for knowledge-graph-aware codebase navigation.",
      "version": "0.1.0",
      "category": "workflow",
      "tags": ["sdd", "spec", "planning", "workflow", "development", "graphify"],
      "author": {
        "name": "René Berndt @ GameFixxer",
        "url": "https://github.com/GameFixxer"
      },
      "homepage": "https://github.com/GameFixxer/spec-driven-development",
      "license": "MIT"
    }
  ]
}
```

### Plugin manifest — `.claude-plugin/plugin.json` (enriched)

```json
{
  "name": "spec-driven-development",
  "version": "0.1.0",
  "description": "Spec-Driven Development (SDD) — a structured, specification-first workflow for planning and executing software features with Claude. Covers five phases: initialize, specify, plan, task, and execute. Includes Graphify integration for knowledge-graph-aware codebase navigation.",
  "author": {
    "name": "René Berndt @ GameFixxer",
    "url": "https://github.com/GameFixxer"
  },
  "homepage": "https://github.com/GameFixxer/spec-driven-development",
  "repository": {
    "type": "git",
    "url": "https://github.com/GameFixxer/spec-driven-development.git"
  },
  "license": "MIT",
  "icon": "icon.png",
  "keywords": ["sdd", "spec", "planning", "workflow", "development", "graphify"]
}
```

Note: `version`, `description`, `keywords`, and `author.name` stay in sync between `plugin.json` and `marketplace.json`. Drift between the two is the single most common cause of subtle install bugs; the implementation plan must assert they match.

### Icon — `icon.png`

A 256×256 PNG rendered at implementation time. Primary approach: a simple generated glyph (text "SDD" on a solid background) using Python/PIL, committed at `./icon.png`. Fallback if PIL is not available in the execution environment: commit a 1×1 transparent placeholder PNG and add a clearly-worded `TODO` in `CHANGELOG.md` so the author replaces it before the next release. The `plugin.json` `icon` field is populated either way — the field is valid even if the PNG is a placeholder.

### Changelog — `CHANGELOG.md`

Keep-a-Changelog style. Opens with:

```
# Changelog

All notable changes to this project are documented here.
The format is based on Keep a Changelog and this project adheres to Semantic Versioning.

## [0.1.0] — 2026-04-24
### Added
- Initial public release of the Spec-Driven Development plugin.
- `skills/sdd` skill covering the five-phase workflow (initialize, specify, plan, task, execute) plus Sprint Mode.
- Graphify integration in the Plan phase.
- Claude Code marketplace manifest for one-command install from GitHub.
```

### README install section (replacement)

Replace the current "Drag and drop" paragraph with:

```
## Installation

Install via the Claude Code marketplace:

    /plugin marketplace add GameFixxer/spec-driven-development
    /plugin install spec-driven-development@spec-driven-development

That's it — the `sdd` skill becomes available immediately.
```

The old drag-and-drop line is removed because the repo does not ship a prebuilt `.plugin` bundle; keeping that instruction would mislead users. If a bundle is produced in a later release, the install section can be expanded then.

## Error handling / edge cases

- **Private repo regression:** if the repo is ever flipped back to private, `marketplace add` will fail with an auth error. Not something this spec fixes; it's an operational invariant.
- **JSON drift:** `version` and `description` are duplicated across `plugin.json` and `marketplace.json`. Implementation plan will include a validation step (`jq` equality check) in the verification phase, not a CI job (YAGNI).
- **Icon missing:** covered above — placeholder-with-TODO path is acceptable.
- **Install name collision:** the marketplace name and plugin name are both `spec-driven-development`, so the explicit install form `spec-driven-development@spec-driven-development` (plugin@marketplace) is used in docs to remove any ambiguity.

## Testing / verification

After implementation, the following must pass:

1. `jq empty .claude-plugin/plugin.json` — valid JSON.
2. `jq empty .claude-plugin/marketplace.json` — valid JSON.
3. `jq -r '.version' .claude-plugin/plugin.json` equals `jq -r '.plugins[0].version' .claude-plugin/marketplace.json`.
4. `jq -r '.description' .claude-plugin/plugin.json` equals `jq -r '.plugins[0].description' .claude-plugin/marketplace.json`.
5. `test -f icon.png && file icon.png | grep -q PNG` — icon exists and is a real PNG.
6. `grep -q 'plugin marketplace add GameFixxer/spec-driven-development' README.md` — install docs updated.
7. Manual smoke test in a fresh Claude Code session:
   - `/plugin marketplace add GameFixxer/spec-driven-development` succeeds.
   - `/plugin install spec-driven-development@spec-driven-development` succeeds.
   - The `sdd` skill is listed under available skills afterwards.

## Rollout

1. Land all file changes in one commit on `main`.
2. Tag `v0.1.0` on the resulting commit so the marketplace entry resolves to a stable ref.
3. Run the manual smoke test from §Testing in a clean environment.
4. Share the two install commands wherever users are being directed.
