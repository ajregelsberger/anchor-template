# Scheduled Agents — Overview

> The five OTL (Outside-the-Line) agents that run autonomously between sessions. Each drops output into `inbox/` and appends a summary to `ops/event-log.md`.

---

## Agent Portfolio

### 1. Daily True Up
**Cadence:** Weekdays, ~5:05pm (example — adjust to your timezone)
**Output:** `inbox/true_up_YYYY-MM-DD.md`

**What it does:**  
Cross-references the calendar (Google Calendar) against available meeting transcripts for that day. For each calendar event:
- If a transcript exists in inbox/ → marks as available for ingestion
- If a transcript is missing → flags as a gap
- Excludes hold blocks and focus time

**Why it matters:** Ensures no meeting goes un-ingested. You review it once to see what context is waiting.

**Status codes:** GREEN (all matched), YELLOW (gaps present), RED (system issue)

---

### 2. Slack Signal Monitor
**Cadence:** Weekdays, ~4:02pm (example — adjust to your timezone)
**Output:** `inbox/slack_signal_YYYY-MM-DD.md`

**What it does:**  
Scans the configured Slack channels for signal items — decisions, commitments, blockers, status changes, and urgent flags. Filters out noise (reactions, thread replies without substance, casual chatter).

**Channels monitored (configure for your team's scope):**
- `{{SLACK_CHANNEL_1}}` (primary team channel)
- `{{SLACK_CHANNEL_2}}` (project or domain-specific channel)
- `{{SLACK_CHANNEL_3}}` (cross-functional or stakeholder channel)
- Direct @-mentions of {{YOUR_NAME}}

> These are examples — replace with your team's actual Slack channels.

**Why it matters:** Surfaces decisions happening in Slack that should be in your project tracker or memory but aren't yet.

**Status codes:** GREEN (scanned, signal captured), YELLOW (scan incomplete), RED (Slack connection issue)

---

### 3. Weekly Anchor Audit
**Cadence:** Fridays, ~4:18pm (example — adjust to your timezone)
**Output:** `memory/anchor-audit-report.md`

**What it does:**  
Meta-agent — checks the anchor's own health. Validates:
- **Memory freshness:** Are memory files being updated, or going stale?
- **Routing integrity:** Do all CLAUDE.md routing pointers resolve to real files?
- **Agent health:** Are the other 4 agents producing output on schedule?
- **Inbox hygiene:** Are old items accumulating without being processed or archived?
- **Session discipline:** Is the event log being written through?

**Why it matters:** Catches drift before it becomes rot. The audit report is the mechanism that surfaces exactly what needs fixing.

**Status codes:** GREEN (all healthy), YELLOW (minor drift), RED (broken pointers, missing files, agent failures)

---

### 4. Commitment & Follow-Up Tracker
**Cadence:** Wednesdays, ~9:06am (example — adjust to your timezone)
**Output:** `inbox/commitment_tracker_YYYY-MM-DD.md`

**What it does:**  
For every "waiting on others" item in `memory/project-state.md` and `ops/board.md`, searches Slack and Gmail for activity. Flags items that:
- Have had no activity in >5 business days
- Had a stated deadline that has passed
- Show conflicting signals (e.g., someone said it was done but it's still in Intake on your project tracker)

**Why it matters:** {{YOUR_NAME}}'s role spans multiple teams. Commitments from others can go stale without a systematic check.

Key people to track: {{TEAM_MEMBER_1}}, {{TEAM_MEMBER_2}}, external partners

> These are examples — replace with your team members.

**Status codes:** GREEN (all active), YELLOW (items approaching stale), RED (items definitely stale)

---

### 5. Monday Morning Briefing
**Cadence:** Mondays, ~7:38am (example — adjust to your timezone)
**Output:** `inbox/monday_briefing_YYYY-MM-DD.md`

**What it does:**  
Weekly rollup — one file to start the week. Aggregates:
- Summary of prior week's agent outputs (true-ups, signal items, commitment status)
- Calendar look-ahead: what's on deck this week
- Open commitments and their staleness
- Any RED or YELLOW audit findings still unresolved
- Force-ranked priority suggestion for the week

**Why it matters:** Replaces the mental overhead of scanning 5 separate inboxes. One read, then you can ask: *"Let's start on the top priority."*

**Status codes:** GREEN (full synthesis), YELLOW (partial data), RED (multiple agent failures)

---

## How to Use Inbox Outputs

1. **Read at session start** — the `session-start` skill surfaces inbox files automatically
2. **Process or archive** — act on the signal, then move the file to `inbox/archived/`
3. **Don't let them pile up** — the weekly audit flags unprocessed items >14 days old as stale

---

## Agent Health Check

Run by the weekly anchor audit automatically. If you need a quick manual check:

- Open Cowork scheduled tasks (sidebar)
- Confirm all 5 are enabled and show a recent `lastRunAt`
- If a task shows "never run" and it's been more than one cadence cycle, disable and re-enable to reset

---

*Agent design principles: `reference/aom-framework.md` (OTL zone rules)*  
*Scheduled task prompts: `Scheduled/` directory*
