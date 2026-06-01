## 1. Clean up skills distributable

- [x] 1.1 Delete `skills/skills/openspec-apply-change/` directory and its contents
- [x] 1.2 Delete `skills/skills/openspec-explore/` directory and its contents
- [x] 1.3 Delete `skills/skills/openspec-propose/` directory and its contents
- [x] 1.4 Delete `skills/skills/openspec-archive-change/` directory and its contents

## 2. Update domain skill init subcommand

- [x] 2.1 Add a skills-configuration phase at the end of the `init` subcommand in `skills/skills/openspec-domain/SKILL.md`
- [x] 2.2 After domain KB files are confirmed and written, run `openspec install skills` to configure all skills in `openspec/skills/`
- [x] 2.3 Add handling for when `openspec/skills/` is absent: warn and skip, do not fail
- [x] 2.4 Report installed skill adapters and target environments in the init completion message
