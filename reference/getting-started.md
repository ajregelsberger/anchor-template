# Getting Started — {{YOUR_NAME}}'s Anchor

> **What this is:** A five-minute orientation for using this Cowork anchor. Read once, then just work.

---

## What the anchor does

This folder is a **file-backed Claude workspace** — every session reads from it and writes back to it. Memory compounds across sessions. You don't need to recap anything at the start of a session; Claude reads the context files and picks up where you left off.

The folder is self-contained. Claude only knows what's in this folder plus what you say in the current session.

---

## Four files to know

| File | When to read |
|------|-------------|
| `CLAUDE.md` | Routing table — tells Claude which file to load for any task. Read first, every session. |
| `memory/project-state.md` | Current team state, open items, project tracker board. Check before any project discussion. |
| `memory/MEMORY.md` | Full index of memory files. |
| `ops/event-log.md` | Append-only history. Source of truth for "what happened." |

---

## How to open a session

Open Cowork pointed at this folder and just talk. The `session-start` skill fires automatically when you ask orienting questions:

- *"Catch me up — what's the current state of the team's open items?"*
- *"Where did we leave off?"*
- *"What's on deck for today?"*
- *"Morning — what do I have going on?"*

---

## What you can do in a session

| Task | What to say |
|------|-------------|
| Ingest a meeting transcript | Drop the file or paste content — *"Here's this morning's meeting transcript"* |
| Check your project tracker | *"What's the current state of the {{PROJECT_TRACKER}} backlog?"* |
| Write a spec for Claude Code | *"Help me write the spec for [your current project]"* |
| Draft a Slack message | *"Draft the Slack message to a key stakeholder about your critical external deliverable"* |
| Investigate a domain-specific issue or anomaly | *"What's the status of [issue] and what do we need to do?"* |
| Close a session | *"Let's close out"* or *"That's it for today"* |

---

## How memory works

Claude doesn't remember across sessions automatically — it reads the files in this folder instead.

- **Session start:** Claude reads `memory/MEMORY.md` and relevant memory files to orient.
- **During work:** Claude writes findings back to `memory/` and appends events to `ops/event-log.md`.
- **Session close:** The `session-close` skill generates a next-session prompt in `sessions/` so context carries forward.

**The core loop:** You correct Claude's memory files → they get more accurate → future sessions start with better context. You don't manage the system manually — you correct it.

---

## Skills that fire automatically

| Skill | Triggers |
|-------|----------|
| `session-start` | "catch me up", "morning", "where did we leave off", "what's on deck" |
| `context-ingestion` | Any transcript/doc upload or "process this meeting" |
| `meta-analysis` | "meta-analyze this session", "retrospective" |
| `session-close` | "close out", "wrap up", "that's it for today" |

---

## Scheduled agents (run between sessions)

Five agents run automatically and drop outputs into `inbox/`:

- **Daily True Up** — weekdays ~5pm: cross-references calendar vs. transcripts
- **Slack Signal Monitor** — weekdays ~4pm: surfaces decisions/blockers from key channels  
- **Weekly Anchor Audit** — Fridays ~4pm: checks anchor health + agent health
- **Commitment & Follow-Up Tracker** — Wednesdays ~9am: flags stale waiting-on items
- **Monday Morning Briefing** — Mondays ~7:30am: weekly rollup

> Cadence times are examples — configure each agent to suit your schedule and timezone.

Review `inbox/` files at session start — they're pre-digested for you.

---

## Where to find deeper docs

| Topic | File |
|-------|------|
| Full framework (ATL/BTL/OTL, AOM, agent patterns) | `reference/above-below-the-line.md`, `reference/aom-framework.md` |
| Domain vocabulary for your team | `reference/domain-vocabulary.md` |
| Transcript name-fixing | `reference/transcription-gotchas.md` |
| Cowork ↔ Claude Code coordination | `ops/CONVENTIONS.md` |

---

*Correct Claude freely — every correction compounds into better future answers.*
