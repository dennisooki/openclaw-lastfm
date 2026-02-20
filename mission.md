# Mission: Last.fm OpenClaw Skill Verification

This document defines the success criteria and verification steps for the Last.fm OpenClaw skill implementation.

---

## Pre-Implementation Checklist

- [ ] Directory structure matches plan
- [ ] Git repository initialized with clean history
- [ ] All required files created

---

## Security Verification (Critical)

### Must Pass All Checks

| Check | Verification Method | Status |
|-------|---------------------|--------|
| No hardcoded API keys | `grep -r "api_key.*=" skill/` returns no secrets | [ ] |
| No hardcoded secrets | `grep -rE "(password|secret|token).*=.*['\"]" skill/` returns no values | [ ] |
| No external endpoints | `grep -r "http" skill/` only shows `ws.audioscrobbler.com` | [ ] |
| No install hooks | No `setupCommand`, `postInstall`, or lifecycle scripts | [ ] |
| No obfuscated code | All files are human-readable markdown/shell | [ ] |
| No shell escape abuse | No `eval`, `exec`, or unescaped variables to shell | [ ] |
| No prompt injection | No dynamic instruction loading from external sources | [ ] |

---

## File Structure Verification

```
lastfm/
├── SKILL.md                    [ ] Exists
├── plan.md                     [ ] Exists
├── mission.md                  [ ] Exists
├── README.md                   [ ] Exists
├── references/
│   ├── api-endpoints.md        [ ] Exists
│   └── auth-guide.md           [ ] Exists
└── scripts/
    └── lastfm-api.sh           [ ] Exists
```

---

## SKILL.md Verification

### Required Frontmatter Fields

- [ ] `name: lastfm`
- [ ] `description` is clear and concise
- [ ] `metadata.openclaw.requires.env` includes `LASTFM_API_KEY`
- [ ] `metadata.openclaw.requires.env` includes `LASTFM_USERNAME`
- [ ] `metadata.openclaw.primaryEnv` set to `LASTFM_API_KEY`

### Required Sections

- [ ] `## What it does` - Clear description
- [ ] `## Required Environment Variables` - Documents required env vars
- [ ] `## Workflow` - Step-by-step execution flow
- [ ] `## Supported Commands` - Lists all available commands
- [ ] `## Guardrails` - Security and safety constraints

---

## Functionality Verification

### Read Operations (No Auth Required)

| Command | Expected Output | Status |
|---------|-----------------|--------|
| `/lastfm now-playing` | Current or most recent track with artist, album | [ ] |
| `/lastfm top-tracks 7day` | Top tracks for 7-day period | [ ] |
| `/lastfm top-artists 1month` | Top artists for 1-month period | [ ] |
| `/lastfm top-albums overall` | Top albums all time | [ ] |
| `/lastfm loved` | List of loved tracks | [ ] |
| `/lastfm profile` | User profile information | [ ] |

### Write Operations (Auth Required)

| Command | Expected Output | Status |
|---------|-----------------|--------|
| `/lastfm love "Artist" "Track"` | Confirmation of loved track | [ ] |
| `/lastfm unlove "Artist" "Track"` | Confirmation of unloved track | [ ] |

### Error Handling

| Scenario | Expected Behavior | Status |
|----------|-------------------|--------|
| Missing `LASTFM_API_KEY` | Clear error message requesting setup | [ ] |
| Missing `LASTFM_USERNAME` | Clear error message requesting setup | [ ] |
| Invalid API key | Error from Last.fm API, no crash | [ ] |
| Rate limit exceeded | Graceful handling, retry suggestion | [ ] |
| User not found | Clear error message | [ ] |
| Write op without `LASTFM_SESSION_KEY` | Clear error explaining auth required | [ ] |

---

## API Endpoint Verification

| Endpoint | Method | Used For | Status |
|----------|--------|----------|--------|
| `ws.audioscrobbler.com/2.0/` | `user.getInfo` | Profile info | [ ] |
| `ws.audioscrobbler.com/2.0/` | `user.getRecentTracks` | Now playing, recent | [ ] |
| `ws.audioscrobbler.com/2.0/` | `user.getTopTracks` | Top tracks | [ ] |
| `ws.audioscrobbler.com/2.0/` | `user.getTopArtists` | Top artists | [ ] |
| `ws.audioscrobbler.com/2.0/` | `user.getTopAlbums` | Top albums | [ ] |
| `ws.audioscrobbler.com/2.0/` | `user.getLovedTracks` | Loved tracks | [ ] |
| `ws.audioscrobbler.com/2.0/` | `track.love` | Love track (write) | [ ] |
| `ws.audioscrobbler.com/2.0/` | `track.unlove` | Unlove track (write) | [ ] |

---

## Configuration Verification

Sample config in `~/.openclaw/openclaw.json`:

```json5
{
  skills: {
    entries: {
      lastfm: {
        enabled: true,
        env: {
          LASTFM_API_KEY: "YOUR_KEY",
          LASTFM_USERNAME: "your_username",
          LASTFM_SESSION_KEY: "optional"
        }
      }
    }
  }
}
```

- [ ] Config example provided in README
- [ ] Config example provided in auth-guide.md
- [ ] All env vars documented

---

## Documentation Verification

### README.md

- [ ] Installation instructions
- [ ] Configuration instructions
- [ ] Usage examples
- [ ] API key setup link
- [ ] Security notes

### references/auth-guide.md

- [ ] How to get API key
- [ ] How to get session key (for write ops)
- [ ] Desktop auth flow explained

### references/api-endpoints.md

- [ ] All used endpoints documented
- [ ] Parameter requirements listed
- [ ] Response format examples

---

## ClawHub Publishing Verification

| Requirement | Status |
|-------------|--------|
| No hardcoded credentials | [ ] |
| Clear author identity | [ ] |
| No malicious patterns | [ ] |
| Valid skill structure | [ ] |
| Passes skill validator | [ ] |

---

## Final Sign-Off

- [ ] All security checks passed
- [ ] All functionality tests passed
- [ ] Documentation complete
- [ ] Ready for ClawHub submission

---

## Notes

_Add any implementation notes or deviations from plan here:_
