---
name: dm-check
description: Daily routine. Read new or unread LinkedIn DMs the owner hasn't answered, draft a suggested reply for each into DM Desk, run the voice-note check, and email a summary. Never sends.
---

# LinkedIn DMs: check and draft replies

Before starting read `${CLAUDE_PLUGIN_ROOT}/references/config.md`, `safety.md`, `schemas.md`, `logging.md`, then `~/.claude/linkedin-outreach-desk/config.json`. This job NEVER sends on LinkedIn; the only change it may make is re-marking opened conversations unread.

Start: ArtifactData `list` on `messages` (limit 1000, out_dir in scratchpad) and `get` `meta/status`.

## Step 1 - Find conversations needing a reply
1. Open https://www.linkedin.com/messaging/ in your own tab; apply safety rules 2-3.
2. From Focused (and "Other" if present) collect conversations whose newest message is NOT from the owner AND that are unread OR arrived in the last 3 days. Scroll until older than 7 days. Read the list via `javascript_tool`/get_page_text before opening anything and note which were unread beforehand. Cap at `limits.dmDraftsPerRun`, newest first; mention overflow.
3. Skip group threads with >3 participants unless addressed by name, and LinkedIn system notices. Sponsored/InMail ads: include with `category: "sponsored"`, empty `suggestedReply`, `aiNote: "Sponsored message, suggest Skip"`.

## Step 2 - Read each conversation
Open it, take thread id from the URL, extract the last <=6 messages, the sender's name, headline and profile URL.
- If the owner's message is actually newest, skip. If the doc id exists, skip.
- An OLDER doc for the same `threadId` that is `pending` and `edited:false` becomes `status:"superseded"` (update with if_version). Never touch approved/edited/sent/skipped/failed docs.
- If the conversation was unread before you opened it, mark it unread again via its options menu. If unavailable, say so in the summary.

## Step 2b - Voice-note check (skip for sponsored / sales pitch)
For every new doc and every existing pending/approved/failed doc whose `voiceNote` is missing or `not_sent`:
1. `profileUrl`: use `senderUrl` if it is /in/<slug>/; otherwise open it and take the redirected vanity URL. Read only; never click anything on a profile.
2. Tracker (read-only here): `get` `state/tracker` on the relay artifact (`artifacts.relay`) and match by slug, then exact name. `voiceNoteSent` true → `voiceNote:"sent"`, `voiceNoteSource:"tracker"`.
3. Else scan the thread (up to ~100 messages) for an audio clip sent by the owner → `sent`, source `thread`.
4. Else `not_sent` plus `voiceNoteIdea`: one specific talking-point sentence, no promises of jobs, meetings, rates or introductions.
Never flip `sent` back to `not_sent`; never touch voice fields when source is `dashboard`. On existing docs update only `profileUrl` and the voice fields.

## Step 2c - Meetings (if `meetings.mode` is not `off`; in `link-only` mode skip slot lookup and only offer `meetings.bookingLink`, never `book`; availability = `curl -s -H "cal-api-version: 2024-09-04" "https://api.cal.com/v2/slots?eventTypeId=<meetings.calcom.eventTypeId>&start=<date>&end=<date>&timeZone=<owner.timezone>"` (public, no key; a time is open only if its exact start is listed); on failure skip this step and log "Cal.com unavailable")
Only for new docs, never sponsored / sales pitch / spam. Always use `meetings.calcom.eventTypeId` (Google Meet or attendee in-person locations).
- `offer`: they want to talk, or a short call is the natural next step and no time is chosen. `meetings.slotsToOffer` (default 2) open slots from availability in `meetings.offerHours` on different days → `meetingSlots` + `bookingLink`. `meetingFormat` = per `meetings.format`: `coffee_or_meet` if they're in `meetings.inPersonAreas` and the format allows coffee, else `google_meet`. Reply offers those times (owner tz labels) and "or grab any time that suits here: <bookingLink>". aiNote "Meeting: check the times before approving".
- `book`: they picked an offered/proposed time AND their email is in the thread AND format is settled (Meet, or coffee with a named venue). Re-check the slot; set `bookingRequest`, `bookMeeting:true`, confirmation reply, aiNote "Approving books the meeting". Slot gone → `offer` with fresh slots. Never book in this skill.
- `ask_details`: time picked but email / format / venue missing → confirm time, ask only for what's missing.
- otherwise `none`.

## Step 3 - Draft replies
`suggestedReply` in the owner's voice, using `owner.about`, `owner.english`, `voice.dmLength`, `voice.extraRules`:
- Start with their first name. Answer what they actually said; move things forward with a real question or next step.
- Never commit the owner to a time, rate, salary, job, introduction or promise; use a holding reply ("Let me check and come back to you this week") and set `aiNote`.
- No hashtags, emoji, links (except the owner's own `meetings.bookingLink`), attachments, phone numbers or emails unless already in the thread from the owner.
- `aiNote` (few words) for: meeting/call asks, rates or fees, specific roles or candidates, personal/sensitive, complaints, legal, sales pitches ("suggest Skip or a polite no"), spam (leave reply empty).
New docs: `status:"pending"`, `edited:false`, `replyText = suggestedReply`, `fetchedAt` now. Write everything with one ArtifactData `batch` (50 per batch), then update `meta/status` with `lastCheckedAt` and `lastCheckSummary`. Close your tab. If `delivery.push` and there are new drafts, PushNotification (<200 chars): "N LinkedIn DMs have suggested replies ready in DM Desk" plus the first sender.

## Step 3b - Daily summary (optional)
If `delivery.dailyEmail`, send ONE plain-text email to `owner.email` only (Gmail send_message; on failure create a draft and log it). Subject `LinkedIn DM Desk: N new, M waiting`. Body: summary line; "New today:" (name, category, first ~100 chars of reply or aiNote); "Waiting for your approval: M" (name + category, max 15); "Voice notes to send:" (name, profileUrl, idea; or "none"); "Meetings:" (offers with times, bookings awaiting approval); the DM Desk link. No full message text, no links from messages. If `delivery.slack.enabled`, post the same summary to that channel instead of or as well as the email.

## Step 4 - Log
Per `logging.md`. Hard rules: the only email is the optional summary to the owner; never modify decisions (approved/edited/skipped/sent).
