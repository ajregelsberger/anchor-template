# AI Development Modes — {{YOUR_COMPANY}}

> **Owner:** AI Orchestration
> **Last Updated:** 2026-03-02
> **Purpose:** Reference guide for choosing and executing the right AI development pattern. Three modes, each with its own workflow, templates, and use cases.

---

## The Three Modes

{{YOUR_COMPANY}} uses three distinct modes of AI-assisted development. Each maps to different work types, skill levels, and process needs. Choosing the right mode matters — it determines how much planning you do, how Claude Code receives instructions, and how work gets reviewed.

| | Mode 1: Session Pipeline | Mode 2: Rapid Prototype | Mode 3: Maintenance & Tweaks |
|---|---|---|---|
| **When to use** | Greenfield builds with clear requirements, multi-session arcs | Exploratory work, fast iteration, building from scratch with domain expert in the loop | Bug fixes, small features, UI polish, ongoing maintenance on existing codebases |
| **Upfront planning** | Heavy — PRD, session plan, dependency map | Light — Blueprint + project context, conversational scoping | Minimal — describe the issue, point to the code |
| **Who generates the prompt** | Cowork (via session-prompt skill or manually) | No prompt — direct conversation with Claude Code | Cowork (via maintenance template) |
| **Git discipline** | One session = one branch = one PR | Informal or no branching (prototyping) | One issue = one branch = one PR |
| **Review pipeline** | Copilot review → remediation → merge | Domain expert review (visual, functional) | Optional Copilot review, lighter remediation |
| **Session continuity** | Formal closeouts + carry-forwards | Memory file + process learnings | Standalone (each tweak is self-contained) |
| **Best for** | large-scale product rebuild (multi-session) | exploratory prototype (domain expert in the loop) | Post-build maintenance by anyone |
| **Template** | `claude_code_session_template_production.md` | `claude_code_session_template_prototype.md` | `claude_code_maintenance_template.md` |

---

## Mode 1: Session Pipeline

**Pattern origin:** large-scale product rebuild (multi-session)
**Proven through:** 8 sessions, 7 PRs merged, 44+ process learnings

### How It Works

The Session Pipeline is a Cowork-to-Claude-Code relay. Cowork owns the planning, context assembly, and prompt generation. Claude Code owns the code generation. A human reviewer gates every merge.

```
┌─────────────────────────────────────────────────────────────┐
│                      COWORK LAYER                           │
│                                                             │
│  PRD + Session Plan + Carry-Forwards + Project CLAUDE.md    │
│           ↓                                                 │
│  Generate session prompt (skill or manual)                  │
│           ↓                                                 │
│  Prompt includes: goal, numbered steps, reference files,    │
│  rules, delivery instructions, carry-forward items          │
└──────────────────────┬──────────────────────────────────────┘
                       │ paste into Claude Code
┌──────────────────────▼──────────────────────────────────────┐
│                    CLAUDE CODE LAYER                         │
│                                                             │
│  Read CLAUDE.md → Read reference files → Plan → Build →     │
│  Test → Commit → Push → Open PR                            │
└──────────────────────┬──────────────────────────────────────┘
                       │ PR created
┌──────────────────────▼──────────────────────────────────────┐
│                    REVIEW LAYER                              │
│                                                             │
│  Copilot review → Extract feedback → Remediation prompt →   │
│  Claude Code fixes → Merge                                  │
└──────────────────────┬──────────────────────────────────────┘
                       │ session complete
┌──────────────────────▼──────────────────────────────────────┐
│                    CLOSEOUT LAYER                            │
│                                                             │
│  Cowork writes closeout incorporating Claude Code's         │
│  session doc → Carry-forwards feed next session prompt      │
└─────────────────────────────────────────────────────────────┘
```

### When to Use Mode 1

- Building a new product from a defined PRD
- Work that will be reviewed by engineers and merged to production
- Multi-session arcs where continuity between sessions matters
- When the orchestrator (not the coder) is driving the process

### Artifacts Produced

