# Marketplace Distribution Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make `spec-driven-development` installable for any Claude Code user via two `/plugin` commands by turning the existing public GitHub repo into a self-hosted single-plugin marketplace.

**Architecture:** Add a `.claude-plugin/marketplace.json` manifest that points at the repo root (`source: "./"`), enrich the existing `.claude-plugin/plugin.json` with standard distribution metadata (homepage, repository, license, icon), add a real `icon.png` and a Keep-a-Changelog `CHANGELOG.md`, and rewrite the README install section. Tag `v0.1.0` at the end so the marketplace entry resolves to a stable ref.

**Tech Stack:** JSON manifests, Markdown, a one-shot Python/PIL script for icon generation, `jq` for JSON validation, `git` for commits and tagging. No runtime dependencies, no test framework — verification is JSON-validity, file-existence, and content grep.

**Source spec:** `docs/superpowers/specs/2026-04-24-marketplace-distribution-design.md`

---

## File Structure

| Path | Status | Responsibility |
| --- | --- | --- |
| `CHANGELOG.md` | Create | Project version history (Keep-a-Changelog). |
| `icon.png` | Create | 256×256 marketplace icon, referenced by `plugin.json`. |
| `.claude-plugin/plugin.json` | Modify | Plugin manifest — add homepage/repository/license/icon/author.url. |
| `.claude-plugin/marketplace.json` | Create | Marketplace manifest — declares this repo as a single-plugin marketplace. |
| `README.md` | Modify | Replace drag-and-drop install section with `/plugin marketplace` commands. |

Boundaries: each file has one responsibility. `plugin.json` and `marketplace.json` deliberately duplicate `version`, `description`, and `keywords`/`tags`; Task 6 verifies they stay in sync.

**Note on commits:** the source spec's rollout section suggested "one commit." This plan deviates intentionally — per-task commits give cleaner history and easier reverts for a low-risk config-only change. The `v0.1.0` tag at the end of Task 7 still produces the single addressable release the spec wanted.

---

### Task 1: Add `CHANGELOG.md`

**Files:**
- Create: `CHANGELOG.md`

- [ ] **Step 1: Write the changelog**

Create `/Users/rene.berndt/IdeaProjects/spec-driven-development/CHANGELOG.md` with exactly this content:

```markdown
# Changelog

All notable changes to this project are documented here.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] — 2026-04-25

### Added
- Initial public release of the Spec-Driven Development plugin.
- `skills/sdd` skill covering the five-phase workflow (initialize, specify, plan, task, execute) plus Sprint Mode.
- Graphify integration in the Plan phase.
- Claude Code marketplace manifest (`.claude-plugin/marketplace.json`) for one-command install from GitHub.
- Plugin icon (`icon.png`).
```

- [ ] **Step 2: Verify file exists and is non-empty**

Run: `test -s CHANGELOG.md && wc -l CHANGELOG.md`
Expected: a positive line count (≥10 lines), exit code 0.

- [ ] **Step 3: Commit**

```bash
git add CHANGELOG.md
git commit -m "$(cat <<'EOF'
Add CHANGELOG.md with 0.1.0 entry

Keep-a-Changelog format. Documents the initial public release
that ships with the marketplace manifest.

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>
EOF
)"
```

---

### Task 2: Generate `icon.png`

**Files:**
- Create: `icon.png` (binary, 256×256 PNG)

- [ ] **Step 1: Generate the icon with Python/PIL**

Run from the repo root:

```bash
python3 - <<'PY'
from PIL import Image, ImageDraw, ImageFont
size = 256
img = Image.new("RGBA", (size, size), (24, 32, 48, 255))  # deep navy background
draw = ImageDraw.Draw(img)

# Solid rounded-square accent
margin = 24
draw.rounded_rectangle(
    [(margin, margin), (size - margin, size - margin)],
    radius=32,
    fill=(255, 196, 64, 255),  # warm amber
)

# "SDD" glyph centered
text = "SDD"
try:
    font = ImageFont.truetype("/System/Library/Fonts/Supplemental/Arial Bold.ttf", 84)
except OSError:
    font = ImageFont.load_default()

bbox = draw.textbbox((0, 0), text, font=font)
tw, th = bbox[2] - bbox[0], bbox[3] - bbox[1]
draw.text(
    ((size - tw) / 2 - bbox[0], (size - th) / 2 - bbox[1]),
    text,
    fill=(24, 32, 48, 255),
    font=font,
)

img.save("icon.png", "PNG", optimize=True)
print("wrote icon.png")
PY
```

Expected output: `wrote icon.png`.

- [ ] **Step 2: Verify the icon is a real 256×256 PNG**

Run: `file icon.png && python3 -c "from PIL import Image; im=Image.open('icon.png'); print(im.size, im.format)"`
Expected: `file` reports `PNG image data, 256 x 256`; Python prints `(256, 256) PNG`.

