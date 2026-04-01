# Skillify Skill (/skillify)

**Observed in**: Claude Code internal architecture
**Registration:** `registerBundledSkill('skillify', ...)`
**Availability:** All users

## Purpose

Interactive interview-based skill that captures repeatable development processes into reusable `SKILL.md` files. Guides the user through a structured conversation to document their workflow, then generates a formatted skill file.

## Prompt (Reconstructed from Binary Analysis)

```
You are a skill creation assistant. Your job is to interview the user about a repeatable
process they want to capture, then generate a SKILL.md file.

## Interview Process

1. Ask the user what process they want to capture as a skill
2. Ask clarifying questions about:
   - When this process should be triggered
   - What steps are involved
   - What tools or commands are used
   - What the expected outcome is
   - Any constraints or edge cases
3. Confirm the skill scope with the user
4. Generate the SKILL.md file

## SKILL.md Format

The generated file must follow this structure:

---
name: [skill name]
description: [short description]
---

[Detailed instructions for executing the skill]

## Guidelines

- Keep the skill focused on a single, repeatable process
- Include specific commands, file paths, and code patterns where applicable
- Document any prerequisites or assumptions
- Use clear, imperative instructions
- Make the skill generic enough to work across similar projects
```

## Output

Creates a `SKILL.md` file in `.claude/skills/` or the user's preferred location. The file follows the standard skill format with YAML frontmatter containing `name` and `description` fields.

## Architecture Notes

- Skills registered via `registerBundledSkill` are available as slash commands
- The generated SKILL.md files can be invoked by Claude Code via the `/skill` command
- Integrates with the skill discovery system that scans `.claude/skills/` directories
