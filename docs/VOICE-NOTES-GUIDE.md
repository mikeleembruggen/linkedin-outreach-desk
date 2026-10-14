# Voice notes guide

Voice notes are short audio messages you send in a LinkedIn conversation. In this system they are the personal touch that follows a blank connection request. The system finds who deserves one and reminds you; **you** record and send it.

## Why a human sends them

- LinkedIn only allows recording and sending voice notes from its **mobile app** (iOS / Android). There is no desktop or API route.
- Nothing in the plugin sends, records or generates voice. Do not use synthetic voices; the value is that it is really you.

## Prerequisites

- LinkedIn mobile app, signed in to the same account, with microphone permission.
- The `voice-notes` cloud routine enabled and the Gmail connector on (new connections are detected from LinkedIn's "accepted your invitation" emails).
- Push notifications from the Claude app on your phone, or the optional daily email.

## Daily flow

1. **You choose the time** you will sit down and send voice notes during setup. The routine runs ~15 minutes before it.
2. **You receive one message:** "Voice notes today: Sam, Priya, ..." with a line per person (name, role, location) and a link.
3. **Tap a link.** It opens a relay page that records the click (so they are not suggested again) and forwards you to their LinkedIn profile.
4. **Open the message thread** in the LinkedIn mobile app and **hold the microphone icon** to record; release to send. Keep it short; LinkedIn caps the length at about a minute.
5. **Done.** Opening the link already excludes them from future picks. You can also tap "I sent a voice note" on their DM Desk card, and `dm-check` marks threads where it sees an audio clip from you.

Picks with no reply after being suggested three times (configurable) are dropped from the list.

## What to say (template)

Aim for 15-30 seconds.

1. **Hello and why now (3-5 s):** "Hi <First>, it's <You>. Thanks for connecting."
2. **Specific hook (5-10 s):** one thing you actually noticed: their role, a post, a project, a shared connection. This is the talking point in the nudge (`voiceNoteIdea`).
3. **Relevance (5-10 s):** why you'd be worth knowing, in one plain sentence. No pitch.
4. **Soft close (3-5 s):** a low-pressure question: "Curious what you're working on at the moment."

Rules of thumb:
- Do not promise jobs, rates, introductions or meetings in the first note.
- No selling. The goal is a reply.
- Speak at normal pace; don't read a script.
- Say their name; it is what makes it feel personal.

## Customising it

All set during `setup` (and editable by re-running it).

| Setting | What it controls |
|---|---|
| `sendTime` | When you receive the list (routine runs 15 min earlier) |
| `count` | People per day (3-5 recommended) |
| `priorityLocations` | Only people in these places are picked; empty = no filter. You choose these |
| `priorityTitles` | Titles ranked first (founder, CEO, director, owner...) |
| `excludeKeywords` | Titles to skip |
| `perCompanyCap` | Max from one employer per day |
| `maxTimesRecommended` | Stop suggesting someone after this many days without a click |
| Delivery | Push, optional email to you, optional Slack |

## State (kept in your relay artifact)

Per person: slug, name, title, connected date, location (looked up once and cached), whether a voice note was sent, times suggested, last suggested. Clicks are logged separately and permanently exclude a person.

## Reply handling after a voice note

When someone replies, the next `dm-check` drafts a response in your voice (and, if they want to talk, offers meeting times). See METHODOLOGY.md stage 4-5.

## Troubleshooting

| Symptom | Fix |
|---|---|
| No list today | No new connections, or none in your priority locations. Widen the locations or wait |
| Someone appears twice | They were not detected as sent; press "I sent a voice note" on DM Desk |
| Voice note not recognised as sent | Tap the profile link or "I sent a voice note" on the DM Desk card |
| Someone already contacted appears | Open their link once (logs a click) or mark them sent on DM Desk |
| No notification | Claude app push permission, routine enabled, Gmail connector connected |
| LinkedIn won't let me record | Update the app and check microphone permission; recording is mobile-only |

## Don'ts

- Don't send the same note to many people.
- Don't send voice notes to people who have already said no or ignored you.
- Don't send more than you can sustain; 3-5 a day is plenty.
