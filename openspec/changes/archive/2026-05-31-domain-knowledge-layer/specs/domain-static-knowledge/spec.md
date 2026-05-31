## ADDED Requirements

### Requirement: Static knowledge directory structure
Static domain knowledge SHALL be stored under `openspec/domain/static/` with the following layout:
- `concepts/` — domain entities and objects (one file per concept)
- `processes/` — workflows and multi-step flows (one file per process)
- `glossary.md` — terminology definitions
- `rules.md` — business constraints and invariants

#### Scenario: Directory structure is created by domain init
- **WHEN** the user runs `/opsx:domain init`
- **THEN** `openspec/domain/static/concepts/`, `openspec/domain/static/processes/`, `openspec/domain/static/glossary.md`, and `openspec/domain/static/rules.md` SHALL be created

### Requirement: Concept and process files have required frontmatter
Every file under `openspec/domain/static/concepts/` and `openspec/domain/static/processes/` SHALL have YAML frontmatter containing at minimum: `name` (string), `type` (one of: `concept`, `process`, `rule`, `actor`), `aliases` (array, may be empty), and `related` (array of other concept names, may be empty).

#### Scenario: Valid concept file passes frontmatter check
- **WHEN** a concept file is read during index regeneration
- **THEN** it SHALL parse without error and contain all required frontmatter fields

#### Scenario: Concept file with missing required field causes warning
- **WHEN** a concept file is missing a required frontmatter field during index regeneration
- **THEN** the skill SHALL warn the user and skip that file rather than failing silently

### Requirement: Concept files have a prose body
Every concept and process file SHALL contain a prose body after the frontmatter with a human-readable definition. The prose body MAY include: edge cases, examples, known states, and links to related concepts.

#### Scenario: Prose body present in scaffolded file
- **WHEN** `/opsx:domain init` or `/opsx:domain add` creates a concept file
- **THEN** the file SHALL include a non-empty prose body below the frontmatter

### Requirement: Concept frontmatter optional fields
Concept files MAY include these additional frontmatter fields: `states` (array of valid state names), `rules` (array of rule IDs from `rules.md`), `actors` (array of actor names relevant to this concept).

#### Scenario: Optional fields appear in index when present
- **WHEN** a concept file includes `states` in its frontmatter
- **AND** the index is regenerated
- **THEN** the `states` array SHALL appear in the index entry for that concept

### Requirement: Glossary and rules files are flat markdown
`glossary.md` and `rules.md` SHALL be plain markdown files without frontmatter. `glossary.md` SHALL use a `### Term` heading per entry. `rules.md` SHALL use a `### Rule: <id>` heading per entry with a one-line description.

#### Scenario: Glossary terms appear in index
- **WHEN** `glossary.md` exists and contains `### Term` entries
- **AND** the index is regenerated
- **THEN** each term SHALL appear in the index under a `glossary` key with its one-line definition
