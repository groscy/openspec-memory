# Changelog

## Unreleased

### Breaking Changes

- **`.claude/commands/opsx/` is no longer the canonical source for OpenSpec skills.** The canonical source has moved to `openspec/skills/<name>/SKILL.md`. The adapter files in `.claude/commands/opsx/` (and the new `.github/copilot-instructions/` and `.idea/runConfigurations/` files) are derived output — do not edit them directly. To update a skill, edit `openspec/skills/<name>/SKILL.md` and regenerate the adapter file manually.

### Added

- `openspec/skills/` — vendor-neutral directory holding all OpenSpec skill files. Each skill is `openspec/skills/<name>/SKILL.md` with an agent-agnostic frontmatter block (`name`, `description`, `triggers`, `agents`, optional `args` and `version`).
- `.github/copilot-instructions/<name>.md` — pre-built Copilot Chat adapter files. Copy to your project's `.github/copilot-instructions/` to use skills in GitHub Copilot Chat.
- `.idea/runConfigurations/opsx_<name>.xml` — pre-built JetBrains run configurations. Copy to your project's `.idea/runConfigurations/` to surface skills in the IntelliJ Run menu.

### Migration Guide

If you have an existing project with hand-edited `.claude/commands/opsx/` files:

1. Create `openspec/skills/<name>/SKILL.md` for each skill — copy the prose body from your existing command file and add the frontmatter block (`name`, `description`, `triggers`, `agents`).
2. Regenerate the Claude adapter: copy the prose body (without frontmatter) into `.claude/commands/opsx/<name>.md`, prefixed with a `<!-- Generated … -->` comment header.
3. Create Copilot and IntelliJ adapter files from the same source if needed.

To roll back: restore `.claude/commands/opsx/` from git history. The skill content is preserved — no data loss.
