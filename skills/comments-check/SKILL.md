---
name: comments-check
description: Daily routine. Read new comments on the owner's own recent LinkedIn posts and draft a suggested reply for each into Comment Desk. Read-only on LinkedIn.
---

# LinkedIn comments: check and draft replies

Read `${CLAUDE_PLUGIN_ROOT}/references/safety.md`, `schemas.md`, `logging.md` and the config first. READ-ONLY on LinkedIn: never comment, reply, react, message, connect or follow.

Start: ArtifactData `list` `comments` (limit 1000) to know existing ids. Never modify existing docs; only create new ones.

## Step 1 - Find recent posts
Open https://www.linkedin.com/in/me/recent-activity/all/ (safety rules 2-3). Collect the owner's OWN posts (not reposts or comments elsewhere) from the last `limits.commentLookbackDays` days that show comments. Get each URL (`/feed/update/urn:li:activity:<id>/`). Cap 15 posts.

## Step 2 - Read comments
Open each post, expand all comments and replies, and extract per comment: name, headline, profile URL, text, relative time, comment URN (`data-id`, `urn:li:comment:(...)`). A comment NEEDS A REPLY only if: written by someone else; the owner hasn't replied in that thread after it; its doc id is new; within the lookback window. Replies by others inside a thread are comments in their own right.

## Step 3 - Draft and save
`suggestedReply` per `owner.about`, `owner.english`, `voice.commentLength`: respond to the specific point, add a small insight or genuine question, first name at the start when natural (plain text, no @). No hashtags, emoji, links, pitch, "Great post!" filler, promises of jobs/meetings/calls, or unsure facts. Spam, hostile, bare "agreed/congrats", or needs a personal/factual answer: draft the best short reply (empty for pure spam) and set `aiNote` ("Looks like spam, suggest Skip", "Asks about a specific role, check before approving").
New docs `status:"pending"`, `edited:false`, `replyText = suggestedReply`. One ArtifactData `batch` of `set` ops (50 per batch), then update `meta/status` (`lastCheckedAt`, `lastCheckSummary`). Close the tab. PushNotification if new: "N new LinkedIn comments with suggested replies ready to review" plus the first commenter.

## Step 4 - Log (per `logging.md`)