- Session prompts: `sessions/<project>_session<N>_prompt.md`
- Session closeouts: `sessions/<project>_session<N>_closeout.md`
- Findings: `sessions/<project>_session<N>_findings.md` or `findings/NNN_<slug>.md`
- PRs: One per session, targeting main

### Key Principle

The orchestrator never writes code. Cowork generates the prompt. Claude Code executes. The human reviews. The closeout feeds the next session. The loop compounds.

---

## Mode 2: Rapid Prototype

**Pattern origin:** exploratory prototype (domain expert in the loop)
**Proven through:** Working portal prototype with config-driven rating engine

### How It Works

Rapid Prototype is a direct conversation with Claude Code. There is no Cowork layer generating prompts — the user talks to Claude Code in real time, iterating on a working application. Context comes from persistent files (CLAUDE.md, Blueprint, PROJECT.md), not from session prompts.

```
┌─────────────────────────────────────────────────────────────┐
│                    CONTEXT LAYER                             │
│                                                             │
│  CLAUDE.md (who I am, how to work with me)                  │
│  + BLUEPRINT.md (company context, strategy)                 │
│  + PROJECT.md (what we're building, phases, constraints)    │
│  + memory/ (process learnings, open items)                  │
│                                                             │
│  Claude Code reads all of these at session start.           │
└──────────────────────┬──────────────────────────────────────┘
                       │ context loaded
┌──────────────────────▼──────────────────────────────────────┐
│                    BUILD LOOP                                │
│                                                             │
│  "What are we building today?" →                            │
│  Back and forth: plan → build → screenshot → iterate →      │
│  "Does this look right?" → adjust → next piece              │
│                                                             │
│  Dev server running. Visual verification after each step.   │
│  Session continues until the user is satisfied.             │
└──────────────────────┬──────────────────────────────────────┘
                       │ session ends
┌──────────────────────▼──────────────────────────────────────┐
│                    CAPTURE                                   │
│                                                             │
│  Update memory/context.md with what was learned     │
│  (process-learning entries + deferred items section)│
│  Working prototype is the primary deliverable               │
└─────────────────────────────────────────────────────────────┘
```

### When to Use Mode 2

- Exploratory or greenfield work where requirements are discovered during building
- The person building is the domain expert (they know the business rules)
- Speed matters more than process discipline
- Visual verification (screenshots, working UI) is the primary QA method
- The builder is non-technical and needs Claude to explain decisions

### Key Context Files

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Who the user is, how to communicate, working preferences |
| `BLUEPRINT.md` | Company identity, strategy, team, domain glossary |
| `PROJECT.md` | What's being built, phases, constraints, open questions |
| `memory/context.md` | Accumulated technical and process decisions |
| `memory/context.md` | Deferred items and questions for domain experts |

### Key Principles

- **Context layering:** Project instructions (the "what") live separately from CLAUDE.md (the "who"). This lets you reuse the project for multiple programs.
- **Session = conversation.** No formal prompt. The user and Claude Code co-discover the solution.
- **Human review is domain-expert review**, not code review. The domain expert validates the business logic, not an engineer reviewing the PR.
- **The Blueprint is the multiplier.** It encodes the full strategic context so every output is informed by the big picture.

---

## Mode 3: Maintenance & Tweaks

**Pattern origin:** Hybrid — takes Cowork prompt generation from Mode 1, drops the PRD/session plan dependency, keeps git discipline but scopes to individual issues.

### How It Works

Maintenance & Tweaks is for ongoing work on an existing codebase. The codebase already has patterns, conventions, and a working CLAUDE.md. You don't need a PRD or session plan — you need to describe the issue, point to the relevant code, and let Claude Code fix it.

Cowork's role here is lighter than Mode 1: help the user articulate the issue clearly, identify the right reference files, and generate a focused prompt.