- [ ] **Step 3: Commit**

```bash
git add icon.png
git commit -m "$(cat <<'EOF'
Add icon.png for marketplace listing

256x256 PNG generated with PIL. Solid amber rounded square on a
deep navy background with "SDD" glyph. Referenced by the icon
field added to plugin.json in the next commit.

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>
EOF
)"
```

---

### Task 3: Enrich `.claude-plugin/plugin.json`

**Files:**
- Modify: `.claude-plugin/plugin.json` (full rewrite)

- [ ] **Step 1: Capture the current contents for the diff message**

Run: `cat .claude-plugin/plugin.json`
Expected: shows the current minimal manifest (name, version, description, author, keywords).

- [ ] **Step 2: Replace the file with the enriched manifest**

Write `.claude-plugin/plugin.json` with exactly this content:

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

- [ ] **Step 3: Verify JSON validity and that the new fields are present**

Run:
```bash
jq empty .claude-plugin/plugin.json && \
jq -r '.homepage, .repository.url, .license, .icon, .author.url' .claude-plugin/plugin.json
```

Expected (each on its own line, no nulls):
```
https://github.com/GameFixxer/spec-driven-development
https://github.com/GameFixxer/spec-driven-development.git
MIT
icon.png
https://github.com/GameFixxer
```

- [ ] **Step 4: Verify the icon file the manifest now points at actually exists**

Run: `test -f icon.png && echo OK`
Expected: `OK`.

- [ ] **Step 5: Commit**

```bash
git add .claude-plugin/plugin.json
git commit -m "$(cat <<'EOF'
Enrich plugin.json with distribution metadata

Adds homepage, repository, license, icon, and author.url so the
manifest carries everything a marketplace listing or plugin
review pipeline expects. License remains MIT (matches LICENSE).

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>
EOF
)"
```

---

### Task 4: Add `.claude-plugin/marketplace.json`

**Files:**
- Create: `.claude-plugin/marketplace.json`

- [ ] **Step 1: Write the marketplace manifest**

Create `.claude-plugin/marketplace.json` with exactly this content:

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

- [ ] **Step 2: Verify JSON validity and shape**

Run:
```bash
jq empty .claude-plugin/marketplace.json && \
jq -r '.name, .plugins[0].name, .plugins[0].source, .plugins[0].version' .claude-plugin/marketplace.json
```

Expected:
```
spec-driven-development
spec-driven-development
./
0.1.0
```

- [ ] **Step 3: Commit**

```bash
git add .claude-plugin/marketplace.json
git commit -m "$(cat <<'EOF'
Add marketplace.json — single-plugin marketplace manifest

Declares this repo as a Claude Code marketplace containing one
plugin sourced from the repo root. Enables installation via:

    /plugin marketplace add GameFixxer/spec-driven-development
    /plugin install spec-driven-development@spec-driven-development

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>
EOF
)"
```

---

### Task 5: Update the README install section

**Files:**
- Modify: `README.md` (replace the "Installation" section, lines 25-27 in the current file)

- [ ] **Step 1: Confirm the current install paragraph**

