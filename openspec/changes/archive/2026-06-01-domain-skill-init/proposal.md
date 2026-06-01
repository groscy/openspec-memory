## Why

The exported `skills/` package currently ships 5 separate skills, requiring users to install each one individually. Reducing the package to a single domain skill that auto-configures the rest on `init` lowers the installation surface and makes project onboarding a one-step process.

## What Changes

- Remove `openspec-apply-change`, `openspec-explore`, `openspec-propose`, and `openspec-archive-change` from the `skills/` distributable package
- Extend the domain skill's `init` subcommand to discover and install all skills found in the project's `openspec/skills/` directory after domain KB setup
- The domain skill becomes the single distributable entry point: install it once, run `init`, and the full skill suite is configured for the current environment

## Capabilities

### New Capabilities
- `domain-init-skill-configure`: The `init` subcommand gains a skills-configuration phase — after writing domain KB files it discovers all `SKILL.md` files under `openspec/skills/` and installs adapter files for the current AI environment (equivalent to `openspec install skills`)

### Modified Capabilities
- `domain-skill`: The `init` subcommand is extended with a post-KB-setup phase that configures all project skills

## Impact

- The `skills/` distributable shrinks from 5 skills to 1 (domain skill only)
- The `domain-skill` spec gains new behavior for its `init` subcommand
- Users who previously installed all 5 skills individually can now install only the domain skill and let `init` configure the rest
