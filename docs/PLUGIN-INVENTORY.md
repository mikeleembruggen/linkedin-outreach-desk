# Plugin inventory: LinkedIn Outreach Desk v0.2

Scope: LinkedIn connections, voice notes, DMs and comments only. Not included: Talent Wire (separate), Veo B-roll, Phoenix Rising, Meta ads, second brain / Obsidian logging, the old five push-nudge tasks, the old Slack/email tracker report, the prospects dashboard, the local tracker CSV and its dashboard.

## Skills

| Skill | Purpose | Writes to LinkedIn | Host | Source today |
|---|---|---|---|---|
| `setup` | Client interview, config, artifacts, Cal.com, routines | no | one-off | new |
| `connect` | Find (feed + "People similar to X") and send blank requests, per client rules and daily cap, no approval step | requests only | Cowork routine | Cowork "LinkedIn networking (2x/day)" |
| `voice-notes` | New connections from Gmail, daily picks, one message at the client's chosen time | never | Cloud routine | cloud "voice note Top 5" (rebuilt with config) |
| `dm-check` | Draft DM replies, meeting offers, voice-note status | no | Cowork routine | `linkedin-dms-check` |
| `dm-send` | Send approved replies, book meetings first | approved only | Cowork routine | `linkedin-dms-send-replies` |
| `comments-check` | Draft replies to comments on own posts | no | Cowork routine | `linkedin-comments-check` |
| `comments-post` | Post approved replies | approved only | Cowork routine | `linkedin-comments-post-replies` |

## Artifacts (created by `setup` per client, private)
`relay.html` (click relay + tracker/state db), `dm-desk.html`, `comment-desk.html`.

## Config
`~/.claude/linkedin-outreach-desk/config.json` (schema: `references/config.md`), mirrored to the relay artifact for cloud routines. Everything client-specific is asked during setup.

## Integrations
| Integration | For | Required |
|---|---|---|
| Claude in Chrome (signed in to LinkedIn) | connect, DMs, comments | yes |
| Artifact + ArtifactData | dashboards, state | yes |
| Gmail connector | new-connection detection, optional daily email | yes |
| Cowork routines / scheduled tasks | all runs | yes |
| LinkedIn mobile app | recording voice notes | yes (by hand) |
| Cal.com + `@calcom/cal-mcp` + API key | automatic booking | optional, client sets up |
| Any booking link | link-only meetings | optional |
| Slack channel | extra copy of reports | optional |
