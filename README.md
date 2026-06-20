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
| **Think** | Claude Chat (claude.ai) | Research, brainstorming, Q&A |
| **Do** | Claude Cowork (this anchor) | File-backed workflows, persistent memory, planning, stakeholder comms |
| **Build** | Claude Code (CLI) | Code generation on real repos — receives specs from the Do layer |

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

---

## Prerequisites

- **Claude Cowork** (Claude Desktop app, Cowork tab)
- **Connectors** for whatever tools you use — as relevant to your role:
  - Google Calendar, Google Drive (for daily true-up agent)
  - Slack (for signal monitor + commitment tracker)
  - Gmail (for commitment tracker)
  - Linear/Jira/Asana or GitHub Issues (for project tracker sync — Linear connector available natively)
  - GitHub (for GitHub sync agent, if applicable)

---

## The Core Idea

Quality in = quality out. The richer and more accurate the context in `memory/` and `reference/`, the better every session. You don't maintain this system manually — you *correct* it, and the corrections compound.

See `reference/getting-started.md` for a full orientation.

---

*Built on the ATL/BTL/OTL framework. Sourced from the Bluefields/vQuip Anchor, built June 2026.*
