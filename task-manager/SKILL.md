---
name: task-manager
description: >
  Manage the user's persistent working memory under ~/agent-vault/ — daily diaries, task folders
  (ongoing vs done), weekly/monthly rollups, and the tasks MEMORY.md fast-catch-up file. Use this
  whenever the user asks about their tasks, progress, plan, what to do next, what they did today,
  or references their diary. Also trigger on "update tasks", "close the X task", "start a task on
  Y", "summarize this week", "summarize this month". Each invocation commits its edits to git.
---

# Task Manager

Maintain the user's working memory at `~/agent-vault/`. Three data areas: `diaries/`, `tasks/`, `reports/`. This skill owns `diaries/` and `tasks/`; `reports/` is owned by the `report-agent` skill.

## Vault layout (authoritative)

```
~/agent-vault/
├── MEMORY.md                       # (optional) top-level catch-up
├── OVERVIEW.md                     # top-level index
├── diaries/
│   ├── MEMORY.md                   # ≤500 words: last 3 days + tasks in flight
│   ├── OVERVIEW.md                 # per-day index (newest on top)
│   ├── daily/   YYYY-MM-DD.md
│   ├── weekly/  YYYY-Www.md        # ISO week
│   └── monthly/ YYYY-MM.md
└── tasks/
    ├── MEMORY.md                   # ≤500 words: active tasks + top open items
    ├── OVERVIEW.md                 # per-task index (ongoing + done tables)
    ├── ongoing/<task-name>/
    │   ├── OVERVIEW.md             # task dashboard (frontmatter + open items + reports)
    │   ├── notes/                  # optional raw notes
    │   └── data/                   # optional raw data
    └── done/<task-name>/           # moved here when closed
```

## Read order — fast path first

Any question about "what did I do", "what's next", "do I remember X" — follow this order, stop as soon as you have the answer:

