# Skills

A collection of custom Claude Code skills for development workflows.

## Skills

| Skill | Description |
|-------|-------------|
| [design-system-patterns](design-system-patterns/) | Principled design system for building SaaS/product UI at the quality level of Linear, Stripe, Notion, and Vercel |
| [kysely](kysely/) | Type-safe SQL query builder assistant grounded in official Kysely documentation |
| [optimize-prompt](optimize-prompt/) | Prompt engineering assistant optimized for Claude models — drafts and refines system, task, and agentic prompts |
| [second-opinion](second-opinion/) | Spawns a skeptical critic subagent to pressure-test plans and proposals before execution |
| [update-docs](update-docs/) | Scans project documentation for staleness and inaccuracies, then updates it to reflect the current codebase |

## Installation

Install all skills:

```bash
npx skills add zackbart/skills
```

Or install a single skill:

```bash
npx skills add zackbart/skills/kysely
npx skills add zackbart/skills/design-system-patterns
npx skills add zackbart/skills/optimize-prompt
npx skills add zackbart/skills/second-opinion
npx skills add zackbart/skills/update-docs
```

Browse skills at [skills.sh/zackbart/skills](https://skills.sh/zackbart/skills). Requires [Claude Code](https://claude.com/claude-code).
