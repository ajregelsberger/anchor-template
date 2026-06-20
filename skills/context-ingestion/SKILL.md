---
name: context-ingestion
description: >
  Process meeting transcripts, spreadsheets, vendor docs, and other raw inputs into this anchor's structured
  memory + reference system. Use this skill whenever the user mentions transcripts, meeting notes, data refresh,
  ingestion, context updates, processing inputs, diffing spreadsheets, or uploads a transcript (.md),
  a spreadsheet (.xlsx), a strategy document, or a vendor proposal and asks to extract signal or update context.
  Also trigger when the user says "ingest this", "process this meeting", "diff it", "update memory from this",
  or any variation of processing raw information into structured memory. Even if the user just drops a file
  without explicit instructions, this skill should activate.
---

# Context Ingestion

You are helping the user process raw inputs (transcripts, spreadsheets, strategy docs, vendor proposals, or any mix) into the anchor's structured reference and memory system. This is a repeatable workflow that runs every time new material comes in.

The goal: turn messy inputs into clean, trusted, routable context — without introducing "almost right" information that silently propagates errors downstream.

## Before You Start

Load these files in this order (the sequence matters — earlier files shape how you interpret later ones):

1. **`CLAUDE.md`** (anchor root) — folder structure and routing rules
2. **`CLAUDE.md` routing table** — already loaded in Step 1; use it to find any file in this anchor. No separate routing-map file exists.
3. **`ops/CONVENTIONS.md`** — coordination framework contract (event log format, board updates, handoff creation, triage classification). Ingestion produces events, may update the board, and may create handoffs.
4. **`reference/transcription-gotchas.md`** — known name/term corrections for this anchor. Cross-check every person mentioned in inputs against this file. Full org-directory: `reference/org-directory.md` (a local file for this anchor — see SETUP.md for how to create it)
5. **`reference/ingestion-guidelines.md`** — the confidence tier framework. This is the processing rulebook — read it fully before touching any input.
6. **`memory/project-state.md`** — current state and workstream snapshot. Needed to distinguish new signal from already-tracked items.
7. **`memory/MEMORY.md`** — index of all memory files; read alongside `memory/project-state.md` for the full picture.

All paths are relative to the anchor root.

## Identify Your Inputs

Check `inbox/` (and `inbox/archived/` for prior context) first. The inbox is the landing zone for automated inputs — scheduled transcript pulls, email extracts, spreadsheet updates. If there are unprocessed files there, those are typically your inputs.

A common naming convention for inbox files:

```
<source>_<YYYY-MM-DD>_<slug>.<ext>
```

Examples:
- `transcript_2026-03-26_product-council.md`
- `spreadsheet_2026-03-26_workstream-status-update.xlsx`
- `proposal_2026-04-12_vendor-name.pdf`
- `projecttracker_YYYY-MM-DD_PM.md`

If the inbox is empty or the user provides inputs directly (upload, paste, or file path), use those.

| Input Type | What to Look For |
|-----------|-----------------|
| **Transcript only** | A meeting transcript (`.md`, pasted notes, or recorded dialogue) |
| **Spreadsheet only** | A structured data file (often operational or financial) |
| **Strategy doc / vendor proposal** | A long-form document (`.docx`, `.pdf`, `.md`) with proposals, terms, or analysis |
| **Combined** | Multiple inputs covering the same period — the transcript often explains *why* the spreadsheet changed |

After processing, move inbox files to `inbox/archived/` (or an appropriate subfolder of `sessions/`) so the inbox stays clean for the next pull.

## Phase 1: Process Transcripts

Skip this phase if no transcript was provided.

### Step 1: Name + Term Correction Pass

Before extracting any signal, cross-reference **every name and product/term mention** against `reference/org-directory.md`. Auto-transcription tools regularly mangle names and domain terminology.

- Check the "Transcription Gotchas" tables for known errors
- If a name doesn't match anyone and could be a transcription error, flag it — don't create a new person entry
- Conference room identifiers are not speakers — verify by reading what was said and matching to the speaker's known role + content patterns

### Step 2: Signal Extraction

Read the full transcript and extract structured signal into these categories:

