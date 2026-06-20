# Scheduled Tasks (the agent portfolio)

These are autonomous Cowork **scheduled tasks** — the "Outside-the-Line" layer that runs on a cron between your working sessions, does a recurring information-gathering job, and drops its output into your `inbox/` for review. All five are live + enabled on the reference anchor.

## The portfolio

| Task | Cadence | Output |
|------|---------|--------|
| **daily-true-up** | Weekdays (late afternoon) | `inbox/true_up_<date>.md` |
| **slack-signal-monitor** | Weekdays (mid-afternoon) | `inbox/slack_signal_<date>.md` |
| **weekly-anchor-audit** | Fridays | `memory/anchor-audit-report.md` |
| **commitment-follow-up-tracker** | Mid-week | `inbox/commitment_tracker_<date>.md` |
| **monday-morning-briefing** | Mondays (morning) | `inbox/monday_briefing_<date>.md` |

> See each agent's SKILL.md for recommended cadence.

They form a loop: **true-up** captures meetings → **signal monitor** captures Slack → **commitment tracker** chases follow-ups → **monday briefing** synthesizes the week ahead → **weekly audit** is the meta-watchdog that checks the anchor's health *and the other four agents'* health.

## How to recreate one in your Cowork
Each file has an **"Adapt before you wire this up"** block at the top — read it first. Then the simplest path: hand the file to your Claude and say *"set this up as a scheduled task, swapping in my paths, connectors, and channels."* The **`schedule`** skill turns plain English ("every weekday at 4pm, scan these channels and write a digest…") into a real scheduled task, with the adapted prompt as the task body. You can also create/edit tasks in Cowork's settings.

## Two hard constraints they all share
- **Runtime:** a scheduled session does **not** have your anchor folder mounted, and can't use the normal file tools or bash to reach it. Scheduled agents need the full absolute path to your anchor — this differs by OS (Windows vs. Mac). See SETUP.md.
- **Connectors are yours to reconnect.** The MCP connector IDs in the source agent prompts are specific to the original anchor — reconnect your own connectors and Claude will generate the correct IDs automatically.

## Output convention (so the audit can watch them)
Every run writes a file with a standard header and appends an `agent-run` event to `ops/event-log.md`:
```
> Run: <ISO UTC> · Status: GREEN|YELLOW|RED · Items found: <n> · Channels scanned: <…> · Next run: <…>
```
The **weekly-anchor-audit** reads those events to confirm each agent ran on cadence — if one goes silently dark, the audit flags it RED. That self-monitoring is the point.

## Design principles (the part worth internalizing)
- **Build the *second* one, not the first.** Solve a recurring problem by hand at least once before automating it — you'll design a far better agent.
- **Group by capability domain, not department.** ingestion · monitoring · tracking · synthesis · maintenance. Cross-cutting by nature.
- **Pace to your review capacity.** Five agents producing daily output is only useful if you actually read the inbox. Start with one or two (signal-monitor + monday-briefing are the highest-leverage for most roles), prove the value, then add.
