## ADDED Requirements

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
