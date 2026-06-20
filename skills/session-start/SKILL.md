---
name: session-start
description: >
  Warm-up orientation for new Cowork sessions — loads the context chain, registers in ops/, checks for staleness,
  finds queued work, and helps the user decide what to work on. Use this skill at the beginning of any session when
  the user hasn't given a specific task yet, or when they say things like "let's get started", "new session",
  "what's the state of things", "catch me up", "what's on deck", "where did we leave off", "what's queued",
  "morning", "what should I work on", or any signal that they're orienting rather than diving into a specific task.
  Also trigger when the session opens and the user's first message is broad or exploratory rather than task-specific.
  This skill is read-only orientation PLUS ops/ registration — it helps the user see the landscape and establishes
  the session's participation in the coordination framework before any work begins.
---

# Session Start

You are helping the user orient at the beginning of a new Cowork session. The goal is to load context, register this session in the `ops/` coordination framework, surface what's current, what's stale, and what's queued — then let the user decide what to work on.

Think of this as the "morning standup" for the user's work — quick status, what's hot, what needs attention, and where to focus. It also establishes the behavioral contract for the rest of the session: write-through to the event log, board awareness, and triage-conscious handoff handling.

## Step 1: Load the Context Chain

Read these files in order. The sequence matters — each file builds on the previous one's context:

1. **`CLAUDE.md`** (anchor root) — identity, folder structure, routing rules, working surfaces
2. **`CLAUDE.md` routing table** — already loaded in Step 1; the routing table maps every file in this anchor. Use it as the navigation surface — no separate routing-map file exists.
3. **`reference/transcription-gotchas.md`** — known name/term corrections for this anchor (always load — applies to every session). Full org-directory: `reference/org-directory.md (create this file for your anchor — see SETUP.md)`
4. **`ops/board.md`** — activity board showing what's in flight, queued handoffs, blockers
5. **`ops/CONVENTIONS.md`** — the coordination framework contract (event log format, handoff lifecycle, triage classification, assumptions register, write-through discipline). You need this before doing any `ops/` writes.
6. **`memory/project-state.md`** — current state, open items, and workstream snapshot for this anchor.
7. **`memory/MEMORY.md`** — index of all memory files; read alongside `memory/project-state.md` for the full picture
8. **Any role-specific vocabulary or reference files you've added to `reference/`**

If any of these files are absent in a given anchor, skip them and note the gap.

## Step 1b: Register in ops/

This session is now part of the coordination framework. Before doing any other work, register it:

1. **Append a `session-start` event** to `ops/event-log.md`. Include what the session's focus area appears to be (or "orienting — focus TBD" if the user hasn't indicated direction yet). Use the event format from `ops/CONVENTIONS.md`.

2. **Add an "In Flight" entry** to `ops/board.md` with owner `cowork`, a brief description of the session's focus, and today's date.

This establishes the write-through contract from the very first action. For the rest of the session, events should be appended to the log as they happen — not batched until closeout.

## Step 2: Check Freshness

For each file you loaded that has a "Last Updated" date in its header, check freshness:

| Freshness | Meaning |
|-----------|---------|
| Updated within 3 days | **Current** — no action needed |
| Updated 4–7 days ago | **Aging** — mention it, but not urgent |
| Updated 8+ days ago | **Stale** — flag prominently, recommend updating before starting new work |

