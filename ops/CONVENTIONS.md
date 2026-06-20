# Ops Conventions — Event Sourcing & Coordination Infrastructure

> **Owner:** {{OPERATOR_NAME}}
> **Created:** 2026-03-30
> **This is a living document.** Update conventions as the system evolves.

---

## Purpose

This directory is the coordination layer between Cowork (above the line) and Claude Code (below the line). It implements event sourcing principles: every action is recorded as an immutable event, current state is derived from the event stream, and handoffs flow bidirectionally between the two tools.

The `ops/` directory is tool-neutral. Both Cowork and Code read from and write to it. Scheduled agents participate as event sources. {{OPERATOR_NAME}} is the only human who approves cross-boundary handoffs.

---

## Directory Structure

```
ops/
├── CONVENTIONS.md       ← You are here. Rules of the system.
├── event-log.md         ← Append-only event stream (source of truth)
├── board.md             ← Activity board — current state (materialized view)
├── assumptions.md       ← Judgment call register — reviewed weekly by {{OPERATOR_NAME}}
└── handoff/
    ├── above/           ← Code → Cowork (pending items)
    │   └── handled/     ← Completed handoffs (visible history)
    └── below/           ← Cowork → Code (pending items)
        └── handled/     ← Completed handoffs (visible history)
```

---

## Event Log

### What It Is

The event log is an append-only record of meaningful work events across all tools and agents. It is the source of truth for "what happened." The board, handoffs, and knowledge base are all downstream views.

### Event Format

Every event follows this structure:

```markdown
## YYYY-MM-DD HH:MM | <source> | <event-type>

<1-3 line description of what happened and why it matters>

→ <action taken or artifact produced, if any>
```

**Source values:**
- `cowork` — Cowork session (above the line)
- `code` — Claude Code terminal session (below the line)
- `code(ide)` — Claude Code IDE session (below the line, visual QA capability)
- `agent:<name>` — Scheduled agent (e.g., `agent:slack-signal-monitor`)
- `operator` — Manual entry by {{OPERATOR_NAME}}

**Source identity rule:** The source field reflects **which tool is writing the event**, not which tool's work is being discussed. A Code session that summarizes Cowork activity still writes `code | session-start`. A Cowork session consuming a Code handoff still writes `cowork | handoff-consumed`. Your identity is set at session start and does not change based on content.

**Event types:**
- `session-start` — A session opened and oriented
- `session-close` — A session completed with closeout
- `handoff-created` — Work item queued for the other side
- `handoff-consumed` — Queued item picked up and acted on
- `finding` — Discovery worth persisting (architectural, process, domain)
- `decision` — A decision was made that affects future work
- `blocker` — Something is blocked and needs resolution
- `delivery` — A concrete output was produced (PR, doc, artifact)
- `agent-run` — Scheduled agent completed a run
- `knowledge-update` — Memory or knowledge base was updated

### Writing Rules

1. **Append only.** Never edit or delete existing events. The log is immutable.
2. **Write as you go.** Don't wait for session close. Record events when they happen.
3. **One event per action.** Don't batch multiple things into one event.
4. **Be specific.** "Worked on a feature" is noise. "Generated routing scaffold for Feature Build v2, 12 routes across 3 modules" is signal.
5. **Include the "so what."** Every event should make clear why a future session would care.
6. **Newest at the bottom.** Append to the end of the file. Chronological order, always.

### Volume Rotation (Never Delete)

The event log is the permanent audit trail. Events are **never deleted or summarized** — they are the immutable record of everything that happened across all tools. When the active log exceeds ~500 lines, rotate to a new volume:

1. Rename the current log to `ops/archive/event-log-YYYY-MM.md` (e.g., `event-log-2026-03.md`)
2. Start a fresh `ops/event-log.md` with a header noting the previous volume
3. The full history is always reconstructable by reading volumes in order
4. The board remains current regardless — it's the derived view, not the raw stream

**The rule:** the event log is the audit trail. If it's not in the log, it didn't happen. If it was in the log, it's never lost.

---

## Session Identity

### What It Is

Every Cowork session has a name assigned by Cowork (visible in the left sidebar). When writing to `ops/` (event log, board.md, closeouts, handoffs, snapshots), use this Cowork-assigned identifier as the session's name.

