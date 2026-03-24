# Skills (v0.1.2)

A collection of custom Claude Code skills for development workflows.

## Agent Tools

| Skill | Description |
|-------|-------------|
| [ui-design-ethos](agent-tools/ui-design-ethos/) | Principled design ethos for building SaaS/product UI at the quality level of Linear, Stripe, Notion, and Vercel |
| [optimize-prompt](agent-tools/optimize-prompt/) | Prompt engineering assistant optimized for Claude models — drafts and refines system, task, and agentic prompts |
| [update-docs](agent-tools/update-docs/) | Scans project documentation for staleness and inaccuracies, then updates it to reflect the current codebase |

## Technical

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

## Recommended External Skills

| Skill | Description |
|-------|-------------|
| [anthropics/skill-creator](https://github.com/anthropics/skills/tree/main/skills/skill-creator) | Create, modify, evaluate, and optimize skills with benchmarking and triggering accuracy tests |
| [cursor/deslop](https://github.com/cursor/plugins/tree/main/cursor-team-kit/skills/deslop) | Removes AI-generated code slop — unnecessary comments, defensive checks, `any` casts, and deeply nested code inconsistent with codebase style |
| [emilkowalski/emil-design-eng](https://github.com/emilkowalski/skill/tree/main/skills/emil-design-eng) | Animation philosophy, spring physics, gesture mechanics, and micro-interaction craft from Emil Kowalski |
| [getsentry/sentry-fix-issues](https://github.com/getsentry/sentry-agent-skills/tree/main/skills/sentry-fix-issues) | Find and fix Sentry issues using MCP — analyzes stack traces, breadcrumbs, and context to identify root causes |
| [kepano/defuddle](https://github.com/kepano/obsidian-skills/tree/main/skills/defuddle) | Extracts clean markdown from web pages using Defuddle CLI, stripping clutter like navigation and ads to save tokens |
| [railwayapp/railway-skills](https://github.com/railwayapp/railway-skills) | Operate Railway infrastructure — create projects, provision services/databases, manage storage, deploy code, and troubleshoot failures |
| [vercel-labs/agent-browser](https://github.com/vercel-labs/agent-browser/tree/main/skills/agent-browser) | Browser automation CLI for AI agents — navigation, form filling, clicking, screenshots, data extraction, and web app testing |
| [wshobson/architecture-patterns](https://github.com/wshobson/agents/tree/main/plugins/backend-development/skills/architecture-patterns) | Implement Clean Architecture, Hexagonal Architecture, and Domain-Driven Design for maintainable backend systems |
| [wshobson/code-review-excellence](https://github.com/wshobson/agents/tree/main/plugins/developer-essentials/skills/code-review-excellence) | Systematic code review practices for constructive feedback, catching bugs early, and fostering knowledge sharing |
| [zackbart/cleenup](https://github.com/zackbart/cleenup) | Scan and redact leaked secrets from Claude Code and Codex CLI session logs |

## Recommended Plugins

| Plugin | Source | Description |
|--------|--------|-------------|
| [claude-hud](https://github.com/jarrodwatts/claude-hud) | `jarrodwatts/claude-hud` | Configurable status line showing model, context, git, usage, and activity |
| [context7](https://github.com/anthropics/claude-plugins-official) | `claude-plugins-official` | Retrieve up-to-date documentation and code examples for any library via MCP |
| [impeccable](https://github.com/pbakaus/impeccable) | `pbakaus/impeccable` | 20+ design skills for UI polish — animations, typography, color, layout, accessibility, and more |
| [motif](https://github.com/zackbart/motif) | `zackbart/motif` | 5-stage dev workflow orchestrator (Research, Plan, Scaffold, Build, Validate) with critic and validator agents |
| [understand-anything](https://github.com/Lum1104/Understand-Anything) | `Lum1104/Understand-Anything` | Codebase analysis via interactive knowledge graphs — onboarding, architecture, and diff analysis |

## Installation

Install all skills from this repo:

```bash
npx skills add zackbart/skills --all -g -y
```

Install all recommended external skills:

```bash
npx skills add -g cursor/plugins --skill deslop --agent claude-code -y
npx skills add -g kepano/obsidian-skills --skill defuddle --agent claude-code -y
npx skills add -g vercel-labs/agent-browser --skill agent-browser --agent claude-code -y
npx skills add -g getsentry/sentry-agent-skills --skill sentry-fix-issues --agent claude-code -y
npx skills add -g railwayapp/railway-skills --skill use-railway --agent claude-code -y
npx skills add -g zackbart/cleenup --skill cleenup --agent claude-code -y
npx skills add -g anthropics/skills --skill skill-creator --agent claude-code -y
npx skills add -g emilkowalski/skill --skill emil-design-eng --agent claude-code -y
npx skills add -g wshobson/agents --skill code-review-excellence --agent claude-code -y
npx skills add -g wshobson/agents --skill architecture-patterns --agent claude-code -y
```

Install recommended plugins:

```bash
claude plugin marketplace add jarrodwatts/claude-hud && claude plugin install claude-hud@claude-hud
claude plugin install context7@claude-plugins-official
claude plugin marketplace add pbakaus/impeccable && claude plugin install impeccable@impeccable
claude plugin marketplace add zackbart/motif && claude plugin install motif@motif
claude plugin marketplace add Lum1104/Understand-Anything && claude plugin install understand-anything@understand-anything
```

Browse skills at [skills.sh/zackbart/skills](https://skills.sh/zackbart/skills).
