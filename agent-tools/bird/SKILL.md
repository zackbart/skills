---
name: bird
description: >
  Bird CLI assistant for interacting with Twitter/X from the terminal. Use this skill whenever the
  user wants to post tweets, reply to tweets, read tweets or threads, search Twitter, check mentions,
  browse their home timeline, manage bookmarks or likes, follow/unfollow users, view followers or
  following lists, look up user profiles, get trending topics, or do anything involving Twitter/X
  from the command line. Also trigger when the user mentions "bird", "bird cli", "bunx bird",
  "npx bird", tweets, Twitter, X posts, timeline, or wants to automate any Twitter/X workflow.
  This skill covers bird v0.8.0 run via `bunx bird` or `npx bird`.
---

# Bird CLI (v0.8.0)

You are an expert at using the bird CLI to interact with Twitter/X. Bird talks to X's internal GraphQL endpoints using browser session cookies — no API keys needed.

## Setup

Bird is run via `bunx bird` (preferred) or `npx bird` as a fallback. Use whichever package runner is available — try `bunx` first, then `npx`. It authenticates by reading cookies from your browser. The user's Chrome "Default" profile is already configured and working.

Every command needs auth. If a command fails with credential errors, try adding `--chrome-profile "Default"` explicitly, or run `bunx bird check` (or `npx bird check`) to diagnose.

## Core Principles

1. **Always use `--plain` when piping output or parsing results** — it disables emoji and color codes, giving stable machine-readable output.
2. **Use `--json` for structured data** — most read commands support it. Prefer JSON when you need to extract specific fields or chain commands.
3. **Respect rate limits** — don't fire off dozens of requests in tight loops. Use `--delay` on paginated commands and keep `--max-pages` reasonable.
4. **Confirm before posting** — tweeting, replying, following, and unfollowing are visible actions. Always confirm with the user before executing write operations.
5. **Media attachments** — up to 4 images or 1 video via `--media <path>` (repeatable). Add `--alt <text>` for accessibility.

## How to Use This Skill

The SKILL.md covers the most common workflows. For detailed per-command options and pagination patterns, load the relevant reference file:

- `references/commands.md` — Full command reference with all options, pagination, and JSON output details

## Quick Reference

### Identity & Auth

```bash
# Check who you're logged in as
npx bird whoami

# Verify credentials are available
npx bird check

# Look up a user's profile info
npx bird about @username --json
```

### Posting

```bash
# Post a tweet
npx bird tweet "hello world"

# Tweet with media
npx bird tweet "check this out" --media /path/to/image.png --alt "description"

# Tweet with multiple images (up to 4)
npx bird tweet "gallery" --media img1.png --media img2.png --alt "first" --alt "second"

# Reply to a tweet
npx bird reply <tweet-id-or-url> "nice post"

# Reply with media
npx bird reply <tweet-id-or-url> "here's a screenshot" --media /path/to/screenshot.png
```

### Reading

```bash
# Read a single tweet
npx bird read <tweet-id-or-url>

# Read as JSON (for parsing)
npx bird read <tweet-id-or-url> --json

# Shorthand — just pass the ID/URL directly
npx bird <tweet-id-or-url>

# View a full conversation thread
npx bird thread <tweet-id-or-url>

# List replies to a tweet
npx bird replies <tweet-id-or-url>
```

### Timelines & Feeds

```bash
# Home timeline ("For You")
npx bird home

# Chronological "Following" feed
npx bird home --following

# A user's tweets
npx bird user-tweets @handle

# Your mentions
npx bird mentions

# Someone else's mentions
npx bird mentions -u @handle
```

### Search

```bash
# Basic search
npx bird search "claude code"

# Search with count
npx bird search "from:anthropic" -n 20

# Search as JSON for processing
npx bird search "AI agents" --json

# Paginate through all results
npx bird search "topic" --all --max-pages 3
```

### Social

```bash
# Follow / unfollow
npx bird follow @username
npx bird unfollow @username

# List who you follow
npx bird following --json

# List your followers
npx bird followers --json

# Check another user's following/followers (needs user ID, not handle)
npx bird following --user <userId>
npx bird followers --user <userId>
```

### Bookmarks & Likes

```bash
# View bookmarks
npx bird bookmarks

# View likes
npx bird likes

# Remove a bookmark
npx bird unbookmark <tweet-id-or-url>
```

### Lists

```bash
# View your lists
npx bird lists --json

# Get tweets from a list
npx bird list-timeline <list-id-or-url>
```

### Trending & News

```bash
# AI-curated news and trending
npx bird news

# Specific tabs
npx bird news --trending-only
npx bird news --news-only
npx bird news --for-you

# Include related tweets
npx bird news --with-tweets
```

## Common Patterns

### Get tweet data for processing

```bash
# Read a tweet as JSON and extract text
npx bird read <url> --json --plain
```

### Paginate through large result sets

Most list commands support `--all` (fetch everything) or `--max-pages N` (cap pages). Use `--cursor` to resume from where you left off. The `--delay` flag (default 1000ms) controls time between page fetches.

```bash
# Get all bookmarks
npx bird bookmarks --all --json

# Get 3 pages of search results
npx bird search "query" --all --max-pages 3 --json

# Resume pagination
npx bird search "query" --cursor "DAABCg..."
```

### Thread expansion for bookmarks

Bookmarks have special thread-expansion flags for pulling context around bookmarked tweets:

```bash
# Include full author reply chains
npx bird bookmarks --author-chain --json

# Include parent tweet for non-root bookmarks
npx bird bookmarks --include-parent

# Full reply chain connected to bookmark
npx bird bookmarks --full-chain-only
```

## Configuration

Bird reads config from `~/.config/bird/config.json5` (global) and `./.birdrc.json5` (project-local). Both are JSON5 format.

```json5
{
  chromeProfile: "Default",
  timeoutMs: 30000,
  cookieTimeoutMs: 5000,
  quoteDepth: 1,
}
```

Environment variables: `NO_COLOR`, `BIRD_TIMEOUT_MS`, `BIRD_COOKIE_TIMEOUT_MS`, `BIRD_QUOTE_DEPTH`.

## Key Warnings

- **`--all` on list-timeline** can get your account banned per the CLI's own warning — use `--max-pages` instead
- **Tweet IDs are strings** — they're too large for JavaScript number precision. When working with JSON output, treat them as strings
- **Query IDs rotate** — bird auto-refreshes and caches GraphQL query IDs. If you hit weird errors, run `npx bird query-ids --fresh` to force a refresh
- **Cookie extraction requires keychain access** — the user may need to enter their macOS password when bird first reads Chrome cookies