Session naming convention depends on your Cowork version — ask {{OPERATOR_NAME}} at orientation.

### Rules

1. **The Cowork sidebar is the authoritative source for session identity.** Never synthesize a session ID by incrementing the highest value found in `board.md`'s In Flight column — `board.md` is a downstream artifact, not the source.
2. **Get the session ID at orientation.** At session start, before writing any ops/ event, identify this session's Cowork name. Two paths:
   - **Preferred:** Ask {{OPERATOR_NAME}} at orientation if it's not already obvious from context ("what's this session's name in your sidebar?"). One short message.
   - **Programmatic (limited):** `mcp__session_info__list_sessions` returns session names for *other* sessions visible to you, but **does not include the current session in its own results**. So this tool helps cross-reference siblings, not self-identify.
3. **Cowork session names are not guaranteed globally unique.** Use the name as a disambiguator within a working window, not as a primary key.
4. **If you discover a wrong session ID mid-session, fix the writes immediately.** Use sed or equivalent across ops/event-log.md and ops/board.md before continuing. Do not silently leave inconsistent writes.
5. **Non-standard names are fine.** If the session is a scheduled agent run or has a different prefix format, use that name as-is.

---

## Activity Board

### What It Is

The board is a materialized view of current state. It answers: "What's happening right now across both tools?" Any session (Cowork or Code) reads the board to orient before starting work. It replaces the need to read multiple memory files to understand current state.

### Board Structure

The board has five sections:

1. **In Flight** — Work actively being done. Each item has an owner (cowork/code/agent), a one-line description, and when it started.
2. **Handoff Queue** — Items waiting to cross the line. Each item has a direction (above/below), a summary, and a link to the handoff file.
3. **Handled** — Handoffs that have been picked up and completed. Visible history of what's been done. Each item shows direction, summary, who handled it, and when. Items age off to the archive after 30 days.
4. **Recently Completed** — Last 10 completed work items (not handoffs — those go to Handled). Provides context for what just happened. Items age off after 7 days.
5. **Blockers** — Anything stuck. Each item has an owner, what's blocked, and what's needed to unblock.

### Update Rules

1. **Update the board when you update the event log.** Every event that changes current state should be reflected on the board. If you append a `session-start` event, add an entry to In Flight. If you append a `delivery` event, move the item from In Flight to Recently Completed.
2. **The board is mutable.** Unlike the event log, the board is edited in place. It reflects current state, not history.
3. **Keep it lean.** The board should be readable in 30 seconds. If it takes longer, something needs to move to Recently Completed or be archived.
4. **Agents can update the board.** When an agent run produces a relevant state change (e.g., signal monitor finds a blocker), it should update the board.

---

## Handoff Protocol

### Directions

- **Above** (`handoff/above/`): Items flowing from below-the-line (Code) to above-the-line (Cowork). These are technical findings, architectural questions, blockers needing product/strategic decisions, or research results that inform planning.
- **Below** (`handoff/below/`): Items flowing from above-the-line (Cowork) to below-the-line (Code). These are session prompts, build requests, technical research asks, spike requests, or bug reports with reproduction steps.

### Handoff File Format

Each handoff is a separate file:

```
handoff/<direction>/<date>_<slug>.md
```

Example: `handoff/below/[date]_feature-build-session8-scaffold.md`

Contents:

```markdown
# <Title>

> **From:** <source tool>
> **To:** <target tool>
> **Created:** <date>
> **Priority:** <high / medium / low>
> **Triage:** 🟢 green | ⚡ yellow | 🔴 red
> **Status:** pending | in-progress | handled

## Context
<What the receiving side needs to know to act on this>

## Request
<Specific ask — what should the receiving side do?>

## Acceptance Criteria
<How do we know this is done?>

## Related Events
<Links to relevant event log entries, if any>

## Assumptions (yellow triage only)
<What the receiving tool decided and why — for {{OPERATOR_NAME}} to review>

## Resolution (added when handled)
<What was done, by whom, and when>
```

### Pre-Handoff Quality Gate: Codebase Audit

**Applies to:** Mode 1 governed prompts flowing below the line (Cowork → Code).

