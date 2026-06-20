---
name: session-close
description: >
  Standardized two-phase session close: capture what happened in this session, then generate a handoff prompt
  for the next session. Use this skill whenever the user signals end-of-session — "close out", "wrap up",
  "session done", "end this session", "closeout", "carry forward", "what did we do", "document this session",
  or any variation that implies they want the session's work captured and handed off. Even a casual "I think
  we're good here" or "that's it for today" should activate this skill. Trigger proactively — don't wait for
  exact magic words.
---

# Session Close

You are helping the user close out a Cowork session using a two-phase pattern. The goal is to capture everything valuable from this session and set up the next one — so knowledge compounds instead of evaporating.

Think of it like a git commit + merge: the session is the branch, the anchor's reference + memory files are main, and closeout is how the work gets persisted.

## Context to Load

Before starting the closeout, make sure you've read:

1. **`ops/CONVENTIONS.md`** — the coordination framework contract. Governs event log format, board updates, handoff creation, triage classification, and the assumptions register.
2. **`ops/board.md`** — current activity board state (so you can reconcile it)
3. **`ops/event-log.md`** — recent events (so you know what was already written mid-session via write-through)
4. **`memory/project-state.md`** — current running memory state and workstream snapshot
5. **`memory/MEMORY.md`** — index of all memory files

## Phase 1: Capture (The Closeout)

Review everything that happened in this session. Look at the full conversation — tool calls, file reads, file writes, decisions discussed, todo items, and artifacts produced. If the session hit a context compaction, reconstruct the full arc from both pre- and post-compaction work.

### What to Capture

Extract these categories from the session:

**Decisions Made** — What was decided? By whom? What alternatives were considered and rejected?

**Patterns Discovered** — Did we find something that works well? A new convention? A useful approach?

**Problems Encountered** — What went wrong? How was it resolved (or not)?

**Files Changed** — What files were created, updated, or reorganized? Include the path and a brief description.

**Memory Updates Needed** — Based on the session's work, which reference / memory files need updating?

### Update Memory + Reference Files

Update files in this anchor that the session changed the meaning of:

| File | Update When... |
|------|---------------|
| `memory/project-state.md` | New decisions, workstream changes, vendor relationships, or preferences emerged |
| `reference/transcription-gotchas.md` | New name corrections or known transcription errors discovered |
| `reference/org-directory.md` | New people or roles confirmed |
| `reference/[your-vocabulary-file].md` | New terms entered the working vocabulary that future sessions will need to recognize |
| Any other `reference/*.md` | Content drifted; needs refresh |

The bar for adding a new entry: would a future session need this to be correct in 30 days?

### Write the Closeout Document

Save to `sessions/` (create a subfolder if useful for organization) using the naming convention:

```
<workstream-or-topic>_<YYYY-MM-DD>_<short-slug>_closeout.md
```

The closeout document should include:

```markdown
# Session Closeout: [Brief Title]
**Date:** YYYY-MM-DD
**Workstream/Topic:** [Primary focus]
**Duration:** [Approximate]

## What Was Done
[Narrative summary of the session — what was the goal, what was accomplished]

## Key Decisions
- [Decision]: [Rationale]

## Problems & Resolutions
- [Problem]: [How resolved, or "unresolved — carry forward"]

## Memory / Reference Updates
| File | Change | Why |
|------|--------|-----|
| [file] | [what changed] | [why it matters] |

## Carry-Forward Items
- [ ] [Item] — Target: [which next session or surface should pick this up]

## Routing Check
- [x] All new/updated files reachable from `CLAUDE.md` routing table
- [ ] [Any routing entries that need adding]
```

### What Does NOT Belong in a Closeout

- Raw conversation logs
- Speculative ideas that weren't tested or discussed
- Duplicate information already captured in reference / memory files
- Detailed code diffs (those belong in PRs, not closeouts)

## Phase 1b: ops/ Reconciliation

After capturing session content and updating memory / reference files, reconcile the `ops/` coordination layer.

**If write-through was practiced during the session** (events appended as they happened, board updated on state changes), this step is lightweight — just a final consistency check.

**If write-through was not practiced**, this reconciliation is heavier. Backfill key events to the event log now. To prevent this next time, ensure the `session-start` skill is used at session open — it establishes the write-through contract.

### Event Log

Append a `session-close` event to `ops/event-log.md` summarizing what the session accomplished. If mid-session write-through was missing, backfill key events. See `ops/CONVENTIONS.md` for event format.

### Board Update

Reconcile `ops/board.md` to reflect end state:
- Move this session's In Flight entry to Recently Completed (if it was registered via `session-start`)
- If this session was not registered in In Flight, skip but note the gap
- Remove any Blockers the session resolved
- If the session consumed handoffs from Code (`ops/handoff/above/`), move those files to `ops/handoff/above/handled/`, add a `## Resolution` section to each, and move them from Handoff Queue to the board's Handled section

