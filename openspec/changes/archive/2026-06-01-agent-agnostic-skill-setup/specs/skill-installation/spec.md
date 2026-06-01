## ADDED Requirements

### Requirement: Install skills CLI command
The `openspec` CLI SHALL provide an `install skills` subcommand that reads all skills from `openspec/skills/` and generates per-environment adapter files. The command SHALL accept an `--env` flag accepting `claude`, `copilot`, `intellij`, or `all` (default: `all`).

#### Scenario: Install generates adapters for all environments
- **WHEN** the user runs `openspec install skills`
- **THEN** adapter files SHALL be written for every environment listed in each skill's `agents` field
- **THEN** the command SHALL print a summary of files written and skipped

#### Scenario: Install is idempotent
- **WHEN** the user runs `openspec install skills` twice with no changes to skill files
- **THEN** the adapter files SHALL be byte-identical on both runs
- **THEN** no warning or error SHALL be emitted

#### Scenario: Install with targeted environment flag
- **WHEN** the user runs `openspec install skills --env copilot`
- **THEN** only Copilot adapter files SHALL be written
- **THEN** Claude and IntelliJ adapter files SHALL remain untouched

#### Scenario: Install reports validation errors without writing
- **WHEN** a SKILL.md has missing required frontmatter fields
- **THEN** the command SHALL print a validation error for that skill
- **THEN** no adapter files SHALL be written for the invalid skill
- **THEN** all valid skills SHALL still be processed

### Requirement: Claude adapter output
For each skill with `claude` in its `agents` field, `openspec install skills` SHALL write a file to `.claude/commands/opsx/<name>.md` containing the skill's prose body prefixed with the description and trigger metadata as a comment header.

#### Scenario: Claude adapter file is written correctly
- **WHEN** `openspec install skills --env claude` runs for a skill named `propose`
- **THEN** `.claude/commands/opsx/propose.md` SHALL be created or overwritten
- **THEN** the file SHALL contain the skill's full prose body
- **THEN** the file SHALL include the trigger (`/opsx:propose`) and description in a header comment

#### Scenario: Removed skill cleans up Claude adapter
- **WHEN** a skill directory is deleted from `openspec/skills/`
- **THEN** running `openspec install skills --env claude` SHALL delete the corresponding `.claude/commands/opsx/<name>.md`

### Requirement: Copilot adapter output
For each skill with `copilot` in its `agents` field, `openspec install skills` SHALL write a file to `.github/copilot-instructions/<name>.md` containing the skill trigger pattern and prose body formatted for GitHub Copilot Chat consumption.

#### Scenario: Copilot adapter file is written correctly
- **WHEN** `openspec install skills --env copilot` runs for a skill named `propose`
- **THEN** `.github/copilot-instructions/propose.md` SHALL be created or overwritten
- **THEN** the file SHALL begin with the trigger pattern and usage instructions
- **THEN** the skill's prose body SHALL follow verbatim

#### Scenario: Copilot adapter includes trigger instruction
- **WHEN** a user opens the generated Copilot adapter file
- **THEN** they SHALL see a human-readable instruction explaining how to invoke this skill in Copilot Chat

### Requirement: IntelliJ adapter output
For each skill with `intellij` in its `agents` field, `openspec install skills` SHALL write a JetBrains run configuration XML to `.idea/runConfigurations/opsx_<name>.xml` that invokes the skill via the OpenSpec IntelliJ plugin action.

#### Scenario: IntelliJ run configuration is written
- **WHEN** `openspec install skills --env intellij` runs for a skill named `propose`
- **THEN** `.idea/runConfigurations/opsx_propose.xml` SHALL be created or overwritten
- **THEN** the run config SHALL reference the correct plugin action ID and skill name

#### Scenario: IntelliJ plugin reads skills directory directly
- **WHEN** the OpenSpec IntelliJ plugin loads a project containing `openspec/skills/`
- **THEN** it SHALL discover all SKILL.md files and register them as available IDE actions
- **THEN** each skill's `name` and `description` frontmatter fields SHALL appear in the IDE action list

### Requirement: Skill migration from Claude-specific layout
The `openspec` CLI SHALL provide a `migrate skills` subcommand that converts existing `.claude/commands/opsx/*.md` files into the portable `openspec/skills/<name>/SKILL.md` format, adding generated frontmatter based on file name and content heuristics.

#### Scenario: Migration moves content to portable location
- **WHEN** the user runs `openspec migrate skills`
- **THEN** each `.claude/commands/opsx/<name>.md` SHALL produce a corresponding `openspec/skills/<name>/SKILL.md` with inferred frontmatter
- **THEN** the original files SHALL remain untouched until the user explicitly removes them

#### Scenario: Migration warns on local modifications
- **WHEN** a `.claude/commands/opsx/<name>.md` file has been locally modified since it was last generated
- **THEN** the migration command SHALL print a warning listing the modified file
- **THEN** it SHALL ask the user to confirm before overwriting the portable skill with migrated content

#### Scenario: Migration runs install after completion
- **WHEN** `openspec migrate skills` completes successfully
- **THEN** it SHALL automatically run `openspec install skills --env all`
- **THEN** the user SHALL see adapter files regenerated for all target environments