Run: `sed -n '25,27p' README.md`
Expected output (verbatim — confirms we're replacing the right lines):
```
## Installation

Drag and drop `spec-driven-development.plugin` into Claude Code or Cowork to install.
```

- [ ] **Step 2: Replace the install section**

Use Edit to replace exactly this block in `README.md`:

Old:
```
## Installation

Drag and drop `spec-driven-development.plugin` into Claude Code or Cowork to install.
```

New:
```
## Installation

Install via the Claude Code marketplace:

```text
/plugin marketplace add GameFixxer/spec-driven-development
/plugin install spec-driven-development@spec-driven-development
```

That's it — the `sdd` skill becomes available immediately.
```

(Note: the inner fence uses ```text to keep the outer Markdown fence un-confused. If the existing README does not use language-tagged fences elsewhere, you may use a plain ``` fence — the rendered output is the same.)

- [ ] **Step 3: Verify the new install commands are present and the stale line is gone**

Run:
```bash
grep -q 'plugin marketplace add GameFixxer/spec-driven-development' README.md && \
grep -q 'plugin install spec-driven-development@spec-driven-development' README.md && \
! grep -q 'Drag and drop `spec-driven-development.plugin`' README.md && \
echo OK
```

Expected: `OK`.

- [ ] **Step 4: Commit**

```bash
git add README.md
git commit -m "$(cat <<'EOF'
Rewrite README install section for marketplace distribution

Replaces the drag-and-drop instruction (which referenced a
.plugin bundle that does not ship in the repo) with the two
/plugin marketplace commands users actually run.

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>
EOF
)"
```

---

### Task 6: Cross-file consistency validation

No file changes. This task asserts the invariants the spec demands between `plugin.json` and `marketplace.json`. If any check fails, fix the offending file in the appropriate earlier task and re-run all checks before proceeding.

- [ ] **Step 1: Both manifests are valid JSON**

Run:
```bash
jq empty .claude-plugin/plugin.json && \
jq empty .claude-plugin/marketplace.json && \
echo "json OK"
```
Expected: `json OK`.

- [ ] **Step 2: Versions match across the two manifests**

Run:
```bash
PV=$(jq -r '.version' .claude-plugin/plugin.json)
MV=$(jq -r '.plugins[0].version' .claude-plugin/marketplace.json)
[ "$PV" = "$MV" ] && echo "version match: $PV"
```
Expected: `version match: 0.1.0`.

- [ ] **Step 3: Descriptions match across the two manifests**

Run:
```bash
PD=$(jq -r '.description' .claude-plugin/plugin.json)
MD=$(jq -r '.plugins[0].description' .claude-plugin/marketplace.json)
[ "$PD" = "$MD" ] && echo "description match"
```
Expected: `description match`.

- [ ] **Step 4: Keywords (plugin.json) match tags (marketplace.json) as sets**

Run:
```bash
PK=$(jq -c '.keywords | sort' .claude-plugin/plugin.json)
MT=$(jq -c '.plugins[0].tags | sort' .claude-plugin/marketplace.json)
[ "$PK" = "$MT" ] && echo "tags match: $PK"
```
Expected: `tags match: ["development","graphify","planning","sdd","spec","workflow"]`.

- [ ] **Step 5: Icon file exists and is a PNG**

Run: `file icon.png | grep -q 'PNG image data, 256 x 256' && echo "icon OK"`
Expected: `icon OK`.

- [ ] **Step 6: Working tree is clean (nothing accidentally left unstaged)**

Run: `git status --porcelain`
Expected: empty output.

(No commit for this task — it's pure verification.)

---

### Task 7: Tag the release

**Files:** none.

- [ ] **Step 1: Confirm we are on `main` and up to date**

Run:
```bash
git rev-parse --abbrev-ref HEAD
git status --porcelain
```
Expected: branch is `main`; porcelain is empty.

- [ ] **Step 2: Confirm no v0.1.0 tag already exists**

Run: `git tag -l v0.1.0`
Expected: empty output.

- [ ] **Step 3: Create the annotated tag**

```bash
git tag -a v0.1.0 -m "v0.1.0 — initial public release via Claude Code marketplace"
```

- [ ] **Step 4: Verify the tag points at HEAD**

Run: `git rev-parse v0.1.0 && git rev-parse HEAD`
Expected: both commands print the same SHA.

- [ ] **Step 5: Push commits and the tag** *(only run after a human has eyeballed the commits — see manual smoke test below)*

```bash
git push origin main
git push origin v0.1.0
```

- [ ] **Step 6: Manual smoke test in a fresh Claude Code session**

In a clean working directory, in a fresh Claude Code session:

```
/plugin marketplace add GameFixxer/spec-driven-development
/plugin install spec-driven-development@spec-driven-development
```

Verify:
- `marketplace add` reports success and lists `spec-driven-development` as a known marketplace.
- `install` reports success.
- The `sdd` skill appears in the available-skills list.
- Invoking the skill (e.g., asking "set up SDD for this project") triggers the SDD flow.

If any step fails, do not delete the tag — investigate and ship a `v0.1.1` patch through the same plan template.

---

## Self-Review

**Spec coverage** (against `docs/superpowers/specs/2026-04-24-marketplace-distribution-design.md`):

| Spec section | Plan task |
| --- | --- |
| Marketplace manifest (`marketplace.json`) | Task 4 |
| Plugin manifest enrichment | Task 3 |
| Icon (`icon.png`, real PNG, PIL primary path) | Task 2 |
| Changelog (Keep-a-Changelog, 0.1.0) | Task 1 |
| README install rewrite (drop drag-and-drop) | Task 5 |
| JSON validity checks | Task 6 steps 1-2 |
| Version/description sync | Task 6 steps 2-3 |
| Tag `v0.1.0` after commits land | Task 7 |
| Manual smoke test | Task 7 step 6 |

No spec gaps. The placeholder-icon fallback in the spec is not invoked because PIL is available in the local environment (verified before plan was written).

**Placeholder scan:** No "TBD", "TODO", "implement later", or vague-tests-for-the-above patterns. Every code block is the exact content to write.

**Type/name consistency:** `spec-driven-development` is used identically as marketplace name and plugin name. Install command form `spec-driven-development@spec-driven-development` (plugin@marketplace) is used identically in spec, plan Task 5, README, and Task 7 smoke test. `0.1.0` is the only version string used anywhere. Icon path is `icon.png` (repo root) in every reference. Author block uses the same `name` and `url` in both manifests.
