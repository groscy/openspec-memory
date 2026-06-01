## MODIFIED Requirements

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
