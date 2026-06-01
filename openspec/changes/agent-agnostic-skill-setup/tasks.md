## 1. Portable Skill Format

- [x] 1.1 Create `openspec/skills/` directory and document the canonical layout in `openspec/skills/README.md`
- [x] 1.2 Define and document the SKILL.md frontmatter schema (name, description, triggers, agents, args, version fields) with validation rules
- [x] 1.3 Migrate existing `.claude/commands/opsx/` skill files to `openspec/skills/<name>/SKILL.md`, adding inferred frontmatter for each
- [x] 1.4 Update `openspec/domain/static/concepts/skill.md` to reflect the new canonical location and frontmatter schema
- [x] 1.5 Add `openspec/skills/` to the project's `.gitignore` exclusion list (remove adapter dirs; keep skills dir committed)

## 2. CLI — `openspec install skills`

- [x] 2.1 Add `install skills` subcommand to the `openspec` CLI with `--env <claude|copilot|intellij|all>` flag
- [x] 2.2 Implement frontmatter parser and validator (required fields check; emit per-skill errors without halting other skills)
- [x] 2.3 Implement Claude adapter writer: reads SKILL.md body, writes `.claude/commands/opsx/<name>.md` with header comment
- [x] 2.4 Implement Copilot adapter writer: writes `.github/copilot-instructions/<name>.md` with trigger pattern and prose body
- [x] 2.5 Implement IntelliJ adapter writer: writes `.idea/runConfigurations/opsx_<name>.xml` run configuration
- [x] 2.6 Implement orphan cleanup: detect and delete adapter files whose source skill no longer exists in `openspec/skills/`
- [x] 2.7 Add idempotency tests: run install twice, assert byte-identical adapter output

## 3. CLI — `openspec migrate skills`

- [x] 3.1 Add `migrate skills` subcommand that reads `.claude/commands/opsx/*.md` and produces `openspec/skills/<name>/SKILL.md` with generated frontmatter
- [x] 3.2 Implement local-modification detection: compare file against last-generated hash; warn and prompt before overwriting
- [x] 3.3 Auto-run `openspec install skills --env all` at the end of a successful migration
- [x] 3.4 Write migration integration test: start from `.claude/commands/` only state, run migrate, assert portable skills and all adapter files exist

## 4. IntelliJ OpenSpec Plugin

- [x] 4.1 Add a `SkillLoader` component to the plugin that scans `openspec/skills/` on project open and registers each skill as an IDE action
- [x] 4.2 Display skill `name` and `description` in the IDE action list / Run menu
- [x] 4.3 Trigger skill execution by forwarding the SKILL.md content to the active AI assistant session
- [x] 4.4 Add a project listener so the plugin reloads skills when `openspec/skills/` changes on disk

## 5. Specs & Domain Sync

- [x] 5.1 Run `openspec status --change agent-agnostic-skill-setup` and verify all spec files are detected as `done`
- [x] 5.2 Update `openspec/specs/domain-skill/spec.md` to reflect that the domain skill canonical location is now `openspec/skills/domain/SKILL.md`
- [ ] 5.3 Run `/opsx:archive` to promote specs and sync the domain index once all implementation tasks are complete

## 6. Documentation & Validation

- [x] 6.1 Add a "Getting Started" section to the OpenSpec README covering `openspec install skills` for first-time setup
- [x] 6.2 Document the three supported environments (Claude, Copilot, IntelliJ) with step-by-step install instructions
- [x] 6.3 Add a CHANGELOG entry noting the breaking change: `.claude/commands/opsx/` is now generated output
- [x] 6.4 Run `openspec install skills --env all` on the repo itself and commit the regenerated adapter files as the new baseline
