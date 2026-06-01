## Context

The `skills/` distributable package currently exports 5 skills: `openspec-domain`, `openspec-apply-change`, `openspec-explore`, `openspec-propose`, and `openspec-archive-change`. Users must install all 5 to get a working openspec skill suite. The domain skill already serves as the knowledge-management entry point; extending its `init` to also configure the other skills makes it the natural single bootstrapper.

The project already has `openspec install skills` as a CLI command that reads `openspec/skills/` and writes adapter files per environment. The skill's `init` flow can delegate to this command rather than reimplementing adapter logic.

## Goals / Non-Goals

**Goals:**
- Reduce the `skills/` distributable to a single skill (domain)
- Extend `init` with a skills-configuration phase that installs all skills found in `openspec/skills/`
- Make the domain skill the only artifact a user needs to copy to bootstrap a project

**Non-Goals:**
- Changing the content or behavior of the 4 removed skills themselves
- Adding a new CLI subcommand; the skill calls the existing `openspec install skills`
- Modifying how adapters are generated (that logic stays in the CLI)

## Decisions

### 1. Domain skill is the sole distributable; other skills are project-local

Rationale: The other 4 skills (`apply`, `propose`, `explore`, `archive`) are workflow skills that live in the project's `openspec/skills/` directory (per the portable-skill-format spec). They should be installed on demand from that source, not bundled separately. Only the bootstrapper needs to be pre-installed by the user.

Alternatives considered:
- Keep all 5 in the distributable: more installation friction, redundant with the portable skill format
- Make a separate "installer" skill: unnecessary indirection when the domain skill already plays this role

### 2. Init delegates to `openspec install skills` for adapter generation

Rationale: The CLI already handles per-environment adapter generation correctly and idempotently. The skill calls it rather than reimplementing file-writing logic.

Alternatives considered:
- Skill writes adapter files directly: duplicates CLI logic, breaks when adapter format changes

### 3. Skills discovery uses `openspec/skills/` as the source

Rationale: This is the canonical portable-skill location per the `portable-skill-format` spec. The init command scans it, so any skill added to that directory is automatically configured on the next `init` or re-run.

### 4. Skills-configuration phase is non-blocking when `openspec/skills/` is absent

Rationale: A project may legitimately have no portable skills yet (e.g., fresh checkout before any `migrate skills` has run). Init should complete the domain KB setup and warn, rather than fail.

## Risks / Trade-offs

- **Users with old 5-skill installs**: They will have stale adapters for the 4 removed skills unless they manually clean up. → Mitigation: document the migration in the domain skill's init output ("If you had previous skill adapters, remove them from `.claude/skills/`").
- **`openspec install skills` not available**: If the CLI is not installed or out of date, skill configuration silently skips. → Mitigation: warn the user and suggest they run `openspec install skills` manually.

## Migration Plan

1. Delete `skills/skills/openspec-apply-change/`, `skills/skills/openspec-explore/`, `skills/skills/openspec-propose/`, `skills/skills/openspec-archive-change/`
2. Update `skills/skills/openspec-domain/SKILL.md` to add the skills-configuration phase at the end of the `init` subcommand
3. Users who previously installed the 4 removed skills should remove the old adapters from their AI environment manually (e.g., `.claude/skills/`)
