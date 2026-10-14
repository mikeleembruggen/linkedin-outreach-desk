# Dashboard database schemas (ArtifactData)

Each dashboard artifact owns a small db. All ids/URLs come from config `artifacts.*`.

## DM Desk: collection `messages`, doc `meta/status`
Doc id `m-<threadId>-` + first 12 hex of sha1(lastInboundText) (`printf %s "..." | shasum`); strip chars outside [A-Za-z0-9_-] from threadId, cap 120.
Fields: `threadId`, `threadUrl`, `senderName`, `senderHeadline`, `senderUrl`, `profileUrl` (vanity /in/slug/), `messages` (last <=6 as `{from:"them"|"me",name,text,at}`), `lastInboundText`, `lastInboundAt`, `inboundCount`, `category` (candidate|client|recruiter|network|sales pitch|sponsored|personal|other), `suggestedReply`, `replyText`, `aiNote`, `status` (pending → approved|skipped → sent|failed; `superseded`), `edited`, `fetchedAt`, `decidedAt`, `sentAt`, `failedAt`, `error`, `note`, `voiceNote` (sent|not_sent), `voiceNoteDate`, `voiceNoteSource` (tracker|thread|dashboard), `voiceNoteIdea`.
`meta/status`: `lastCheckedAt`, `lastCheckSummary`, `lastSentAt`, `lastSendSummary`.

DM meeting fields: `meetingIntent` (`none`|`offer`|`book`|`ask_details`), `meetingFormat` (`coffee_or_meet`|`google_meet`), `meetingSlots` ([{start ISO UTC, label}] offered in the reply), `bookingLink`, `bookingRequest` ({start, label, attendeeName, attendeeEmail, timeZone, eventTypeId, location `google_meet`|`in_person`, venue}), `bookMeeting` (bool, dashboard toggle), `bookingUid`, `bookedAt` (set by dm-send after create_booking).

## Comment Desk: collection `comments`, doc `meta/status`
Doc id `c-<activityId>-<commentId>` from the comment URN (digits); fallback `h-<activityId>-` + sha1(commenterName+commentText)[:12].
Fields: `postUrl`, `postSnippet`, `postId`, `commenterName`, `commenterHeadline`, `commenterUrl`, `commentText`, `commentUrl`, `commentUrn`, `commentedAt`, `suggestedReply`, `replyText`, `aiNote`, `status` (pending → approved|skipped → posted|failed), `edited`, `fetchedAt`, `decidedAt`, `postedAt`, `error`, `note`.
`meta/status`: `lastCheckedAt`, `lastCheckSummary`, `lastPostedAt`, `lastPostSummary`.

## Relay artifact (`artifacts.relay`)
- `state/tracker`: `{updatedAt, rows:[{slug,name,title,connected (YYYY-MM-DD),voiceNoteSent (bool),location,timesRecommended (int),lastRecommended}]}`. Only the `voice-notes` skill writes it.
- `state/connect`: `{date, sentToday, companies:{<company>:count}}`. Only `connect` writes it.
- `clicks/<slug>`: `{url, firstClickedAt, lastClickedAt, count}`, written by the relay page when the owner opens a relay link.
- `config`: copy of the client's config for cloud routines.
- `meta/status`: `lastRunSummary`.
