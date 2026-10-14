---
name: connect
description: Routine, one to a few runs a day. Find relevant new people by browsing the owner's LinkedIn feed and "People similar to X", then send blank connection requests to the best matches within the caps in config. No approval step; the owner's targeting rules and caps are the authorisation.
---

# LinkedIn connections: find and connect

Read `${CLAUDE_PLUGIN_ROOT}/references/safety.md`, `logging.md` and `~/.claude/linkedin-outreach-desk/config.json` first. This job reads the feed and profiles and sends connection requests ONLY. It never messages, follows, reacts, endorses or accepts/declines invitations. Everything it targets comes from `config.prospecting`; never invent targeting rules, and if `prospecting` is missing stop and tell the owner to run `setup`.

## Config used
`prospecting.offer` (what the owner sells, one line), `prospecting.types[]` (each: `name`, `description`, `titleKeywords`, `locationRule` strict|prefer|any, `locations`), `prospecting.excludeKeywords`, `prospecting.perCompanyCap` (default 2), `prospecting.requestsPerRun` ({min,max}, default 5 to 8), `prospecting.maxPerDay` (default 10), `prospecting.note` (default empty = blank request).

## Step 0 - Day cap
ArtifactData `get` `state/connect` on the relay artifact (`artifacts.relay`): `{date, sentToday, companies:{}}`. If `date` is not today in `owner.timezone`, reset. If `sentToday >= maxPerDay`, log and finish. This run may send at most `min(requestsPerRun.max, maxPerDay - sentToday)`.

## Step 1 - Browse for candidates
1. Open https://www.linkedin.com/feed/ in your own tab; apply safety rules 2-3.
2. Scroll the feed at a human pace and read post authors' name, headline, company, degree. Candidates must be 2nd or 3rd degree. Skip existing 1st-degree connections, pending invitations ("Pending"), company pages, promoted content and posts with no clear individual author.
3. Open a promising person's profile and use "People similar to X" / "More profiles for you" as an additional source. Never use LinkedIn search.
4. Bound the browse: roughly 15-20 minutes of human-style effort, then stop.

## Step 2 - Qualify
A person qualifies only if they match one of `prospecting.types` by title/headline/company AND pass that type's location rule (read the location from the profile; if a strict-location type's location is not clearly inside `locations`, skip) AND match no `excludeKeywords`.
- `perCompanyCap`: no more than that many requests to the same employer per run (also check `state/connect.companies` for today). After the cap, move to a fresh seed profile or the feed.
- Prefer, between similarly strong matches, people whose location is in a type's `locations`.
- Fewer than `requestsPerRun.min` qualified is fine: send fewer and say why. Never pad with weak matches.

## Step 3 - Send
For each qualified person, one at a time:
1. Use the feed card's Connect button, or on the profile click Connect (sometimes under "More"). If a "Add a note" dialog appears, click "Send without a note" (or add `prospecting.note` verbatim if one is configured).
2. Take a screenshot or re-read the page to confirm the state changed to "Pending". If unclear, reload and check; never click Connect twice for the same person.
3. Wait 20-60 s (vary it) before the next profile. Close any tab you opened when done.
Stop the whole run on a rate limit, "you've reached the weekly invitation limit", restriction, captcha or "unusual activity" notice. Never try to bypass it.

## Step 4 - Record and report
Update `state/connect` (`sentToday`, `companies`) with `if_version`. Do NOT write the people to the voice-note tracker; the `voice-notes` routine learns about them when they accept.
Report (final message and, if `delivery.push`, PushNotification only on problems or when `delivery.reportEveryRun` is true): who was connected (name, role, company, location, type, one-line reason), companies that hit the cap, and why fewer than the minimum if so. Log per `logging.md` with counts.

## Hard rules
No messages of any kind. No notes unless configured. Never connect to someone matching no type. Respect the day cap even if asked to continue. If logged out or challenged: close the tab, notify, log, finish.
