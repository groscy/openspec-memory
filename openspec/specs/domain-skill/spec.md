## Portable Format Requirements

### Requirement: Domain skill stored in portable format
The `/opsx:domain` skill SHALL be stored as `openspec/skills/domain/SKILL.md` with valid portable-skill frontmatter. The `.claude/commands/opsx/domain.md` file SHALL be a generated adapter, not the canonical source.

#### Scenario: Domain skill canonical location is portable
- **WHEN** a developer checks out the OpenSpec project
- **THEN** the canonical domain skill content SHALL be found at `openspec/skills/domain/SKILL.md`
- **THEN** the Claude adapter at `.claude/commands/opsx/domain.md` SHALL be regeneratable via `openspec install skills --env claude`

#### Scenario: Domain skill is invokable from IntelliJ
- **WHEN** a user has the OpenSpec IntelliJ plugin installed and runs `openspec install skills --env intellij`
- **THEN** the domain skill SHALL appear as an IDE action
- **THEN** invoking it from IntelliJ SHALL execute the same workflow as invoking `/opsx:domain` from Claude

#### Scenario: Domain skill is invokable from Copilot
- **WHEN** a user has the Copilot adapter installed via `openspec install skills --env copilot`
- **THEN** the domain skill trigger SHALL be discoverable in GitHub Copilot Chat
- **THEN** using the trigger SHALL instruct Copilot to follow the domain skill workflow

---

## ADDED Requirements

### Requirement: Domain skill init command
The `/opsx:domain` skill SHALL support an `init` subcommand that scaffolds the full domain knowledge base from scratch and then configures all project skills. `init` SHALL be conversational: it interviews the user to extract concepts, processes, terminology, and rules, then drafts files for user confirmation before writing. After writing domain KB files, it SHALL run skill configuration for the current project.

#### Scenario: Init creates directory structure
- **WHEN** the user runs `/opsx:domain init`
- **THEN** `openspec/domain/static/concepts/`, `openspec/domain/static/processes/`, `openspec/domain/static/glossary.md`, `openspec/domain/static/rules.md`, and `openspec/domain/discovered/` SHALL be created

#### Scenario: Init interviews user before writing
- **WHEN** the user runs `/opsx:domain init`
- **THEN** the skill SHALL ask the user to describe their domain in plain language before writing any files

#### Scenario: Init reads codebase when it exists
- **WHEN** the user runs `/opsx:domain init` and source code files exist in the project
- **THEN** the skill SHALL read relevant files (models, schemas, routes) and use them to enrich the drafted concepts

#### Scenario: Init shows draft before writing
- **WHEN** the skill has gathered sufficient information
- **THEN** it SHALL show the user a summary of concepts, processes, and rules it intends to create
- **THEN** it SHALL write files only after the user confirms

#### Scenario: Init generates index after writing
- **WHEN** all domain KB files are written during `init`
- **THEN** the skill SHALL generate `_index.yaml` automatically

#### Scenario: Init configures project skills after domain KB setup
- **WHEN** domain KB files have been written and `openspec/skills/` exists
- **THEN** the skill SHALL run `openspec install skills` to configure all skills in `openspec/skills/`
- **THEN** the skill SHALL report how many adapters were written and for which environments

### Requirement: Domain skill add command
The `/opsx:domain` skill SHALL support an `add` subcommand that creates a single concept, process, rule, or actor entry and updates the index.

#### Scenario: Add creates a concept file
- **WHEN** the user runs `/opsx:domain add concept Invoice`
- **THEN** the skill SHALL create `openspec/domain/static/concepts/invoice.md` with valid frontmatter and a prose body
- **THEN** the skill SHALL regenerate `_index.yaml`

#### Scenario: Add prevents duplicate entries
- **WHEN** the user runs `/opsx:domain add` with a name that already exists in the index
- **THEN** the skill SHALL warn the user and ask whether to update the existing entry or cancel

### Requirement: Domain skill sync command
The `/opsx:domain` skill SHALL support a `sync` subcommand that regenerates `_index.yaml` from all current frontmatter without modifying any concept or discovery files.

#### Scenario: Sync regenerates index from current files
- **WHEN** the user runs `/opsx:domain sync`
- **THEN** `_index.yaml` SHALL be rewritten to reflect all current files in `openspec/domain/static/` and `openspec/domain/discovered/`

#### Scenario: Sync reports what changed
- **WHEN** sync completes
- **THEN** the skill SHALL report how many entries were added, updated, or removed compared to the previous index
