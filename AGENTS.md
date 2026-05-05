This folder contains the source for a Skilled Agent originally built for the Valet runtime. Changes should follow the Skilled Agent open standard.

## Setup

### Connectors

- **loops** (CLI): The Loops command-line tool, available on PATH as `loops`. The agent uses it to list donor campaigns, pull aggregate metrics (sent / opens / clicks / replies), and read per-contact engagement events. Writes (sending a transactional email, adding or removing a tag) only happen when a Slack user explicitly asks and confirms. See `skills/loops/SKILL.md` for invocation rules.

### Channels

- **slack** (slack): The agent's per-agent Slack bot. Listens for @mentions and replies in-thread, and posts the daily donor engagement digest to whichever channels the bot has been invited to. Slack writes use the auto-injected outbound Slack connector.
- **cron** (cron): Fires the daily donor engagement digest at 8am Pacific, Monday through Friday (`0 8 * * 1-5`, `America/Los_Angeles`). Declared inline in `valet.yaml`, so it's created automatically by the dashboard setup flow.

### Secrets

- `LOOPS_API_KEY`: Loops API key, scoped to the workspace whose donor list you want the agent to read. Generate it in Loops → Settings → API. Read access is sufficient for the digest and Q&A; transactional-send permission is required only if you want the agent to be able to send thank-you emails on confirmation. Set at the org level so other Loops-powered agents can share the connector.

### External Setup

1. After deploy, invite the agent's Slack bot to whichever channel(s) you want the daily donor digest in. The agent posts the digest to every channel it's a member of — invite it to one focused fundraising channel, or several. If the bot has not been invited anywhere, the digest is sent as a DM to the workspace install user with a one-line nudge to invite it somewhere.
2. Invite the bot to any additional channels where teammates should be able to @mention it for ad-hoc donor questions (e.g. a development team channel for "how's the year-end appeal performing?" follow-ups).
3. The first cron fire is the next 8am Pacific weekday after deploy. To smoke-test sooner, @mention the bot in Slack with a question like *"who's warmed up most this week?"* — that exercises the Slack + Loops path without waiting for the cron.

## Customizing

- **Change the schedule**: edit the `cron` and `timezone` on the `cron` channel in `valet.yaml`, then redeploy. The default is `0 8 * * 1-5` `America/Los_Angeles` (8am PT, Monday through Friday).
- **Tune what counts as "warming up"**: the rule lives in `SOUL.md` → *Daily Donor Workflow → Phase 1*. Default is "opened the last 3+ emails OR clicked in the last 7 days OR replied in the last 14 days." Edit the thresholds (window length, minimum open streak, click recency) to match your fundraising rhythm — a quarterly appeals shop will want a longer window than a weekly newsletter.
- **Control where the digest posts**: invite or remove the bot from channels in Slack — that's the only signal the agent uses. There is no channel name in the configuration.
- **Limit which list/segment the agent watches**: if you want the agent scoped to a specific donor segment (not all contacts), set a `LOOPS_DONOR_SEGMENT` env var on the agent with the segment name or ID and reference it in your `loops` queries.
