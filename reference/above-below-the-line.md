# Above / Below / Outside the Line

> The three-zone coordination model for AI work at {{YOUR_COMPANY}}. Everything in the anchor system maps to one of these zones.

---

## The Three Zones

### ATL — Above the Line
**Tool:** Cowork / Claude Desktop (this anchor)

The analysis, planning, and coordination layer. Claude here reads and writes files, processes transcripts, drafts communications, investigates data issues, and hands specs to Build.

**Owns:** Analysis, process design, planning, stakeholder communications, warehouse specs
**Writes:** Specs, memos, query writeups, Slack drafts, meeting notes, work outputs
**Never:** Ships production code directly to repos; sends external messages without approval

### BTL — Below the Line
**Tool:** Claude Code (CLI)

The implementation layer. Receives specs from ATL via `ops/handoff/below/`. Builds warehouse models, pipeline jobs, scripts, tests.

**Owns:** Repos, dbt models, Prefect jobs, pipeline scripts, tests, PRs
**Writes:** Code, tests, PRs, technical findings in `ops/handoff/above/`
**Never:** Sends stakeholder communications; makes unilateral architectural decisions

### OTL — Outside the Line
**Tool:** Scheduled agents (Cowork cron tasks)

Autonomous agents running on a cadence between sessions. They collect, monitor, audit, and synthesize — then drop outputs into `inbox/` for review. They do not act externally on {{YOUR_NAME}}'s behalf.

**Owns:** Recurring information collection and monitoring
**Writes:** Inbox digests, audit reports, signal summaries
**Never:** Sends messages, creates Linear tickets, or takes external action without review

---

## The Shared Layer: `ops/`

All three zones share one coordination layer — `ops/`:

| File | Purpose |
|------|---------|
| `ops/event-log.md` | Append-only event stream. Source of truth for "what happened." |
| `ops/board.md` | Materialized view of current in-flight work and handoff queue. |
| `ops/handoff/above/` | BTL → ATL: code findings, blockers, decisions needing ATL judgment |
| `ops/handoff/below/` | ATL → BTL: specs, schemas, task definitions for Claude Code to build |
| `ops/assumptions.md` | Logged assumptions; any session can validate or invalidate |
| `ops/CONVENTIONS.md` | Event format, handoff protocol, triage rules — read before coordinating |

---

## Handoff Triage

Every item crossing the ATL/BTL line carries a triage color:

| Color | Meaning | What the receiving tool does |
|-------|---------|------------------------------|
| 🟢 GREEN | Informational, reversible, or follows an approved pattern | Acts immediately — no hold needed |
| 🟡 YELLOW | Judgment call with a reasonable default | Acts on best judgment + surfaces the decision for review |
| 🔴 RED | Changes scope, timeline, architecture, or affects others | Blocks until {{YOUR_NAME}} approves |

**The human governor rule:** Only {{YOUR_NAME}} acts externally on others' behalf. Agents surface, Cowork drafts, Code builds. {{YOUR_NAME}} decides what leaves the system.

---

## Practical Mapping for the Data Role

| Work type | Zone |
|-----------|------|
| Pull project tracker, check open items | ATL (this anchor) |
| Investigate a domain issue, write findings | ATL → `ops/board.md` |
| Write implementation spec | ATL → `ops/handoff/below/` |
| Build the implementation, open PR | BTL (Claude Code) |
| Daily signal scan, commitment tracker | OTL (scheduled agents) |
| Ingest meeting transcript into memory | ATL (`context-ingestion` skill) |
| Draft Slack message to team | ATL (drafts only — {{YOUR_NAME}} sends) |

---

*Full AOM constitution (rules all three zones inherit): `reference/aom-framework.md`*
*Coordination conventions: `ops/CONVENTIONS.md`*
