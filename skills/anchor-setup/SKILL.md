---
name: anchor-setup
description: >
  First-time personalization wizard for a fresh anchor clone. Collects the user's
  details and applies them across all {{PLACEHOLDER}} files in the anchor, seeds
  memory files, and guides through scheduled agent selection. Use when: "set me up",
  "configure this anchor", "I just cloned this", "initial setup", "personalize this",
  "first time using this", "new anchor", "help me get started", "customize for me",
  or any signal that the user has a fresh clone and wants to configure it for their
  context. This skill should run BEFORE any other skill on a fresh anchor.
---

# Anchor Setup

You are running the first-time setup wizard for this anchor. The goal: collect the user's
details, apply them across all template files (replacing `{{PLACEHOLDER}}` values), seed
the memory files, and get the user ready for their first real session.

This should feel like a friendly onboarding conversation — not a form to fill out.

---

## Before You Start

Check if this anchor has already been set up. Look for two signals:
1. Does `CLAUDE.md` still contain literal `{{YOUR_NAME}}` text?
2. Does `memory/project-state.md` still contain placeholder text?

If both are already personalized, this anchor is already configured. Tell the user and
offer to update specific sections instead.

---

## Phase 1: Ask the Questions

Ask all questions upfront in one clear message — don't ask one at a time. Tell the user
they can answer in any format; you'll extract the values.

Introduce yourself briefly:
> "I'm going to set up this anchor for you. I'll ask you a few questions, then apply your
> answers across all the template files. This takes about 5 minutes. You can answer these
> in any format — a list, sentences, whatever's easiest."

Then ask:

---

**Group 1 — Identity**
1. What's your full name?
2. What's your job title / role?
3. What company or organization do you work for?
4. What's your department or team called?

