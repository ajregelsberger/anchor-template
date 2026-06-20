---
name: meta-analysis
description: >
  Perform a reflective meta-analysis of a work session — audit comprehension, separate signal from noise,
  identify process patterns, and generate concrete action items for improving how this anchor performs over time.
  Use this skill whenever the user explicitly says "meta-analysis", "meta analyze this session", "run a
  meta-analysis", "retrospective", or close variations. This is NOT a session closeout — closeout captures
  what happened and generates handoffs. Meta-analysis captures how well the system performed and what should
  structurally change. Closeout is a git commit message; meta-analysis is a retrospective that improves the
  pipeline itself.
---

# Meta-Analysis

You are helping the user run a reflective meta-analysis on a work session. This is the feedback loop that makes the anchor self-improving — it's how process learnings, structural improvements, and comprehension corrections compound across sessions.

## What Meta-Analysis IS vs. What It ISN'T

| Meta-Analysis | Session Closeout |
|---|---|
| How well did the system perform? | What happened in this session? |
| What should change about how we work? | What's the handoff for the next session? |
| Signal vs. noise in context / comprehension | Files changed, decisions made |
| Structural improvements to `CLAUDE.md`, memory, conventions, templates | Reference / memory updates, open items |
| Process patterns worth replicating or avoiding | Carry-forward tasks |

Meta-analysis and closeout are complementary but separate. A session might get both — always run meta-analysis first, then closeout. The meta-analysis corrections feed into the closeout's carry-forwards, and the signal / noise separation prevents the closeout from propagating noise into the next session.

Meta-analysis is rarer than closeout — it only happens after sessions with enough substance to reflect on.

## When Meta-Analysis Adds Value

Sessions worth meta-analyzing share these traits:

- **Heavy context loading** — ingested multiple sources (transcripts, spreadsheets, strategy docs, vendor proposals, web research) and had to synthesize understanding
- **Complex reasoning** — worked through business logic, data validation, architectural decisions, or multi-step analysis
- **Domain comprehension** — Claude had to build and demonstrate understanding of a domain
- **Process experimentation** — tried a new workflow, tool, skill, or approach and learned something about what works
- **Memory / convention work** — created, compacted, or restructured persistent context

Quick lookups, simple file edits, and routine comms don't warrant meta-analysis.

## How to Run a Meta-Analysis

### Step 0: Determine the Source

Meta-analysis can run in two modes:

**Current session** (default) — Analyze the conversation you're already in. All context is in the conversation window.

**Cross-session** — The user points you at another session to analyze. Useful when the user wants to analyze a session from a fresh context window (avoiding noise that accumulated during the work itself).

To pull another session's conversation (if your harness supports it):
1. Use the session-listing tool to find the session by name
2. Use the transcript-read tool with the session ID — start with a small limit and increase if the session was long
3. If the transcript is truncated and you're missing early context, pull more messages or ask the user to fill in gaps

You're analyzing another Claude instance's work. That instance had its own context window, loaded different files, and made its own judgment calls. Read the transcript as a reviewer, not as a participant.

### Step 1: Comprehension Narrative

Tell the story of what the session worked on and what you (or the other session) understood about the domain by the end. Write this as a narrative — not bullet points, not a summary, but a story that demonstrates comprehension.

The purpose is to make the comprehension auditable. The user needs to see what Claude "thinks it knows" so they can correct it. This is where signal / noise separation starts.

**If the session hit a context compaction** (common in long sessions), reconstruct the arc from both pre- and post-compaction work. The compaction boundary often hides early-session context loading that shaped later reasoning — try to recover it from file writes, todo list state, or memory files created during the session.

Guidelines for the narrative:

- **Be specific, not vague.** Concrete claims that could be wrong are useful. Vague summaries are not.
- **Show your sources.** When you state something, note where the understanding came from (transcript, file path, doc heading). This helps the user trace whether the source was reliable.
- **Flag confidence levels.** If something is uncertain or inferred rather than stated, say so. "I believe X based on Y, but this wasn't confirmed" is better than stating X as fact.
- **Include things that confused you or seemed contradictory.** These are often the most valuable parts — they reveal where the domain itself is messy or where context was incomplete.

### Step 2: User Red-Lines

Present the narrative and wait for the user's corrections. They will mark:

- **Signal** — carry this forward, it's correct and important
- **Noise** — drop this, it's not relevant or was overfit
- **Wrong** — correct this, the understanding is off
- **Missing** — add this, something important was left out

Don't rush this step. The corrections ARE the meta-analysis. Everything that follows is derived from this exchange.

### Step 3: Process Pattern Analysis

After red-lining, step back and analyze the process itself. Look for:

**What worked well:**
- Did a particular approach to context loading prove effective?
- Did a specific tool, skill, or workflow save time or improve quality?
- Was there a moment where the right context made a complex task easy?

**What created friction:**
- Where did Claude add noise instead of signal? What caused it?
- Were there unnecessary back-and-forths that better context could have prevented?
- Did any tools or skills underperform or get used incorrectly?
- Was context overloaded — too many sources creating confusion rather than clarity?

