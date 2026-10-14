---
name: voice-notes
description: Daily cloud routine at the time the owner chose. Detect new 1st-degree connections from LinkedIn acceptance emails in Gmail, pick today's people to send a voice note to by the owner's rules, and deliver the list as one notification (and optional email). No browser, never writes to LinkedIn.
---

# Voice notes: new connections, today's picks, one daily message

Read `${CLAUDE_PLUGIN_ROOT}/references/safety.md` (rules 4, 6, 10, 11 apply), `schemas.md`, `logging.md` and `~/.claude/linkedin-outreach-desk/config.json`. This routine runs in the cloud: do NOT use a browser or Claude in Chrome and do not try to sign in to LinkedIn. It never sends any LinkedIn message, email to anyone else, or connection request. Voice notes themselves are recorded by the owner in the LinkedIn mobile app.

## Config used
`voiceNotes.count` (default 5), `voiceNotes.sendTime`, `voiceNotes.priorityLocations` (list; empty = no location filter), `voiceNotes.priorityTitles`, `voiceNotes.maxTimesRecommended` (default 3), `voiceNotes.perCompanyCap` (default 2), `voiceNotes.excludeKeywords`, `artifacts.relay`, `owner.timezone`, `delivery`.

## Data (relay artifact, `artifacts.relay`)
- `state/tracker`: `{updatedAt, rows:[{slug,name,title,connected,voiceNoteSent,location,timesRecommended,lastRecommended}]}`.
- `clicks` collection: one doc per slug the owner opened through a relay link. A clicked slug is excluded for good.
- Relay link for every pick: `<artifacts.relay>#<slug>`; it logs the click and forwards to the LinkedIn profile.
- DM Desk `messages` (if `artifacts.dmDesk`): any doc with `voiceNote:"sent"` and a `profileUrl` slug marks that tracker row `voiceNoteSent:true`.

## Steps
1. `get` `state/tracker` (keep `version`; if missing, start with `rows: []`). `list` `clicks` (limit 1000, page with cursor).
2. **New connections from Gmail.** Search `from:invitations@linkedin.com newer_than:4d` (page size 50). Relevant emails look like "<First> accepted your invitation". For each, read the message in full; parse the HTML:
   - The new connection's "<Full Name> <Headline>" follows "has accepted your invitation" (or "you're now connected"), before "Message" / "Suggestions from".
   - Their slug is the FIRST `linkedin.com/comm/in/<slug>` link (stop at `?`, `&`, `/`, `"`). Links after "Suggestions from..." or "People you may know" are NOT connections.
   - Connected date = the email date in `owner.timezone`. If name and slug cannot be extracted with confidence, skip that email.
   Add rows for slugs not already present with `voiceNoteSent:false, location:"", timesRecommended:0, lastRecommended:""`. If Gmail is unreachable, continue with existing rows and say so.
3. **Candidate pool:** rows with `voiceNoteSent` false, slug not in clicks, `timesRecommended < voiceNotes.maxTimesRecommended`, no `excludeKeywords` in the title.
4. **Rank:** (a) title matches any `voiceNotes.priorityTitles` (case-insensitive) first; (b) fewer `timesRecommended`; (c) newest `connected`. Soft rule: at most `perCompanyCap` people whose title names the same company.
5. **Location:** if `priorityLocations` is empty skip this step. Otherwise walk down the ranked list until you have `count` confirmed matches or have done 15 new lookups:
   - A filled `location` (including "Unknown") is final; never re-check.
   - Otherwise WebSearch (standard) for `"<name>" linkedin.com/in/<slug>`, then `"<name>" "<company from title>" linkedin`. Accept a location only if a result clearly is the same person (same slug, or same name and company). Write the city/region, or "Unknown". Do not WebFetch linkedin.com.
   - Qualifies if the location contains a `priorityLocations` entry (case-insensitive) or is clearly a suburb of one.
   - If fewer than `count` are confirmed, fill the rest with the best-ranked "Unknown" location people, marked "(location unverified)". Never include someone with a known location outside the list.
6. **Update tracker:** for each pick, `timesRecommended += 1`, `lastRecommended = today`; set `updatedAt`. `set` with `if_version`; on conflict re-read once, re-apply, retry once.
7. **Deliver one message** (this is the notification; plain text, no headings, no hashtags):
   - Line 1: `Voice notes today: <First names, comma-separated>` (or `No voice-note picks today` plus a short reason).
   - One line per person: `<n>. <Full name>, <short title> (<location>) <artifacts.relay>#<slug>`.
   - One short line only if relevant: how many new connections were found, or a problem.
   If `delivery.dailyEmail` is true, also email the same text to `owner.email` ONLY (Gmail; on failure, create a draft). Never email anyone else.
8. Log per `logging.md`.

## Hard rules
Never contact anyone. Never mark `voiceNoteSent:true` from a guess (only from DM Desk docs). Never reduce `timesRecommended`. Never exceed `count` picks.
