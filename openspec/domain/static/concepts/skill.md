---
name: Skill
type: concept
aliases: [command, slash-command]
related: [Schema, Change]
---
A Skill is a markdown instruction file that Claude loads and executes to implement an OpenSpec workflow command. Skills are invoked via slash commands (e.g., `/opsx:propose`, `/opsx:archive`) or by name through the Skill tool.

Skills live in two parallel locations: `.claude/commands/opsx/<name>.md` (local project commands, invokable as `/opsx:<name>`) and `.claude/skills/<skill-name>/SKILL.md` (marketplace-style skills, invokable via the Skill tool by name). The content is equivalent; the two locations serve different invocation paths.

A skill file contains prose instructions that Claude follows step by step. Skills have access to all Claude tools (Bash, Read, Write, Edit, AskUserQuestion, etc.) and may call the `openspec` CLI for status and instructions.