- **Decisions** — what was decided, by whom, with what rationale
- **Status changes** — workstream / product / project status shifts
- **Timeline updates** — new dates, slipped dates, hard deadlines
- **Ownership changes** — new owners, transitions
- **New initiatives** — new scope, new projects, new commitments
- **Action items** — who owes what, by when
- **Risks / blockers surfaced** — what's at risk; what's blocked
- **Organizational / process signal** — role changes, reporting structures, cadence changes

For each piece of signal, note **who said it** and **whether it's a direct quote or a paraphrased summary**.

### Step 3: Confidence Classification

This is the critical step. For every piece of extracted signal, assign a confidence tier per `reference/ingestion-guidelines.md`:

**Tier 1 (High) — Commit directly:**
- Direct quotes from named speakers
- Confirmed by multiple speakers or sources
- The user has explicitly confirmed it
- Comes from structured data (spreadsheet cells, system exports)

**Tier 2 (Medium) — Commit with source annotation:**
- Meaning inferred from context but not explicitly stated
- From auto-transcribed narrative (not quoted speech)
- Attribution is ambiguous

**Tier 3 (Low) — Flag for the user, do NOT commit:**
- Unrecognized names that could be transcription errors
- Contradicts existing reference / memory entries
- Appears only in the AI-generated summary section (not in the detailed transcript)
- Would change an existing entry's meaning rather than add to it

Remember the core principle: **the cost of "almost right" is higher than the cost of "unknown."** When in doubt, flag rather than commit.

**Source attribution rule (from `ops/CONVENTIONS.md`):** Every claim committed to canonical memory must be traceable to a source — transcript filename + timestamp, Slack permalink, Drive doc, or explicit user confirmation. Inference-from-context claims are Tier 2 at best.

### Step 4: Cross-Reference

Compare extracted signal against the workstream-state snapshot and the running memory:
- Is this genuinely new, or already tracked?
- Does it update an existing item's status, or conflict with it?
- Are there new people, terms, or acronyms that need org-directory entries?

## Phase 2: Process Spreadsheets

Skip this phase if no spreadsheet was provided.

### Step 1: Load and Diff