1. **`tasks/MEMORY.md`** and **`diaries/MEMORY.md`** (small, always load first).
2. If unanswered and question is about history/trends: **`diaries/OVERVIEW.md`** or **`tasks/OVERVIEW.md`** (grep, don't read whole).
3. If unanswered: the specific **`daily/YYYY-MM-DD.md`** or **`ongoing/<task>/OVERVIEW.md`** file.
4. If still unanswered: raw files under `notes/` or `data/` of a task.

Don't read every daily or every task — grep the OVERVIEW, then open one file.

## Daily diary format (new format, 2026-04-24 onward)

```yaml
---
date: YYYY-MM-DD
tags: [short, distinctive, keywords]
tasks_touched:
  - <task-name>
  - <task-name>
done:
  - "bullet describing what was accomplished today (past tense, specific)"
notes: |
  optional freeform
---
```

**No `todo:` field.** Open todos live on each task's `OVERVIEW.md`. The daily is an event log, not a backlog.

## Tasks folder format

Each task has an `OVERVIEW.md` with this frontmatter:

```yaml
---
name: kebab-case-task-name
status: ongoing     # or: done
created: YYYY-MM-DD
closed: null        # or: YYYY-MM-DD
owner_request: <who asked — self, Denis, Putti-san, team social, etc>
priority: high      # high | medium | low
reports: [<report-name-without-extension>, ...]
diaries: [YYYY-MM-DD, ...]
---
```

Body sections: `## Why`, `## Status`, `## Open items` (checklist of what's left), `## Reports` (links).

## Workflows

### When the user shares something new in chat (the "Diary Update Feature")

1. Load `tasks/MEMORY.md` and `diaries/MEMORY.md`.
2. Decide which existing tasks this information touches (check tags/names — Denis, Putti-san, rendering, HEVC, BBQ, etc).
3. Append to today's `daily/YYYY-MM-DD.md` (create if missing). Record the event in `done:` and add task names to `tasks_touched:`.
4. Update each touched task's `OVERVIEW.md`:
   - Append new open items to the task's `## Open items` section if the news introduces new work.
   - If a todo in that task is now done (based on what the user said), strike it or move it to a `## Resolved` section with the outcome.
   - Bump `diaries:` frontmatter to include today's date.
5. **Hybrid task creation (type C rule):** if the news introduces a long-lived work item that doesn't fit any existing task, **suggest** creating a new task folder — don't auto-create. Ask: "Looks like `<short description>` is a new long-lived task — want me to create `tasks/ongoing/<kebab-name>/`?"
6. Refresh `diaries/MEMORY.md` and `tasks/MEMORY.md` (see "MEMORY.md maintenance" below).
7. Refresh `diaries/OVERVIEW.md` and `tasks/OVERVIEW.md` blocks for what changed.
8. **Commit** (see "Git" section).
9. Present the Task Summary (format below).

### Starting a new task (explicit)

User says "start a task on X" / "create a task for Y":

1. Pick a kebab-case name (descriptive, not dated).
2. `mkdir -p ~/agent-vault/tasks/ongoing/<name>/`.
3. Write `OVERVIEW.md` with frontmatter + `## Why` + `## Status` + `## Open items`.
4. Append a new block to `tasks/OVERVIEW.md` Ongoing table.
5. Refresh `tasks/MEMORY.md`.
6. Commit.

### Closing a task

User says "close the X task" / "mark X done":

1. `git mv ~/agent-vault/tasks/ongoing/<name>/ ~/agent-vault/tasks/done/<name>/`.
2. Update the task's `OVERVIEW.md` frontmatter: `status: done`, `closed: YYYY-MM-DD`. Add a `## Outcome` section (ask the user for the one-line result if not obvious).
3. Move the row in `tasks/OVERVIEW.md` from Ongoing to Done.
4. Refresh `tasks/MEMORY.md`.
5. Commit.

### "Update tasks" — todo reconciliation

When the user says "update tasks", do NOT silently rewrite:

1. Read each ongoing task's `## Open items`.
2. If ≤5 open items total across tasks, ask per-item. If more, group by task and ask which tasks have progress.
3. Wait for answers.
4. For each confirmed-done item, move it to `## Resolved` on its task (with one-line outcome if user provided).
5. Refresh MEMORY and OVERVIEW files.
6. Commit.

### Weekly / monthly rollup

User says "summarize this week" / "summarize April":

- **Weekly:** determine ISO week. Write `diaries/weekly/YYYY-Www.md` summarizing the 7 days: tasks that shipped, tasks that started, stuck items, top 3 decisions. Link to daily files. Keep under ~400 words.
- **Monthly:** write `diaries/monthly/YYYY-MM.md` aggregating the month's weeklies. Keep under ~600 words.
- Commit.

## MEMORY.md maintenance (≤500 words each, HARD CAP)

Three MEMORY.md files to keep fresh:
- `diaries/MEMORY.md` — last 3 days (one-liner each) + active context.
- `tasks/MEMORY.md` — active tasks list + top 5 open items across all tasks.
- (optional) `~/agent-vault/MEMORY.md` — global, if useful.

Rebuild at end of every turn that changed tasks or diaries. Fail loudly (refuse to write) if a MEMORY.md would exceed 500 words — trim older entries instead.

Template:

```markdown
---
updated: YYYY-MM-DD
---

# <Area> MEMORY — last updated YYYY-MM-DD

## <section>
- ...
```

## Git

At the end of every invocation that made edits:

```bash
cd ~/agent-vault && git add -A && git commit -m "task-manager: <short action>"
```

Don't push here — a SessionEnd hook pushes both `~/agent-vault` and `~/.agents/skills` at session end. If the commit fails because there's nothing to commit, don't error — just proceed.

## Output — Task Summary

Use this exact shape when reporting back:

```
## Task Summary — <date>

### ✅ Done
- <item>

### 📋 Open (by task)
- **<task-name>:** <top 1-3 open items>
- **<task-name>:** <top 1-3 open items>

### ℹ Notes
- <any suggestions, e.g. "this looks like a candidate for a new task — want me to create it?">
```

If there's nothing, say "Nothing recorded yet." for that section.

## Rules

- **Never invent** task names, todos, results, or dates. If the user is vague, ask.
- **Never move a task to done** without explicit user confirmation.
- **Never rewrite historical diaries** to the new format unless the user asks.
- **Absolute dates only** in stored files ("2026-04-24", not "yesterday").
- **Kebab-case task names.** No `task1`, no dates as names.
- **500-word MEMORY cap** is hard. Trim older entries first.
- **Data integrity** — numbers, names, results are transcribed verbatim from what the user said. No fabrication.
