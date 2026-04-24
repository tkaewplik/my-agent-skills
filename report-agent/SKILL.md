---
name: report-agent
description: >
  Create and maintain free-form markdown reports under ~/agent-vault/reports/. Every report links
  to a parent task in ~/agent-vault/tasks/ via YAML frontmatter (`task: <task-name>`). Use this
  whenever the user wants to write, update, or look up a report — "start a report on X", "add this
  to the HEVC report", "write up the benchmark results", "find my notes on Z". Reads reports
  MEMORY.md first for a fast catch-up, then OVERVIEW.md, then the specific report.
---

# Report Agent

Own the reports area of the user's vault at `~/agent-vault/reports/`. Every report is attached to a task managed by the `task-manager` skill.

Data reports (benchmark numbers, latency measurements, throughput comparisons, config matrices) are high-value artifacts — the numbers drive real decisions and get cited later. **Treat the data as sacred.** Transcribe verbatim. Never round silently, never fill missing cells, never fabricate samples. If a value is unclear or missing, ask the user.

## Location

```
~/agent-vault/reports/
├── MEMORY.md                 # ≤500 words: most recent reports, what's in flight
├── OVERVIEW.md               # per-report index (newest-touched on top)
└── <report-name>.md          # kebab-case, topic-describing — no dates in the name
```

Tasks live in `~/agent-vault/tasks/ongoing/<task-name>/` and `~/agent-vault/tasks/done/<task-name>/`. Each report file's frontmatter `task:` field points to one of those.

## Report file format

Every report starts with YAML frontmatter:

```yaml
---
task: <task-name>         # REQUIRED — must match an existing folder under tasks/
task_status: ongoing      # or: done (mirrors the task's status for fast grep)
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# <Report Title>
...
```

Below the frontmatter, shape the content around what the user gave — headings, tables, fenced code, Mermaid diagrams as appropriate. Free format means free *structure*, not free *quality*.

## Read order — fast path first

1. **`reports/MEMORY.md`** — check if the question is answerable from the last-few-reports summary.
2. If not: **`reports/OVERVIEW.md`** — grep for topic/keyword, identify the candidate file.
3. Open only that file.
4. If the question is about a task's reports collectively: look at the task's `OVERVIEW.md` under `## Reports` instead.

## Workflows

### Adding information / creating a report

1. Load `reports/MEMORY.md` (and `tasks/MEMORY.md` so you know the active task names).
2. Decide:
   - **Append to existing report** if topic matches → add a dated subsection `### Update — YYYY-MM-DD`.
   - **New report** otherwise → pick a kebab-case name.
3. **Require the `task:` link.** If the user didn't say which task:
   - If one task is an obvious fit, propose it: "I'll attach this to `hevc-gdextension-migration` — OK?"
   - If no task fits, ask: "Which task does this belong to?" or offer to create a new task first (and let `task-manager` handle it).
4. Write the report (with frontmatter).
5. Update the parent task's `OVERVIEW.md`:
   - Add the report name to the `reports:` frontmatter list.
   - Add a link under the `## Reports` section.
6. Update `reports/OVERVIEW.md` — refresh (or insert at top) the block for this report, including the `**Task:**` line.
7. Refresh `reports/MEMORY.md` — put this report at the top of "most recent".
8. **Commit** (see Git section).

### Looking up a report

1. Read `reports/MEMORY.md`.
2. If not found, grep `reports/OVERVIEW.md` (topics + keywords).
3. Open the matching file.
4. Answer, citing the file path so the user can open it directly.

If `OVERVIEW.md` has no matching block but a report file exists, read the file and **backfill** its `OVERVIEW.md` block before answering.

## OVERVIEW.md block format

```
## <report-name>
**File:** `<report-name>.md`
**Task:** [<task-name>](../tasks/ongoing/<task-name>/OVERVIEW.md)   <!-- or tasks/done/... -->
**Created:** YYYY-MM-DD · **Updated:** YYYY-MM-DD
**Topic:** one-sentence description.
**Covers:** short digest of main sections / findings / numbers.
**Keywords:** searchable terms — tools, people, filenames, error messages, numbers.
```

Keep each block ≤6 lines. Be generous with distinctive keywords.

## MEMORY.md maintenance (≤500 words, HARD CAP)

Rebuild `reports/MEMORY.md` at the end of every invocation that wrote anything. Structure:

```markdown
---
updated: YYYY-MM-DD
---

# Reports MEMORY — last updated YYYY-MM-DD

## Most recent reports
- **<report-name>.md** (YYYY-MM-DD, task: <task-name>) — one-line summary.

## Older (archived under <done-task>)
- ...

## What's in flight (not yet a report)
- <thing> (task: <task-name>)

## How to catch up fast
- (pointers)
```

Fail loudly if it would exceed 500 words — trim older entries instead.

## Data reports — tables and charts

**Tables:** markdown tables for per-sample measurements. Right-align numeric columns, include units in headers, bold an **Avg** / **Total** / **Median** row only if the user provided it or it's trivial arithmetic on the exact values given.

**Charts:**
1. **Mermaid** fenced blocks (`xychart-beta`, `gantt`, `sequenceDiagram`, etc) for quick visual summaries.
2. ASCII bars (`█████░░░`) for inline sparkline-style.
3. Generated image file only if the user provides or explicitly asks for one.

**Data integrity checklist — run before writing:**
- Every number came from the user's message (or is a transparent arithmetic aggregate of those numbers).
- Units, precision, and sample count match the user's input.
- No samples were dropped, reordered, or renamed without noting it.
- Ambiguities are called out in the report; not silently picked.
- Missing values → stop and ask, not fill.

## Git

At the end of every invocation that made edits:

```bash
cd ~/agent-vault && git add -A && git commit -m "report-agent: <short action>"
```

Don't push — a SessionEnd hook pushes both `~/agent-vault` and `~/.agents/skills`. Empty commits ("nothing to commit") are not errors — just proceed.

## Rules

- **Every report MUST have a `task:` frontmatter field** pointing to an existing task folder. If none fits, coordinate with `task-manager` to create one first.
- **Never overwrite** an existing report without confirmation — append a dated subsection instead.
- **Never invent** numbers, names, or facts.
- **Don't create a new report** just because the wording is different — prefer appending if the topic matches.
- **Keep `OVERVIEW.md` and the parent task's `reports:` list in sync on every write.**
- **Report names:** kebab-case, topic-describing. No dates as names, no `report1.md`.
- **Absolute dates only** ("2026-04-24", never "yesterday").
