# OpenSpec Skill: domain

**Trigger**: `/opsx:domain`

**Description**: Manage the OpenSpec domain knowledge base - initialize, add entries, and sync the index

**Usage**: Start a Copilot Chat message with `/opsx:domain <subcommand>` to invoke this skill.

Subcommands: `init`, `add <type> <name>`, `sync`

<!-- Source: openspec/skills/domain/SKILL.md -->

---

Manage the OpenSpec domain knowledge base.

**Input**: A subcommand: `init`, `add <type> <name>`, or `sync`. If none provided, show the list and stop.

---

## Subcommand: init

Scaffold the domain KB from scratch.

1. Check if `openspec/domain/` already exists — warn and ask to reinitialize or cancel if so.
2. Interview the user: "Describe your project's domain — main concepts, processes, terminology, and business rules."
3. Optionally scan source directories (`src/`, `models/`, etc.) to enrich concepts.
4. Show a draft summary and ask for confirmation before writing any files.
5. Create directories: `openspec/domain/static/concepts/`, `static/processes/`, `discovered/`
6. Write concept/process files, `glossary.md`, and `rules.md`.
7. Generate `openspec/domain/_index.yaml` (see Index Generation Procedure below).
8. **Configure OpenSpec skills**: if `openspec/skills/` exists, read each `openspec/skills/<name>/SKILL.md` and write a `.claude/commands/opsx/<name>.md` adapter (description frontmatter + header comment + prose body).
9. Report what was created, including which skill files were configured.

## Subcommand: add

`/opsx:domain add <type> <name>` — type is `concept`, `process`, `rule`, or `actor`.

1. Check `openspec/domain/` exists; error if not.
2. Check for duplicate names in `_index.yaml`.
3. Interview for content (aliases, states, related, constraints).
4. Write the file; regenerate `_index.yaml`.

## Subcommand: sync

Regenerate `_index.yaml` from all current frontmatter and report changes (added/updated/removed).

## Index Generation Procedure

1. Scan `openspec/domain/static/concepts/`, `static/processes/`, `discovered/`
2. Parse `glossary.md` (one `### Term` per entry) and `rules.md` (one `### Rule: <id>` per entry)
3. Validate frontmatter; skip invalid files with a warning
4. Write `openspec/domain/_index.yaml` with keys: `concepts`, `processes`, `glossary`, `rules`, `discoveries`
5. Use first sentence of prose body as `def`; emit `[]` for categories with no entries

**Required concept/process frontmatter fields**: `name`, `type`, `aliases`, `related`

**Guardrails**
- Never hand-edit `_index.yaml` — always regenerate
- Only write files after explicit user confirmation during `init`
- Always regenerate the index after `add` or `init`
