---
name: bird-fast
description: Post tweets, read threads, search X/Twitter from the terminal using the bird CLI. Use when automating Twitter/X, posting from scripts, analyzing tweet threads, monitoring mentions, or working with Twitter/X via browser-cookie auth.
---

# bird CLI

A command-line interface for X/Twitter that authenticates via an existing browser session — no API keys or OAuth setup required.

## Safety

- Treat X/Twitter writes as external actions: ask for explicit approval before posting, replying, deleting, following/unfollowing, muting, blocking, or sending DMs unless the user has given a clear automation rule.
- Never ask the user to paste `auth_token`, `ct0`, cookies, or browser credential files into chat.
- Avoid using secret CLI flags in agent sessions (`--auth-token`, `--ct0`) because they can leak through logs/history/context.
- Prefer browser-cookie auth: have the user log into `x.com` in Safari, Chrome, or Firefox, then verify with `bird whoami` / `bird check`.
- Do not read or print browser cookie databases.

## Overview

`bird` is a TypeScript-based CLI that uses Twitter's internal GraphQL API with automatic query ID management. It extracts authentication cookies from Safari, Chrome, or Firefox.

Key features:

- Zero-config authentication via browser cookies
- Full tweet lifecycle: post, reply, read, search
- Media uploads with alt text
- Multiple output formats: human-readable, JSON, plain
- Automatic GraphQL query ID refresh and REST API fallback

## When to Use

- Posting tweets or replies from scripts or automation
- Reading and analyzing tweet threads programmatically
- Searching for mentions or specific content
- Monitoring Twitter activity from the CLI

## Prerequisites

- Node.js >= 22 OR Homebrew standalone binary
- Browser login: must be logged into `x.com` in Safari, Chrome, or Firefox

## Installation

```bash
# Homebrew (macOS — standalone binary, no Node.js required)
brew install steipete/tap/bird

# npm
npm install -g @steipete/bird

# pnpm
pnpm add -g @steipete/bird

# bun
bun add -g @steipete/bird

# One-shot execution (no install)
bunx @steipete/bird whoami
npx @steipete/bird whoami
```

## Authentication

`bird` uses this priority chain:

1. CLI flags: `--auth-token`, `--ct0` (avoid in agent sessions)
2. Environment variables: `AUTH_TOKEN`, `CT0`, `TWITTER_AUTH_TOKEN`, `TWITTER_CT0` (avoid asking user to share these)
3. Browser cookies: auto-extracted from Safari → Chrome → Firefox (preferred)

Verify authentication:

```bash
bird whoami
bird check
```

If cookie access fails on macOS, the Terminal/OpenClaw process may need Full Disk Access or the user may need to log into `x.com` in a supported browser.

## Quick Start

```bash
# Read-only/safe checks
bird whoami
bird check
bird read https://x.com/user/status/1234567890123456789
bird search "from:elonmusk" -n 20
bird mentions

# External writes: require user approval first
bird tweet "Hello from the terminal!"
bird reply 1234567890123456789 "Great point!"
```

## Command Reference

### Writing Commands

Post a new tweet with optional media:

```bash
bird tweet "<text>" [options]

# Options
--media <path>    Attach media file (repeatable, up to 4 images/GIFs or 1 video)
--alt <text>      Alt text for corresponding --media (repeatable)
```

Reply to an existing tweet:

```bash
bird reply <tweet-id-or-url> "<text>" [options]
```

### Reading Commands

Fetch a single tweet. Handles standard tweets, Notes, and Articles:

```bash
bird read <tweet-id-or-url>
bird 1234567890123456789
bird https://x.com/user/status/1234567890123456789
```

Read replies or a thread:

```bash
bird replies <tweet-id-or-url>
bird thread <tweet-id-or-url>
```

### Search Commands

```bash
bird search "<query>" [-n <count>]
bird search "from:anthropic"
bird search "@username"
```

Supports Twitter advanced search syntax: `from:`, `to:`, `since:`, `until:`, `filter:links`, etc.

Mentions/bookmarks:

```bash
bird mentions [-u <handle>] [-n <count>]
bird bookmarks [-n <count>]
```

### Utility Commands

```bash
bird whoami
bird check
bird query-ids [--fresh] [--json]
bird help [command]
```

## Global Options

- `--cookie-source <source>`: browser priority: `safari`, `chrome`, `firefox` (repeatable)
- `--chrome-profile <name>`: Chrome profile name
- `--firefox-profile <name>`: Firefox profile name
- `--timeout <ms>`: request timeout in milliseconds
- `--plain`: stable output, no emoji/color (good for scripts)
- `--no-emoji`: disable emoji
- `--no-color`: disable ANSI colors
- `--json`: output JSON where supported

## Media Attachments

- Images: `.jpg`, `.jpeg`, `.png`, `.webp` — up to 4
- GIFs: `.gif` — up to 4
- Videos: `.mp4`, `.m4v`, `.mov` — 1 only

Cannot mix videos with images/GIFs. Always include alt text for images when practical.

```bash
bird tweet "Three photos" \
  --media photo1.jpg --alt "Beach sunset" \
  --media photo2.jpg --alt "Mountain view" \
  --media photo3.jpg --alt "City skyline"
```

## Output Formats

- Human-readable: default
- Plain: `--plain` for scripting/logs
- JSON: `--json` for programmatic parsing

```bash
bird read 1234567890123456789 --json | jq '.text'
bird mentions -n 50 --json | jq '.[] | {user: .user.screen_name, text: .text}'
```

## Configuration

Optional JSON5 config files:

- `~/.config/bird/config.json5` — global
- `./.birdrc.json5` — project-specific

Example:

```json5
{
  cookieSource: ["firefox", "safari"],
  firefoxProfile: "default-release",
  timeoutMs: 20000
}
```

## Best Practices

- Use `--json` for scripting.
- Use `--plain` in CI/logs.
- Add `--alt` when attaching images.
- Run `bird query-ids --fresh` if bird has not been used in a few days or GraphQL errors appear.
- Handle rate limits; avoid tight loops and add delays for bulk reads.

## Troubleshooting

### Missing credentials / no auth token found

- Verify the user is logged into `x.com` in Safari, Chrome, or Firefox.
- Try a specific browser source: `bird whoami --cookie-source safari`.
- On macOS, grant Full Disk Access to the terminal/process running OpenClaw if Safari cookie reads fail with `EPERM`.
- Do not ask the user to paste cookie values into chat.

### GraphQL 404 errors

Refresh the query ID cache:

```bash
bird query-ids --fresh
```

### Error 226 automation detected

Wait a few minutes and retry. `bird` may fall back to REST API, but X can still throttle.

### Wrong account detected

Use a specific browser/profile:

```bash
bird whoami --cookie-source chrome --chrome-profile "Profile 1"
```

## Exit Codes

- `0`: success
- `1`: runtime error (network, auth, API)
- `2`: invalid usage

## Resources

- GitHub: https://github.com/steipete/bird
- Original skill source: https://github.com/ckorhonen/claude-skills/blob/main/skills/bird-fast/SKILL.md
