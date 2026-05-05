# Daily Donor Engagement Digest

The cron channel fires once on its schedule (8am Pacific,
Monday through Friday). There is no payload to parse — your
job is to run the digest workflow and post the result to Slack.

## Steps

1. Follow the **Daily Donor Workflow** in SOUL.md (Phases 1–3):
   pull yesterday's donor campaigns from Loops, compute opens /
   clicks / replies, identify the top 5 "warming up" donors
   (and any "cooled off" candidates), and compose the digest
   message.
2. Resolve target channels per the SOUL **Where to post**
   section: list every channel the bot is a member of and post
   once to each. If the bot is in zero channels, DM the
   workspace install user instead with the digest and a
   one-line invite hint.
3. Post exactly once per resolved destination. Do not retry on
   failure — log the error in your session and continue with
   the remaining destinations. The next cron fire is the
   recovery.
4. Do not send any follow-ups, reactions, or thread replies
   after the initial post. Your turn ends after the posts
   complete.

## Skip conditions

Skip posting (and stop silently) if any of these are true:

- No donor campaigns were sent in the last 24 hours AND no
  donors qualify as "warming up" or "cooled off" in the
  trailing window. Nothing meaningful to report.
- The Loops API is unreachable or returns an auth error — log
  it and stop. The next fire is the recovery.

If a campaign was sent but no donors qualify as warming, post
the campaign metrics line on its own — that is still useful
information for the development team.
