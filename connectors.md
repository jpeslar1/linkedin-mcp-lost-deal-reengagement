# Connectors

Every MCP server this prompt uses, with setup and the placeholders to replace.

## Zevari — LinkedIn MCP for Claude (REQUIRED, the core of the prompt)

**What it does:** Live LinkedIn profile lookup (`linkedin_get_profile`), recent posts and comments (`linkedin_get_user_posts`, `linkedin_get_user_comments`), intent signals (`signals_list`), company intelligence (`agents_company_intelligence`), revival message generation (`agents_generate_messages`), target save (`targets_save`).

**Why it's the core of this workflow:** Every revival tool can fire on the calendar. None of them can tell you the contact moved jobs yesterday, or that they commented on a competitor-pricing post last week, or that someone from your team's domain just viewed their profile. Those signals only exist on LinkedIn, and Zevari is the only way to query LinkedIn safely on a schedule.

**Setup:** [zevari.ai](https://zevari.ai) → connect LinkedIn → copy the MCP server URL into your Claude Code config (`~/.claude/mcp.json` or via `claude mcp add`).

**Placeholders:**
- `[ZEVARI_MCP_ID]` — your Zevari MCP server ID (looks like `mcp__a1b2c3d4-...`)

**Endpoints used:**
- `zevari_context_get` / `zevari_context_confirm` — confirm active org + LinkedIn account
- `linkedin_get_profile` — current title + company (job change detection)
- `linkedin_get_user_posts` — last 30 days of posts (category engagement)
- `linkedin_get_user_comments` — last 30 days of comments (same intent question)
- `signals_list` — intent signals (profile views, content engagement)
- `agents_company_intelligence` — company-side signals (funding, layoffs, exec changes)
- `profile_get` — current ICP rules + category keywords for the post scanner
- `agents_generate_messages` — draft the revival message in the rep's voice
- `targets_save` + `targets_create_list` — save the contact and build the weekly revival list
- `library_context_get` (`revival-outreach` or `craft-outreach`) — voice rules and formatting

## Pipedrive — Lost-deal source + stage updates (REQUIRED)

**What it does:** Sources the closed-lost watchlist via a filter. Receives stage updates (`Revival - {SIGNAL_TYPE}`) and audit notes after each sweep.

**Setup:** Any Pipedrive MCP server. I use `mcp__pipedrive__` from the community list.

**You need to set up before running:**

1. A Pipedrive filter that returns closed-lost deals 60 days to 18 months old. Save the filter ID — that's `[PIPEDRIVE_LOST_FILTER_ID]`.
   - Status = `lost`
   - Lost time ≥ 60 days
   - Lost time ≤ 18 months
   - Primary contact has LinkedIn URL field populated
2. Four new pipeline stages (in addition to your existing Lost stage):
   - `Revival - Job Change`
   - `Revival - Competitor Trouble`
   - `Revival - Funding Signal`
   - `Revival - Intent Signal`
3. A `linkedin_url` custom field on contacts (most teams have this already).
4. The `lost_reason` field actively used by reps on every closed-lost deal. If reps aren't filling it in, the classifier has nothing to compare signals against — fix that first.

**Placeholders:**
- `[PIPEDRIVE_LOST_FILTER_ID]` — your closed-lost filter

**Endpoints used:**
- `list_deals` / `search_deals` — pull the filter
- `get_deal` — get full deal context including notes
- `update_deal` — move to revival stage
- `add_note` — append the signal-evidence note
- `update_person` — write back new title on `JOB_CHANGE`
- `update_organization` / `create_organization` — record new company on `JOB_CHANGE`

## Instantly — Optional autopilot enrollment

**What it does:** If a rep opts into autopilot for safe signal types (`JOB_CHANGE`, `FUNDING_SIGNAL`), the workflow auto-enrolls the contact in a revival cold sequence instead of waiting for Slack approval.

**Why it's optional:** Default for new reps is DM-first. You only flip autopilot ON once you trust the classifier and the message drafts for that rep's voice. Most revival messages benefit from rep review — the rep knows the relationship history Claude doesn't.

**Setup:** [Instantly MCP docs](https://developer.instantly.ai/mcp). Build a campaign called something like "Lost Deal Revival" with 2-3 emails: the revival message (passed in as a custom variable), a follow-up nudge, and a breakup. Get the campaign UUID.

**Placeholders:**
- `[INSTANTLY_MCP_ID]` — your Instantly MCP server ID
- `[INSTANTLY_REVIVAL_CAMPAIGN_ID]` — the revival campaign UUID

**Endpoints used:**
- `list_leads` — dedupe check before enrollment
- `add_leads_to_campaign_or_list_bulk` — enrollment

## Claude in Chrome — Slack webhook + per-rep DMs

**What it does:** Posts to `hooks.slack.com` for the run summary and DMs each rep individually with their drafts for approval. The bash sandbox blocks `hooks.slack.com` — Chrome is the only reliable way.

**Setup:** [Claude in Chrome](https://www.anthropic.com/news/claude-for-chrome).

**Tools used:**
- `mcp__Claude_in_Chrome__tabs_context_mcp` — open or find a tab
- `mcp__Claude_in_Chrome__javascript_tool` — fire the Slack webhook with the right `<@USER_ID>` ping

**Placeholders:**
- `[SLACK_WEBHOOK_URL]` — your incoming webhook
- `[REVIVAL_CHANNEL]` — channel where the weekly summary goes (cosmetic, for the prompt body)
- `[ALERT_CHANNEL]` — channel for errors / approval-required alerts
- `[YOUR_SLACK_USER_ID]` — your Slack member ID (`U...`)
- `[REP_DM_USER_ID]` — Slack member ID for the rep whose deal is being surfaced (mapped per-deal via the per-rep dictionary in `prompt.md`)

## WebSearch

Built into Claude Code. Used for competitor news in Step 4 — layoffs, price hikes, acquisitions, product issues. Also useful for confirming a funding announcement found in `agents_company_intelligence` is real and current.

No placeholder.
