# Scheduled Tasks (the agent portfolio)

These are autonomous Cowork **scheduled tasks** — the "Outside-the-Line" layer that runs on a cron between your working sessions, does a recurring information-gathering job, and drops its output into your `inbox/` for review. All five are live + enabled on the reference anchor.

## The portfolio

| Task | Cadence | Output |
|------|---------|--------|
| **daily-true-up** | Weekdays (late afternoon) | `inbox/true_up_<date>.md` |
| **slack-signal-monitor** | Weekdays (mid-afternoon) | `inbox/slack_signal_<date>.md` |
| **weekly-anchor-audit** | Fridays | `memory/anchor-audit-report.md` |
| **commitment-follow-up-tracker** | Mid-week | `inbox/commitment_tracker_<date>.md` |
| **monday-morning-briefing** | Mondays (morning) | `inbox/monday_briefing_<date>.md` |

> See each agent's SKILL.md for recommended cadence.

They form a loop: **true-up** captures meetings → **signal monitor** captures Slack → **commitment tracker** chases follow-ups → **monday briefing** synthesizes the week ahead → **weekly audit** is the meta-watchdog that checks the anchor's health *and the other four agents'* health.

---

## How to recreate one in your Cowork
Each file has an **"Adapt before you wire this up"** block at the top — read it first. Then the simplest path: hand the file to your Claude and say *"set this up as a scheduled task, swapping in my paths, connectors, and channels."* The **`schedule`** skill turns plain English into a real scheduled task. You can also create/edit tasks in Cowork's settings.

---

## Two hard constraints they all share
- **Runtime:** a scheduled session does **not** have your anchor folder mounted. Scheduled agents need the full absolute path to your anchor — this differs by OS. See SETUP.md.
- **Connectors are yours to reconnect.** The MCP connector IDs in the source prompts are specific to the original anchor — reconnect your own and Claude will generate the correct IDs automatically.

---

## Required patterns — apply to every agent you build

These are non-negotiable for accurate agent output. They emerged from production failures on real inboxes.

### 1. Direction check (inbox agents)

Before flagging any email thread as "needs reply," check the last element of `messages[]`:

```
Last message labelIds contains "SENT" → we replied last → SKIP (awaiting their response)
Last message labelIds does NOT contain "SENT" → they sent last → flag as needs reply
```

Without this: agents surface threads where the user already replied, creating false alarms that erode trust in the agent.

### 2. Full thread context before any summary

Always call `get_thread(threadId, messageFormat="FULL_CONTENT")` before writing a thread summary. Snippet fields miss rate agreements, shipment confirmations, and timeline changes mid-conversation.

### 3. Multi-source context loading

Every agent should load context from multiple sources before acting:

1. **Notion** (live project board — most authoritative)
2. **iMessage** (confirms rates/approvals before email follows)
3. **Google Drive** `memory/project-state.md` (standing backlog)
4. **Google Calendar** (deadlines, calls)

Trust order: Notion / iMessage → Gmail → Drive.

### 4. Scan window recommendations

| Use case | Recommended window |
|----------|--------------------|
| Daily triage (new inbound) | `newer_than:2d` |
| New vendor/contact scan | `newer_than:30d` |
| Per-contact full history | `newer_than:90d` |
| Weekly briefing new inbound | `newer_than:7d` |
| Weekly briefing pipeline | `newer_than:60d` |

> Use 30d (not 14d) for new inbound scans. Threads go quiet for weeks and reactivate — 14d misses them.

### 5. iMessage as a context source

For any agent tracking relationships, deals, or commitments: search iMessage for contact names before writing status summaries. People confirm decisions over text before sending a formal email. Missing iMessage context means the agent is one step behind reality.

---

## Output convention (so the audit can watch them)
Every run writes a file with a standard header and appends an `agent-run` event to `ops/event-log.md`:
```
> Run: <ISO UTC> · Status: GREEN|YELLOW|RED · Items found: <n> · Next run: <…>
```
The **weekly-anchor-audit** reads those events to confirm each agent ran on cadence — if one goes silently dark, the audit flags it RED.

---

## Alternatives: on-demand access

### Claude Chat Projects
Scheduled tasks run on a clock. Claude Chat Projects (claude.ai) give you the same behavior on-demand — triggered by opening a chat. Projects support the same MCP connectors and work on any device.

| Need | Use |
|------|-----|
| Automated output without opening anything | Scheduled tasks |
| Mobile / browser, any device | Claude Chat Project |
| iMessage + Notion context | Claude Chat Project or Cowork |

See `reference/project-instructions.md` for a working example.

### Gemini Gem (Google-native)
If your stack is Google Workspace-first, a Gemini Gem covers session-open triage natively using @-handle extensions (@Gmail, @Google Calendar, @Google Tasks, @YouTube, etc.) without needing Cowork installed.

| Need | Use |
|------|-----|
| Automated cron runs (no-touch) | Scheduled tasks |
| Google-only stack, any device | Gemini Gem |
| @Google Tasks write + @YouTube | Gemini Gem |
| iMessage + Notion | Claude Chat Project or Cowork |

See `reference/gemini-gem-instruction.md` for a working template.

---

## Design principles (the part worth internalizing)
- **Build the *second* one, not the first.** Solve a recurring problem by hand at least once before automating it.
- **Group by capability domain, not department.** ingestion · monitoring · tracking · synthesis · maintenance.
- **Pace to your review capacity.** Five agents producing daily output is only useful if you actually read the inbox.
- **Trust multi-source context over single-source.** An agent that reads only Gmail misses iMessage, Notion, and calendar context.
- **Direction check first, summarize second.** Never flag a thread without checking who sent last.
