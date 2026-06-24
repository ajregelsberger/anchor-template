# anchor-template

A template for building a personal AI workspace (an "Anchor") using Claude Cowork. Clone this, fill in your details, and get a fully functional AI assistant that persists context across sessions and runs autonomous background agents.

---

## What This Is

An **Anchor** is a file-backed Claude Cowork workspace that lives on your machine. It persists memory, skills, and context across sessions — so every conversation builds on the last. This template gives you the full folder structure, reusable skills, scheduled agent scripts, and framework documentation to set one up for your own role.

---

## The Mental Model: TDB

Three Claude tools, three jobs:

| Zone | Tool | What it does |
|------|------|--------------|
| **Think** | Claude Chat (claude.ai) | Research, brainstorming, Q&A — and now: session-open automation on mobile/browser via Project Instructions |
| **Do** | Claude Cowork (this anchor) | File-backed workflows, persistent memory, planning, stakeholder comms |
| **Build** | Claude Code (CLI) | Code generation on real repos — receives specs from the Do layer |

> **New in June 2026:** Claude Chat Projects now support the same MCP connectors (Gmail, Drive, Calendar, Notion, iMessage) as Cowork. You can generate a `reference/project-instructions.md` from your anchor to get session-open automation on any device — phone, browser, or desktop — without needing the Cowork app. See [Claude Chat Projects](#claude-chat-projects--mobile--browser-access) below.

---

## ATL / BTL / OTL — The Three Operating Zones

| Zone | Name | What happens here |
|------|------|--------------------|
| **ATL** | Above the Line (Cowork) | Analysis, planning, drafts, specs. Never ships code directly. |
| **BTL** | Below the Line (Claude Code) | Implementation. Never sends stakeholder comms. |
| **OTL** | Outside the Line (Scheduled agents) | Autonomous monitoring between sessions. Never takes external action. |

See `reference/above-below-the-line.md` for the full model.

---

## What's in This Repo

| Path | Purpose |
|------|---------|
| `CLAUDE.md` | Identity + routing table (read every session) |
| `memory/` | Running context that compounds across sessions |
| `reference/` | Framework docs + role-specific context |
| `ops/` | Coordination layer: event log, board, handoffs |
| `skills/` | Reusable Cowork workflows (session-start, context-ingestion, etc.) |
| `Scheduled/` | Autonomous agent skill scripts (daily true-up, Slack signal monitor, etc.) |
| `templates/` | Claude Code dev-mode templates + scheduled task design pattern |
| `inbox/` | Landing zone for scheduled agent output |
| `saddle/` | Scratchpad for raw/unsorted work |
| `sessions/` | Session closeouts + next-session prompts |
| `work/` | Active outputs |

---

## Quick Start

1. Clone this repo
2. Open `SETUP.md` and work through every `{{PLACEHOLDER}}`
3. Point Claude Cowork at this folder (Settings → Cowork → Add Folder)
4. Say: *"Set me up"* to trigger the `anchor-setup` skill — it walks you through personalization interactively
5. Optionally wire up scheduled agents (see `Scheduled/` and `templates/scheduled-tasks/README.md`)
6. Optionally generate `reference/project-instructions.md` for Claude Chat / mobile access

---

## Prerequisites

**Claude Cowork** (Claude Desktop app, Cowork tab)

**Connectors** — connect the ones relevant to your role:

| Connector | Used by |
|-----------|---------|
| Google Calendar | daily-true-up, monday-briefing, commitment-tracker |
| Google Drive | all agents (read/write anchor files) |
| Gmail | commitment-tracker, inbox-triage, session-open |
| Notion | all agents (live project board — authoritative source for deal/task status) |
| iMessage | all agents (real-time context: rates confirmed, deals closed, shipments via text) |
| Slack | slack-signal-monitor, commitment-tracker |
| Linear / Jira / Asana | project tracker sync |
| GitHub | github-sync agent |

> **Connector priority:** When multiple sources have the same information, trust them in this order: iMessage / Notion (most recent, directly updated) → Gmail → Drive project-state.md. Always note discrepancies rather than silently picking one.

---

## Key Agent Patterns (from production use)

These patterns emerged from running real agents against real inboxes. Apply them when building or adapting any scheduled agent.

### Direction Check — the single most important inbox pattern

Before flagging any email thread as "needs reply," check the `messages[]` array last element:

```
Last message labelIds contains "SENT" → we replied last → skip (awaiting their response)
Last message labelIds does NOT contain "SENT" → they sent last → flag as needs reply
```

Without this check, agents incorrectly flag threads where the user already replied. This is the #1 false positive in inbox monitoring agents.

### Always load full thread context

Call `get_thread(threadId, messageFormat="FULL_CONTENT")` before writing any summary about a thread. Snippet-only context misses:
- Rate agreements buried in earlier messages
- Products that were shipped but not mentioned in the latest reply
- Contracts or timelines that changed mid-conversation

### Multi-source context loading

Before any agent acts, load context from all available sources:
1. **Notion** (live project board — most authoritative)
2. **iMessage** (real-time updates that predate email)
3. **Google Drive** project-state.md (standing backlog)
4. **Google Calendar** (upcoming deadlines, calls)

Cross-reference sources. If they conflict, note the discrepancy and trust the most recently updated one.

### 30-day scan window for commitment tracking

Use `newer_than:30d` for new inbound scans in commitment trackers, not 14 days. Threads often go quiet for 3–6 weeks and reactivate — a 14-day window misses them.

### Day-aware session-open behavior

For inbox-heavy roles, vary scan depth by day of week rather than running the same query every day:

| Day | Scan depth |
|-----|-----------|
| Monday | 7-day + full vendor pipeline (60–90d per vendor) |
| Mon / Thu | 90-day commitment tracker pass |
| All other days | 2-day triage |

This eliminates redundant deep scans on non-trigger days while keeping Monday comprehensive.

### iMessage as a first-class context source

People confirm rates, approve concepts, and close deals over text before emailing. If an agent only reads Gmail, it has an incomplete picture. Add an iMessage scan for all active vendor/contact names before writing any deal status summary.

---

## Sub-Anchor Pattern — Specialized Inbox Monitors

If you have a public-facing email account (e.g., `hello@company.com`, `info@`, `support@`) that needs dedicated monitoring, consider a **sub-anchor**: a second Cowork session authenticated to that account, sharing the same Drive folder.

**How it works:**
- Primary session: authenticated as your main account — handles planning, memory, all ventures
- Sub-anchor session: authenticated as your public account — inbox monitoring only, writes to `inbox/` folder

**Rules for sub-anchors:**
- Only write to `inbox/` — never update `ops/board.md` or `memory/project-state.md`
- Source every claim to a specific email (sender, subject, date)
- Draft replies but never send — primary account holder sends
- Share the Drive folder with the sub-account as Editor

See `reference/hello-session.md` for a full example.

---

## Claude Chat Projects — Mobile / Browser Access

Claude Chat Projects (claude.ai) support the same MCP connectors as Cowork and are available on any device — phone, browser, desktop. You can replicate session-open automation in a Project so you get a briefing automatically when you open a chat.

**To generate project instructions from your anchor:**

Ask Claude in your Cowork session: *"Generate project-instructions.md for this anchor"*

The output covers:
- Session-open automation (day-aware Gmail scan, Notion/iMessage/Calendar load)
- Venture / role context
- Multi-account routing rules
- Agent rules (direction check, full thread context, draft-only)
- Key Drive file IDs

**What works in Projects:** All MCP connector tools (Gmail, Drive, Calendar, Notion, iMessage). Session-open automation fires when you open a new chat.

**What doesn't work in Projects:** Scheduled tasks (automated cron runs). Those stay in Cowork. The Project is the on-demand layer; Cowork is the automated layer.

See `reference/project-instructions.md` for a working template.

---

## Gemini Gem — Alternative for Google-first Workflows

If your role is heavily Google Workspace (Gmail, Calendar, Drive, Tasks, Keep) a **Gemini Gem** can cover session-open triage natively — without needing Cowork or the Claude desktop app.

**Advantages over Claude Chat Projects:**
- Native @-handle syntax: `@Gmail`, `@Google Calendar`, `@Google Drive`, `@Google Tasks`, `@Google Keep`, `@YouTube`, `@GitHub`
- Multi-account is handled by signing into Gemini as the target account — no connector setup
- `@Google Tasks` write access works natively (add action items directly after triage)
- `@YouTube` for verifying video posts — not available in Claude

**Limitations vs. Claude:**
- No iMessage connector
- No Notion connector
- Can't auto-write files to Drive (manual copy-paste to save output)
- No scheduled/cron runs — on-demand only

**To generate a Gemini Gem from your anchor:**

Ask Claude in your Cowork session: *"Generate a Gemini Gem instruction for this anchor"*

See `reference/gemini-gem-instruction.md` for a working template. Key sections:
- Session-open routine (Drive → Tasks → Keep → Calendar → Gmail scan → YouTube check)
- Day-aware scan table (same pattern as Cowork scheduled tasks)
- Direction check rule
- Cross-app command patterns (`@Gmail find X and @Google Tasks add action items`)
- Comparison table: Gem vs. Claude Chat Project vs. Cowork Scheduled Tasks

> **Which to use:** Gem for Google-only on any device; Claude Chat Project for iMessage + Notion context; Cowork Scheduled Tasks for fully automated (no-touch) monitoring.

---

## The Core Idea

Quality in = quality out. The richer and more accurate the context in `memory/` and `reference/`, the better every session. You don't maintain this system manually — you *correct* it, and the corrections compound.

See `reference/getting-started.md` for a full orientation.

---

## Changelog

### June 2026 — hello@ sub-anchor patterns
- Added iMessage and Notion as first-class context sources for all agents
- Added direction check rule (prevent false-positive "needs reply" flags)
- Added full thread context requirement (`get_thread FULL_CONTENT`)
- Changed commitment tracker new inbound window from 14d → 30d
- Added day-aware session-open behavior pattern
- Added sub-anchor pattern documentation (dedicated inbox monitor)
- Added Claude Chat Projects section (mobile/browser access via project-instructions.md)
- Added Gemini Gem section (Google-native alternative with @-handle extensions)
- Added multi-source context loading with priority order

---

*Built on the ATL/BTL/OTL framework. hello@ and Gemini patterns added June 2026.*
