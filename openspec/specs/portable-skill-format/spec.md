## Requirements

### Requirement: Vendor-neutral skill directory layout
Every OpenSpec Skill SHALL be stored as `openspec/skills/<name>/SKILL.md` in the project repository, where `<name>` is the kebab-case skill identifier. This location SHALL be the canonical source of truth; per-environment adapter files are derived output.

#### Scenario: Skill file is present in portable location
- **WHEN** a skill named `propose` exists
- **THEN** its content SHALL be found at `openspec/skills/propose/SKILL.md`
- **THEN** no adapter-specific location SHALL be required to host the primary content

#### Scenario: Canonical location survives adapter deletion
- **WHEN** a user deletes all `.claude/commands/` adapter files
- **THEN** the skill content in `openspec/skills/` SHALL remain intact and re-installable

### Requirement: Agent-agnostic frontmatter schema
Every SKILL.md SHALL begin with a YAML frontmatter block containing at minimum: `name` (string), `description` (string), `triggers` (list of strings), and `agents` (list of strings). Additional optional fields: `args` (string describing accepted arguments), `version` (semver string).

#### Scenario: Valid frontmatter is parseable
- **WHEN** `openspec install skills` reads a SKILL.md
- **THEN** it SHALL parse the frontmatter without error if all required fields are present
- **THEN** it SHALL emit a validation error listing missing fields if any required field is absent

#### Scenario: agents field controls adapter generation
- **WHEN** a skill has `agents: [claude, copilot]` and install runs with `--env all`
- **THEN** adapters SHALL be generated for `claude` and `copilot` only
- **THEN** no IntelliJ adapter SHALL be written for that skill

#### Scenario: Frontmatter does not appear in generated adapters
- **WHEN** an adapter file is written for any environment
- **THEN** the frontmatter YAML block SHALL NOT be copied verbatim into the adapter content
- **THEN** the adapter SHALL use only fields extracted from frontmatter (name, description, trigger)

### Requirement: Prose body is agent-agnostic instruction text
The prose body of a SKILL.md (everything after the frontmatter) SHALL be written as plain markdown instructions executable by any capable AI agent. It SHALL NOT reference Claude-specific tool names, API identifiers, or SDK method names directly.

#### Scenario: Skill body uses generic action language
- **WHEN** a SKILL.md instructs the agent to read a file
- **THEN** it SHALL use phrases like "read the file at `<path>`" rather than tool-specific invocations
- **THEN** it SHALL describe the intended action in natural language that any instruction-following AI can act on

#### Scenario: Claude-specific tooling referenced as optional hints
- **WHEN** a skill body needs to hint at a specific tool for performance reasons
- **THEN** it SHALL wrap the hint in an optional block: `<!-- Claude: use the Read tool -->`
- **THEN** other agents SHALL be able to ignore such hints and still execute the skill correctly
