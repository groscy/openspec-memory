## ADDED Requirements

### Requirement: Skills check for domain directory on entry
Every OpenSpec skill (`explore`, `propose`, `apply`, `archive`) SHALL check for the existence of `openspec/domain/` at the start of execution. If it exists, the skill SHALL load `_index.yaml`. If it does not exist, the skill SHALL proceed without domain context.

#### Scenario: Skill loads index when domain present
- **WHEN** any OpenSpec skill starts and `openspec/domain/_index.yaml` exists
- **THEN** the skill SHALL read `_index.yaml` into its active context before proceeding

#### Scenario: Skill proceeds normally when domain absent
- **WHEN** any OpenSpec skill starts and `openspec/domain/` does not exist
- **THEN** the skill SHALL proceed without loading any domain files and without error

### Requirement: Concept files loaded by name from change scope
Skills SHALL load individual concept files only when the concept name appears in the current change description, task description, or conversation context. Skills SHALL NOT load all concept files by default.

#### Scenario: Relevant concept file loaded during propose
- **WHEN** `/opsx:propose` is creating a proposal about "invoice approval"
- **AND** the index contains an `Invoice` concept
- **THEN** the skill SHALL load `openspec/domain/static/concepts/invoice.md`

#### Scenario: Unrelated concept files not loaded
- **WHEN** `/opsx:propose` is working on a change unrelated to any indexed concept
- **THEN** the skill SHALL load only the index and no individual concept files

### Requirement: Frontmatter read before prose body
When a skill loads a concept file, it SHALL prefer reading only the frontmatter for proposal and explore phases. The prose body SHALL be read only during the apply (implementation) phase when depth is needed.

#### Scenario: Propose reads frontmatter only
- **WHEN** `/opsx:propose` loads a concept file for context
- **THEN** it SHALL use the frontmatter fields (name, aliases, related, states, rules) as context
- **THEN** it SHALL NOT require the prose body to complete the proposal

#### Scenario: Apply reads full file body
- **WHEN** `/opsx:apply` loads a concept file for an implementation task
- **THEN** it SHALL read the full file including the prose body for complete context

### Requirement: Skills use domain terminology in output
When domain knowledge is loaded, skills SHALL use the canonical concept names, aliases, and terminology from the domain KB in their output — proposals, spec names, task names, and implementation identifiers.

#### Scenario: Proposal uses canonical concept name
- **WHEN** `/opsx:propose` has loaded an `Invoice` concept from the domain KB
- **THEN** the proposal SHALL refer to the concept as "Invoice" (or its defined aliases) consistently

### Requirement: Skills flag domain conflicts
When a proposal or spec contradicts a known domain rule or concept definition, the skill SHALL surface the conflict to the user before finalizing the artifact.

#### Scenario: Proposal contradicts a known rule
- **WHEN** `/opsx:propose` generates a proposal that would violate a rule in `domain/static/rules.md`
- **THEN** the skill SHALL note the conflict and ask the user whether to proceed or revise
