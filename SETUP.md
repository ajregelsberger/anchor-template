# SETUP.md — Personalization Checklist

Work through this file top to bottom before your first session. Every `{{PLACEHOLDER}}` in this repo is listed here with the file(s) it appears in.

**Faster option:** Say *"Set me up"* in Claude Cowork — the `anchor-setup` skill walks you through all of this interactively and applies values automatically.

---

## Section 1: Before You Start

Have these ready:

- [ ] Your full name
- [ ] Your job title / role
- [ ] Your company or organization name
- [ ] Your department or team name
- [ ] The full path to this folder on your machine (e.g., `C:\Users\yourname\Documents\Anchor` or `/Users/yourname/Documents/Anchor`)
- [ ] Who set up this anchor — you, or someone else? (Their name = `{{OPERATOR_NAME}}`)
- [ ] Your 2–3 key collaborators (name, role, one-line relationship)
- [ ] Your main production systems (3–5 tools/databases you work with)
- [ ] Your project tracker (Linear, Jira, Asana, GitHub Issues, Notion, or other)
- [ ] Your most important Slack channels (up to 5)
- [ ] Your OS: Windows or Mac (affects path format in agent prompts)

---

## Section 2: CLAUDE.md

| Placeholder | What to fill in | Example |
|-------------|-----------------|---------|
| `{{YOUR_NAME}}` | Your full name | "Jordan Smith" |
| `{{YOUR_ROLE}}` | Your job title | "Head of Engineering" |
| `{{YOUR_DEPARTMENT}}` | Your team/function | "Engineering" |
| `{{YOUR_COMPANY}}` | Your employer | "Acme Corp" |
| `{{OPERATOR_NAME}}` | Who set up the anchor | "Jordan Smith" (or another name if someone else) |
| `{{TEAM_MEMBER_1}}`, `{{TEAM_MEMBER_2}}` | Key collaborators | "Sam Lee", "Alex Kim" |
| `{{CREATION_DATE}}` | Date you set this up | "2026-06-20" |

**Routing table:** Replace the `[Your key domain]` row with routing rules that match your role. Each row answers: "When the user asks about X, which file should Claude read?" Add rows for every domain-specific reference file you create in `reference/`.

**Conventions section:** Replace the `[Add your domain-specific conventions here]` line with terminology rules, data handling rules, communication norms specific to your domain.

---

## Section 3: memory/ — Seed Your Memory Files

Memory files are excluded from git (`.gitignore`) — they're personal and accumulate over time.

- [ ] `memory/user-YOURNAME.md` — rename this file to `memory/user-[your-slug].md` (e.g., `user-jordan.md`). Fill in your role, team, and operating style. Claude will also add to it as it learns your preferences.
- [ ] `memory/project-state.md` — fill in your current open items, blockers, and recent decisions.
- [ ] `memory/reference-systems.md` — fill in your key systems, key people, and external resources.
- [ ] `memory/feedback-working-style.md` — optionally seed with known preferences before your first session; Claude will add to it automatically.
- [ ] `memory/MEMORY.md` — update `{{YOUR_NAME}}` in the header.

---

## Section 4: reference/ — What to Customize

| File | Action |
|------|--------|
| `reference/decision-principles.md` | Replace the `## Domain-Specific Decisions` section with rules specific to your role |
| `reference/transcription-gotchas.md` | Add mis-transcriptions for your team's names and domain terms |
| `reference/agents-overview.md` | Update the 5 agents' channel lists and key people lists with your own |
| `reference/company-strategic-context.md` | Fill in your current priorities, background context, and key historical decisions |
| `reference/above-below-the-line.md` | No changes needed — the framework is generic |
| `reference/aom-framework.md` | No changes needed — the principles are generic |
| `reference/getting-started.md` | No changes needed — examples are already generic |
| `reference/ingestion-guidelines.md` | No changes needed — confidence tiers are generic |

**Role-specific reference files:** Add files for your domain (e.g., `reference/your-domain-vocabulary.md`, `reference/your-stack-overview.md`) and update the routing table in `CLAUDE.md` to point to them.

---

## Section 5: Scheduled Agents — Wiring Them Up

Each agent in `Scheduled/` has a `SKILL.md` with an **"Adapt before wiring this up"** block at the top.

For each agent you want to enable:

1. Open the `SKILL.md` file
2. Follow the "Adapt before wiring" instructions (replace `{{ANCHOR_ROOT_PATH}}`, channel names, people names)
3. In Cowork → Settings → Scheduled Tasks → New Task: set name, cron schedule, paste the adapted prompt
4. Save and enable

| Agent | Cadence | Requires |
|-------|---------|----------|
| `daily-true-up` | Weekdays 5pm | Google Calendar + Google Drive |
| `slack-signal-monitor` | Weekdays 4pm | Slack |
| `weekly-anchor-audit` | Mondays 8am | Nothing extra |
| `commitment-follow-up-tracker` | Weekdays 3pm | Slack + Gmail |
| `monday-morning-briefing` | Mondays 7am | All of the above |
| `github-sync` *(optional)* | Weekdays 6pm | GitHub |
| `linear-sync` *(optional)* | Twice daily via -am/-pm wrappers | Linear |

---

## Section 6: ops/ — Initialize Your Ops Files

Ops files are excluded from git — they're runtime state.

- [ ] `ops/event-log.md` — start fresh (or copy the template from this repo)
- [ ] `ops/board.md` — fill in your current quarterly goals and in-flight work
- [ ] `ops/assumptions.md` — start empty

---

## Section 7: First Session

Once you've worked through this checklist, open Cowork pointed at this folder and say:

> *"Catch me up — what's the current state of my open items?"*

This triggers the `session-start` skill, which reads your memory files and helps you decide what to work on.

Or start even simpler: *"Morning."*

See `reference/getting-started.md` for the full primer.
