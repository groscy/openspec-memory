---
name: Skill
type: concept
aliases: [command, slash-command]
related: [Schema, Change]
---
A Skill is a markdown instruction file stored in `openspec/skills/<name>/SKILL.md` that any capable AI agent or IDE can load and execute to implement an OpenSpec workflow command. Skills are invoked via slash commands (e.g., `/opsx:propose`, `/opsx:archive`) or through IDE integrations.

The canonical source of truth for every skill is `openspec/skills/<name>/SKILL.md`. Per-environment adapter files (`.claude/commands/opsx/<name>.md`, `.github/copilot-instructions/<name>.md`, IntelliJ run configs) are **generated output** produced by running `openspec install skills`. They must never be edited directly.

Every SKILL.md begins with an agent-agnostic YAML frontmatter block declaring `name`, `description`, `triggers`, and `agents` (required), plus optional `args` and `version` fields. The prose body contains plain markdown instructions that any instruction-following AI can act on; Claude-specific tool hints may appear as HTML comments (`<!-- Claude: use the Read tool -->`).

Skills may call the `openspec` CLI for status and instructions, and have access to file read/write operations for implementing changes.
