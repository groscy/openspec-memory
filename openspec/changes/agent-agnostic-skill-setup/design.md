## Context

OpenSpec Skills are currently stored in Claude-specific directories (`.claude/commands/opsx/`, `.claude/skills/`). Teams using JetBrains IntelliJ with the OpenSpec plugin or GitHub Copilot cannot participate in OpenSpec workflows because there is no adapter layer between those environments and the skill files. The fix is to establish a single vendor-neutral source of truth and generate per-environment adapter files from it on demand.

## Goals / Non-Goals

**Goals:**
- Define a vendor-neutral `openspec/skills/<name>/SKILL.md` format as canonical home for all skills
- Provide `openspec install skills` CLI to regenerate per-environment adapters
- Support Claude, Copilot, and IntelliJ OpenSpec plugin as first-class targets
- Allow existing projects to migrate without re-authoring skill content

**Non-Goals:**
- Runtime execution of skills (still delegated to each AI agent / IDE)
- Supporting agents beyond the three listed targets in the first iteration
- Automatic detection or hot-reload of new skills (install is an explicit step)

## Decisions

### D1: Single source, generated adapters (not multi-copy)

Keep one SKILL.md per skill in `openspec/skills/`. Each `openspec install skills` run writes thin adapter files per target (`.claude/commands/`, `.github/copilot-instructions/`, IntelliJ run config XML). Adapter files are derived output — gitignored or regenerated, never edited directly.

**Alternative considered:** maintain separate files per environment in parallel. Rejected: divergence is inevitable; any edit must be replicated N times.

### D2: Frontmatter-driven adapter generation

SKILL.md files carry a YAML frontmatter block that declares `name`, `description`, `args`, `triggers`, and `agents` (list of compatible targets). The install command reads frontmatter only — not the prose body — to generate each adapter, keeping adapter generation fast and the skill body unmodified.

```yaml
---
name: propose
description: "Propose a new change and generate all artifacts"
args: "<description>"
triggers:
  - /opsx:propose
agents: [claude, copilot, intellij]
---
```

**Alternative considered:** a separate `skill.yaml` manifest alongside SKILL.md. Rejected: increases file count and risks the manifest going stale relative to the prose.

### D3: Copilot adapter as `.github/copilot-instructions/<name>.md`

Copilot doesn't have a native slash-command system at the file level. The adapter writes an `@workspace`-style instructions file that includes the skill's trigger pattern and references the SKILL.md content inline. Copilot Chat can then be prompted with the trigger to load context.

**Alternative considered:** a single aggregate `copilot-instructions.md` combining all skills. Rejected: merging and splitting on re-run is fragile; one file per skill is idempotent.

### D4: IntelliJ adapter as run configuration XML

The OpenSpec IntelliJ plugin reads `openspec/skills/` directly at project open. For the `openspec install skills --env intellij` path, the installer also writes a `.idea/runConfigurations/opsx_<name>.xml` run config so skills appear in the Run menu without requiring a plugin version bump.

**Alternative considered:** require a plugin update to scan `openspec/skills/` automatically (no install step). Preferred long-term, but install-based run configs work with the current plugin version as an interim solution.

### D5: Migration path — `openspec migrate skills`

A migration subcommand reads existing `.claude/commands/opsx/*.md` files, moves their content to `openspec/skills/<name>/SKILL.md` (adding frontmatter), then runs `openspec install skills --env all`. This is a one-time, non-destructive operation (originals are left in place until the user removes them).

## Risks / Trade-offs

- **Copilot trigger fidelity** → Copilot doesn't enforce slash-command syntax; adapter relies on user using the trigger text. Mitigation: document the trigger pattern clearly in the adapter file header.
- **IntelliJ plugin lag** → Plugin may not auto-detect new skills until project reload. Mitigation: document that `openspec install skills` should be re-run after adding a skill; plugin roadmap item to add file-watcher.
- **BREAKING: `.claude/commands/` is now generated** → Existing projects that hand-edited those files will lose edits on next install. Mitigation: `migrate skills` warns if it detects local modifications before overwriting; migration guide in CHANGELOG.

## Migration Plan

1. Add `openspec/skills/` directory with all existing skill content + frontmatter
2. Run `openspec install skills --env all` to regenerate adapters
3. Add generated adapter directories to `.gitignore` (or keep them committed — both are valid)
4. For existing projects: run `openspec migrate skills` once, then remove old `.claude/commands/opsx/` entries

Rollback: restore `.claude/commands/opsx/` from git history; no data loss since skills are content-identical.

## Open Questions

- Should `openspec install skills` be part of `openspec init` by default (run automatically on project setup)?
- Should adapter files be committed to git (portable, visible to CI) or gitignored (clean, always regenerated)? Leaning toward committed with a note that they are generated.
- IntelliJ plugin version that gains direct `openspec/skills/` reading — timeline TBD with plugin team.
