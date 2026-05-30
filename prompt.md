# Lost Deal Re-Engagement — Weekly Signal Sweep

A Claude Code prompt that runs every Monday against the "Closed Lost" deals in Pipedrive (60 days to 18 months old), sweeps each one for **live LinkedIn signals** via the LinkedIn MCP (Zevari), classifies a revival angle, drafts a revival message, and pings the original rep for approval before anything sends.

The reason this prompt exists: every other lost-deal revival tool fires on the calendar — "ping every closed-lost deal 6 months after close." That's noise. Most lost deals stayed lost for a reason that hasn't changed. The 5-10% worth reviving are the ones where the buyer's situation changed (job change, funding, competitor trouble, content engagement) — and those signals only show up live on LinkedIn.

---

## Overview

You are running the weekly lost-deal revival sweep. For each Closed Lost deal in the configured Pipedrive filter:

1. Pull the deal + primary contact + original loss reason from Pipedrive.
2. Run a **live Zevari sweep** on the contact: profile (job change?), recent posts, recent comments, intent signals.
3. Run **company intelligence** on the lost-deal account (funding, layoffs, exec changes).
4. WebSearch the competitor that won (if loss reason names one): layoffs, price hikes, acquisitions, product issues.
5. Classify a revival angle: `JOB_CHANGE`, `COMPETITOR_TROUBLE`, `FUNDING_SIGNAL`, `INTENT_SIGNAL`, or `NO_SIGNAL`.
6. For signals: draft a revival message via Zevari `agents_generate_messages` using the deal context + new signal + rep's voice.
7. Update Pipedrive: move to `Revival - {SIGNAL_TYPE}` stage, append a signal-evidence note.
8. Slack DM the original rep with the deal, signal, and draft — they approve or edit before send.
9. (Optional) Auto-enroll in Instantly revival campaign if the workflow is configured for autopilot.
10. Save to a Zevari list `lost-deal-revival-YYYY-MM-DD`.
11. Slack summary at the end.

Cap: ~25 lost deals per weekly sweep to stay under LinkedIn's ~40 lookups/hour rate limit (each deal = ~3 LinkedIn calls: profile + posts + comments).

---

## ERROR & APPROVAL NOTIFICATIONS

If anything stops the task or needs approval, send to `[ALERT_CHANNEL]` via Chrome (bash sandbox blocks `hooks.slack.com`):

1. Call `mcp__Claude_in_Chrome__tabs_context_mcp` with `createIfEmpty: true`
2. If the tab is on `chrome://`, navigate to `https://google.com` first
3. Run via `mcp__Claude_in_Chrome__javascript_tool`:

```js
fetch('[SLACK_WEBHOOK_URL]', {
  method: 'POST',
  mode: 'no-cors',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({
    text: "<@[YOUR_SLACK_USER_ID]> ⚠️ *Lost Deal Revival — Action Needed*\n\n<error or approval ask>"
  })
}).then(r => ({status: r.status, type: r.type})).catch(e => ({error: e.message}))
```

`{status: 0, type: "opaque"}` = success.

---

## CONNECTORS

- **Zevari (LinkedIn MCP):** `mcp__[ZEVARI_MCP_ID]` — live profile, posts, comments, intent signals, company intelligence, message generation, target save
- **Pipedrive:** `mcp__pipedrive__` — lost-deal source, stage updates, note appends
- **(Optional) Instantly:** `mcp__[INSTANTLY_MCP_ID]` — autopilot enrollment if configured
- **Chrome:** `mcp__Claude_in_Chrome__` — Slack webhook + per-rep DMs
- **WebSearch:** competitor news + funding announcements

See `connectors.md`.

---

## STEP 0 — CONFIRM ZEVARI CONTEXT

At the start of every fresh chat, before any Zevari read, call `zevari_context_get`. Confirm the active organization and LinkedIn account match what the rep expects. Call `zevari_context_confirm` with the matching IDs from `available_contexts`. Do NOT skip this — another session can flip the context mid-day.

---

## STEP 1 — LOAD LOST DEALS

