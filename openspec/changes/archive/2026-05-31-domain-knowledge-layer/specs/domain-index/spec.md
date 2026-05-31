## ADDED Requirements

### Requirement: Index file location and format
The system SHALL maintain a domain index at `openspec/domain/_index.yaml` using structured YAML format. The index SHALL NOT be hand-edited by users.

#### Scenario: Index exists when domain directory is present
- **WHEN** `openspec/domain/` exists
- **THEN** `openspec/domain/_index.yaml` SHALL exist and be valid YAML

#### Scenario: Index structure covers all entry types
- **WHEN** the index is read
- **THEN** it SHALL contain top-level keys: `concepts`, `processes`, `rules`, `discoveries`
- **THEN** each entry SHALL include at minimum: `def` (one-line definition) and `file` (relative path to source file)

### Requirement: Index is auto-generated from frontmatter
The system SHALL generate `_index.yaml` by reading the YAML frontmatter of all files under `openspec/domain/static/` and `openspec/domain/discovered/`. The index SHALL reflect the current state of all frontmatter at generation time.

#### Scenario: New concept file is reflected in index after regeneration
- **WHEN** a new concept file is added to `openspec/domain/static/concepts/`
- **AND** the index is regenerated
- **THEN** the new concept SHALL appear in `_index.yaml` under the `concepts` key

#### Scenario: Deleted concept file is removed from index after regeneration
- **WHEN** a concept file is deleted from `openspec/domain/static/`
- **AND** the index is regenerated
- **THEN** the deleted concept SHALL NOT appear in `_index.yaml`

### Requirement: Index regeneration triggers
The index SHALL be regenerated in two situations: at the end of every `/opsx:archive` run, and when `/opsx:domain sync` is explicitly called.

#### Scenario: Archive triggers index regeneration
- **WHEN** `/opsx:archive` completes a change
- **THEN** `_index.yaml` SHALL be regenerated before the change is marked archived

#### Scenario: Manual sync regenerates index
- **WHEN** the user runs `/opsx:domain sync`
- **THEN** `_index.yaml` SHALL be regenerated from all current frontmatter

### Requirement: Skills load the index when domain exists
Any OpenSpec skill SHALL load `_index.yaml` at the start of its execution if `openspec/domain/` exists. Skills SHALL NOT load the index if `openspec/domain/` does not exist.

#### Scenario: Index loaded when domain directory present
- **WHEN** a skill starts and `openspec/domain/_index.yaml` exists
- **THEN** the skill SHALL read and use `_index.yaml` as context

#### Scenario: No domain overhead when domain directory absent
- **WHEN** a skill starts and `openspec/domain/` does not exist
- **THEN** the skill SHALL NOT attempt to load any domain files
