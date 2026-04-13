# Skills (v0.1.3)

A collection of custom Claude Code skills for development workflows, plus a record of globally installed skills and plugins.

## Repo Skills

Skills in this repo are installable per-project or globally.

### Agent Tools

| Skill | Description |
|-------|-------------|
| [bird](agent-tools/bird/) | Bird CLI assistant for Twitter/X - tweeting, replying, reading, searching, and managing timelines from the terminal |
| [ethos](agent-tools/ethos/) | Generate and maintain project vision, principles, personas, and non-goals - the 50k-foot "why behind the what" for agents and humans |
| [optimize-prompt](agent-tools/optimize-prompt/) | Prompt engineering assistant optimized for Claude models - drafts and refines system, task, and agentic prompts |
| [ui](agent-tools/ui/) | Practical UI design system with concrete values for hierarchy, spacing, typography, color, shadows, and polish -- makes Claude Code produce professionally designed interfaces |
| [update-docs](agent-tools/update-docs/) | Scans project documentation for staleness and inaccuracies, then updates it to reflect the current codebase |

### Technical

| Skill | Description |
|-------|-------------|
| [android-device-owner](technical/android-device-owner/) | Android Device Owner Mode (DPC) assistant for DevicePolicyManager, app management, web filtering, kiosk mode, and user restrictions |
| [fastify](technical/fastify/) | Fastify web framework expert for building HTTP servers and APIs |
| [ios-mdm](technical/ios-mdm/) | Apple iOS/iPadOS MDM protocol expert — commands, configuration profiles, payloads, DDM, and supervision |
| [nanomdm](technical/nanomdm/) | NanoMDM Apple MDM server assistant for check-in handlers, command queues, and APNs push |
| [node-pg-migrate](technical/node-pg-migrate/) | PostgreSQL migration expert using node-pg-migrate |
| [sparkle-mac](technical/sparkle-mac/) | Sparkle macOS auto-update framework for integrating updates, appcasts, and EdDSA signing |
| [sparkle-win](technical/sparkle-win/) | WinSparkle auto-update framework for Windows desktop apps with appcast feeds and EdDSA signing |
| [typebox](technical/typebox/) | TypeBox JSON Schema type builder assistant for TypeScript |

Install a skill from this repo into a project:

```bash
npx skills add zackbart/skills --skill <name> -y
```

Browse skills at [skills.sh/zackbart/skills](https://skills.sh/zackbart/skills).

## Global Skills

These 16 skills are installed globally (`~/.agents/skills/`) on this machine. Codex reads them directly; Claude Code and Cursor get symlinks.

### From this repo (4)

| Skill | Install |
|-------|---------|
| [bird](agent-tools/bird/) | `npx skills add -g zackbart/skills --skill bird --agent claude-code -y` |
| [ethos](agent-tools/ethos/) | `npx skills add -g zackbart/skills --skill ethos --agent claude-code -y` |
| [optimize-prompt](agent-tools/optimize-prompt/) | `npx skills add -g zackbart/skills --skill optimize-prompt --agent claude-code -y` |
| [ui](agent-tools/ui/) | `npx skills add -g zackbart/skills --skill ui --agent claude-code -y` |

### From external repos (12)

| Skill | Source | Install |
|-------|--------|---------|
| agent-browser | `vercel-labs/agent-browser` | `npx skills add -g vercel-labs/agent-browser --skill agent-browser --agent claude-code -y` |
| claude-md-improver | `anthropics/claude-plugins-official` | `npx skills add -g anthropics/claude-plugins-official --skill claude-md-improver --agent claude-code -y` |
| cleenup | `zackbart/cleenup` | `npx skills add -g zackbart/cleenup --skill cleenup --agent claude-code -y` |
| defuddle | `kepano/obsidian-skills` | `npx skills add -g kepano/obsidian-skills --skill defuddle --agent claude-code -y` |
| deslop | `cursor/plugins` | `npx skills add -g cursor/plugins --skill deslop --agent claude-code -y` |
| emil-design-eng | `emilkowalski/skill` | `npx skills add -g emilkowalski/skill --skill emil-design-eng --agent claude-code -y` |
| find-skills | `vercel-labs/skills` | `npx skills add -g vercel-labs/skills --skill find-skills --agent claude-code -y` |
| frontend-design | `anthropics/claude-plugins-official` | `npx skills add -g anthropics/claude-plugins-official --skill frontend-design --agent claude-code -y` |
| playwriter | `remorses/playwriter` | `npx skills add -g remorses/playwriter --skill playwriter --agent claude-code -y` |
| sentry-fix-issues | `getsentry/sentry-agent-skills` | `npx skills add -g getsentry/sentry-agent-skills --skill sentry-fix-issues --agent claude-code -y` |
| skill-creator | `anthropics/skills` | `npx skills add -g anthropics/skills --skill skill-creator --agent claude-code -y` |
| use-railway | `railwayapp/railway-skills` | `npx skills add -g railwayapp/railway-skills --skill use-railway --agent claude-code -y` |

### Install all global skills

```bash
# From this repo
npx skills add -g zackbart/skills --skill bird --agent claude-code -y
npx skills add -g zackbart/skills --skill ethos --agent claude-code -y
npx skills add -g zackbart/skills --skill optimize-prompt --agent claude-code -y
npx skills add -g zackbart/skills --skill ui --agent claude-code -y

# External
npx skills add -g vercel-labs/agent-browser --skill agent-browser --agent claude-code -y
npx skills add -g anthropics/claude-plugins-official --skill claude-md-improver --agent claude-code -y
npx skills add -g zackbart/cleenup --skill cleenup --agent claude-code -y
npx skills add -g kepano/obsidian-skills --skill defuddle --agent claude-code -y
npx skills add -g cursor/plugins --skill deslop --agent claude-code -y
npx skills add -g emilkowalski/skill --skill emil-design-eng --agent claude-code -y
npx skills add -g vercel-labs/skills --skill find-skills --agent claude-code -y
npx skills add -g anthropics/claude-plugins-official --skill frontend-design --agent claude-code -y
npx skills add -g remorses/playwriter --skill playwriter --agent claude-code -y
npx skills add -g getsentry/sentry-agent-skills --skill sentry-fix-issues --agent claude-code -y
npx skills add -g anthropics/skills --skill skill-creator --agent claude-code -y
npx skills add -g railwayapp/railway-skills --skill use-railway --agent claude-code -y
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
