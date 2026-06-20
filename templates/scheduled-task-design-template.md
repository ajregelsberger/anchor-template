# Scheduled Task Design Template

> Pattern for designing new OTL (Outside The Line) scheduled agents that run on your anchor. Use this when you want an agent to run autonomously on a cadence and drop output into `inbox/` for review.

---

## When to design a scheduled task

A scheduled task is appropriate when:

- **The work has a clear cadence** (daily, weekly, hourly) and shouldn't depend on you opening a session manually
- **The output is informational, not action-taking** — it surfaces signal, gathers intelligence, runs audits. It doesn't send messages to other people or modify production state.
- **The work is read-only or write-to-inbox-only** — Per AOM-1, agents surface to the governor (you). They don't take external action.
- **The pattern has been validated at least once manually** — Don't design a scheduled task for work that hasn't been proven through at least one manual run. (Decision Principle #1: build the second one, not the first.)

If the work is one-off, conversational, or requires real-time judgment, it's not a scheduled task — it's a Cowork session or skill.

---

## Design template

Fill this in before creating the scheduled task in Cowork.

```markdown
# Scheduled Task — <agent name>

## Identity
- **Task ID** (Cowork-assigned): <e.g., daily-true-up>
- **Capability domain**: <ingestion / monitoring / tracking / synthesis / maintenance / notification / reporting>
- **Tier**: <Tier 1 operational | Tier 2 roadmap candidate>

## Cadence
- **Days**: <weekdays / weekends / specific day(s)>
- **Time**: <e.g., 7:30am ET, 5:00pm ET>
- **Time zone**: <your local time zone, e.g. America/New_York>

## Purpose
<One-paragraph description: what this agent does and why it matters to your role.>

## Inputs (what the agent reads)
- <Source 1 — e.g., Google Calendar via MCP for the last 24h>
- <Source 2 — e.g., Drive folder for new transcripts>
- <Source 3 — e.g., specific Slack channels via Slack MCP>

## Process
1. <Step 1>
2. <Step 2>
3. <Step 3>
4. <Final step: produce the output>

## Output
- **Location**: `inbox/<filename>_<date>.md` (always lands in inbox, not in a final destination)
- **Format**: <markdown / JSON / both>
- **Header**: Standard agent output header (see below)

## Guardrails
- Never sends messages to anyone other than {{OPERATOR_NAME}} (DM-only if Slack DM)
- Never modifies production state outside the anchor
- Never auto-publishes to channels or external systems
- Tier 3 / low-confidence items get flagged, not committed to canonical memory
- If a finding rises to "urgent" — flag in the output header, do NOT auto-update the board

## Event log entry
- Every run appends an `agent-run` event to `ops/event-log.md`:
  ```
  ## YYYY-MM-DD HH:MM | agent:<task-id> | agent-run

  <Status: GREEN / YELLOW / RED> — <1-line summary of what was found>

  → Output: <inbox/path>
  ```

## Standard output header
Every output file starts with:
```
# <Agent Name> — <run date>

**Run**: <YYYY-MM-DD HH:MM ET>
**Status**: GREEN | YELLOW | RED
**Items found**: <count>
**Channels/sources scanned**: <list>
**Next run**: <YYYY-MM-DD HH:MM ET>

---

## Headline
<1-2 line summary of the most important finding(s)>

---

## Details
<full output>
```

## Eval / verification plan
- <How will you know this agent is working? E.g., "compare output to manual run for 1 week before trusting"?>
- <What's the calibration threshold for promoting from Tier 2 → Tier 1?>

## Maintenance signals
- Output freshness — when did it last run? (Weekly Anchor Audit checks this.)
- Run-history pattern — any silent failures? Empty outputs that should have content?
- Prompt drift — is the agent still calibrated to the right inputs?
```

---

## Patterns from the operational portfolio

The 5 agent skills in `Scheduled/` are the reference designs. Each covers a different capability domain. Common patterns:

**Ingestion agent** (e.g., daily-true-up):
- Pulls calendar + Drive transcripts for the last 24h
- Cross-references calendar against transcripts to find unprocessed meeting notes
- Stages unprocessed transcripts to `inbox/archived/` with consistent naming

**Monitoring agent** (e.g., slack-signal-monitor):
- Scans 3–8 configured channels for last 24h activity
- Filters for decisions, commitments, blockers, status changes (not noise)
- Output: 1–2 paragraphs of headline signal + structured channel-by-channel detail

**Maintenance / meta-agent** (e.g., weekly-anchor-audit):
- CLAUDE.md freshness (last-updated date)
- Memory file staleness (when did each get touched last?)
- Routing-table integrity (do all referenced paths exist?)
- Other agents' health (output freshness, run history)
- Surfaces issues for {{OPERATOR_NAME}} to fix or delegate

**Tracking agent** (e.g., commitment-follow-up-tracker):
- Reads commitments captured in memory files + event log
- Flags items approaching or past their follow-up date
- Output: prioritized list of "you said you'd do X by Y — status?"

**Synthesis agent** (e.g., monday-morning-briefing):
- Aggregates the week's prior agent outputs
- Adds calendar look-ahead for upcoming week
- Pulls top open items + pending decisions
- One read to start the week

**Path format depends on OS** — use the full absolute path to your anchor folder (Windows: backslashes, Mac: forward slashes). Scheduled agents need this to write files correctly.

---

## Designing new agents

For each new agent candidate:

1. **Manual prove the pattern once** before designing the scheduled task
2. **Fill in the template above** — every section
3. **Wire up in Cowork's scheduled task UI** with the prompt drawn from your template
4. **First week = Tier 2 (roadmap)** — review every output manually, calibrate prompt, watch for drift
5. **Promote to Tier 1 (operational)** only after a full cycle of verified-correct outputs

---

## What to NOT design as a scheduled task

- Anything that sends external comms (Slack to other people, email, posts to channels)
- Anything that modifies production state outside the anchor
- Anything that requires real-time judgment {{OPERATOR_NAME}} would normally make in-session
- Anything that hasn't been manually validated at least once
- Anything that would duplicate work an existing Tier 1 agent already does

If in doubt, design it as a Cowork skill or a one-off session prompt first. Promote to scheduled task only when the cadence and value are both proven.
