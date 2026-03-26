# Bird CLI Command Reference (v0.8.0)

Complete reference for all bird commands, options, and output modes.

## Table of Contents

- [Global Options](#global-options)
- [Posting Commands](#posting-commands) — tweet, reply
- [Reading Commands](#reading-commands) — read, thread, replies
- [Timeline Commands](#timeline-commands) — home, mentions, user-tweets
- [Search](#search)
- [Social Commands](#social-commands) — follow, unfollow, following, followers
- [Bookmarks & Likes](#bookmarks--likes) — bookmarks, unbookmark, likes
- [Lists](#lists) — lists, list-timeline
- [Discovery](#discovery) — news/trending
- [User Info](#user-info) — whoami, about
- [Utility](#utility) — check, query-ids

---

## Global Options

These flags work on every command:

| Flag | Description |
|------|-------------|
| `--auth-token <token>` | Twitter auth_token cookie (override browser) |
| `--ct0 <token>` | Twitter ct0 cookie (override browser) |
| `--chrome-profile <name>` | Chrome profile name for cookie extraction |
| `--chrome-profile-dir <path>` | Chrome/Chromium profile directory or cookie DB path |
| `--firefox-profile <name>` | Firefox profile name for cookie extraction |
| `--cookie-timeout <ms>` | Cookie extraction timeout (keychain/OS helpers) |
| `--cookie-source <source>` | Cookie source for browser extraction (repeatable) |
| `--media <path>` | Attach media file (repeatable, up to 4 images or 1 video) |
| `--alt <text>` | Alt text for corresponding `--media` (repeatable) |
| `--timeout <ms>` | Request timeout in milliseconds |
| `--quote-depth <depth>` | Max quoted tweet depth (default: 1; 0 disables) |
| `--plain` | Plain output — no emoji, no color, stable for scripting |
| `--no-emoji` | Disable emoji output |
| `--no-color` | Disable ANSI colors (or set `NO_COLOR` env) |

---

## Posting Commands

### tweet

Post a new tweet.

```
bunx bird tweet <text>
```

| Argument | Description |
|----------|-------------|
| `text` | Tweet text (required) |

Supports `--media` and `--alt` global flags for attachments.

### reply

Reply to an existing tweet.

```
bunx bird reply <tweet-id-or-url> <text>
```

| Argument | Description |
|----------|-------------|
| `tweet-id-or-url` | Tweet ID or full URL to reply to |
| `text` | Reply text |

Supports `--media` and `--alt` global flags for attachments.

---

## Reading Commands

### read

Fetch and display a single tweet.

```
bunx bird read <tweet-id-or-url>
```

Shorthand: `bunx bird <tweet-id-or-url>` (just pass the ID/URL directly).

| Option | Description |
|--------|-------------|
| `--json` | Output as JSON |
| `--json-full` | JSON with raw API response in `_raw` field |

### thread

Show the full conversation thread containing a tweet.

```
bunx bird thread <tweet-id-or-url>
```

| Option | Description |
|--------|-------------|
| `--all` | Fetch all thread tweets (paged) |
| `--max-pages <n>` | Fetch N pages (implies pagination) |
| `--delay <ms>` | Delay between page fetches (default: 1000) |
| `--cursor <string>` | Resume pagination from cursor |
| `--json` | Output as JSON |
| `--json-full` | JSON with raw API response |

### replies

List replies to a tweet.

```
bunx bird replies <tweet-id-or-url>
```

| Option | Description |
|--------|-------------|
| `--all` | Fetch all replies (paged) |
| `--max-pages <n>` | Fetch N pages |
| `--delay <ms>` | Delay between page fetches (default: 1000) |
| `--cursor <string>` | Resume from cursor |
| `--json` | Output as JSON |
| `--json-full` | JSON with raw API response |

---

## Timeline Commands

### home

Get your home timeline.

```
bunx bird home
```

| Option | Description |
|--------|-------------|
| `-n, --count <n>` | Number of tweets (default: 20) |
| `--following` | Get "Following" feed (chronological) instead of "For You" |
| `--json` | Output as JSON |
| `--json-full` | JSON with raw API response |

### mentions

Find tweets mentioning a user.

```
bunx bird mentions
```

| Option | Description |
|--------|-------------|
| `-u, --user <handle>` | User handle (defaults to current user) |
| `-n, --count <n>` | Number of tweets (default: 10) |
| `--json` | Output as JSON |
| `--json-full` | JSON with raw API response |

### user-tweets

Get tweets from a user's profile timeline.

```
bunx bird user-tweets <handle>
```

| Option | Description |
|--------|-------------|
| `-n, --count <n>` | Number of tweets (default: 20) |
| `--max-pages <n>` | Stop after N pages (max: 10) |
| `--delay <ms>` | Delay between pages (default: 1000) |
| `--cursor <string>` | Resume from cursor |
| `--json` | Output as JSON |
| `--json-full` | JSON with raw API response |

---

## Search

```
bunx bird search <query>
```

Query supports Twitter search operators like `from:username`, `@mention`, `"exact phrase"`, etc.

| Option | Description |
|--------|-------------|
| `-n, --count <n>` | Number of tweets (default: 10) |
| `--all` | Fetch all results (paged) |
| `--max-pages <n>` | Stop after N pages with `--all` |
| `--cursor <string>` | Resume pagination |
| `--json` | Output as JSON |
| `--json-full` | JSON with raw API response |

---

## Social Commands

### follow / unfollow

```
bunx bird follow <username-or-id>
bunx bird unfollow <username-or-id>
```

Accepts username with or without `@`, or a user ID.

### following

Get users that you (or another user) follow.

```
bunx bird following
```

| Option | Description |
|--------|-------------|
| `--user <userId>` | User ID to query (defaults to you) |
| `-n, --count <n>` | Users per page (default: 20) |
| `--cursor <cursor>` | Pagination cursor |
| `--all` | Fetch all (paginate automatically) |
| `--max-pages <n>` | Stop after N pages |
| `--json` | Output as JSON |

### followers

Get users that follow you (or another user).

```
bunx bird followers
```

Same options as `following`.

---

## Bookmarks & Likes

### bookmarks

Get your bookmarked tweets.

```
bunx bird bookmarks
```

| Option | Description |
|--------|-------------|
| `-n, --count <n>` | Number to fetch (default: 20) |
| `--folder-id <id>` | Bookmark folder (collection) ID |
| `--all` | Fetch all bookmarks (paged) |
| `--max-pages <n>` | Stop after N pages |
| `--cursor <string>` | Resume pagination |
| `--json` | Output as JSON |
| `--json-full` | JSON with raw API response |

**Thread expansion flags:**

| Flag | Description |
|------|-------------|
| `--expand-root-only` | Only expand threads when bookmarked tweet is root |
| `--author-chain` | Only include author self-reply chains connected to bookmark |
| `--author-only` | Include all tweets from bookmarked author in thread |
| `--full-chain-only` | Save entire reply chain connected to bookmarked tweet |
| `--include-ancestor-branches` | Include sibling branches for ancestors (with `--full-chain-only`) |
| `--include-parent` | Include direct parent tweet for non-root bookmarks |
| `--thread-meta` | Add metadata fields (isThread, threadPosition, etc.) |
| `--sort-chronological` | Sort output oldest to newest |

### unbookmark

Remove bookmarked tweets.

```
bunx bird unbookmark <tweet-id-or-url...>
```

Accepts one or more tweet IDs or URLs.

### likes

Get your liked tweets.

```
bunx bird likes
```

| Option | Description |
|--------|-------------|
| `-n, --count <n>` | Number to fetch (default: 20) |
| `--all` | Fetch all likes (paged) |
| `--max-pages <n>` | Stop after N pages |
| `--cursor <string>` | Resume pagination |
| `--json` | Output as JSON |
| `--json-full` | JSON with raw API response |

---

## Lists

### lists

Get your Twitter lists.

```
bunx bird lists
```

| Option | Description |
|--------|-------------|
| `--member-of` | Show lists you're a member of (instead of owned) |
| `-n, --count <n>` | Number to fetch (default: 100) |
| `--json` | Output as JSON |

### list-timeline

Get tweets from a list timeline.

```
bunx bird list-timeline <list-id-or-url>
```

| Option | Description |
|--------|-------------|
| `-n, --count <n>` | Number to fetch (default: 20) |
| `--all` | Fetch all tweets. **WARNING: may get account banned** |
| `--max-pages <n>` | Fetch N pages (safer than `--all`) |
| `--cursor <string>` | Resume pagination |
| `--json` | Output as JSON |
| `--json-full` | JSON with raw API response |

---

## Discovery

### news / trending

Fetch AI-curated news and trending topics from Explore tabs.

```
bunx bird news
bunx bird trending
```

| Option | Description |
|--------|-------------|
| `-n, --count <n>` | Number of items (default: 10) |
| `--ai-only` | Show only AI-curated news items |
| `--with-tweets` | Also fetch related tweets per item |
| `--tweets-per-item <n>` | Tweets per news item (default: 5) |
| `--for-you` | Fetch from For You tab |
| `--news-only` | Fetch from News tab |
| `--sports` | Fetch from Sports tab |
| `--entertainment` | Fetch from Entertainment tab |
| `--trending-only` | Fetch from Trending tab |
| `--json` | Output as JSON |
| `--json-full` | JSON with raw API response |

---

## User Info

### whoami

Show which Twitter account the current credentials belong to.

```
bunx bird whoami
```

No additional options beyond global flags.

### about

Get account origin and location information for a user.

```
bunx bird about <username>
```

| Option | Description |
|--------|-------------|
| `--json` | Output as JSON |

---

## Utility

### check

Check credential availability. Shows which cookies were found and from which source.

```
bunx bird check
```

### query-ids

Show or refresh cached Twitter GraphQL query IDs.

```
bunx bird query-ids
```

| Option | Description |
|--------|-------------|
| `--json` | Output as JSON |
| `--fresh` | Force refresh (downloads X client bundles) |

---

## JSON Output

Commands that support `--json` output structured data. Add `--json-full` to include the raw API response in a `_raw` field (tweet commands only).

Commands with JSON support: `read`, `replies`, `thread`, `search`, `mentions`, `bookmarks`, `likes`, `following`, `followers`, `about`, `lists`, `list-timeline`, `user-tweets`, `query-ids`.

## Pagination Pattern

Commands with pagination follow a consistent pattern:

| Flag | Behavior |
|------|----------|
| `--all` | Fetch all pages automatically |
| `--max-pages <n>` | Cap the number of pages |
| `--cursor <string>` | Resume from a specific cursor |
| `--delay <ms>` | Delay between page fetches (default: 1000ms) |

When using `--json`, the cursor for the next page is included in the response.
