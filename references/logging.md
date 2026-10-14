# Run logging (every run, even when nothing happened)

Append ONE line to `~/.claude/linkedin-outreach-desk/run-log.md` (create if missing): `YYYY-MM-DD HH:MM <skill> <outcome + numbers>` in `owner.timezone`. Cloud routines that cannot write local files put the same line in `meta/status.lastRunSummary` of their artifact instead.

If the run needs the owner (logged out, captcha, rate-limit warning, anything unexpected), also PushNotification a short message (e.g. "LinkedIn is signed out in Chrome, <skill> skipped") and add a line starting `NEEDS OWNER:` at the top of `run-log.md` stating the exact step.

Never write passwords, tokens or message bodies (beyond a few words) into logs.
