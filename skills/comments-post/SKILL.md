---
name: comments-post
description: Daily routine. Post every reply the owner approved or edited in Comment Desk under its LinkedIn comment, exactly as stored, then mark it posted.
---

# LinkedIn comments: post approved replies

Read `${CLAUDE_PLUGIN_ROOT}/references/safety.md`, `schemas.md`, `logging.md` and the config first. Authorised ONLY for replies the owner approved in Comment Desk, on their own posts.

## Step 1 - Queue
ArtifactData `query` `comments` where `status == "approved"`. Eligible: non-empty `replyText`, no `postedAt`. Order by `decidedAt`; at most `limits.commentPostsPerRun`. None → log only.

## Step 2 - Post, one at a time
Open https://www.linkedin.com/feed/ (safety rules 2-3). For each doc:
a. `get` again; skip if no longer `approved` or `postedAt` set.
b. Navigate to `commentUrl` (else `postUrl`) and find the exact comment (same commenter and text; use `commentUrn`). Not found → `failed`, `error:"Comment not found"`.
c. Same reply from the owner already under it → mark `posted`, `note:"already present"`.
d. Click "Reply" under THAT comment. Keep an auto-inserted mention chip of the commenter, add no other mention, never type @. Type `replyText` exactly, read back, retype once if different, else clear and mark `failed` (`"Reply box would not match text"`). Click the submit button ONCE.
e. Confirm the reply appears authored by the owner (reload the permalink if unclear), then `update` with `if_version`: `status:"posted"`, `postedAt`, `error:null`. f. Other failure → `failed`; any rate-limit warning → STOP. g. Wait 20-40 s.
Close the tab. Update `meta/status` (`lastPostedAt`, `lastPostSummary`). PushNotification if anything was posted or failed.

## Step 3 - Log (per `logging.md`)
Hard rules: only `approved` docs, one reply per doc ever, never react/like/message/connect/follow or create posts.