**What was surprising:**
- Did something unexpected surface during the work?
- Were there domain insights that weren't in any existing context file?
- Did a tool or approach work better (or worse) than expected?

**Comprehension failure modes** (if applicable). Common patterns to look for:

- **Overfit** — drew conclusions from a narrow or point-in-time source, treating a snapshot as current truth
- **Hallucinated connections** — linked two things that weren't actually related because they shared surface-level similarity
- **Recency bias** — overweighted recent context at the expense of established facts or decisions
- **Source confusion** — mixed up which system produced which data, or attributed information to the wrong source
- **Terminology drift** — used incorrect terms that revealed a misunderstanding
- **Bulk-load noise** — loaded too many sources at once, creating confusion rather than clarity. The signal got buried.
- **Historical / current conflation** — didn't distinguish between how something was originally built and how it currently works

### Step 4: Structural Recommendations

Based on the corrections and process analysis, identify concrete changes that would improve the system for future sessions. These should be specific and actionable:

**Memory updates** — New facts, corrections to existing memories, or stale memories to remove. Draft the actual content, don't just say "update memory."

**`CLAUDE.md` changes** — Routing additions, convention updates. Be specific about what to add / change.

**Reference doc improvements** — Updates to `org-directory.md`, the vocab file, the workstream-state snapshot. Surface the gap and propose the addition.

**Template improvements** — If a template was used and it could be better, propose the improvement.

**Convention changes** — New rules, modified workflows, updated definitions in `ops/CONVENTIONS.md`, `reference/decision-principles.md`, or `reference/ingestion-guidelines.md`.

**Skill / tool gaps** — Skills that should exist but don't, or existing skills that need enhancement. Tools that would have helped but aren't connected.

**Context loading strategy** — If the session revealed something about what context is useful vs. polluting for a given type of work, capture that.

### Step 5: Generate the Report

Write the meta-analysis report and save it. The report has two sections: the analysis (for reference) and the action items (for execution).

**Location:** `sessions/meta-analyses/`
**Naming:** `meta-analysis_<YYYY-MM-DD>_<slug>.md`

```markdown
# Meta-Analysis: [Title]
**Date:** YYYY-MM-DD
**Source session:** [Session name / ID]
**Workstream / topic:** [Primary focus]

## Comprehension Narrative (Corrected)
[The narrative from Step 1, incorporating all corrections from Step 2. This is the "clean" version — signal only, noise removed, errors fixed.]

## Process Patterns
### What Worked
[From Step 3]

### What Created Friction
[From Step 3]

### Comprehension Failure Modes
[From Step 3, if any — what went wrong in Claude's reasoning and why]

## Action Items
### Memory / Reference Updates
| Action | File | Content / Change |
|--------|------|------------------|
| [Create / Update / Remove] | [path] | [specific change] |

### Structural Changes
| Target | Change | Rationale |
|--------|--------|-----------|
| [`CLAUDE.md` / template / convention] | [specific change] | [why] |

### Skill / Tool Gaps
| Gap | Description | Priority |
|-----|-------------|----------|
| [what's missing] | [what it would do] | [how urgent] |

## Meta-Insight
[One paragraph — the single most important takeaway from this meta-analysis that should influence how future sessions approach similar work. The thing you'd tell a fresh Claude session if you could only tell it one thing.]
```

### Step 6: Apply or Queue

After the user reviews the report:

- **Immediate applies** — memory updates, small `CLAUDE.md` fixes, terminology corrections. Do these now.
- **Queued improvements** — larger structural changes, new skills, template rewrites. Add these to a next-session prompt in `sessions/next-session-prompts/`.

Ask the user which action items to apply now vs. queue.

## Cross-Session Mode: Additional Notes

When analyzing another session's transcript:

- You're reading a conversation between the user and a different Claude instance. That instance had its own context window and made its own decisions.
- Don't assume the other instance had access to the same context you do.
- Pay attention to where the other instance got confused, backtracked, or was corrected — these are high-value signals.
- The transcript may be truncated. If critical context seems missing, pull more messages or ask the user to fill gaps.

## Quality Bar

A good meta-analysis should:

- Surface at least one comprehension correction that, if left uncaught, would have propagated to future sessions
- Identify at least one process pattern (positive or negative) that's replicable across sessions
- Produce at least two concrete, specific action items (not vague "improve X")
- Be concise enough to read in 5 minutes — this is a working document, not a thesis

A meta-analysis that produces no corrections or improvements either means the session was perfect (rare) or the analysis wasn't deep enough (more common). Push harder on the comprehension narrative if you're not finding anything.

## Relationship to Other Artifacts

**Process learnings** — If a meta-analysis surfaces a genuinely new AI process insight (not business content, not a restatement), it belongs as a durable entry in `memory/project-state.md` (or wherever this anchor stores accumulated process insights). The bar is high: the insight should be actionable for future sessions and not already captured. Most meta-analyses won't produce a new process learning — that's fine.

**Memory files** — Meta-analysis often produces memory updates (corrections, new facts, stale removals). Apply or queue in Step 6, not deferred indefinitely.

**Closeout** — If both meta-analysis and closeout happen in the same session, the meta-analysis report is a reference file for the closeout. The closeout can point to it rather than duplicating the analysis.
