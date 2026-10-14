---
name: dm-send
description: Daily routine. Send every DM reply the owner approved or edited in DM Desk, in its own LinkedIn conversation, exactly as stored, then mark it sent.
---

# LinkedIn DMs: send approved replies

Read `${CLAUDE_PLUGIN_ROOT}/references/safety.md`, `schemas.md`, `logging.md` and the config first. The owner has authorised sending ONLY replies they approved in DM Desk; approving or saving an edit is approval of that exact text for that exact conversation.

## Step 1 - Queue
ArtifactData `query` `messages` where `status == "approved"` (limit 1000). Eligible: non-empty `replyText`, no `sentAt`. Order by `decidedAt` ascending; at most `limits.dmSendsPerRun`. None → log only, no notification.

## Step 2 - Send, one at a time
Open https://www.linkedin.com/messaging/ (safety rules 2-3). For each doc:
a. `get` it again; if no longer `approved` or `sentAt` set, skip.
b. Navigate to `threadUrl`; confirm the header name equals `senderName`. Missing/mismatch → `status:"failed"`, `failedAt`, `error:"Conversation not found"`.
c. Compare the thread with the doc:
   - Same text from the owner already present → mark `sent` with `note:"already present"`, do not resend.
   - Owner sent any other message after `lastInboundText` → `status:"skipped"`, `note:"You replied directly on LinkedIn"`.
   - The other person wrote again after `lastInboundText` → do NOT send unless the doc has `failedAt` and `decidedAt` is later (re-approved); else `failed` with `error:"<first name> sent a new message after this draft. Open the conversation, then Approve again to send anyway or Edit."`.
c2. Calendar (if the doc has `meetingSlots`, or `bookingRequest` with `bookMeeting` not false; check slots with the public endpoint described in dm-check Step 2c; book with `mcp__cal-com__createBooking` loaded via ToolSearch `+cal-com`, args `{"params":{"cal-api-version":"2024-08-13"},"body":{start (UTC), eventTypeId, attendee{name,email,timeZone}, location}}`, location `{"type":"integration","integration":"google-meet"}` or `{"type":"attendeeAddress","address":venue}`; if either is unavailable mark these docs `failed` "Calendar unavailable, could not check the times"): every offered slot must still be free or `failed` "<label> is no longer free in your calendar. Edit the times, then approve again.". For a booking: if `bookingUid` set, never book again; else re-check the slot, then `createBooking` (shape above; in-person with no venue → `failed` "No coffee venue set"). On success `update` `bookingUid`, `bookedAt` immediately, then send. Booking error → do NOT send, `failed` "Calendar booking failed: <reason>". Approving a card with a bookingRequest is the owner's approval of that one booking.
d. Click the message box, type `replyText` exactly (shift+Enter for newlines, never plain Enter), read the box back and compare; retype once if different; if still different clear it and mark `failed` (`"Message box would not match text"`). No attachments, GIFs or emoji.
e. Click Send ONCE. f. Confirm it appears as sent by the owner (reload to check if unclear). Then `update` with `if_version`: `status:"sent"`, `sentAt`, `error:null`. g. Other failure → `failed` + short `error`; rate-limit warning → STOP the run. h. Wait 30-60 s.
Close the tab. Update `meta/status` (`lastSentAt`, `lastSendSummary`). PushNotification if anything was sent or held back.

## Step 3 - Log (per `logging.md`)
Hard rules: only `approved` docs, one message per doc ever, never start new conversations or message anyone else. At most one Cal.com booking per doc; never cancel, reschedule or edit other bookings or event types.