```
┌─────────────────────────────────────────────────────────────┐
│                      COWORK LAYER                           │
│                                                             │
│  User describes the issue/tweak in plain language           │
│  Cowork reads project CLAUDE.md for current state           │
│  Cowork asks clarifying questions if needed                 │
│           ↓                                                 │
│  Generate maintenance prompt:                               │
│  - Branch name                                              │
│  - Goal (one sentence)                                      │
│  - What to change (specific files, behaviors)               │
│  - Reference files (2-4 files showing existing patterns)    │
│  - Rules (match patterns, don't refactor, test it)          │
│  - Delivery instructions (commit, push, PR)                 │
└──────────────────────┬──────────────────────────────────────┘
                       │ paste into Claude Code
┌──────────────────────▼──────────────────────────────────────┐
│                    CLAUDE CODE LAYER                         │
│                                                             │
│  Read CLAUDE.md → Read reference files → Fix/Build →        │
│  Test → Commit → Push → Open PR                            │
└──────────────────────┬──────────────────────────────────────┘
                       │ PR created
┌──────────────────────▼──────────────────────────────────────┐
│                    REVIEW (OPTIONAL)                         │
│                                                             │
│  For small fixes: self-review + merge                       │
│  For anything touching business logic: Copilot or human     │
└─────────────────────────────────────────────────────────────┘
```

### When to Use Mode 3

- Bug fixes on a shipped or in-progress codebase
- Small feature additions (new field, new button, UI tweak)
- Dependency updates, config changes, environment fixes
- Anything that doesn't warrant a full session plan
- When you want Cowork to help you frame the issue before handing to Claude Code
- When you want to run the app and browse/test/tweak interactively (Explore & Refine variant)

### Two Variants

Mode 3 has two prompt variants:

- **Targeted Fix** — You know the issue upfront. Cowork generates a specific prompt with branch, issue, numbered changes, and delivery instructions. One-shot execution.
- **Explore & Refine** — You want to run the app, look at it, and make design decisions interactively. Cowork generates a setup prompt, then you direct Claude Code in real time with screenshots. This is the "product person testing their own app" pattern — useful when you haven't touched the codebase in a while or want to do UI polish.

### Key Difference from Mode 1

Mode 1 prompts are generated from a PRD and session plan with carry-forwards. Mode 3 prompts are generated from a plain-language description of the issue plus the project's existing context. No PRD needed, no session plan needed, no closeout needed.

### Key Difference from Mode 2

Mode 2 is a direct conversation with Claude Code. Mode 3 uses Cowork as a prompt generator, then hands a structured prompt to Claude Code. This is useful when the person doing the work isn't deeply familiar with the codebase (or hasn't touched it in a while) and needs Cowork to help assemble the context.

---

## Choosing the Right Mode

```
Is this a multi-session build with a PRD?
  ├── Yes → Mode 1: Session Pipeline
  └── No
       ├── Am I the domain expert building interactively?
       │    └── Yes → Mode 2: Rapid Prototype
       └── Is this a specific fix/tweak on an existing codebase?
            └── Yes → Mode 3: Maintenance & Tweaks
```

### Mode Transitions

Projects naturally move between modes over time:

- **Mode 2 → Mode 1:** A prototype matures and needs production discipline (git, reviews, PRs)
- **Mode 1 → Mode 3:** A product ships and enters maintenance
- **Mode 2 → Mode 3:** A prototype is "good enough" and just needs tweaks
- **Mode 3 → Mode 1:** A "small tweak" turns out to need a session plan (scope creep signal)

---

## Template Locations

All templates live in `templates/` in this anchor:

| Template | Mode | File |
|----------|------|------|
| Session Pipeline (Production) | Mode 1 | `claude_code_session_template_production.md` |
| Rapid Prototype | Mode 2 | `claude_code_session_template_prototype.md` |
| Maintenance & Tweaks | Mode 3 | `claude_code_maintenance_template.md` |
| Modes Overview (this doc) | Reference | `ai_development_modes_overview.md` |

---

*Three modes, one goal: get the right work done at the right speed with the right amount of process.*
*Choose the mode that matches the work. Switch modes when the work changes.*
