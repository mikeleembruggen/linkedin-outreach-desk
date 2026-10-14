# LinkedIn Outreach Desk: client setup and customisation guide

You get a system that finds relevant people, sends blank connection requests, tells you each day who to send a personal voice note to (you record it in the LinkedIn mobile app), watches your DMs and comments, drafts replies in your voice, and books coffees or calls. **You approve every reply before it goes out.** Connection requests follow the targeting rules and daily cap you set.

Read METHODOLOGY.md for the big picture and VOICE-NOTES-GUIDE.md for the voice note step.

## 0. What you need

| Item | Why |
|---|---|
| Claude desktop app with Cowork routines | Hosts the scheduled runs |
| Chrome with Claude in Chrome, **signed in to LinkedIn** | Connection requests, DMs and comments run through your own browser |
| Computer on and awake, Chrome open, at the browser routines' times | Those routines drive your Chrome |
| Gmail connector | Detects new connections from LinkedIn's acceptance emails (the voice-note routine) |
| LinkedIn mobile app | Recording voice notes |
| Optional: Cal.com account and API key, or any booking link | Meetings |
| Optional: Slack channel, daily email to yourself | Extra copies of reports |

You never give Claude your LinkedIn password. If LinkedIn signs out or shows a check, routines stop and notify you.

## 1. Install and run setup

1. Add the plugin.
2. Sign in to LinkedIn in Chrome and leave it open.
3. Run the **`setup`** skill. It asks the questions below, writes your config, and creates your private dashboards and routines. You can re-run it any time to change an answer.

## 2. The questions setup asks you

Nothing is pre-filled: every value is yours.

### Who you are
Name exactly as on LinkedIn, first name, timezone, English variant, one or two sentences on what you do and who you talk to. Paste 5-10 messages you have written so replies sound like you.

### Who you want to meet
1. What you sell or do, in one line.
2. One or two **candidate types**. For each: title/headline keywords, and a location rule: **strict** (only these places), **prefer** (tiebreak) or **any**, plus the places.
3. Exclusions (competitors, students, vendors, certain industries).
4. Company cap (max requests to one employer per run; default 2).
5. Volume: requests per run (min and max) and per day. Start with 5-8 per run and 10 per day; raise slowly.
6. Blank request (default) or a short note.

How it finds people: your home feed and "People similar to X" on profiles that already matched. It never uses LinkedIn search.

### Voice notes
- **The time of day you will sit down and send them** (`voiceNotes.sendTime`). The routine runs 15 minutes earlier and delivers your list then.
- How many per day (3-5 is plenty).
- Priority locations (cities or regions you want to send to; leave empty for no filter).
- Which titles count as decision-makers.

### Replies
Length, rules (hashtags, emoji, links), and your writing samples.

### Meetings
| Mode | What happens | You set up |
|---|---|---|
| `calcom` | Offers 2 real open times, books the chosen one when you approve | Cal.com event, API key, MCP (below) |
| `link-only` | Reply includes your booking link; they pick a time | any booking URL (Calendly, Google, TidyCal...) |
| `off` | No meeting suggestions | nothing |

Also: format (coffee or video, video only, coffee only, phone call), the cities where you offer coffee, and offer hours.

### Delivery
Push notification (default on), optional daily email to your own address, optional Slack channel (give the channel id; off by default).

## 3. Cal.com setup (you do this yourself, only for `calcom` mode)

1. In Cal.com create an event type (e.g. "Meeting", 30 min) with locations **Google Meet** and **In person (attendee)**.
2. Settings > Developer > API Keys: create a key.
3. In your own terminal (Claude never sees the key):
   ```
   claude mcp add --scope user cal-com -e CAL_API_KEY=<your key> -- npx -y @calcom/cal-mcp@latest
   ```
4. Give `setup` your booking link (`cal.com/<user>/<slug>`); it reads the event id and length.

Using another service? Choose `link-only` and give setup your URL; no key needed.

## 4. Where each routine runs and when

| Routine | Skill | Suggested time | Runs on |
|---|---|---|---|
| Connect | `connect` | e.g. 09:00 and 15:00 weekdays | Cowork routine (needs your computer and Chrome) |
| Comments check | `comments-check` | 07:15 daily | Cowork routine |
| DM check | `dm-check` | 08:15 daily | Cowork routine |
| **Voice-note picks** | `voice-notes` | your `sendTime` minus 15 minutes | **Cloud routine (works with the computer off)** |
| DM send | `dm-send` | 13:00 daily | Cowork routine |
| Comments post | `comments-post` | 17:30 daily | Cowork routine |

Why the split: anything that clicks around LinkedIn needs your signed-in Chrome, so it runs as a Cowork routine on your computer. The voice-note routine only reads Gmail and writes to your artifact, so it runs in the cloud and fires even if the computer is asleep. Keep the order: check before send, and spread the connect runs across the day.

## 5. Rollout order

1. Run `setup`; it does one supervised `dm-check`, `comments-check` and a small `connect` run.
2. Enable the read-only routines: `comments-check`, `dm-check`, `voice-notes`.
3. For a week: approve replies by hand; refine voice and rules.
4. Enable `dm-send` and `comments-post`.
5. Enable `connect` at low volume; review the first reports.
6. After two weeks, review acceptance, reply and booking rates before raising limits.

## 6. Approving replies
Open DM Desk or Comment Desk, read each card, then Approve, Edit-and-approve or Skip. Approved text is sent exactly as stored at the next send run. If the other person wrote again since the draft, the reply is held back for you.

## 7. Safety and limits
- Never share passwords, 2FA codes or API keys with Claude.
- A captcha, verification or "unusual activity" notice stops the run; resolve it yourself.
- Dashboards are **private** (they hold your messages).
- Automating LinkedIn can breach LinkedIn's terms and your account may be restricted. Keep volumes low.
- LinkedIn UI changes may need skill tweaks.

## 8. Troubleshooting
| Problem | Likely cause / fix |
|---|---|
| "LinkedIn is signed out" notification | Sign in in Chrome; re-run the routine |
| Browser routine didn't run | Computer asleep or Chrome closed |
| No voice-note list | No new connections or none in your priority locations; check Gmail connector |
| Wrong location or someone missing | Location comes from web search; correct it in `state/tracker` |
| Dashboard empty | Run `dm-check` / `comments-check` once; check artifact URLs in config |
| Booking fails | API key, event type id, `cal-com` MCP connected |
| Replies sound off | Add more writing samples and rules |
| Duplicate runs | An old task with the same job is still enabled; disable it |

## 9. Where things live
- Config: `~/.claude/linkedin-outreach-desk/config.json` (copy in the relay artifact for cloud runs)
- Run log: `~/.claude/linkedin-outreach-desk/run-log.md`
- Dashboards and tracker state: your private artifacts (links in config)
