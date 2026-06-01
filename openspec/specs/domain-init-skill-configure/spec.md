## Requirements

### Requirement: Domain init configures project skills
After completing domain KB setup, the `/opsx:domain init` subcommand SHALL discover all `SKILL.md` files under `openspec/skills/` in the current project and invoke `openspec install skills` to generate adapter files for the current AI environment.

#### Scenario: Init installs skills after domain KB is written
- **WHEN** the user confirms the domain KB draft and files are written
- **THEN** the skill SHALL run `openspec install skills` to configure all skills found in `openspec/skills/`
- **THEN** the skill SHALL report how many skill adapters were written

#### Scenario: Init skips skill configuration when openspec/skills/ is absent
- **WHEN** the user runs `/opsx:domain init` and no `openspec/skills/` directory exists
- **THEN** the skill SHALL complete domain KB setup normally
- **THEN** the skill SHALL warn: "No `openspec/skills/` directory found — skipping skill configuration"

#### Scenario: Init reports skill configuration results
- **WHEN** `openspec install skills` completes successfully during init
- **THEN** the skill SHALL list the skill names that were configured
- **THEN** the skill SHALL show the environments targeted (e.g., `claude`, `copilot`)

### Requirement: Skills distributable contains only the domain skill
The `skills/` distributable package SHALL contain only the domain skill (`skills/skills/openspec-domain/SKILL.md`). The `apply-change`, `explore`, `propose`, and `archive-change` skills SHALL NOT be present in `skills/skills/`.

#### Scenario: Distributable has single skill entry
- **WHEN** a user copies the `skills/` directory to their project
- **THEN** only `skills/skills/openspec-domain/` SHALL be present under `skills/skills/`
- **THEN** no other skill subdirectories SHALL exist in `skills/skills/`

#### Scenario: Other skills are installed via init, not distributable
- **WHEN** the user runs `/opsx:domain init` in a project with `openspec/skills/` populated
- **THEN** the apply, propose, explore, and archive skills SHALL be configured from `openspec/skills/`
- **THEN** they SHALL NOT be sourced from the `skills/` distributable package
