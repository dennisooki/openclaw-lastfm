# Last.fm OpenClaw Skill - Implementation Plan

## Overview
A secure, configurable skill that provides access to Last.fm user profile data including now playing, top tracks/artists/albums by time period, loved tracks, and optional write operations (love/unlove, scrobble).

---

## Skill Structure

```
lastfm/
├── SKILL.md                    # Main skill definition
├── references/
│   ├── api-endpoints.md        # Last.fm API reference
│   └── auth-guide.md           # Authentication setup guide
└── scripts/
    └── lastfm-api.sh           # Helper script for API calls
```

---

## SKILL.md Frontmatter (Security-Focused)

```yaml
---
name: lastfm
description: Access Last.fm user profile, now playing, top tracks/artists/albums by period, loved tracks, and optionally love/unlove tracks.
metadata:
  {
    "openclaw":
      {
        "requires": { "env": ["LASTFM_API_KEY", "LASTFM_USERNAME"] },
        "primaryEnv": "LASTFM_API_KEY",
        "emoji": "🎵"
      }
  }
---
```

**Key security decisions:**
- No hardcoded API keys or secrets
- Environment variables only (injected via `~/.openclaw/openclaw.json`)
- Read-only by default; write operations require `LASTFM_SESSION_KEY` (optional)

---

## Features (User-Configurable)

| Feature | API Method | Auth Required | Config Flag |
|---------|-----------|---------------|-------------|
| Now Playing | `user.getRecentTracks` | No | `read_enabled` (default: true) |
| Profile Info | `user.getInfo` | No | `read_enabled` |
| Top Tracks | `user.getTopTracks` | No | `read_enabled` |
| Top Artists | `user.getTopArtists` | No | `read_enabled` |
| Top Albums | `user.getTopAlbums` | No | `read_enabled` |
| Loved Tracks | `user.getLovedTracks` | No | `read_enabled` |
| Recent Tracks | `user.getRecentTracks` | No | `read_enabled` |
| Love Track | `track.love` | Yes | `write_enabled` (default: false) |
| Unlove Track | `track.unlove` | Yes | `write_enabled` |
| Scrobble | `track.scrobble` | Yes | `write_enabled` |

---

## Configuration Schema (`~/.openclaw/openclaw.json`)

```json5
{
  skills: {
    entries: {
      lastfm: {
        enabled: true,
        apiKey: "YOUR_LASTFM_API_KEY",
        env: {
          LASTFM_API_KEY: "YOUR_LASTFM_API_KEY",
          LASTFM_USERNAME: "your_username",
          LASTFM_SESSION_KEY: "optional_for_write_ops"
        },
        config: {
          read_enabled: true,
          write_enabled: false,
          default_period: "7day"
        }
      }
    }
  }
}
```

---

## Security Best Practices (To Avoid Malicious Flagging)

| Practice | Implementation |
|----------|----------------|
| **No hardcoded secrets** | All credentials via env vars |
| **No network exfiltration** | Only calls `ws.audioscrobbler.com` |
| **No shell escape sequences** | Clean, validated inputs |
| **No obfuscated code** | Clear, documented SKILL.md |
| **No install hooks** | No `setupCommand` or lifecycle scripts |
| **Explicit permissions** | Documents exactly what data is accessed |
| **Fail-safe defaults** | Write ops disabled by default |
| **No prompt injection** | No dynamic instruction loading |
| **Clear author identity** | Include `author`, `homepage` in metadata |

---

## Slash Command Approach

Based on OpenClaw conventions, the skill will be **user-invocable by default** (`user-invocable: true` is default). This allows:
- Slash command: `/lastfm now-playing`, `/lastfm top-tracks 7day`
- Natural language: "What am I listening to on Last.fm?"

---

## Last.fm Authentication Flow (For Write Operations)

1. User requests API key from Last.fm: https://www.last.fm/api/account/create
2. For write ops, user must authorize via desktop auth flow:
   - Get token: `auth.getToken`
   - User visits authorization URL
   - Exchange token for session key: `auth.getSession`
3. Session key stored in `LASTFM_SESSION_KEY` env var

---

## Sample SKILL.md Content

```markdown
---
name: lastfm
description: Access Last.fm user profile, now playing, top tracks/artists/albums, and optionally love/unlove tracks.
metadata:
  {
    "openclaw":
      {
        "requires": { "env": ["LASTFM_API_KEY", "LASTFM_USERNAME"] },
        "primaryEnv": "LASTFM_API_KEY",
        "emoji": "🎵",
        "homepage": "https://github.com/YOUR_USERNAME/openclaw-skill-lastfm"
      }
  }
---

# Last.fm Profile Skill

## What it does
Retrieves Last.fm user listening data including now playing, top tracks/artists/albums by time period, and loved tracks. Optionally supports write operations (love/unlove tracks, scrobble) when `LASTFM_SESSION_KEY` is configured.

## Required Environment Variables
- `LASTFM_API_KEY`: Your Last.fm API key (get one at https://www.last.fm/api/account/create)
- `LASTFM_USERNAME`: Your Last.fm username

## Optional Environment Variables
- `LASTFM_SESSION_KEY`: Required for write operations (love/unlove, scrobble)

## Workflow
1. Validate required environment variables are present
2. Construct API request to `ws.audioscrobbler.com/2.0/`
3. Execute request with appropriate method and parameters
4. Parse and format response for user

## Supported Commands
- `now-playing` / `np`: Current or most recent track
- `top-tracks [period]`: Top tracks (period: 7day, 1month, 3month, 6month, 12month, overall)
- `top-artists [period]`: Top artists
- `top-albums [period]`: Top albums
- `loved`: Loved tracks
- `profile`: User profile info
- `love <artist> <track>`: Love a track (requires auth)
- `unlove <artist> <track>`: Unlove a track (requires auth)

## Guardrails
- Never log or expose API keys or session keys
- Rate limit: respect Last.fm's 5 requests/second limit
- Write operations fail gracefully if `LASTFM_SESSION_KEY` not set
- All user inputs are URL-encoded before API calls
- Only connect to `ws.audioscrobbler.com` - no external endpoints
```

---

## Additional Feature Suggestions

| Feature | Description |
|---------|-------------|
| **Weekly listening report** | Generate a formatted summary of week's listening |
| **Compare with friend** | Compare top artists with another Last.fm user |
| **Track recommendations** | Based on top artists, suggest similar artists |
| **Listening clock** | Show listening patterns by time of day |
| **Genre breakdown** | Aggregate top tags into genre percentages |

---

## Publishing Checklist (For ClawHub Safety Review)

- [ ] No hardcoded credentials anywhere in skill
- [ ] All network calls to documented Last.fm endpoints only
- [ ] Clear README with setup instructions
- [ ] No install hooks or lifecycle scripts
- [ ] No obfuscated or minified code
- [ ] Explicit permission documentation
- [ ] Author identity verifiable (GitHub profile linked)
- [ ] Test with `openclaw skills validate ./lastfm`
- [ ] Run through skill verifier before publishing