Before handing a governed session prompt to Code, run a **pre-handoff codebase audit** against the actual repo. This is a ~15-minute investment that prevents ~2-3 hours of downstream friction.

**What to audit:**

1. **Schema:** Read the actual evolution files and model classes. Does the prompt's proposed DDL duplicate existing columns, use wrong types, or conflict with established naming conventions?
2. **Existing code:** Grep for anything the prompt says to "create." Does it already exist? Are file paths correct?
3. **Routes and guards:** Read the actual route file and guard implementations. Does the prompt overlap with existing endpoints or tell Code to build something that's already there?
4. **Service interfaces:** Read the services the prompt references. Does the prompt assume methods that don't exist, or miss methods that do?
5. **PRD cross-check:** Compare work items AND exit criteria against the relevant PRD requirements. Are all requirements either in scope or explicitly deferred in Out of Scope?

**How to run it:** Use a `code-explorer` subagent with the prompt file and repo path. The agent reads both and produces a structured diff of "what the prompt says" vs "what actually exists." Fix all mismatches before handing down.

**When to skip:** Mode 2 (Rapid Prototype) and Mode 3 (Maintenance) prompts are typically short enough that the overhead isn't justified. The audit is for Mode 1 governed prompts with 10+ work items where factual drift between plan and codebase is likely.

---

### Handoff Lifecycle

1. **Created** (`pending`) — Source tool writes the file, appends a `handoff-created` event to the log, adds it to the board's Handoff Queue.
2. **Picked up** (`in-progress`) — Receiving tool reads the file, updates status to `in-progress`, appends a `handoff-consumed` event.
3. **Handled** (`handled`) — Receiving tool finishes the work, updates status to `handled`, adds a `## Resolution` section to the handoff file describing what was done, **moves the file to `handled/` subfolder**, moves it from Handoff Queue to the board's Handled section, appends a `delivery` or `finding` event with the result.
4. **Volume rotation** — Handled handoffs stay in the `handled/` subfolder permanently. They are the receipts. When a `handled/` folder accumulates 20+ files, move the oldest to `ops/archive/handoffs/` — but they're still readable, never deleted.

**Why the `handled/` subfolder:** Listing an inbox directory shows only what's pending. Listing `handled/` shows what's been done. No need to open files to check status. The handoff file itself is still a self-contained record: what was asked, who asked, what was done, and when.

### Handoff Triage — When to Act vs. When to Hold

Every handoff gets a triage classification that determines whether the receiving tool can act independently or must wait for {{OPERATOR_NAME}}. The creating tool assigns the triage level. If uncertain, default to yellow.

#### GREEN — Auto-flow

The receiving tool picks up and acts immediately. {{OPERATOR_NAME}} sees it in the event log and board but doesn't need to review before action.

**The test:** Is this informational, reversible, or following an already-approved pattern?

| Green examples | Why it's green |
|---------------|----------------|
| Status report / onboarding confirmation | Informational only — no action needed beyond reading |
| Finding that confirms an existing assumption | Doesn't change direction — reinforces what's already planned |
| Session closeout summary | Retrospective, not prospective — no decision needed |
| Technical research result within approved scope | Operator already approved the scope; the result is just data |
| Routine session prompt for a pre-approved session plan | Operator signed off on the plan; individual sessions are execution |

**Board behavior:** Green handoffs appear briefly in the Handoff Queue, get picked up immediately, and move to Handled. They don't block.

#### YELLOW — Proceed but Flag

The receiving tool can pick up and begin work, but {{OPERATOR_NAME}} should review at their next session. The tool notes any assumptions it made that {{OPERATOR_NAME}} might override.

**The test:** Does this introduce something new, suggest a different approach within existing scope, or require a judgment call that has a reasonable default?

| Yellow examples | Why it's yellow |
|----------------|-----------------|
| Technical finding that suggests a better approach (but current approach still works) | Has a sensible default (keep going), but operator might want to pivot |
| New dependency discovered mid-build | Work can continue around it, but operator should know |
| Question where the tool has a confident answer but it's not certain | Acts on best judgment, flags for review |
| A carry-forward item that might be stale | Tool can skip it for now, but operator should confirm |
| Handoff that's green-pattern but first time for this specific workflow | Treat first-of-kind cautiously; future instances become green |

