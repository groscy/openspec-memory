## Why

OpenSpec Skills are currently authored and installed as Claude-specific artifacts (`.claude/commands/`, `.claude/skills/`), making them inaccessible to teams using JetBrains IntelliJ via the OpenSpec plugin or GitHub Copilot. Decoupling Skills from Claude's invocation paths and introducing a portable installation mechanism lets any AI-backed IDE participate in OpenSpec workflows without re-authoring content.

## What Changes

- Skills are stored in a vendor-neutral location (`openspec/skills/<name>/SKILL.md`) and described with an agent-agnostic frontmatter schema (name, description, args, triggers)
- A new `openspec install skills` CLI command writes the appropriate adapter files for each target environment (Claude, Copilot, IntelliJ OpenSpec plugin) from the single source of truth
- The OpenSpec IntelliJ plugin reads `openspec/skills/` directly and surfaces skills as IDE actions / run configurations
- The Copilot adapter emits `@workspace`-compatible prompt files that forward to each skill's SKILL.md
- **BREAKING**: `.claude/commands/opsx/` and `.claude/skills/` are no longer the canonical home for OpenSpec skills; the install step regenerates them as derived output

## Capabilities

### New Capabilities

- `portable-skill-format`: A vendor-neutral skill file format and directory layout (`openspec/skills/<name>/SKILL.md`) that any agent or IDE can load, with frontmatter describing name, aliases, args, triggers, and compatible agents
- `skill-installation`: The `openspec install skills` CLI command and its adapter logic — reads `openspec/skills/`, writes per-environment adapter files (`.claude/commands/`, `.github/copilot-instructions/`, IntelliJ run configs), and keeps them in sync

### Modified Capabilities

- `domain-skill`: The existing domain-skill spec describes skill file locations and invocation as Claude-specific; requirements must change to reference the new portable format and treat Claude adapter files as generated output

## Impact

- `openspec/skills/` — new canonical directory (tracked in git)
- `.claude/commands/opsx/` — becomes generated; should be git-ignored or regenerated on install
- `.github/copilot-instructions/` — new generated adapter files for Copilot
- IntelliJ OpenSpec plugin — must gain a skill-loader that reads `openspec/skills/`
- `openspec` CLI — gains `install skills [--env <claude|copilot|intellij|all>]` subcommand
- Existing projects can migrate by running `openspec install skills` once
