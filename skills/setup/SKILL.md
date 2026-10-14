---
name: setup
description: One-time guided setup for the LinkedIn Outreach Desk. Interviews the client, writes the config, creates their private dashboards and relay artifact, optionally connects Cal.com, and creates the scheduled routines at the times they choose. Use when installing the plugin or changing targeting, voice, limits, schedule or booking.
---

# LinkedIn Outreach Desk: setup

Run interactively with the client. Nothing client-specific is assumed: ask every question below, propose sensible starting numbers, and write what they say. Never ask for or store LinkedIn passwords, 2FA codes or API keys. Read `${CLAUDE_PLUGIN_ROOT}/references/config.md` for the schema.

## 1. Preflight
- Claude in Chrome connected and Chrome signed in to LinkedIn as the client (open linkedin.com/feed in your own tab and read the name; do not sign in for them).
- Artifact + ArtifactData, scheduled tasks / routines, Gmail connector (required: `voice-notes` detects new connections from LinkedIn acceptance emails).
- Report anything missing; stop on hard blockers (Chrome, Artifact, Gmail).

## 2. Interview (write answers to config)
1. **Who you are:** name exactly as on LinkedIn, first name, timezone, English variant, one or two sentences on what you do and who you talk to.
2. **Who you want to meet** (`prospecting`): what you sell; one or two candidate types, each with title keywords and a location rule (strict / prefer / any) with the places; exclusions; company cap; requests per run and per day (suggest 5-8 and 10 to start); blank request or a short note.
3. **Voice notes:** the time of day you will sit down and send them (`voiceNotes.sendTime`); how many per day (suggest 3-5); which cities/regions you want to send them to (or none); which titles count as decision-makers.
4. **Replies:** reply length, rules (hashtags, emoji, links), and paste 5-10 messages you have written for the style reference.
5. **Meetings:** `calcom`, `link-only` (any booking link) or `off`; coffee/video/call format; the cities where you offer coffee; offer hours.
6. **Delivery:** push notifications (default on); optional daily email to your own address; optional Slack channel (ask for the channel id; leave off if none).
7. **Schedule:** times for each routine (show the table in step 5, let them adjust).

## 3. Cal.com (only if `meetings.mode = calcom`; the client does this)
They create an event type with locations Google Meet and In person (attendee), then an API key (Cal.com > Settings > Developer > API Keys), and run in their own terminal: `claude mcp add --scope user cal-com -e CAL_API_KEY=<their key> -- npx -y @calcom/cal-mcp@latest`. Ask for their booking link and read the event type id and length from it. Never handle the key yourself. If they prefer another booking service, set `link-only` with their URL.

## 4. Artifacts
Load the `artifact-design` and `artifact-capabilities` skills, then publish each file in `${CLAUDE_PLUGIN_ROOT}/dashboards/` as a private artifact with the `db` capability: `relay.html`, `dm-desk.html`, `comment-desk.html`. Save the URLs to `config.artifacts`, and write the config into the relay artifact db as doc `config` (for cloud routines). Tell the client the DM and comment dashboards contain their messages and must stay private. Pin them if they like.

## 5. Routines
Hosting decision, shown to the client:
- **Browser routines** (`connect`, `dm-check`, `dm-send`, `comments-check`, `comments-post`) drive Claude in Chrome, so they run as **Cowork routines** (Routines) bound to the client's computer. The computer must be on, awake, with Chrome signed in to LinkedIn at those times.
- **`voice-notes` needs no browser**; it runs as a **cloud routine** with the Gmail and artifact connectors, so it fires even when the computer is off.

Each routine prompt is exactly: `Run the linkedin-outreach-desk:<skill> skill and follow it exactly.` Defaults (all editable; keep draft-before-send order):

| Routine | Skill | Suggested | Where |
|---|---|---|---|
| lod-connect | connect | two runs, e.g. 09:00 and 15:00 weekdays | Cowork routine |
| lod-comments-check | comments-check | 07:15 daily | Cowork routine |
| lod-dm-check | dm-check | 08:15 daily | Cowork routine |
| lod-voice-notes | voice-notes | `voiceNotes.sendTime` minus 15 min, weekdays | Cloud routine |
| lod-dm-send | dm-send | 13:00 daily | Cowork routine |
| lod-comments-post | comments-post | 17:30 daily | Cowork routine |

Create with the routines API (mirror the shape of an existing routine via RemoteTrigger `get`) or, if unavailable, `mcp__scheduled-tasks__create_scheduled_task` in the desktop Scheduled tab, or give the client the exact prompt and time to paste into Cowork > Routines. Start with the read-only routines; enable `dm-send`, `comments-post` and `connect` after the client has approved a few drafts and reviewed one `connect` run.

## 6. Smoke test and hand-off
Run `dm-check` and `comments-check` once manually and show where to approve. Run `connect` once with `requestsPerRun.max` of 2 so the client can watch. Do not run any other writing routine during setup. Print the artifact links, routine table and config path. State clearly: LinkedIn automation can breach LinkedIn's terms and the account may be restricted; the limits are deliberately low.