**Group 2 — Setup**
5. Who set up this anchor — you, or someone else? (If someone else: what's their name?)
6. What's the **full path** to this anchor folder on your machine?
   - Windows example: `C:\Users\yourname\Documents\Anchor`
   - Mac example: `/Users/yourname/Documents/Anchor`
7. Are you on **Windows or Mac**? (This affects how agent prompts write file paths.)

**Group 3 — Team**
8. Who are your 2–3 key collaborators? For each: name, role, and one sentence on how they relate to your work.
9. Are there key external partners or vendors you track? (Optional — skip if not applicable.)

**Group 4 — Tools**
10. What project tracker do you use? (Linear, Jira, Asana, GitHub Issues, Notion, other — or none)
11. What are your most important Slack channels? (List up to 5 by name.)
12. What are your main production systems — the tools or databases you work with or report on regularly? (List 3–5 with a short description of each.)

**Group 5 — Domain**
13. What's your primary work domain? (e.g., engineering, operations, product, finance, data, marketing, legal — helps pick example routing rules)
14. Are there any names or terms in your environment that commonly get mis-transcribed by AI meeting notes tools? (Optional — these go in the transcription gotchas file.)

**Group 6 — Agents (Optional)**
15. Which scheduled agents do you want to enable? (You can skip this now and enable them later via SETUP.md.)
    - **Daily True Up** — cross-references calendar vs. meeting transcripts (requires: Google Calendar + Google Drive)
    - **Slack Signal Monitor** — scans Slack channels for decisions/blockers (requires: Slack)
    - **Weekly Anchor Audit** — meta-agent that checks the anchor's health (requires: nothing extra)
    - **Commitment Tracker** — chases stale commitments from Slack + Gmail (requires: Slack + Gmail)
    - **Monday Morning Briefing** — weekly orientation rollup (requires: all of the above)
    - **GitHub Sync** (optional) — tracks PR/commit activity (requires: GitHub)
    - **Project Tracker Sync** (optional, Linear only) — pulls issue board twice daily (requires: Linear)

---

## Phase 2: Confirm the Values

After the user answers, extract and confirm the values in a clean table before applying:

```
Here's what I'll use:

YOUR_NAME:          [value]
YOUR_ROLE:          [value]
YOUR_COMPANY:       [value]
YOUR_DEPARTMENT:    [value]
OPERATOR_NAME:      [value]
ANCHOR_ROOT_PATH:   [value]
OS:                 [Windows / Mac]
TEAM_MEMBER_1:      [name] — [role]
TEAM_MEMBER_2:      [name] — [role]
PROJECT_TRACKER:    [value]
SLACK_CHANNEL_1:    [value]
SLACK_CHANNEL_2:    [value]
SYSTEM_1:           [value]
SYSTEM_2:           [value]
[...etc.]
```

Does this look right? I'll apply these and you can adjust anything after.

Wait for confirmation before proceeding.

---

## Phase 3: Apply Values Across Files

Once confirmed, work through every file systematically. For each file:
1. Read the current content
2. Replace all `{{PLACEHOLDER}}` occurrences with the confirmed values
3. Write the file back

**Files to update (in order):**

### Critical files (do these first)
1. `CLAUDE.md` — core identity + routing
2. `memory/MEMORY.md` — update `{{YOUR_NAME}}` references
3. `memory/project-state.md` — update header
4. `memory/user-YOURNAME.md` — rename the file to `user-[their-slug].md` and fill in with their details

### Reference files
5. `reference/above-below-the-line.md`
6. `reference/aom-framework.md`
7. `reference/agents-overview.md`
8. `reference/decision-principles.md`
9. `reference/getting-started.md`
10. `reference/ingestion-guidelines.md`
11. `reference/transcription-gotchas.md` — populate with any mis-transcriptions the user listed
12. `reference/company-strategic-context.md`

### Memory files
13. `memory/feedback-working-style.md` — update header only (body stays as empty template)
14. `memory/reference-systems.md` — populate with their systems, key people, and tracker
15. `memory/user-[slug].md` — populate with name, role, team, operating style from their answers

### Ops files
16. `ops/CONVENTIONS.md`
17. `ops/board.md` — update header and owner fields
18. `ops/assumptions.md` — update header

### Scheduled agents (only the ones they selected)
For each selected agent: replace `{{ANCHOR_ROOT_PATH}}` with their actual path, replace `{{YOUR_NAME}}` with their name, replace channel lists with their channels, replace team member lists with their team.

If OS is **Mac**: use forward slashes and `/`-separated paths throughout agent prompts.
If OS is **Windows**: use backslashes and `\`-separated paths.

### Skills
19. `skills/session-start/SKILL.md` — update path references
20. `skills/context-ingestion/SKILL.md` — update path references
21. `skills/session-close/SKILL.md` — update path references

---

## Phase 4: Seed the Ops Files

Initialize the files that need real starting content (not just placeholder swaps):

### `ops/event-log.md`
Write a `session-start` event documenting this setup session:
```markdown
## [TODAY'S DATE] [TIME] | cowork | session-start

Anchor setup — first configuration session. Anchor personalized for [YOUR_NAME] / [YOUR_COMPANY].

→ Configured: CLAUDE.md, all memory files, reference files, ops files, selected scheduled agents.
→ Run anchor-setup skill to reconfigure if anything changes.
```

### `ops/board.md`
Update the "In Flight" section to reflect that the anchor was just set up and is ready to use.

---

## Phase 5: Agent Wiring Summary

For each scheduled agent the user selected, provide a ready-to-copy summary:

```
## Agents to wire up in Cowork Settings → Scheduled Tasks

For each agent below:
1. Open Cowork → Settings → Scheduled Tasks → New Task
2. Set the name, description, and cron schedule
3. Paste the prompt from the SKILL.md file in your Scheduled/ folder
4. Save and enable

Agent: Daily True Up
  Schedule: Weekdays 5pm (your timezone)
  Cron: 0 17 * * 1-5
  Prompt: [contents of Scheduled/daily-true-up/SKILL.md]
  Requires: Google Calendar connector + Google Drive connector

[...repeat for each selected agent]
```

If the user didn't select any agents, tell them they can wire them up later using `templates/scheduled-tasks/README.md`.

---

## Phase 6: First Session Checklist

Finish with a clear "you're ready" summary and the exact prompt to use for their first real session:

```
## Your anchor is ready. ✓

Here's what was set up:
- CLAUDE.md personalized for [YOUR_NAME] at [YOUR_COMPANY]
- Memory files seeded (project-state, user profile, reference systems)
- [N] scheduled agents configured (wire them up in Cowork settings — details above)
- Framework docs anonymized and ready

## Your first real session

Open Cowork pointed at this folder and say:

> "Catch me up — what's the current state of my open items?"

This will trigger the session-start skill, which reads your memory files and helps
you decide what to work on. From there, just talk to it.

## What to do next

1. Wire up your scheduled agents (if you selected any above)
2. Add your current projects and open items to `memory/project-state.md`
3. Add any role-specific reference files you need to `reference/`
4. Update the routing table in `CLAUDE.md` to point to those files
5. Run `meta-analysis` after your first real session to calibrate
```

---

## What This Skill Does NOT Do

- Does not create Linear/Jira tickets or external records
- Does not send Slack messages
- Does not wire up scheduled agents (gives you the prompts and instructions — you do the wiring)
- Does not replace SETUP.md — that file documents every placeholder manually for reference; this skill automates the application

---

## If Something Goes Wrong

If a file fails to update or a placeholder was missed:

1. Check SETUP.md for a manual list of every placeholder and where it appears
2. Use your editor's find/replace to apply any missed values
3. Re-run anchor-setup and answer just the question for the missed section
