# Skills Repository (v0.1.2)

This repo contains reusable Claude Code skills. Each skill lives in its own directory with a `SKILL.md` and optional `references/` folder.

## Quick Reference

- `AGENTS.md` is a symlink to `CLAUDE.md` — edit only `CLAUDE.md`
- Skills are installed via `npx skills add zackbart/skills/<skill-name>`
- README.md contains the full skill table — update it when adding/removing skills

## Structure

- Each skill is a self-contained directory with a `SKILL.md` (required) and optional `references/` for supplementary docs
- Skills are general-purpose and not tied to any specific project
- Reference files are loaded on-demand to keep context lean

## SKILL.md Format

- Must have YAML frontmatter with `name`, `description`, and optionally `metadata` (version, source)
- `description` is used for skill triggering — be specific about when to activate
- `name` must match the directory name

## Conventions

- Keep `SKILL.md` under 500 lines; use `references/` for detailed content
- Skill descriptions should be specific about when to trigger
- All code examples in reference files should come from official documentation where applicable
- Test skills before committing changes
- When adding a new skill, also update `README.md` with the skill table entry and install command