**Board behavior:** Yellow handoffs show a ⚡ indicator on the board. The receiving tool picks them up and adds a `## Assumptions` section to the handoff file documenting what it decided and why. {{OPERATOR_NAME}} reviews at next session and either confirms (handoff stays as-is) or overrides (new handoff with corrections).

#### RED — Hold for {{OPERATOR_NAME}}

The handoff sits in the queue. Neither tool acts until {{OPERATOR_NAME}} explicitly approves. The event log records the hold.

**The test:** Does this change scope, timeline, architecture, or affect people outside the AI system?

| Red examples | Why it's red |
|-------------|--------------|
| Architectural decision with significant tradeoffs | Irreversible (or expensive to reverse) — needs human judgment |
| Blocker that requires a product/strategic decision | Only the operator has the organizational context to decide |
| Anything that affects stakeholders outside the AI system | Decision Principle #4 — never act on the operator's behalf with other people |
| Scope change — "this feature is bigger than we thought" | Changes timeline/resources, needs human prioritization |
| First handoff in a new workstream (no established pattern) | Start narrow — operator validates the pattern before it becomes routine |
| Security, compliance, or data-sensitive findings | Risk tolerance is a human call |

**Board behavior:** Red handoffs show a 🔴 indicator on the board and remain in the Handoff Queue until {{OPERATOR_NAME}} explicitly moves them. If a red handoff sits for 48+ hours, the next agent run or session should surface a reminder.

#### Triage Escalation

A handoff can be **escalated** (green → yellow, yellow → red) by either tool at any time if new information changes the risk level. Add an event to the log noting the escalation and reason. Handoffs should never be **de-escalated** without {{OPERATOR_NAME}}'s approval — only {{OPERATOR_NAME}} can move something from red to yellow or yellow to green.

#### How Triage Interacts with Decision Principle #4

DP #4 says: "Surface to the operator, never act on their behalf." Triage refines what "act on behalf" means:

- **Green:** The tool isn't acting on the operator's behalf — it's doing its own job within established boundaries. Surfacing still happens via the event log and board. The operator has full visibility without being a bottleneck.
- **Yellow:** The tool acts on its best judgment but explicitly flags the decision for review. The operator can override retroactively. This is "act, then surface" rather than "surface, then wait."
- **Red:** The tool surfaces and waits. This is the original DP #4 behavior, reserved for decisions where acting first would be harmful or irreversible.

DP #4 is not weakened — it's calibrated. The spirit (operator maintains control) is preserved. The letter (every handoff blocks on the operator) is refined to avoid the operator becoming a bottleneck on informational flow.

---

## Continuous Write-Through

### The Behavioral Shift

The old model: work happens in a session, learnings accumulate in conversation context, everything gets captured in a closeout at the end.

The new model: learnings, decisions, and state changes are written to persistent storage **as they happen.** The closeout still exists but becomes lighter — it's a final reconciliation, not the primary capture mechanism.

### What Gets Written Mid-Session

| Discovery | Write To | Example |
|-----------|----------|---------|
| Something happened | Event log | "Discovered Angular Material CDK path changed in v18" |
| Current state changed | Board | Move item from In Flight → Completed |
| Work needs to cross the line | Handoff file + board | Technical finding that affects product planning |
| A durable learning emerged | Knowledge base / memory | Process learning, architectural pattern, domain insight |
| A decision was made | Event log + decision-principles.md (if broadly applicable) | "Chose standalone components over NgModules for Feature Build" |

### What Still Happens at Closeout

- Final reconciliation: ensure the board matches reality
- Carry-forward items: anything that didn't get captured mid-session
- Next-session prompt generation (if applicable)
- Memory/knowledge base updates that require synthesis across the session

### The Rule

**If you learned it, write it now. Don't wait for closeout.** The event log is cheap to append to. The board is cheap to update. A finding written in the moment has full context. A finding reconstructed at closeout is a summary of a memory.

---

## Source Attribution for Canonical Memory Writes

### The Rule

**Every claim committed to canonical memory must be traceable to a source.** Allowed sources:

