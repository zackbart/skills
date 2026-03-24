# Skills (v0.1.2)

A collection of custom Claude Code skills for development workflows.

## Global Skills Install

These 12 skills are installed globally (`~/.agents/skills/`) on this machine. Codex reads them directly; Claude Code and Cursor get symlinks.

### From this repo (3)

| Skill | Install |
|-------|---------|
| [ui-design-ethos](agent-tools/ui-design-ethos/) | `npx skills add -g zackbart/skills --skill ui-design-ethos --agent claude-code -y` |
| [optimize-prompt](agent-tools/optimize-prompt/) | `npx skills add -g zackbart/skills --skill optimize-prompt --agent claude-code -y` |
| [update-docs](agent-tools/update-docs/) | `npx skills add -g zackbart/skills --skill update-docs --agent claude-code -y` |

### From external repos (9)

| Skill | Source | Install |
|-------|--------|---------|
| agent-browser | `vercel-labs/agent-browser` | `npx skills add -g vercel-labs/agent-browser --skill agent-browser --agent claude-code -y` |
| architecture-patterns | `wshobson/agents` | `npx skills add -g wshobson/agents --skill architecture-patterns --agent claude-code -y` |
| cleenup | `zackbart/cleenup` | `npx skills add -g zackbart/cleenup --skill cleenup --agent claude-code -y` |
| code-review-excellence | `wshobson/agents` | `npx skills add -g wshobson/agents --skill code-review-excellence --agent claude-code -y` |
| defuddle | `kepano/obsidian-skills` | `npx skills add -g kepano/obsidian-skills --skill defuddle --agent claude-code -y` |
| deslop | `cursor/plugins` | `npx skills add -g cursor/plugins --skill deslop --agent claude-code -y` |
| emil-design-eng | `emilkowalski/skill` | `npx skills add -g emilkowalski/skill --skill emil-design-eng --agent claude-code -y` |
| find-skills | `vercel-labs/skills` | `npx skills add -g vercel-labs/skills --skill find-skills --agent claude-code -y` |
| sentry-fix-issues | `getsentry/sentry-agent-skills` | `npx skills add -g getsentry/sentry-agent-skills --skill sentry-fix-issues --agent claude-code -y` |
| skill-creator | `anthropics/skills` | `npx skills add -g anthropics/skills --skill skill-creator --agent claude-code -y` |
| use-railway | `railwayapp/railway-skills` | `npx skills add -g railwayapp/railway-skills --skill use-railway --agent claude-code -y` |

### Install all global skills

```bash
# From this repo
npx skills add -g zackbart/skills --skill ui-design-ethos --agent claude-code -y
npx skills add -g zackbart/skills --skill optimize-prompt --agent claude-code -y
npx skills add -g zackbart/skills --skill update-docs --agent claude-code -y

# External
npx skills add -g vercel-labs/agent-browser --skill agent-browser --agent claude-code -y
npx skills add -g wshobson/agents --skill architecture-patterns --agent claude-code -y
npx skills add -g zackbart/cleenup --skill cleenup --agent claude-code -y
npx skills add -g wshobson/agents --skill code-review-excellence --agent claude-code -y
npx skills add -g kepano/obsidian-skills --skill defuddle --agent claude-code -y
npx skills add -g cursor/plugins --skill deslop --agent claude-code -y
npx skills add -g emilkowalski/skill --skill emil-design-eng --agent claude-code -y
npx skills add -g getsentry/sentry-agent-skills --skill sentry-fix-issues --agent claude-code -y
npx skills add -g anthropics/skills --skill skill-creator --agent claude-code -y
npx skills add -g railwayapp/railway-skills --skill use-railway --agent claude-code -y
npx skills add -g vercel-labs/skills --skill find-skills --agent claude-code -y
```

## Global Plugins

These Claude Code plugins are installed globally alongside the skills.

| Plugin | Source | Install |
|--------|--------|---------|
| claude-hud | `jarrodwatts/claude-hud` | `claude plugin marketplace add jarrodwatts/claude-hud && claude plugin install claude-hud@claude-hud` |
| context7 | `claude-plugins-official` | `claude plugin install context7@claude-plugins-official` |
| impeccable | `pbakaus/impeccable` | `claude plugin marketplace add pbakaus/impeccable && claude plugin install impeccable@impeccable` |
| motif | `zackbart/motif` | `claude plugin marketplace add zackbart/motif && claude plugin install motif@motif` |
| understand-anything | `Lum1104/Understand-Anything` | `claude plugin marketplace add Lum1104/Understand-Anything && claude plugin install understand-anything@understand-anything` |

## Repo Skills (for per-project use)

These skills live in this repo and are installed per-project, not globally.

### Agent Tools

| Skill | Description |
|-------|-------------|
| [ui-design-ethos](agent-tools/ui-design-ethos/) | Principled design ethos for building SaaS/product UI at the quality level of Linear, Stripe, Notion, and Vercel |
| [optimize-prompt](agent-tools/optimize-prompt/) | Prompt engineering assistant optimized for Claude models — drafts and refines system, task, and agentic prompts |
| [update-docs](agent-tools/update-docs/) | Scans project documentation for staleness and inaccuracies, then updates it to reflect the current codebase |

### Technical

| Skill | Description |
|-------|-------------|
| [fastify](technical/fastify/) | Fastify web framework expert for building HTTP servers and APIs |
| [kysely](technical/kysely/) | Type-safe SQL query builder assistant grounded in official Kysely documentation |
| [nanomdm](technical/nanomdm/) | NanoMDM Apple MDM server assistant for check-in handlers, command queues, and APNs push |
| [node-pg-migrate](technical/node-pg-migrate/) | PostgreSQL migration expert using node-pg-migrate |
| [sparkle-mac](technical/sparkle-mac/) | Sparkle macOS auto-update framework for integrating updates, appcasts, and EdDSA signing |
| [sparkle-win](technical/sparkle-win/) | WinSparkle auto-update framework for Windows desktop apps with appcast feeds and EdDSA signing |
| [typebox](technical/typebox/) | TypeBox JSON Schema type builder assistant for TypeScript |
| [vitest](technical/vitest/) | Vitest testing framework with Jest-compatible API, coverage, mocking, and browser mode |
| [zod](technical/zod/) | Zod v4 TypeScript-first validation library with schemas, parsing, type inference, JSON Schema, and error handling |

Install specific skills from this repo into a project:

```bash
npx skills add zackbart/skills --skill <name> -y
```

Browse skills at [skills.sh/zackbart/skills](https://skills.sh/zackbart/skills).
