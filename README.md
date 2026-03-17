# Skills

A collection of custom Claude Code skills for development workflows.

## Agent Tools

| Skill | Description |
|-------|-------------|
| [design-system-patterns](agent-tools/design-system-patterns/) | Principled design system for building SaaS/product UI at the quality level of Linear, Stripe, Notion, and Vercel |
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
| [cursor/deslop](https://github.com/cursor/plugins/tree/main/cursor-team-kit/skills/deslop) | Removes AI-generated code slop — unnecessary comments, defensive checks, `any` casts, and deeply nested code inconsistent with codebase style |
| [emilkowalski/skill](https://github.com/emilkowalski/skill/tree/main/skills/emil-design-eng) | Design and engineering principles for building better user interfaces |
| [kepano/defuddle](https://github.com/kepano/obsidian-skills/tree/main/skills/defuddle) | Extracts clean markdown from web pages using Defuddle CLI, stripping clutter like navigation and ads to save tokens |
| [pbakaus/impeccable](https://github.com/pbakaus/impeccable) | Design guidance and commands to help AI assistants generate better frontend interfaces by steering away from common design mistakes |
| [getsentry/sentry-fix-issues](https://github.com/getsentry/sentry-for-ai/tree/main/skills/sentry-fix-issues) | Find and fix Sentry issues using MCP — analyzes stack traces, breadcrumbs, and context to identify root causes |
| [railwayapp/railway-skills](https://github.com/railwayapp/railway-skills) | Operate Railway infrastructure — create projects, provision services/databases, manage storage, deploy code, and troubleshoot failures |
| [vercel-labs/agent-browser](https://github.com/vercel-labs/agent-browser/tree/main/skills/agent-browser) | Browser automation CLI for AI agents — navigation, form filling, clicking, screenshots, data extraction, and web app testing |
| [wshobson/architecture-patterns](https://github.com/wshobson/agents/tree/main/plugins/backend-development/skills/architecture-patterns) | Implement Clean Architecture, Hexagonal Architecture, and Domain-Driven Design for maintainable backend systems |
| [wshobson/code-review-excellence](https://github.com/wshobson/agents/tree/main/plugins/developer-essentials/skills/code-review-excellence) | Systematic code review practices for constructive feedback, catching bugs early, and fostering knowledge sharing |

## Installation

Install all skills:

```bash
npx skills add zackbart/skills
```

Browse skills at [skills.sh/zackbart/skills](https://skills.sh/zackbart/skills).