### Handoffs (if applicable)

If the session produced work that Code (below the line) needs to act on — session prompts, build requests, research asks, bug reports — create a handoff file in `ops/handoff/below/` using the format in `ops/CONVENTIONS.md`.

Assign a triage color:
- 🟢 **Green** — informational, reversible, or following an already-approved pattern
- ⚡ **Yellow** — introduces something new or requires a judgment call with a reasonable default
- 🔴 **Red** — changes scope, timeline, architecture, or affects people outside the AI system

Append a `handoff-created` event to the log and add it to the board's Handoff Queue.

Per Decision Principle #7 (in `reference/decision-principles.md`): create the handoff at the topic boundary. If carry-forwards target Code, create the handoff now rather than deferring to a future session.

### Assumptions

If any judgment calls were made during the session that aren't covered by existing conventions or decision principles, log them in `ops/assumptions.md`. This is the feedback loop that makes the system self-improving — repeating assumptions get promoted to rules, wrong assumptions get corrected.

## Phase 2: Handoff (Next-Session Prompt)

After the closeout is written, identify carry-forward items and generate next-session prompts.

### Identify Carry-Forwards

Look for:
- **Unfinished work** — tasks that were started but not completed
- **Deferred decisions** — things marked "we'll deal with later"
- **Blocked items** — work that can't proceed until something else happens
- **Open questions** — things that need answers from other people or systems
- **Follow-up tasks** — natural next steps from what was accomplished

### Route Carry-Forwards

Carry-forwards don't all go to the same next session:
- Some may go back to the same workstream
- Some may go to a different workstream entirely
- Some may need user action outside Claude (a Slack message, a meeting ask, etc.)

Generate a separate next-session prompt for each distinct target session.

### Write Next-Session Prompts

Save each to `sessions/next-session-prompts/` using the convention:

```
<workstream-or-topic>_next_session_prompt.md
```

Each prompt should follow this structure:

```markdown
# Next Session Prompt — [Title]
**Generated:** YYYY-MM-DD (from [source session slug])
**Type:** [What kind of session — ingestion, planning, build, etc.]
**Estimated effort:** [Rough time estimate]

---

## Session Objective
[One sentence — what should the next session accomplish?]

## Walk-in State
[What the next session needs to know about the current state when it opens — what's already in flight, what's been decided, what's still open]

## What to Do
1. [Specific task]
2. [Specific task]
3. [Specific task]

## Reference Files
| File | Why |
|------|-----|
| `path/to/file` | [Why the next session needs this] |

## Rules for This Session
- Read all reference files before starting work
- [Any session-specific rules from carry-forward context]
```

The handoff prompt should be specific enough that a fresh Claude session can pick it up and know exactly what to do. "Continue working on the project" is useless. "Draft the evaluation memo using the structure in `reference/build-vs-buy.md` — pull cost data from the proposal at [path]" is useful.

## Phase 3: Verify

Before declaring the closeout complete:

1. **Routing check** — Can every new or updated file be found by following the `CLAUDE.md` routing table? If not, add the missing routing entries now.
2. **Orphan check** — Did you create any files that aren't referenced from anywhere? Add references.
3. **Memory consistency** — Do the memory / reference updates you just made align with each other?
4. **Board accuracy** — Does `ops/board.md` accurately reflect post-session state? Is this session no longer In Flight? Handoffs queued where appropriate?
5. **Event log completeness** — Does `ops/event-log.md` have a `session-close` event? Were significant mid-session events missed?

If new routing entries are needed, make them. The session that creates the artifact owns the routing update.

## Frequency Guidelines

Not every session needs the full two-phase close:

| Session Type | What to Do |
|-------------|-----------|
| Short focused task (< 1 hour) | Phase 1 closeout. Handoff only if there's unfinished work. |
| Long deep session (multi-hour) | Full two-phase close. Always generate handoff. |
| End of day with multiple sessions | Close out each one; the last session of the day handles dedup and any merge conflicts. |
| Dead-end session | Closeout documents *why* it failed — so the next session doesn't repeat it. |
| Quick question / lookup | No formal closeout needed. |

Adjust the depth of the closeout to match the session's depth. A 20-minute focused task doesn't need a 200-line closeout document.

## What This Skill Does NOT Do

- Does not close out Claude Code sessions (those have their own git/PR convention)
- Does not push anything to an external mount — closeout is a working-session action
- Does not generate stakeholder updates — that's a separate process
- Does not update CLAUDE.md unless routing entries are needed for new artifacts
