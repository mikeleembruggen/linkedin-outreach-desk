# Configuration

All skills read one file: `~/.claude/linkedin-outreach-desk/config.json`, written by the `setup` skill from the client's own answers. Nothing client-specific (names, places, emails, links, ids) is hard-coded in any skill. If the file is missing, stop and tell the user to run `setup`.

Cloud routines cannot read local files, so `setup` also stores the same config as a `config` doc in the relay artifact db; skills that run in the cloud read that copy.

```json
{
  "owner": {
    "name": "Full name exactly as on LinkedIn",
    "firstName": "First",
    "email": "own address, used only for the optional daily email",
    "timezone": "IANA zone, e.g. Europe/London",
    "english": "e.g. British English",
    "about": "1-2 sentences: what you do and who you talk to. Grounds reply voice."
  },
  "prospecting": {
    "offer": "One line: what you sell or do",
    "types": [
      { "name": "Peers", "description": "...", "titleKeywords": ["..."], "locationRule": "strict|prefer|any", "locations": ["Country or city"] },
      { "name": "Buyers", "description": "...", "titleKeywords": ["..."], "locationRule": "any", "locations": [] }
    ],
    "excludeKeywords": ["competitors, students, vendors..."],
    "perCompanyCap": 2,
    "requestsPerRun": { "min": 5, "max": 8 },
    "maxPerDay": 10,
    "note": ""
  },
  "voiceNotes": {
    "sendTime": "HH:MM local, the time you will sit down and send voice notes",
    "count": 5,
    "priorityLocations": ["cities or regions; empty = no filter"],
    "priorityTitles": ["founder", "ceo", "director", "owner", "head of"],
    "excludeKeywords": [],
    "maxTimesRecommended": 3,
    "perCompanyCap": 2
  },
  "voice": {
    "dmLength": "1-4 short sentences",
    "commentLength": "1-3 short sentences, usually under 280 characters",
    "samples": ["5-10 messages written by the client, as a style reference"],
    "extraRules": ["No hashtags", "No emoji", "No links"]
  },
  "meetings": {
    "mode": "calcom | link-only | off",
    "bookingLink": "client's booking URL (any service)",
    "calcom": { "eventTypeId": 0, "lengthMinutes": 30, "mcpServer": "cal-com" },
    "slotsToOffer": 2,
    "format": "coffee_or_video | video_only | coffee_only | call",
    "inPersonAreas": ["cities where you offer coffee"],
    "offerHours": "e.g. Mon-Fri 9:00-16:30 owner time, not today, next 7 days"
  },
  "artifacts": {
    "relay": "https://claude.ai/artifact/... (created by setup)",
    "dmDesk": "https://claude.ai/artifact/... (created by setup)",
    "commentDesk": "https://claude.ai/artifact/... (created by setup)"
  },
  "delivery": {
    "push": true,
    "dailyEmail": false,
    "reportEveryRun": false,
    "slack": { "enabled": false, "channelId": "" }
  },
  "limits": {
    "dmDraftsPerRun": 25,
    "dmSendsPerRun": 20,
    "commentPostsPerRun": 20,
    "commentLookbackDays": 14
  },
  "schedule": {
    "host": "cowork-routine | desktop-scheduled",
    "times": { "connect": ["09:00", "15:00"], "commentsCheck": "07:15", "dmCheck": "08:15", "voiceNotes": "same as voiceNotes.sendTime minus 15 min", "dmSend": "13:00", "commentsPost": "17:30" }
  }
}
```

Rules:
- Every value comes from the client during `setup`. The plugin ships with no defaults for names, places, links or ids; the numbers shown are suggested starting points.
- `delivery.dailyEmail` only ever emails `owner.email`. `delivery.slack` is optional and off by default; when enabled the daily voice-note message and connect report are also posted to that channel.
- Timestamps in logs use `owner.timezone`.
