# Skills Repository

This repo contains reusable Claude Code skills. Each skill lives in its own directory with a `SKILL.md` and optional `references/` folder.

## Structure

- Each skill is a self-contained directory with a `SKILL.md` (required) and optional `references/` for supplementary docs
- Skills are general-purpose and not tied to any specific project
- Reference files are loaded on-demand to keep context lean

## Conventions

- Keep `SKILL.md` under 500 lines; use `references/` for detailed content
- Skill descriptions should be specific about when to trigger
- All code examples in reference files should come from official documentation where applicable
- Test skills before committing changes