- **Transcript** — filename + approximate timestamp (e.g., `transcript_[date]_meeting-name.md ~00:21:05`)
- **Slack source** — channel + message timestamp or permalink
- **Drive doc** — file ID or permalink + heading section
- **Decision event** — `ops/event-log.md` event date/source
- **Operator-direct** — chat message in current session (cite the session by name + approximate turn position, e.g., "{{OPERATOR_NAME}} direct, [session name] [turn position]")
- **Convention or DP** — explicit citation to the standing rule that justifies the claim

### Why

Cowork sessions routinely synthesize claims from in-session context — speaker attribution from a transcript, owner assignment from inferred org-chart logic, status updates from cross-meeting triangulation. When those claims land in canonical documents as if they were verified facts, downstream sessions inherit them as canonical. Inferred-from-context claims must be downgraded to **tentative** until confirmed.

### What "tentative" looks like

For a claim that isn't yet source-attributable but the session has to commit *something*:

- Mark inline with a clear tag — `*(tentative — Cowork inference from <context>; needs {{OPERATOR_NAME}} confirmation)*`
- Add to `ops/assumptions.md` so it lands in the next weekly review
- Don't write it to a *header* (where downstream readers anchor) — write it to a body subsection (where it reads as in-flight signal)

### The Test

Before writing any claim to canonical memory, ask: **"If a future session reads this in 3 weeks and acts on it, will the source I cited still hold up?"** If the answer is "I'm inferring this from context that isn't in canonical memory," it's tentative.

### What this does NOT mean

- Synthesis sections (delta summaries, closeouts) still use Cowork's own framing; they're transparent about being syntheses.
- The event log doesn't need source attribution — events ARE the source.
- Items already in canonical memory don't get retroactive citation added unless they're being modified.

---

## Knowledge Base — The "What Did We Learn" Layer

### Three Layers of Persistence

The ops/ system has three layers, each serving a different question:

| Layer | Question | Artifact | Mutability |
|-------|----------|----------|------------|
| **Event log** | "What happened?" | `ops/event-log.md` | Append-only, never edited |
| **Board** | "What's happening now?" | `ops/board.md` | Mutable, reflects current state |
| **Knowledge base** | "What did we learn?" | Existing memory/ and reference/ files | Updated when durable learnings emerge |

The fourth layer feeds into the other three:

| Layer | Question | Artifact | Mutability |
|-------|----------|----------|------------|
| **Assumptions register** | "What did we decide without being told?" | `ops/assumptions.md` | Append-only, reviewed by {{OPERATOR_NAME}} |

The event log captures facts. The board captures state. The knowledge base captures **understanding** — durable learnings, patterns, decisions, and context that future sessions need regardless of what's currently in flight. The assumptions register captures **judgment calls** — decisions both tools made on their own that {{OPERATOR_NAME}} should review, promote to standing rules, or correct.

### Where Knowledge Lives

The knowledge base is NOT a new directory. It's the existing memory and reference infrastructure, written to more aggressively:

- **Process learnings** → your process-learnings memory file
- **Architectural/technical learnings** → Code's auto-memory (`~/.claude/projects/.../memory/`)
- **Decision rationale** → your decision-principles reference file
- **Domain knowledge** → Relevant workstream memory/ folders
- **Cross-session auto-memory** → `.auto-memory/` (Cowork) or Code's project memory

### When Events Become Knowledge

Not every event is worth persisting as knowledge. The filter:

- **An event is a fact:** "We discovered the CDK path changed in v18." → stays in the event log.
- **Knowledge is a learning:** "Angular Material CDK paths change between major versions. Always verify import paths by reading node_modules directly, not from memory." → goes to the knowledge base.

The test: **would a session 3 weeks from now benefit from knowing this, even if it's working on something unrelated?** If yes, it's knowledge. If it's only relevant in the context of the current work, it's an event.

### Write-Through to Knowledge Base

Both tools should write to the knowledge base mid-session when a durable learning emerges. Don't wait for closeout. The event log entry records *that* something was learned. The knowledge base entry records *what* was learned in a form future sessions can use.

When writing to the knowledge base, append a `knowledge-update` event to the log referencing what was written and where. This creates a traceable chain: event log → knowledge base update → the knowledge itself.