- Read the new spreadsheet programmatically (use pandas or openpyxl)
- Find and read the prior baseline version (if one exists in this anchor's `saddle/` or a designated baseline folder)
- If no baseline exists, treat the entire spreadsheet as new data and note this

### Step 2: Generate Delta Report

Compare the two versions and report:
- **Added rows** — entirely new entries
- **Removed rows** — entries that disappeared
- **Changed values** — column-by-column changes for each modified row
- **Structural changes** — new columns, removed columns, renamed headers
- **Color changes** — if the spreadsheet uses color coding, extract and compare colors between versions

### Step 3: Save New Baseline

Save the new spreadsheet alongside the baseline as the updated reference for next time.

## Phase 3: Process Strategy Docs / Vendor Proposals

Skip if none provided.

### Step 1: Pull Every Concrete Commitment

When ingesting a vendor proposal, strategy memo, or competitive analysis:
- Pull every number (dates, dollar amounts, percentages, headcounts)
- Pull every named contact (vendor people, internal stakeholders, decision-makers)
- Pull every explicit commitment (SLAs, exclusivity terms, deliverable dates)
- Flag every implicit assumption that hasn't been stated

### Step 2: Compare to Current State

Cross-reference against the workstream-state snapshot:
- Does this align with what we've already decided?
- Where does it conflict with existing constraints or commitments?
- What does it open as a new decision the user has to make?

### Step 3: Surface Trade-offs

Don't synthesize one-sided. If the doc proposes option A and rejects option B, surface what option B's case would have been, and where the proposal has built-in assumptions worth questioning.

## Phase 4: Reconcile (Combined Inputs Only)

If you processed multiple input types:
- The transcript often explains *why* spreadsheet values changed — connect the dots
- The strategy doc often references decisions made in the transcript — verify alignment
- Surface any conflicts between sources
- Use the Source Reliability Hierarchy from `reference/ingestion-guidelines.md` to weigh conflicting sources

## Phase 5: Update Reference + Memory Files + ops/ Layer

Now act on your processed signal. Only commit Tier 1 and Tier 2 items.

### `ops/event-log.md`

Append a `knowledge-update` event noting what was processed and what was updated:

```markdown
## YYYY-MM-DD HH:MM | cowork | knowledge-update

Ingested <input filename(s)>. Updated <count> entries in workstream-state; <count> entries in org-directory; <count> gotchas added. <count> Tier 3 items flagged for user.

→ Delta summary: sessions/<path>/ingestion_<date>_summary.md
```

### `ops/board.md`

If the ingestion surfaces state changes that should be visible at-a-glance:
- **Blockers discovered** — add to Blockers
- **Existing items resolved** — move to Recently Completed
- **Items that need to cross the line** — add to Handoff Queue (see handoffs below)
- **Business state changes affecting in-flight work** — note on relevant In Flight entries

Not every ingestion updates the board — only when ingested signal changes the current operational picture.

### Handoffs (if applicable)

If the ingestion surfaces signal that Code (below the line) needs to know about — a technical decision from a meeting, a new requirement, a scope change, a bug reported — create a handoff in `ops/handoff/below/` using the format in `ops/CONVENTIONS.md`.

Assign a triage color and append a `handoff-created` event.

### `memory/project-state.md`

- Refresh the relevant sections with current state from this ingestion
- Update status indicators, dates, ownership as warranted
- If new information contradicts an existing entry, surface the conflict in the delta summary rather than silently overwriting

### `reference/transcription-gotchas.md`

- Add new name/term corrections — these are always high-value, low-risk additions
- For broader org-directory updates (new people, roles): `reference/org-directory.md`
- Add new terms / acronyms only if explicitly defined or used with consistent meaning
- Never invent definitions by pattern-matching to similar-sounding industry terms

### `memory/project-state.md` (continued)

- Append a dated entry if new context emerged about user preferences, decisions, vendor relationships, working style
- Keep entries short (2–3 lines each)
- Format: `### YYYY-MM-DD` + `[decision/workflow/vendor/strategy/people/preference/correction]` tags

### `ops/assumptions.md`

If any judgment calls were made during ingestion that aren't covered by the ingestion guidelines — interpreting an ambiguous speaker attribution, deciding a Tier 2 item is safe to commit, classifying a handoff triage level — log them for the user's review.

## What Goes Where

| Signal type | Write to |
|-------------|----------|
| Named decision | `memory/project-state.md` |
| Action item (person + task) | `memory/project-state.md` |
| Blocker | `memory/project-state.md` + `ops/board.md` |
| Strategic context | `memory/project-state.md` |
| Domain-specific finding | `memory/project-state.md` + potential project tracker ticket |
| Feedback on working style | `memory/feedback-working-style.md` |
| Info about a person's role | `memory/user-{{YOUR_NAME}}.md` or a reference file |

## Phase 6: Produce Delta Summary

Create a summary document saved to `sessions/`:

```
<workstream-or-topic>_ingestion_<YYYY-MM-DD>_summary.md
```

The summary should include:

1. **Inputs Processed** — what was ingested (filenames, dates, sources)
2. **Spreadsheet Delta** (if applicable) — the diff results
3. **Transcript Signal** (if applicable) — categorized extracted signal with confidence tiers
4. **Strategy Doc Signal** (if applicable) — commitments + open questions extracted
5. **Memory / Reference Files Updated** — table showing which files changed and what
6. **Items Needing User Attention** — all Tier 3 items, decision points, and conflicts. This is the most important section.
7. **Recommended Downstream Actions** — suggestions for what to do next

## Phase 7: Verify

Before finishing:

1. **Routing check** — are all updated / created files reachable via `CLAUDE.md` / `reference/routing-map.md` chain?
2. **Org-directory integrity** — do new entries have correct spelling, role, and disambiguation notes?
3. **New concept reachability** — if new terms or workstreams were introduced, can a future session find them by following the routing surface?
4. **Event log and board consistency** — does the event log have a `knowledge-update` event? Are handoffs (if any) on the board?

Flag any issues found rather than silently ignoring them.

## What This Skill Does NOT Do

- Does not rebuild downstream artifacts (dashboards, project plan spreadsheets) — recommends changes only
- Does not prescribe priority changes — surfaces them as recommendations
- Does not fabricate signal — if the input is ambiguous, says so
- Does not send communications — drafts only, user fires
- Does not write application code — this is orchestration-layer work