If a critical file (the workstream-state snapshot, the org-directory, or this anchor's primary context store) is stale, recommend a refresh before diving into new work. Stale context leads to stale decisions.

## Step 3: Check for Queued Work

Check these sources for pending work:

### Handoff queue (from `ops/board.md`)
The board's Handoff Queue section shows items waiting to cross the ATL/BTL line. If there are handoffs queued for Cowork (direction: ↑ above), surface them. Also check `ops/handoff/above/` for pending handoff files.

For each pending handoff, note its triage color:
- 🟢 **Green** — can be picked up and acted on immediately
- ⚡ **Yellow** — can be picked up, but the user should review any assumptions made
- 🔴 **Red** — holds until the user explicitly approves

### Next-session prompts
Scan `sessions/next-session-prompts/` for queued prompts (files not prefixed with `CONSUMED_`). For each one:
- Read the title and session objective
- Note when it was generated
- Note the session type (planning, ingestion, build, etc.)

### Recent closeouts
Check `sessions/` (and any subfolders) for the most recent closeout document. If it flagged carry-forwards, surface them. This handles the "where did we leave off" case.

### Recent agent events
Check the last 10–15 events in `ops/event-log.md` for recent `agent-run` events from any scheduled agents the user has wired up. Agents may have surfaced signal that the session should know about, even if it didn't create a board-level blocker. Non-blocking agent intelligence gets missed if you only read the board.

### When the user picks a handoff
If the user chooses to work on a pending handoff:
1. Update the handoff file's `**Status:**` field to `in-progress`
2. Append a `handoff-consumed` event to `ops/event-log.md`
3. Update the board to reflect the status change
4. Follow the full handoff lifecycle in `ops/CONVENTIONS.md`

## Step 4: Orient and Present

Give the user a brief, scannable status summary:

### What's Hot
Items that are critical, blocked, or time-sensitive. Keep to 3–5 items max — if everything is "hot," nothing is.

### What's Stale
Files or workstreams that haven't been updated recently. Include why it matters (e.g., "workstream-state snapshot is 9 days old — a refresh may be warranted before strategic planning").

### What's Queued
Pending handoffs (with triage color) and next-session prompts waiting to be picked up, with their one-sentence goals and dates.

### What's Pending Review
Check `ops/assumptions.md` for unreviewed entries. Flag prominently if a review cadence has been set (e.g., Friday weekly review); mention briefly if entries are accumulating.

### Suggested Direction
Based on what you see, suggest what the user might want to focus on. Consider:
- Is there stale context that should be refreshed first?
- Are there queued prompts that are time-sensitive?
- Did a recent closeout flag urgent carry-forwards?
- Are there red-triage handoffs that have been sitting for 48+ hours?
- Did agents surface anything notable?

Frame these as suggestions, not directives. The user makes the call.

**Session-start is not complete until the live sources are checked.** Query your live project tracker ({{PROJECT_TRACKER}}), communication tools, and any other live sources configured for your anchor.

After checking, note the status line:

```
read: CLAUDE.md ✓ memory ✓ board ✓ | live: {{PROJECT_TRACKER}} ✓ {{COMMUNICATION_TOOL}} ✓
```

See `reference/aom-framework.md` for the principle behind live-source checking.

## Step 5: Route to the Right Context

Once the user indicates what they want to work on, use the routing table in `CLAUDE.md` to suggest which additional reference files to load. The routing table is the authoritative navigation surface for this anchor — consult it to find the right entry point for any topic.

## Ops Participation Reminder

This session participates in the `ops/` coordination framework. As you work through it:

- **Event log** (`ops/event-log.md`) is the source of truth. Append events as they occur — decisions, findings, deliveries, handoff creation/consumption.
- **Board** (`ops/board.md`) reflects current state. Update it when state changes (items complete, blockers arise, handoffs move).
- **Write-through** is the expectation. Don't wait for closeout to capture what happened. A finding written in the moment has full context; a finding reconstructed at closeout is a summary of a memory.
- **Assumptions register** (`ops/assumptions.md`). If you make a judgment call not covered by existing conventions or decision principles, log it for the user's review.

## What This Skill Does NOT Do

- Does not auto-execute queued session prompts — presents them as options for the user to choose
- Does not update memory files or reference docs — it reads them for orientation only
- Does not assume the session type — waits for the user to indicate direction
- Does not replace the routing map — it uses the routing map to help the user navigate

## Tone

Keep the orientation brief and scannable. The user wants to quickly see what matters and make a decision. Think dashboard, not report.

If everything is current, nothing is queued, and there's nothing urgent, just say so: "Everything looks current. No queued sessions. What would you like to work on?"
