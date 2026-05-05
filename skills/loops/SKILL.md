The Loops CLI is available on PATH as `loops`. It authenticates with the `LOOPS_API_KEY` environment variable, which the runtime sets for you — do not pass `--api-key` on the command line and never echo the key in your reply.

The agent uses Loops read-only by default: list campaigns, pull aggregate campaign metrics (sent / opens / clicks / replies / unsubs), and read per-contact engagement events for the donor list/segment. Write actions (sending a transactional email, adding or removing a tag on a contact) only happen when a Slack user explicitly asks and confirms.

─── Running commands non-interactively ──────────────────────────────────────────────────────────────

Always invoke the CLI with this shape:

  PAGER=cat loops <root-flags> <resource> <subcommand> <subcommand-flags>

Three rules that bite if you get them wrong:

1. Disable the pager. When stdout is a TTY, `loops` may pipe output
   through `$PAGER` (default `less`) and hang waiting for keypresses.
   Prefix every invocation with `PAGER=cat` (or pipe to `| cat`).

2. Root flags go BEFORE the resource, subcommand flags go AFTER.
   `--format`, `--api-key`, `--debug`, `--yes` / `-y` are root flags
   on `loops` itself — placing them after the subcommand makes them
   unknown flags.

3. Always request JSON. Default human-readable output is unstable
   and hard to parse. Use `--format json` (or `--format jsonl` for
   line-delimited) on every read.

Canonical example — list donor campaigns sent in the last 7 days as JSON:

  PAGER=cat loops --format json campaigns list --since 7d

For per-contact engagement events on a specific campaign:

  PAGER=cat loops --format json events list --campaign <campaign-id> --type open,click,reply

─── Resources the agent uses ────────────────────────────────────────────────────────────────────────

The exact subcommand surface depends on your installed `loops` version. Run `PAGER=cat loops --help` once at the start of a session to confirm the available resources, then use the patterns below.

- **`campaigns`** — list, view, and read aggregate metrics for sent campaigns.
  - List recent: `PAGER=cat loops --format json campaigns list --since 1d`
  - View one: `PAGER=cat loops --format json campaigns view <campaign-id>`
  - Metrics: `PAGER=cat loops --format json campaigns metrics <campaign-id>`
    Returns `sent`, `delivered`, `opens`, `unique_opens`, `clicks`,
    `unique_clicks`, `replies`, `unsubscribes`, `bounces`.
  // TODO: confirm exact subcommand names against your installed `loops` version.

- **`contacts`** — look up a donor by email, list contacts in a segment, read per-contact engagement.
  - Lookup: `PAGER=cat loops --format json contacts find --email <email>`
  - In segment: `PAGER=cat loops --format json contacts list --segment <segment-id-or-name>`
  - Engagement: `PAGER=cat loops --format json contacts events --email <email> --since 30d`

- **`events`** — read engagement events (opens, clicks, replies) across contacts and campaigns.
  - By campaign: `PAGER=cat loops --format json events list --campaign <campaign-id> --type open,click,reply`
  - By contact: `PAGER=cat loops --format json events list --email <email> --since 30d`

- **`transactional`** — send a one-off transactional email. **Write action — confirmation required.**
  - `PAGER=cat loops --format json transactional send --template <template-id> --email <email> --data '<json>'`

- **`tags`** / **`segments`** — add or remove a contact's membership in a segment or tag. **Write action — confirmation required.**
  - Add: `PAGER=cat loops --format json contacts tag add --email <email> --tag <tag-name>`
  - Remove: `PAGER=cat loops --format json contacts tag remove --email <email> --tag <tag-name>`

─── Output handling ─────────────────────────────────────────────────────────────────────────────────

- Always `--format json`. Parse the result programmatically; never paste raw JSON into Slack.
- For Slack messages, summarize: campaign name, sent / opens / clicks / replies as integers and percentages, then a bulleted list of warming donors with **redacted** emails (`m***@x.com`) and a one-line signal + date for each.
- Cap warming-donor lists at 5 for the daily digest and at 10 for ad-hoc Slack questions.

─── Auth ────────────────────────────────────────────────────────────────────────────────────────────

The CLI reads `LOOPS_API_KEY` from the environment. The runtime injects it. If a call returns 401 / unauthorized, stop, do not retry, and surface a one-line error to the operator (DM the install user; never to a public channel).
