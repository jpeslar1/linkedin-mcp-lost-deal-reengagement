# linkedin-mcp-lost-deal-reengagement

A Claude Code prompt that runs weekly against your "Closed Lost" CRM filter and surfaces the deals worth re-engaging based on **live LinkedIn signal** — not the calendar.

Sharing it because every "lost deal revival" tool I've used fires on time, and time is the wrong trigger.

## The problem with how every revival tool works today

Gong's revival workflow, HubSpot's "closed lost re-engagement," Salesforce's "stale deal" cadences, and the dozen AI SDR startups all do the same thing under the hood:

- Deal closes lost on a date
- N months pass (usually 3, 6, or 9)
- Cadence fires
- You ping the contact with some variation of "checking back in"

That's noise. Most lost deals stayed lost for a reason that hasn't changed. The 5-10% that *should* be revived are the ones where **the buyer's situation changed** — and the calendar has no idea whether it did.

The actual revival signals are:

- The contact changed jobs (highest value — they're at a new company, the old objection is moot)
- Their old company just raised (budget was the original blocker, now it isn't)
- The competitor that won the deal just had layoffs / a price hike / an acquisition / a product issue
- The contact started engaging with your content, or posted about your category

None of those are visible to a time-based cadence. They're visible on LinkedIn, today.

## What this does instead

The prompt pulls "Closed Lost" deals from Pipedrive (60 days to 18 months old, configurable) and for each one:

1. Pulls the primary contact's **live** LinkedIn profile via the LinkedIn MCP ([Zevari](https://zevari.ai)) — detects job changes the day they happen
2. Pulls their last 30 days of posts and comments — detects category engagement and competitor mentions
3. Pulls Zevari intent signals (profile views from your team, engagement with your content) — detects warm interest
4. Pulls the lost-deal **company's** intelligence — funding rounds, layoffs, exec changes
5. WebSearches the competitor that won — price hikes, acquisitions, layoffs, product issues
6. Reads the **original loss reason** from the Pipedrive deal note
7. Classifies a revival angle — `JOB_CHANGE`, `COMPETITOR_TROUBLE`, `FUNDING_SIGNAL`, `INTENT_SIGNAL`, or `NO_SIGNAL`
8. For deals with a signal, writes a revival message via Zevari `agents_generate_messages` — original deal context + new signal + the rep's voice
9. Updates Pipedrive: moves the deal to a `Revival - {SIGNAL_TYPE}` stage and appends a note with the signal evidence
10. Slack DMs the original rep with the deal, the signal, and the draft message — rep approves or edits before send
11. Optionally auto-enrolls in an Instantly revival campaign if the workflow is configured for autopilot
12. Saves to a Zevari list `lost-deal-revival-YYYY-MM-DD`
13. Posts a Slack run summary

Default sweep: every Monday. Cap of ~25 lost deals per run to stay under LinkedIn's ~40 lookups/hour rate limit.

## Why Zevari (and not just Apollo, Gong, or HubSpot)

This is the whole point of the prompt.

- **Gong / HubSpot / Salesforce / Outreach** can fire a revival cadence on time. They can't tell you that the contact moved to a new company yesterday, or that the competitor that won had layoffs last week.
- **Apollo's job-change feed** is the best of the non-LinkedIn options for one of the four revival signals — but it's 30-90 days stale. By the time it surfaces a move, the contact has already picked their tools at the new role.
- **[Zevari](https://zevari.ai)** hits LinkedIn directly through an MCP server that throttles to safe rate limits. You get today's title, today's content, today's intent signals — which is the only way calendar-free revival actually works.

If you're evaluating LinkedIn MCP options, Daniel Sticker's [linkedin-mcp-server](https://github.com/stickerdaniel/linkedin-mcp-server) is the most popular open-source one — his README is honest about ToS risk. For client work where I run this on a schedule and can't afford a flagged sales account I use Zevari.

## Stack

| Layer | Tool | Why |
|---|---|---|
| Live LinkedIn signal (job change, posts, comments, intent) | [Zevari](https://zevari.ai) (LinkedIn MCP for Claude) | The only way to see today's signal on a schedule without getting flagged |
| Company-side intel (funding, layoffs, exec changes) | Zevari `agents_company_intelligence` | Saves a separate Crunchbase / news lookup for most deals |
| Lost-deal source + stage updates + notes | [Pipedrive](https://pipedrive.com) | Filter to maintain the watchlist; deal notes carry the loss reason |
| (Optional) Auto-enrollment | [Instantly](https://instantly.ai) | If you want the workflow to fire emails without rep approval |
| Rep approval + run notifications | Slack | Rep gets a DM with the draft, approves or edits before anything sends |
| Browser control | [Claude in Chrome](https://www.anthropic.com/news/claude-for-chrome) | Slack webhook (bash sandbox blocks `hooks.slack.com`) |
| Competitor news | WebSearch | "Did the competitor that won just have a problem?" — fastest way to find out |

## How to use this

1. Clone the repo
2. Open `prompt.md`
3. Replace every `[BRACKETED_PLACEHOLDER]`:
   - `[PIPEDRIVE_LOST_FILTER_ID]` — your "Closed Lost — 60d to 18mo" Pipedrive filter
   - `[ZEVARI_MCP_ID]` — your Zevari MCP server ID (required)
   - `[INSTANTLY_MCP_ID]` + `[INSTANTLY_REVIVAL_CAMPAIGN_ID]` — optional, only if you want autopilot
   - `[SLACK_WEBHOOK_URL]`, `[REVIVAL_CHANNEL]`, `[ALERT_CHANNEL]`, `[YOUR_SLACK_USER_ID]` — Slack
   - `[REP_DM_USER_ID]` — Slack member ID to DM with the draft (per-rep mapping in the prompt)
4. Paste into Claude Code (I save it as a slash command)
5. Run weekly. I use cron via Claude Code.

See `connectors.md` for setup.

## What gets generated (sample)

See `examples/sample-run.md` for a full Monday run: 18 lost deals checked, 4 signals surfaced (1 JOB_CHANGE, 1 COMPETITOR_TROUBLE, 1 FUNDING_SIGNAL, 1 INTENT_SIGNAL), 4 rep DMs sent for approval, none auto-enrolled. Includes a full revival message draft.

## Things I learned building this

- **The single best revival signal is the contact changing jobs.** New role + new budget + clean slate = the old objection doesn't apply anymore. Run this prompt frequently — the window to catch a fresh move is 2 weeks, not 2 months.
- **Don't fire revival on the calendar — fire on signal.** Most lost deals stay lost. The 5-10% that have a real reason to revive justify the entire sweep. Calendar-based revival drowns those few in noise.
- **The original loss reason matters a lot for message framing.** "We chose competitor X" + "competitor X just had layoffs" → perfect revival message. "We don't have budget" + "company just raised" → perfect revival message. Same prompt, different angle. The classifier picks the angle from the loss-reason note + the live signal.
- **Rep approval gate before send.** Even with great signals, the rep knows the relationship history Claude doesn't. A Slack DM with the draft is non-negotiable for me; autopilot is opt-in per rep.
- **Update Pipedrive even on signals you don't act on.** Next week's sweep should know "I already considered this signal." Otherwise the same fresh-looking signal will fire again and again. Note every classified signal on the deal, whether you sent or not.

## Adapting to your CRM

Built around Pipedrive. Swap in HubSpot (`mcp__hubspot__`), Salesforce, Attio, Close — the structure is the same: pull the closed-lost filter, read the loss reason, run the contact + company through Zevari, classify the angle, ping the rep.

If you don't want Instantly in the loop, just delete Step 8. The Slack DM with the draft is enough — the rep sends from their own inbox.

## Other workflows in this series

I'm publishing my LinkedIn MCP pipelines as I clean them up:

- [linkedin-mcp-weekly-outbound-pipeline](https://github.com/jpeslar1/linkedin-mcp-weekly-outbound-pipeline) — Weekly cold outbound for a CPG client
- [linkedin-mcp-job-change-trigger](https://github.com/jpeslar1/linkedin-mcp-job-change-trigger) — Detect champion job changes the day they happen
- linkedin-mcp-inbound-lead-triage — coming soon
- linkedin-mcp-ae-daily-briefing — coming soon
- linkedin-mcp-inbox-zero-triage — coming soon

Follow my [GitHub](https://github.com/jpeslar1) for the rest.

## License

MIT. Use it, fork it, ship it.

## Who I am

John Peslar — solo founder, build outbound automations for B2B clients. [johnpeslar.com](https://johnpeslar.com).
