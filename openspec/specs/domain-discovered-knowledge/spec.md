## ADDED Requirements

### Requirement: Discovered knowledge directory and file structure
Discovered knowledge SHALL be stored as individual markdown files under `openspec/domain/discovered/`. Each file SHALL have a kebab-case filename matching its `name` frontmatter field.

#### Scenario: Discovery file is created in the correct location
- **WHEN** a discovery is captured (via archive or explicit capture)
- **THEN** a file SHALL be created at `openspec/domain/discovered/<name>.md`

### Requirement: Discovery files have required frontmatter
Every file under `openspec/domain/discovered/` SHALL have YAML frontmatter containing: `name` (kebab-case string), `type: discovery` (literal), `discovered_in` (the change name that surfaced this fact), `date` (ISO 8601 date), `relates_to` (array of static concept names), `status` (one of: `active`, `superseded`, `invalidated`).

#### Scenario: New discovery defaults to active status
- **WHEN** a discovery is captured
- **THEN** its `status` field SHALL be `active`

#### Scenario: Discovery with invalid status causes warning
- **WHEN** a discovery file has a `status` value other than `active`, `superseded`, or `invalidated`
- **THEN** the index regeneration SHALL warn the user and skip that file

### Requirement: Discovery status lifecycle
A discovery's `status` SHALL transition from `active` to `superseded` when a later change explicitly contradicts or replaces it. A discovery MAY be marked `invalidated` when it is found to have been incorrect. The `superseded_by` field (change name) SHALL be set when status becomes `superseded`.

#### Scenario: Superseded discovery appears in index with status label
- **WHEN** a discovery has `status: superseded`
- **THEN** the index entry SHALL include `[SUPERSEDED]` in its summary

#### Scenario: Archive detects contradiction with active discovery
- **WHEN** `/opsx:archive` runs and the completed change contradicts an `active` discovery
- **THEN** the skill SHALL surface the contradiction and offer to mark the discovery `superseded`

### Requirement: Archive proposes discovery candidates
During `/opsx:archive`, the skill SHALL review the completed change's implementation and compare it against existing domain knowledge. For any fact in the implementation that is absent from or contradicts the domain KB, the skill SHALL propose it as a discovery candidate for the user to confirm or dismiss.

#### Scenario: Archive proposes candidate from implementation
- **WHEN** the archived change introduces behavior not reflected in any static or discovered knowledge
- **THEN** the skill SHALL name the candidate, describe what was observed, and ask the user whether to capture it

#### Scenario: User dismisses discovery candidate
- **WHEN** the user declines a proposed discovery candidate
- **THEN** no file is created and archiving continues

#### Scenario: User confirms discovery candidate
- **WHEN** the user confirms a proposed discovery candidate
- **THEN** the skill SHALL create the discovery file and include it in the next index regeneration
