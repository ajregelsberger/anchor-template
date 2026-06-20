# CLAUDE.md — {{YOUR_NAME}}'s Anchor

> **Owner:** {{YOUR_NAME}} — {{YOUR_ROLE}}, {{YOUR_COMPANY}}
> **Anchor type:** [Describe your anchor type, e.g., "department-lead anchor", "individual contributor anchor", "founder anchor"]
> **Created:** {{CREATION_DATE}} by {{OPERATOR_NAME}}
> **This is a living document.** Update it as the work evolves.

---

## Who This Is For

{{YOUR_NAME}} — **{{YOUR_ROLE}}**, {{YOUR_DEPARTMENT}} at {{YOUR_COMPANY}}. [1-2 sentences about your scope and what this anchor covers.]

Key collaborators: {{TEAM_MEMBER_1}}, {{TEAM_MEMBER_2}}.

[Brief note on why this anchor is cross-functional, if applicable.]

---

## What This Folder Is

This is an **Anchor** — a file-backed Claude Cowork workspace that lives on your machine. It persists memory, skills, and context across sessions, so every conversation builds on the last.

**Folder:** `{{ANCHOR_ROOT_PATH}}`

**Structure:**

```
{{YOUR_NAME}}'s Anchor/
├── CLAUDE.md              ← this file (read every session)
├── memory/                ← context that compounds across sessions
├── reference/             ← stable framework docs + role-specific context
├── ops/                   ← coordination layer (event log, board, handoffs)
├── skills/                ← reusable Cowork workflows
├── Scheduled/             ← autonomous agent skill scripts
├── templates/             ← Claude Code dev-mode templates
├── inbox/                 ← landing zone for agent output
├── saddle/                ← scratchpad for raw/unsorted work
├── sessions/              ← session closeouts + next-session prompts
└── work/                  ← active outputs
```

**Workflow loop:**

1. **Session start** → `session-start` skill loads memory, checks live sources, surfaces what's on deck
2. **In session** → work happens; Claude drafts, analyzes, investigates, and hands off to Claude Code for implementation
3. **Session close** → `session-close` skill writes back to memory, updates ops board, generates next-session prompt
4. **Between sessions** → OTL scheduled agents run autonomously, drop output in `inbox/`, log events

---

## How It Works (the 60-second version)

- Claude reads `memory/` at session start. The richer the memory, the better the session.
- Claude drafts before asking. You review and approve.
- Claude Code receives specs via `ops/handoff/below/`. It never sends stakeholder comms.
- Scheduled agents drop signal in `inbox/`. Claude processes it during the next session via `context-ingestion`.
- Everything that matters gets written to memory. Corrections compound.

---

## TDB — Think · Do · Build (where you sit)

| Zone | Tool | What it does |
|------|------|--------------|
| **Think** | Claude Chat (claude.ai) | Research, brainstorming, Q&A — ephemeral |
| **Do** (ATL) | Claude Cowork (this anchor) | Persistent memory, planning, drafts, specs, comms |
| **Build** (BTL) | Claude Code (CLI) | Implementation on real repos — receives specs from ATL |

ATL never ships code. BTL never sends stakeholder comms. See `reference/above-below-the-line.md`.

---

## Routing Rules

**Local, in this folder:**

| If the task involves... | Read... |
|------------------------|---------|
| First time in this anchor / how to use it | `reference/getting-started.md` |
| [Your key domain — e.g., "data stack vocabulary"] | `reference/[your-vocab-file].md` |
| Current open items and project state | `memory/project-state.md` + `memory/MEMORY.md` |
| The framework — ATL/BTL/OTL, AOM, agent patterns | `reference/{above-below-the-line,aom-framework,agents-overview,decision-principles,ingestion-guidelines}.md` |
| Cowork ↔ Claude Code coordination | `ops/CONVENTIONS.md` |
| Claude Code dev modes + scheduled-task design | `templates/` |
| [Add rows for your specific systems/projects] | `reference/[your-file].md` |

> **Note for setup:** Replace the rows above with routing rules that match your role. Each row answers: "When the user asks about X, which file should Claude read?" The goal is that Claude never has to guess where to find information — the routing table tells it.

> **Role-specific reference files:** Add files for your domain (e.g., `reference/your-domain-vocabulary.md`) and update the routing table to point to them.

---

## Conventions

- **Memory-first.** Before answering a recurring question, check `memory/project-state.md`. After learning something durable, write it back.
- **Source attribution.** Every claim committed to memory must be traceable to a source — a document, a transcript, a Slack message, or your direct confirmation.
- **Read-only by default on production systems.** Treat any write path as confirm-gated.
- [Add your domain-specific conventions here — terminology rules, data handling rules, communication norms]

---

## First Session

Open Cowork pointed at this folder and just talk to it:

- *"Set me up"* (fires `anchor-setup` — first-time personalization wizard)
- *"Catch me up — what's the current state of my open items?"* (fires `session-start`)
- *"Here's a transcript from this morning's meeting — pull the signal into memory."* (fires `context-ingestion`)
- *"Help me write the spec for [project] so I can hand it to Claude Code."*

See `reference/getting-started.md` for the full primer.

---

*This anchor carries the framework, skills, and templates locally. The context — your projects, your team, your decisions — is yours to build. Correct it freely — that's how it gets smart.*