---

## Meta-Findings Routing

### What It Is

Some findings are scoped to a single session. Others change how every environment works. The meta-findings routing protocol determines where a finding gets written based on its **blast radius** — how many environments or future sessions would benefit from knowing it.

### Classification

| Blast Radius | Where to Write | Example |
|-------------|----------------|---------|
| **Session-scoped** | Event log only | "Discovered CSRF bypass was already configured — no changes needed" |
| **Workstream-scoped** | Event log + workstream memory | "Feature Build evaluations don't persist to the database yet — gap for Session 6" |
| **Org-scoped** | Event log + process-learnings | "Claude Code's consistent ~2 bugs per session pattern holds across v2" |
| **Cross-environment** | Event log + process-learnings + knowledge base + board | "A quality gate applies to prompt generation, not just code — new rule" |
| **Strategic** | All of the above + operator inbox handoff | "The ops/ framework has a structural flaw that affects all environments" |

### Routing Rules

**Session-scoped** findings are the default. Most discoveries stay in the event log. The test: would anyone outside this session's immediate workflow care?

**Workstream-scoped** findings affect a specific project (e.g., Feature Build, Prototype, Maintenance). They go to that project's memory files in addition to the event log. The test: would the next session on this project need to know?

**Org-scoped** findings affect how you work across projects within {{YOUR_COMPANY}}. They go to your process-learnings file. The test: is this a reusable process insight, not just a project-specific fact?

**Cross-environment** findings change how multiple environments operate. These get the full three-hop write:

1. **Knowledge base** — Permanent home. Full write-up with context and "How to apply" section.
2. **Board broadcast** — One-paragraph summary so all environments see it on next session start.
3. **Process learnings** — Numbered entry for accumulation with existing entries.

**Strategic** findings get the three-hop write PLUS a directed handoff to the operator. These are findings that require {{OPERATOR_NAME}}'s explicit attention because they affect system design, organizational decisions, or introduce risk. The test: if {{OPERATOR_NAME}} doesn't see this within 24 hours, could something go wrong?

### The Trigger

After writing any `finding` event to the log, ask: "Who else would benefit from this?" If the answer is "just this session" — done. If broader — route accordingly. **The session that discovers the finding owns the routing.** Don't defer it.

### Event Log Annotation

When a finding is routed beyond the event log, the log event should reference where it was written:

```markdown
## YYYY-MM-DD HH:MM | <source> | finding

<Description of the finding.>

→ Routed: `knowledge/<topic>/<slug>.md`, board broadcast, process learning #NNN.
```

This creates a traceable chain from event → knowledge → board, so future sessions can reconstruct the full context.

---

## Assumptions Register

### What It Is

The assumptions register (`ops/assumptions.md`) is a shared log where both ATL and BTL record judgment calls they made without explicit instruction. It's the feedback loop that makes the triage system self-improving: assumptions that repeat get promoted to rules, assumptions that prove wrong get corrected, and over time the system needs fewer assumptions because the conventions are more precise.

### When to Log an Assumption

- You classified a handoff triage level and it wasn't obvious
- You chose a default when multiple options were reasonable
- You interpreted an ambiguous instruction one way over another
- You decided something was green-triage that could arguably be yellow or red
- You skipped or deprioritized something based on your own judgment
- You made a decision that a future review might question

**Don't over-log.** If the decision was obviously covered by an existing convention or decision principle, it's not an assumption — it's following the rules. Log the gray areas, not the clear-cut ones.

### Review Cadence

{{OPERATOR_NAME}} reviews the register aligned with the **weekly anchor audit**. For each entry, {{OPERATOR_NAME}} marks one of:

- ✅ **Promote** → becomes a convention, decision principle, or instruction
- ⚡ **Correct** → wrong assumption, correction written as a rule
- 🔇 **Discard** → one-off, not worth codifying
- 🔄 **Watch** → pattern not clear yet, keep logging instances

Promoted and corrected entries reference back to the assumption that generated them, creating a traceable chain from judgment call → standing rule.

### The Feedback Loop

