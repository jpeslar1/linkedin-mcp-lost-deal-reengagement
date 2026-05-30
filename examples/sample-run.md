# Sample run

A real Monday sweep (sanitized — names changed) from a recent week.

## Setup

- Closed-lost filter had 142 deals in the 60-day to 18-month window
- Sweep cap: 25 deals (oldest-lost first)
- 18 deals checked this run — 7 were skipped at Step 1 (LinkedIn URL missing on the contact — flagged for a separate enrichment pass)
- All connectors healthy
- Autopilot: OFF for all reps this run

## Step 2-4 — signal sweep (18 deals)

| Deal | Contact | Lost reason | Signal found | Classification |
|---|---|---|---|---|
| Hexel — annual plan | M. Acosta | "wrong year for budget" | Hexel raised \$22M Series B 11 days ago | **FUNDING_SIGNAL** |
| Brightspoke — 3-seat pilot | R. Okafor | "chose Riverbend" | Riverbend announced 18% price hike, 9 days ago | **COMPETITOR_TROUBLE** |
| Lumenfield — enterprise | S. Brar | "no exec sponsor" | profile views from our team's domain, last 14 days | **INTENT_SIGNAL** |
| Northpine — pro plan | D. Vasquez | "chose competitor" | now VP Ops at Coraline Brands (was Northpine, 18 days ago) | **JOB_CHANGE** |
| Calderon Group — multi-seat | T. Hwang | "not a priority right now" | no signal | NO_SIGNAL |
| Mossbank — annual | A. Patel | "chose Riverbend" | no movement at Riverbend; A. Patel still at Mossbank | NO_SIGNAL |
| Stillrun — pro plan | J. Marin | "internal tool instead" | no signal | NO_SIGNAL |
| Goldhaven — enterprise | K. Wexler | "champion left" | no exec change at Goldhaven yet | NO_SIGNAL |
| Pemberton — pilot | L. Quinn | "budget killed" | no funding | NO_SIGNAL |
| Trellis Foods — multi | I. Onyemachi | "too early" | no signal | NO_SIGNAL |
| Driftwood — pro | C. Romanov | "chose Acme Corp" | nothing material at Acme | NO_SIGNAL |
| Northvale — pilot | P. Sandberg | "no exec sponsor" | new VP of Ops hired at Northvale, ICP-aligned title, but no contact engagement yet | NO_SIGNAL (note appended for next sweep) |
| Halewell — annual | E. Bramley | "chose Riverbend" | Riverbend price hike (same as Brightspoke) — already used the angle this run | NO_SIGNAL (de-duped) |
| Cordova Brands — multi | N. Fischer | "wrong year" | no funding | NO_SIGNAL |
| Quayside — pro | V. Holm | "internal tool" | no signal | NO_SIGNAL |
| Westmark — enterprise | F. Tanaka | "champion left" | no signal | NO_SIGNAL |
| Pinewood — pilot | G. Lestrade | "too small" (now ICP-aligned?) | Pinewood headcount doubled, but no contact engagement | NO_SIGNAL (note appended) |
| Brackenpine — annual | H. Aoki | "chose competitor X" | nothing material at competitor X | NO_SIGNAL |

**Net:** 4 signals surfaced (~22%), 14 NO_SIGNAL. The 4 signals split cleanly into one of each category — a coincidence, not typical (most weeks skew toward INTENT_SIGNAL).

## Step 5 — ICP recheck on the 4 signal deals

All 4 still in ICP. D. Vasquez (JOB_CHANGE) moved to Coraline Brands — Coraline is in ICP. No skips.

## Step 6 — drafts written

### Draft 1 — JOB_CHANGE — D. Vasquez (Coraline Brands)

```
Subject: congrats on the coraline move

Hey D — saw you joined Coraline as VP Ops a couple weeks ago, congrats on the move.

We talked back when you were at Northpine about [product] — different company, different problem, so this isn't a "checking back in" note.

Coraline's operations footprint is the exact shape we tend to land well at, and given you already know what we do, I'd rather just send you a 90-second loom of how a team your size at your new shop typically runs it — no pitch, just so you have the context if it's ever relevant.

Want me to send it over?
```

### Draft 2 — COMPETITOR_TROUBLE — R. Okafor (Brightspoke)

```
Subject: heads up on the riverbend price change

Hey R — saw Riverbend announced an 18% price increase last week, effective next renewal cycle.

I'm only pinging because we lost the Brightspoke deal to them eight months ago over pricing — figured you'd want a heads-up before the renewal hits, and I'd rather you hear it from us than be surprised.

Happy to share what the spread looks like now between our pricing and theirs post-increase if it's useful. No pitch otherwise.
```

