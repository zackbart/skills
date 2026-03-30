# My Claude Code Environment

Everything installed globally on this machine as of 2026-03-30.

## Global Skills (15)

Skills live in `~/.agents/skills/` with symlinks in `~/.claude/skills/`.

### From `zackbart/skills` (3)

| Skill | Description |
|-------|-------------|
| bird | Bird CLI assistant for Twitter/X - tweeting, replying, searching from terminal |
| ethos | Generate project vision, principles, personas, and non-goals |
| optimize-prompt | Prompt engineering assistant optimized for Claude models |

### From external repos (12)

| Skill | Source |
|-------|--------|
| agent-browser | `vercel-labs/agent-browser` |
| claude-md-improver | `anthropics/claude-plugins-official` |
| cleenup | `zackbart/cleenup` |
| defuddle | `kepano/obsidian-skills` |
| deslop | `cursor/plugins` |
| emil-design-eng | `emilkowalski/skill` |
| find-skills | `vercel-labs/skills` |
| frontend-design | `anthropics/claude-plugins-official` |
| playwriter | `remorses/playwriter` |
| sentry-fix-issues | `getsentry/sentry-agent-skills` |
| skill-creator | `anthropics/skills` |
| use-railway | `railwayapp/railway-skills` |

## Install Everything

```bash
# From zackbart/skills
npx skills add -g zackbart/skills --skill bird --agent claude-code -y
npx skills add -g zackbart/skills --skill ethos --agent claude-code -y
npx skills add -g zackbart/skills --skill optimize-prompt --agent claude-code -y

# External repos
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
