## ADDED Requirements

### Requirement: Domain skill init command
The `/opsx:domain` skill SHALL support an `init` subcommand that scaffolds the full domain knowledge base from scratch. `init` SHALL be conversational: it interviews the user to extract concepts, processes, terminology, and rules, then drafts files for user confirmation before writing.

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
- **WHEN** all files are written during `init`
- **THEN** the skill SHALL generate `_index.yaml` automatically

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
