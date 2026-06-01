# Changelog

## Unreleased

### Breaking Changes

- **`.claude/commands/opsx/` is no longer committed to the repo.** These files are now written by `/opsx:domain init` from the canonical sources in `openspec/skills/` and are gitignored. Do not edit them directly — they will be overwritten on the next init.

### Added

- `openspec/skills/` — vendor-neutral directory for all OpenSpec skill files. Each skill is `openspec/skills/<name>/SKILL.md` with an agent-agnostic frontmatter block (`name`, `description`, `triggers`, `agents`, optional `args` and `version`).
- `/opsx:domain init` now configures OpenSpec skills as part of domain initialization — it writes `.claude/commands/opsx/<name>.md` adapter files from the `openspec/skills/` sources after setting up the domain KB.

### Migration Guide

If you have an existing project with hand-edited `.claude/commands/opsx/` files:

1. Create `openspec/skills/<name>/SKILL.md` for each skill — copy the prose body from your existing command file and add the frontmatter block (`name`, `description`, `triggers`, `agents`).
2. Add `.claude/commands/opsx/` to your `.gitignore`.
3. Run `/opsx:domain init` to regenerate the adapter files from the new sources.

To roll back: restore `.claude/commands/opsx/` from git history and remove it from `.gitignore`.