Pull deals from Pipedrive filter `[PIPEDRIVE_LOST_FILTER_ID]`:

Filter criteria (set this up in Pipedrive before running):
- Status = `lost`
- Lost time ≥ 60 days ago (don't pester deals that just lost — let the sting fade)
- Lost time ≤ 18 months ago (older than that, the contact almost certainly moved on)
- Has a primary contact with a LinkedIn URL custom field

Pull via `list_deals` or `search_deals`. For each deal, gather:

- `deal_id`
- `title` (the deal name)
- `lost_time`
- `lost_reason` (from `lost_reason` field — REQUIRED for classification)
- Deal notes (last 5 — the loss reason is often in a note, not the field)
- `value` + `currency`
- `owner_id` → map to `[REP_DM_USER_ID]` via the per-rep dictionary below
- Primary `person_id` → `name`, `email`, `linkedin_url`, `last_known_title`, `last_known_company`
- `org_id` → `name`, `website`, `linkedin_url`

Sort by `lost_time` ascending (oldest first — those have had more time for the situation to change). Take the first 25.

### Per-rep DM mapping

Replace this with your team's `owner_id` → Slack `U...` mapping:

```
PIPEDRIVE_OWNER_ID  →  SLACK_USER_ID
12345               →  [REP_DM_USER_ID]
12346               →  U07ABC123
12347               →  U07ABC456
```

If a deal's owner isn't in the map, fall back to `[YOUR_SLACK_USER_ID]` and flag it in the summary.

---

## STEP 2 — LIVE LINKEDIN SWEEP (per deal contact)

For each deal's primary contact, run the following Zevari calls. Process no more than 3 deals concurrently to respect LinkedIn rate limits.

### 2a — Profile (job change check)

Call `linkedin_get_profile` with the contact's LinkedIn URL.

Extract:
- `current_company` (most recent experience entry where `end_date` is null or "Present")
- `current_title`
- `started_at_current_role`

Compare against Pipedrive's `last_known_company` + `last_known_title`:
- Same company + same title → no job change.
- Different company OR title, started < 12 months ago → **JOB_CHANGE candidate** (highest priority signal — short-circuit and classify here)
- Different company OR title, started > 12 months ago → log "stale move" but continue to other signals (they're settled in, treat as normal contact)

### 2b — Recent posts (category engagement)

Call `linkedin_get_user_posts` for the last 30 days.

Scan for:
- Posts about your product category (use the keywords from `profile_get` → `category_keywords`)
- Posts mentioning the competitor that won the deal (from the loss reason — see Step 4)
- Posts about the problem your product solves

If any qualifying post exists → **INTENT_SIGNAL candidate**.

### 2c — Recent comments (same intent question)

Call `linkedin_get_user_comments` for the last 30 days.

Same scan as 2b. Comments are weaker signal than posts but worth checking — a contact commenting on a competitor's post about price hikes is gold.

### 2d — Zevari intent signals

Call `signals_list` filtered to this contact.

Zevari surfaces:
- Profile views from your team / your domain
- Engagement with content posted by your team
- Reactions / comments on your posts

If `signals_list` returns anything fresh (< 30 days) → **INTENT_SIGNAL candidate** (high confidence — they're actively considering you again).

---

## STEP 3 — COMPANY INTELLIGENCE

For each deal's company (the account they were buying for), call `agents_company_intelligence`.

Scan for:
- Funding round announced (any size) in last 90 days → **FUNDING_SIGNAL candidate** if loss reason was "budget" or "timing"
- Layoffs in last 90 days → context only (rarely a revival signal; usually de-prioritize)
- Exec change in your buyer's department in last 90 days → context, may unblock if loss reason was "champion left" / "no exec sponsor"
- New initiative / product launch / market expansion → context for the message body

---

## STEP 4 — COMPETITOR INTELLIGENCE (if applicable)

Parse the loss reason. If it names a competitor ("chose Acme", "went with [competitor]", "bought competitor X 6 months ago"):

1. WebSearch: `"[competitor name]" layoffs OR "price increase" OR acquired OR shutting down site:* -site:[competitor].com` over the last 90 days.
2. WebSearch: `"[competitor name]" customer complaints OR "switching from" OR alternative` over the last 60 days.

If anything material surfaces → **COMPETITOR_TROUBLE candidate**.

Categories that count:
- Layoffs at the competitor (their team is stretched, support is degrading)
- Price hike announcement (now your pricing looks better than it did)
- Acquisition by a larger player (uncertainty, contract risk, roadmap changes)
- Product end-of-life / discontinued feature the customer was using
- Major outage or customer-impacting incident

Skip soft stuff (a snarky tweet doesn't count).

---

## STEP 5 — CLASSIFY THE REVIVAL ANGLE

For each deal, pick **one** angle in this priority order:

1. **JOB_CHANGE** — if Step 2a flagged a fresh move (< 12 months). Highest value. The original objection at the old company is moot — they're at a new company, new role, new budget, clean slate.
2. **COMPETITOR_TROUBLE** — if Step 4 surfaced a material problem AND the loss reason names that competitor. The revival is "you bet on them, here's what's happening."
3. **FUNDING_SIGNAL** — if Step 3 surfaced fresh funding AND the loss reason was "budget" or "timing" or "wrong year for us."
4. **INTENT_SIGNAL** — if Step 2b/2c/2d surfaced fresh engagement with your category or your team.
5. **NO_SIGNAL** — none of the above. Leave the deal alone, run again next week. **Do not** force a message.

If a deal qualifies for multiple, pick the highest-priority one. Note the others in the Pipedrive note for context.

---

## STEP 6 — DRAFT THE REVIVAL MESSAGE

For each deal with a signal (not `NO_SIGNAL`):

1. Call `profile_get` to confirm the lost-deal company still passes ICP. If they don't (e.g. they got acquired into an out-of-ICP parent, or the contact moved into an out-of-ICP industry on a `JOB_CHANGE`), do not draft a message. Update Pipedrive with the classification but skip the rep DM. Log "out of ICP" in the summary.
2. Call `library_context_get` with `skill_slug='revival-outreach'` if you have one set up; otherwise use `craft-outreach`. Read the rep's voice rules and any campaign-specific formatting.
3. Call `agents_generate_messages` with:
   - `lead_id` (the contact, via `targets_save` first if not yet a Zevari target)
   - `context` block containing: original deal title, original loss reason, original loss date, signal type, signal evidence (specific post URL / funding announcement URL / competitor news URL / job change details)
   - `tone` = the rep's voice (passed in from the per-rep config)
   - `length` = "short" (revival messages should be ≤ 6 sentences)

The generated message MUST:

- **Acknowledge the prior conversation honestly.** "We talked back in [month]" — not "long time no chat!" (fake intimacy).
- **State the new signal as the reason for reaching out NOW.** That's the whole point — calendar-free revival means "I'm pinging because X just changed," not "checking back in."
- **Frame the signal against the original objection.** If they lost because of competitor X and competitor X just had layoffs — name both. If they lost because of budget and the company just raised — name both.
- **No pitch in the first message.** The revival earns the reply. Pitch goes in the second touch.
- **No LinkedIn references** ("saw your post on linkedin"). Write as if you noticed the signal in the world.

---

## STEP 7 — UPDATE PIPEDRIVE

For **every** deal Claude classified (even `NO_SIGNAL`):

1. Append a note: `"Revival sweep [YYYY-MM-DD]: classified as {SIGNAL_TYPE}. Evidence: {short summary + URL}. Original loss reason: {loss_reason}."`
2. If the signal is anything other than `NO_SIGNAL`, move the deal to stage `Revival - {SIGNAL_TYPE}` (set these stages up in Pipedrive before running — five stages: `Revival - Job Change`, `Revival - Competitor Trouble`, `Revival - Funding Signal`, `Revival - Intent Signal`, plus the original `Lost` stage which stays for `NO_SIGNAL` deals).
3. If `JOB_CHANGE`, also update the contact's `job_title` to the new title and link a new organization for the new company. The CRM should always reflect the live state.

Logging every classification (including `NO_SIGNAL`) is important: next week's sweep should see "I already considered this signal" and not re-fire on the same thing.

---

## STEP 8 — SLACK DM TO REP (approval gate)

For every deal with a signal, DM the original rep via Chrome:

```js
fetch('[SLACK_WEBHOOK_URL]', {
  method: 'POST',
  mode: 'no-cors',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({
    text: "<@[REP_DM_USER_ID_FROM_MAPPING]> 🔁 *Lost Deal Revival — draft for your approval*\n\n*Deal:* [deal title] (lost [N months] ago, [original loss reason summary])\n*Contact:* [name] — [current title @ current company]\n*Signal:* {SIGNAL_TYPE} — [one-line evidence]\n*Pipedrive:* https://yourorg.pipedrive.com/deal/[deal_id]\n\n*Draft message:*\n```\n[message body]\n```\n\nReply ✅ to approve, ✏️ to edit, or ❌ to skip. Auto-enrollment is OFF for this run."
  })
})
```

If the workflow is configured for autopilot (a per-rep flag), skip the DM and go straight to Step 9. Default for new reps: autopilot OFF, DM ON.

---

## STEP 9 — (OPTIONAL) AUTO-ENROLL IN INSTANTLY

Only if autopilot is enabled for this rep AND the signal type is on the rep's autopilot whitelist (e.g. `JOB_CHANGE` and `FUNDING_SIGNAL` are safe to auto-fire; `COMPETITOR_TROUBLE` and `INTENT_SIGNAL` usually need rep judgment).

Call `add_leads_to_campaign_or_list_bulk` to enroll in `[INSTANTLY_REVIVAL_CAMPAIGN_ID]`.

Required fields: `email`, `first_name`, `last_name`, `company_name` (= current company, post-job-change if applicable), `custom_variables: {signal_type, signal_evidence, original_loss_reason, draft_message}`.

Set `skip_if_in_campaign: true`.

If autopilot is OFF (default), do NOT enroll. The rep ships from their own inbox after approving in Slack.

---

## STEP 10 — SAVE TO ZEVARI

For every deal with a signal:

1. Call `targets_save` to save the contact with the deal context attached as research notes.
2. Create or find a list named `lost-deal-revival-YYYY-MM-DD` via `targets_create_list`.
3. Add each contact to that list.

This builds a historical record of every classified revival signal — auditable later, and feeds into next week's "already considered" check.

---

## STEP 11 — SLACK RUN SUMMARY

Send to `[REVIVAL_CHANNEL]` via Chrome `javascript_tool`:

```
<@[YOUR_SLACK_USER_ID]> 🔁 *Lost Deal Revival — Weekly Sweep Complete*

📅 Run date: [date]
📦 Lost deals checked: [N]
🆕 Signals surfaced: [N]
   • JOB_CHANGE: [N]
   • COMPETITOR_TROUBLE: [N]
   • FUNDING_SIGNAL: [N]
   • INTENT_SIGNAL: [N]
   • NO_SIGNAL: [N]
✅ Rep DMs sent for approval: [N]
🤖 Auto-enrolled (autopilot reps only): [N]
🚫 Out of ICP (skipped): [N]
🔗 LinkedIn cap hit: yes/no — [affected deals if yes]

Drafts pending rep approval:
• [rep name] — [deal title] — {SIGNAL_TYPE} — [one-line signal]
[list all]
```

---

## STEP 12 — REFLECTION

- Any deals where the Zevari sweep failed (LinkedIn profile gone, contact deleted) → flag for manual review and consider archiving the deal.
- Any signals that fired but the rep skipped on review → log so we can tighten the classifier (likely a false positive pattern).
- Any deals classified `NO_SIGNAL` three sweeps in a row → suggest archiving them out of the revival filter.

---

## Schedule

Every Monday at 9:00 AM local time. ~25 deals per sweep. For a 200-deal closed-lost backlog, the full pool gets a fresh live-check every 8 weeks — which is the right cadence for revival signal hunting (signals move week-by-week, not day-by-day).