```
Assumption made → Logged in register → {{OPERATOR_NAME}} reviews
  → Promote: becomes convention/DP/instruction (fewer future assumptions)
  → Correct: becomes correction/feedback (better future judgment)
  → Discard: one-off, no action
  → Watch: accumulate instances until pattern is clear
```

Over time, the register should trend toward fewer entries per week as the rules get sharper. If it's growing instead of shrinking, the conventions need a broader update.

---

## How Each Tool Participates

### Cowork (Above the Line)

- Reads the board at session start to orient
- Appends events as it works (session-start, decisions, findings, handoff-created)
- Updates the board to reflect state changes
- Creates handoff files in `handoff/below/` for Code
- Consumes handoff files from `handoff/above/` (Code's findings)
- Writes to knowledge base mid-session when durable learnings emerge
- Logs judgment calls to the assumptions register

**Orientation-style work routes through the session-start skill — always.** Any request that is semantically orientation (pulse check, ATL/BTL pulse, catch-me-up, state-of-things, "where did we leave off", "what's on deck", morning briefing, any variation thereof) is session-start's job. Do not ad-hoc it. The skill handles ops/ registration, staleness checks, and queued-work discovery. Bypassing it loses those second-order effects even when the ad-hoc response content looks complete. If the request is ambiguous — if it could be orientation or could be direct work — treat it as orientation and run session-start. Better to over-trigger the skill than to silently skip it.

**Session-start is not complete until the live sources are checked and a read-receipt is logged.** Orientation must derive "what's open / due / already shipped" from the live systems of record, NOT from `board.md` or other cached files that drift the moment a tool updates. When the cache and a live source disagree, the live source wins and the cache is flagged stale.

### Claude Code (Below the Line)

- Reads the board at session start to understand what's in flight
- Appends events as it works (session-start, findings, delivery, blockers)
- Creates handoff files in `handoff/above/` for Cowork
- Consumes handoff files from `handoff/below/` (session prompts, research asks)
- Updates the board when work completes or blockers arise
- Writes to its own CLAUDE.md memory for findings that future Code sessions need
- Logs judgment calls to the assumptions register

### Claude Code IDE (Visual QA)

IDE sessions handle visual QA tasks. Their reporting protocol is intentionally lightweight:

1. **Append one event** to `ops/event-log.md` using `code(ide)` as the source. Format:
   ```markdown
   ## YYYY-MM-DD ~HH:MM | code(ide) | delivery

   <What was tested, what passed, what failed, any new findings.>

   → <Summary. Reference the QA handoff filename.>
   ```
2. **Move the QA handoff** to the appropriate `handled/` folder.

**That's it.** IDE sessions do NOT update the board — the terminal session or Cowork reconciles the board on their next pass. This keeps the QA loop fast and avoids merge conflicts on `board.md`.

### Scheduled Agents

- Append `agent-run` events to the event log
- Update the board if they discover blockers or state changes
- Write to their existing output locations AND append an event
- Never create handoff files directly — they surface to {{OPERATOR_NAME}}, who decides

### {{OPERATOR_NAME}} (Manual)

- Can append events manually for decisions made outside of AI tools
- Reviews and approves handoff queue before items cross the line
- Performs periodic board reconciliation (weekly, aligned with anchor audit)

---

## Naming Conventions

| Artifact | Pattern | Example |
|----------|---------|---------|
| Event log | `ops/event-log.md` | — |
| Board | `ops/board.md` | — |
| Handoff (below, pending) | `ops/handoff/below/<date>_<slug>.md` | `[date]_feature-build-session8.md` |
| Handoff (below, handled) | `ops/handoff/below/handled/<date>_<slug>.md` | moved after completion |
| Handoff (above, pending) | `ops/handoff/above/<date>_<slug>.md` | `[date]_angular-cdk-breaking-change.md` |
| Handoff (above, handled) | `ops/handoff/above/handled/<date>_<slug>.md` | moved after completion |
| Assumptions register | `ops/assumptions.md` | — |
| Archived events | `ops/archive/event-log-YYYY-MM.md` | `event-log-2026-03.md` |
| Archived handoffs | `ops/archive/handoffs/<file>` | — |

---

*This conventions file is the contract for how ops/ works. All tools, agents, and sessions that interact with ops/ should read this file first.*