### Draft 3 — FUNDING_SIGNAL — M. Acosta (Hexel)

```
Subject: congrats on the 22M

Hey M — saw the Series B closed last week, congrats.

When we talked last quarter the budget timing was off — totally fair at the time. Different conversation now obviously.

If the annual plan we scoped is still in the realistic zone for the new year, happy to redo the math against your current team size (you've grown a bit since June). Otherwise no rush.
```

### Draft 4 — INTENT_SIGNAL — S. Brar (Lumenfield)

```
Subject: noticed you've been poking around

Hey S — quick note: a couple folks from our side noticed your team's been looking at our stuff over the last two weeks.

I know the original blocker was the exec sponsor situation. If that's shifted I'd rather make myself useful directly than have you piece things together from the website.

Happy to send updated pricing for the team size you scoped, or set up a 20-min call with our solutions lead if it's easier. Either's fine, just let me know which is less work for you.
```

## Step 7 — Pipedrive updates (all 18 deals)

- 4 deals moved to their `Revival - {SIGNAL_TYPE}` stage
- 14 deals stayed in `Lost`, all 14 got a note appended: `"Revival sweep 2026-05-25: classified as NO_SIGNAL. No qualifying signal in profile, posts, comments, intent, company intel, or competitor news. Original loss reason: [...]"`
- D. Vasquez's contact record updated: `job_title = "VP Ops"`, organization changed from Northpine to Coraline Brands (org created since it didn't exist in Pipedrive yet)
- 3 deals (Northvale, Pinewood, Halewell) got extra context notes for next sweep — situation moving but not actionable yet

## Step 8 — Slack DMs to reps

Four DMs sent — one per signal deal. Reps tagged individually with the draft, the deal link, and the signal evidence. None auto-enrolled (autopilot was OFF for all reps this run).

DM thread response (came in over the next ~6 hours):

- D. Vasquez draft: rep replied ✅, sent as-is.
- R. Okafor draft: rep replied ✏️ — wanted to add a line about the renewal contract date he remembered from the original deal. Edited and sent.
- M. Acosta draft: rep replied ❌ — they had a relationship update Claude didn't know about (M. is leaving Hexel in two months, the funding is moot for him personally). Good catch — exactly why the approval gate exists.
- S. Brar draft: rep replied ✅, sent as-is.

## Step 9 — auto-enrollment

None — autopilot OFF.

## Step 10 — Zevari list saved

4 contacts saved to `lost-deal-revival-2026-05-25` list. M. Acosta will still appear in the list even though the draft was skipped — the historical record matters even when the rep makes a judgment call.

## Step 11 — Slack run summary

```
🔁 Lost Deal Revival — Weekly Sweep Complete

📅 Run date: 2026-05-25
📦 Lost deals checked: 18
🆕 Signals surfaced: 4
   • JOB_CHANGE: 1
   • COMPETITOR_TROUBLE: 1
   • FUNDING_SIGNAL: 1
   • INTENT_SIGNAL: 1
   • NO_SIGNAL: 14
✅ Rep DMs sent for approval: 4
🤖 Auto-enrolled (autopilot reps only): 0
🚫 Out of ICP (skipped): 0
🔗 LinkedIn cap hit: no

Drafts pending rep approval:
• T. Wexford — Hexel annual plan — FUNDING_SIGNAL — Series B 11 days ago
• T. Wexford — Brightspoke pilot — COMPETITOR_TROUBLE — Riverbend price hike
• J. Imani — Lumenfield enterprise — INTENT_SIGNAL — profile views from our domain
• J. Imani — Northpine pro plan — JOB_CHANGE — contact now VP Ops at Coraline Brands

Note: 7 deals skipped at Step 1 (missing LinkedIn URL on contact) — flagging for enrichment pass.
```

## Step 12 — reflection notes

- The de-dupe on Halewell (also lost to Riverbend) was the right call — sending two reps the same competitor-trouble draft on the same week looks coordinated and weakens both. Next iteration: if multiple lost deals share a competitor-trouble signal, surface it once, let the reps decide who pings whom.
- The Northvale "new VP hired" note is the kind of thing that becomes a signal in 4-6 weeks once that VP gets settled. Worth a manual check before next month's sweep.
- M. Acosta rejection is exactly the value of the approval gate. The classifier was right (Hexel raised, original objection moot) — but the rep's relationship context overrode it. Bake that into the per-rep memory: "Hexel funding signal — rep flagged as not actionable, don't re-fire for 90 days."
- Total run time: ~22 min including Pipedrive notes on all 18 deals.
