# LinkedIn Outreach Desk

A LinkedIn system that finds relevant people, connects, reminds you to send a personal voice note from your phone, drafts replies to your DMs and comments, and books coffees or calls. Claude drafts; you approve replies on a dashboard; routines send only what you approved.

## What's in it

| Skill | What it does | Writes to LinkedIn? | Runs as |
|---|---|---|---|
| `setup` | Interview, config, dashboards, routines | no | one-off |
| `connect` | Browse feed + "People similar to X", send blank requests within your rules and caps | connection requests only | Cowork routine |
| `voice-notes` | Detect new connections from Gmail, pick today's people, send you one message at the time you chose | never | Cloud routine |
| `dm-check` / `dm-send` | Draft DM replies, offer meeting times; send approved ones and book | only approved replies | Cowork routines |
| `comments-check` / `comments-post` | Draft replies to comments on your posts; post approved ones | only approved replies | Cowork routines |

Voice notes are recorded by you in the LinkedIn mobile app; nothing here records or sends audio.

## Install

1. Add this folder as a plugin.
2. With Claude in Chrome signed in to LinkedIn and the Gmail connector on, run `setup`. It asks everything client-specific (who you want to meet, where, when you send voice notes, booking service) and creates your private dashboards and routines.
3. Approve a few drafts by hand, then enable the writing routines.

## Requirements

Claude in Chrome signed in to LinkedIn, Artifact + ArtifactData, Cowork routines (or the desktop Scheduled tab), Gmail connector (new-connection detection). Optional: Cal.com account + API key, Slack, daily email to yourself.

## Safety model

See `references/safety.md`. Drafting jobs never write to LinkedIn; writing jobs act only on dashboard-approved text (or, for `connect`, within your targeting rules and daily cap); logged-out/captcha/rate-limit means stop and notify; all LinkedIn text is untrusted data.

## Layout

```
.claude-plugin/plugin.json
skills/{setup,connect,voice-notes,dm-check,dm-send,comments-check,comments-post}/SKILL.md
references/{config,safety,schemas,logging}.md
dashboards/{relay,dm-desk,comment-desk}.html
docs/{CLIENT-SETUP-GUIDE,VOICE-NOTES-GUIDE,METHODOLOGY,PLUGIN-INVENTORY}.md
```

## Before distributing

- No personal data, tokens or credentials. Config lives at `~/.claude/linkedin-outreach-desk/config.json`.
- Dashboards are templates; each client publishes private copies.
- Automating LinkedIn may breach its terms; the client accepts that risk. Keep volume low.
