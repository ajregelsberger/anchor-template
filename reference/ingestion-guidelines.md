# Ingestion Guidelines

> How to process transcripts, spreadsheets, and other documents into memory. The confidence-tier system determines what gets committed and what gets flagged.

---

## What "Ingestion" Means

Ingestion = extracting signal from a source document and writing durable facts into `memory/project-state.md` (or other memory files). The goal is to capture what's actually true and actionable — not to summarize the meeting.

**The session-level workflow:** drop a transcript or doc, say "process this," and the `context-ingestion` skill fires. But the underlying rules below govern how that skill operates.

---

## Confidence Tiers

Every fact extracted from a source document is assigned a tier based on certainty.

### Tier 1 — Commit
**Criteria:** Direct, structured, named, unambiguous.

Examples:
- "{{TEAM_MEMBER_1}} will have the critical external deliverable done by Friday"
- "We decided: {{YOUR_PROJECT_TRACKER}} is the single source of truth, [old tool] is sunset"
- "TASK-19 is due 2026-06-24, assigned to {{YOUR_NAME}}"
- A numeric result from a query: `SELECT COUNT(*) FROM records WHERE...` → 1,247 rows

**Action:** Write directly to `memory/project-state.md` or the relevant memory file. No flag needed.

---

### Tier 2 — Commit with flag
**Criteria:** Indirect, inferable, strong contextual signal — but not a direct statement.

Examples:
- "{{YOUR_NAME}} seemed to take ownership of a task without being explicitly assigned"
- "{{TEAM_MEMBER_2}} implied the decision deadline is EOM, but didn't state it explicitly"
- "The transcript said 'we'll fix that' without naming who 'we' is"

**Action:** Write to memory with a 🚩 flag and a source note. Example:
```
🚩 Assumed: {{YOUR_NAME}} owns [task] (inferred from context — confirm)
```

---

### Tier 3 — Surface, don't commit
**Criteria:** Unclear speaker, ambiguous context, possible mishearing, or conflicting signals.

Examples:
- Multiple people talking over each other with no clear resolution
- A term heard but context doesn't make it clear whether it's a domain concept or a named item
- A decision mentioned but contradicted later in the same transcript
- Unknown participant name

**Action:** Surface as a question to {{YOUR_NAME}} in the session response. Do not write to memory. Document the ambiguity.

---

## Source Document Types

### Meeting Transcripts (Google Meet / Granola)
**Primary signal:** decisions, action items, blockers, context

Process:
1. Correct transcription errors using `reference/transcription-gotchas.md` before extracting signal
2. Extract Tier 1: explicit decisions and named action items
3. Extract Tier 2: inferred ownership, implied deadlines
4. Flag Tier 3: anything ambiguous
5. Update `memory/project-state.md` with new items; merge with existing rather than duplicate
6. Append to `ops/event-log.md`: `## YYYY-MM-DD | cowork | finding` with what was ingested

### Spreadsheets / Project Tracker Exports
**Primary signal:** current status of issues, metrics, dates

Process:
1. Check dates: are they current (within the last week)? If older, flag as potentially stale
2. For project tracker data: update the relevant board section of `memory/project-state.md`
3. Numeric outputs from spreadsheets: commit with source date

### Slack Digests (from signal monitor)
**Primary signal:** decisions made in Slack that didn't make it to your project tracker

Process:
1. Review each signal item
2. If it's a decision → assess whether it should create a project tracker ticket (flag for {{YOUR_NAME}})
3. If it's a blocker → update `ops/board.md` and flag in project-state.md
4. If it's informational → no memory write needed; archive the digest

---

## What Goes Where

| Signal type | Destination |
|------------|-------------|
| Named decision | `memory/project-state.md` → Shared / Team Decisions section |
| Action item (person + task) | `memory/project-state.md` → relevant active work section |
| Blocker (waiting on X) | `memory/project-state.md` + `ops/board.md` |
| Strategic context | `memory/project-state.md` → Open Strategic Threads |
| Domain-specific finding | `memory/project-state.md` → relevant active work + potential project tracker ticket |
| Feedback on working style | `memory/feedback-working-style.md` |
| Info about a person's role or relationship | `memory/user-{{YOUR_NAME}}.md` (for you) or a reference file |

---

## What Not to Commit

- **Speculation:** "{{YOUR_NAME}} might want to..." → don't commit
- **Superseded decisions:** if a previous memory entry is overruled, update it — don't add a new conflicting entry
- **Operational detail:** meeting logistics, room bookings, "good to see you" intros
- **Anything not traceable** to a source event → surface as Tier 3 or discard

---

## After Ingestion — Checklist

- [ ] `memory/project-state.md` updated with new items/decisions
- [ ] `ops/event-log.md` has a `finding` entry with what was ingested and what changed
- [ ] Ambiguous items have been surfaced to {{YOUR_NAME}} (not silently dropped)
- [ ] 🚩 flags on Tier 2 items are visible and can be confirmed/denied
- [ ] Source (transcript name + timestamp) is noted for any committed fact

---

*Transcription name/term fixes: `reference/transcription-gotchas.md`*  
*AOM confidence-tier principles: `reference/aom-framework.md`*
