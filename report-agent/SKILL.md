---
name: report-agent
description: >
  Create and maintain free-form markdown reports in ~/reports/. Use this skill whenever the user
  wants to write, update, or look up a report — e.g. "start a report on X", "add this to the HEVC
  report", "write up the benchmark results", "make a report for Putti-san about Y", "what did I
  write in the streaming report", "find my notes on Z". The agent reads the report index first,
  decides whether the new information belongs in an existing report or a new one, and keeps the
  index in sync.
---

# Report Agent

Help the user capture information into well-organized markdown reports under `~/reports/`, and retrieve from them later. Format is free — shape the report around the content the user provides, not a rigid template.

Reports are often **data reports**: benchmark numbers, latency measurements, throughput comparisons, before/after results, configuration matrices. These are high-value artifacts — the numbers drive real decisions, get shown to teammates, and get cited later. **Treat the data as sacred.** Transcribe it exactly as given. Never round silently, never fill missing cells with plausible-looking values, never fabricate samples, never "smooth" inconsistent entries. If a value is unclear or missing, ask the user — do not guess.

## Location

- Reports live in `~/reports/` (i.e. `/home/ficha/reports/`).
- Each report is a single markdown file: `~/reports/<report_name>.md`. Use `kebab-case` names (e.g. `hevc-rtp-streaming.md`, `godot-192-fov.md`).
- A single index file `~/reports/SUMMARY.md` lists every report with a short digest and keywords.

## SUMMARY.md — the lookup index

`SUMMARY.md` is the **first place to read** whenever the user asks about a report or wants to add information. Grep it to find the right file, *then* open the specific report for detail. This keeps you from reading every report on every question.

### Format

Plain markdown, newest-touched on top, one block per report:

```
## <report-name>
**File:** `<report-name>.md`
**Created:** YYYY-MM-DD · **Updated:** YYYY-MM-DD
**Topic:** one-sentence description of what this report is about
**Covers:** short digest of the main sections / findings / decisions inside
**Keywords:** searchable terms — tools, people, files, concepts, error messages
```

Keep each block ≤6 lines. Be generous with distinctive keywords (library names, ticket IDs, people's names, specific numbers, filenames) — those are what you grep against later.

## Workflow — adding information

When the user provides information that should become a report (or extend one):

1. **Read `~/reports/SUMMARY.md`** (create it if missing).
2. **Decide the destination** based on the content's topic:
   - If an existing report clearly covers this topic → append to that file.
   - If no existing report fits → create a new `~/reports/<report-name>.md`.
   - If unsure which of two reports fits best, ask the user briefly before writing.
3. **Shape the content** — don't just dump raw input. Understand the context and:
   - Pick appropriate section headings (`##`, `###`) based on what the user said.
   - Preserve tables, code blocks, numbers, and names verbatim.
   - Add a dated subsection (`### Update — YYYY-MM-DD`) when appending to an existing report so the timeline stays clear.
   - For a new report, start with a `# Title` and a one-paragraph context intro before the detail.
4. **Write / edit** the report file.
5. **Update `SUMMARY.md`**: refresh the block for that report (bump **Updated**, extend **Covers** and **Keywords** as needed), or insert a new block at the top for a new report.
6. **Confirm** to the user: say which file you wrote to, whether it was new or appended, and point out anything you had to guess about routing.

## Workflow — looking up information

When the user asks about past report content ("what did I write about…", "find my notes on…", "remind me of the numbers from…"):

1. **Read or grep `SUMMARY.md` first.** Match against topics and keywords.
2. Identify the candidate report file(s).
3. **Open only those files** and extract the answer. Don't read every report.
4. Answer the user, citing the file path (e.g. `~/reports/hevc-rtp-streaming.md`) so they can open it directly.
5. If `SUMMARY.md` has no matching entry but a report file exists for the topic, read that file and **backfill** its `SUMMARY.md` block before answering.

## Rules

- **Never overwrite** an existing report without explicit confirmation — append instead.
- **Never invent** numbers, names, or facts. If information is missing, ask the user.
- **Don't create a new report** just because the wording is different — prefer appending if the topic matches.
- **Keep `SUMMARY.md` in sync on every write.** An out-of-date index defeats the purpose.
- If `~/reports/` doesn't exist yet, create it on first use.
- Free format means free *structure*, not free *quality* — headings, lists, tables, and code fences should still be used where they make the content clearer.

## Data reports — tables and charts

When the user provides measurements, benchmarks, or comparisons, shape them into readable structures. Use whichever form fits the data.

### Tables

Use markdown tables for per-sample measurements, comparisons, and configuration matrices. Right-align numeric columns, include units in the header, and always include a bold **Avg** / **Total** / **Median** row when the user provided one (or when one is trivially derivable from the given samples — but only arithmetic on the exact numbers provided, no extrapolation).

Example shape (the numbers here are *placeholders for illustration only* — never use them in a real report):

```
| Sample  | readback (ms) | encode (ms) | total (ms) |
|---------|--------------:|------------:|-----------:|
| 1       |        <val> |       <val> |      <val> |
| 2       |        <val> |       <val> |      <val> |
| **Avg** |    **<val>** |   **<val>** |  **<val>** |
```

### Charts

Markdown has no native chart, so use one of these, in order of preference:

1. **Mermaid** fenced blocks (` ```mermaid `) for bar charts, line charts, pie, gantt, sequence, flow — renders in most markdown viewers including GitHub and VS Code. Good default for quick visual summaries of data the user gave you.
2. **ASCII tables or sparkline-style bars** (`█████░░░`) when the user just wants a rough visual next to the numbers.
3. **A link to a generated image** (`![...](./charts/<name>.png)`) only if the user explicitly provides the image or asks for one to be generated — never invent a filename that doesn't exist.

Example Mermaid shape (placeholder — replace every value with the user's actual data):

```mermaid
xychart-beta
  title "<chart title from user>"
  x-axis [<label1>, <label2>, <label3>]
  y-axis "<unit>" 0 --> <max>
  bar [<val1>, <val2>, <val3>]
```

### Data integrity checklist (run before writing)

- Every number in the report came from the user's message (or is a transparent arithmetic aggregate of those numbers).
- Units, precision, and sample count match what the user provided.
- No samples were dropped, reordered, or renamed without noting it.
- If something was ambiguous, the report says so explicitly instead of picking silently.
- If any value is missing, stop and ask the user rather than writing a partial table with blanks filled in.

## Naming guidance

- `<report-name>` should describe the topic, not the date or the author. Good: `udp-packet-fragmentation.md`, `bosch-onboarding.md`, `asio-build-fix.md`. Bad: `report1.md`, `2026-04-14.md`, `my-notes.md`.
- If the user names the report explicitly, use their name verbatim (kebab-case it).
