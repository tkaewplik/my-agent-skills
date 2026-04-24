---
name: task-manager
description: >
  Check the user's latest diary entry in ~/diaries/ and summarize what they have already done
  and what they still need to do. Use this skill whenever the user asks about their tasks,
  progress, what to do next, what they've learned or accomplished, or references their diary.
  Also trigger when the user says things like "what's my plan?", "what did I do today?",
  "what's left to do?", "show my tasks", or "check my diary".
---

# Task Manager

Help the user understand their current progress and next steps by reading their latest diary entry.

## Diary location

Diaries are stored in `~/diaries/` (i.e. `/home/ficha/diaries/`).
Each file is named by date: `YYYY-MM-DD.md`.

## Summary memory file

A single index file at `~/diaries/SUMMARY.md` holds a one-entry-per-day digest of every diary. It is the **first place to look** whenever the user asks a historical or "do you remember…" question — grep it first, then open the specific day file(s) it points to for detail. This avoids reading every diary on every question.

### Format of `SUMMARY.md`

Plain markdown, newest date on top, one block per diary file:

```
## YYYY-MM-DD
**Tags:** tag1, tag2, tag3
**Done:** one-line digest of the day's accomplishments
**Todo carryover:** one-line digest of what was left open
**Keywords:** searchable terms (tools, people, files, concepts)
```

Keep each block ≤5 lines. Tags/keywords are what you grep against, so be generous with distinctive terms (library names, filenames, ticket IDs, people, error messages).

### When to read `SUMMARY.md`
- User asks about *past* work, not just today ("when did I…", "have I ever…", "what was that bug with X", "remind me about Y").
- User asks a question whose answer likely lives in an older diary.
- Before answering from memory or guessing — always check the index first.

Workflow: `Grep` or `Read` `SUMMARY.md` → identify candidate date(s) → `Read` the specific `YYYY-MM-DD.md` files → answer.

### When to update `SUMMARY.md`
Whenever you create or modify a diary file (including the "Diary Update Feature" flow below), you MUST also update its corresponding block in `SUMMARY.md` in the same turn. If no block exists for that date, insert one at the top. If the file is missing entirely, create it.

Also: if the user asks about an older diary that has no `SUMMARY.md` entry, read that diary and backfill its block before answering.

## Steps

1. List all files in `~/diaries/` and find the one with the latest date (sort by filename descending, take the first).
2. Read that file.
3. Extract two categories of information:

   - **Done** — anything described as learned, completed, accomplished, or finished. Look in sections like "What I Learned Today", "Accomplished", `done:` YAML frontmatter, or any past-tense descriptions.
   - **To Do** — anything described as planned, upcoming, or intended for tomorrow/future. Look in sections like "Tomorrow's Plan", "Next Steps", `todo:` YAML frontmatter, or any future-tense descriptions.

4. Present the result using the output format below.

## Output format

Use this structure every time:

```
## Task Summary — <date of the diary entry>

### ✅ Done
- <item>
- <item>

### 📋 To Do
- <item>
- <item>
```

If there are no done items or no to-do items, say "Nothing recorded yet." for that section.
Keep each bullet point short and actionable — one line per item.

## Notes

- If there are no diary files at all, let the user know and suggest they create one.
- Diary files may use YAML frontmatter (with `done:` and `todo:` lists) OR plain markdown sections — handle both.
- Always tell the user which diary file you read (e.g., "Reading `2026-04-01.md`...).

## Diary Update Feature

When the user provides story, narrative, or information about accomplishments and next steps **in the chat conversation** (not just reading the diary), **also update the current date's diary file** with this new information:

1. Extract accomplishments mentioned as completed/finished work
2. Extract next steps and todos mentioned  
3. Add these to the matching date diary file (determined by current date in context)
4. Update both the YAML frontmatter (`done:` and `todo:` lists) and the markdown narrative sections
5. **Update `~/diaries/SUMMARY.md`** — refresh (or insert) the block for that date so the index stays in sync
6. After updating, present the updated Task Summary using the output format above

## "Update tasks" trigger — todo reconciliation

When the user says **"update tasks"** (or any close variant like "update the tasks", "update todos", "reconcile tasks"), do NOT silently rewrite the todo list. Instead, walk through the existing open todos with the user and ask which ones are done.

Workflow:

1. Read the latest diary file and extract its current open `todo:` list.
2. Decide how to present the list for confirmation:
   - **Few open todos (≤5):** ask about each one individually — one bullet per todo, yes/no/partial. Example: "Is `(Putti-san) 4-camera framerate drop` done? If yes, what was the result so I can move it to done?"
   - **Many open todos (>5):** don't spam the user with a long per-item checklist. Summarize them into logical groups (e.g. "Putti-san requests", "streaming pipeline work", "carryover research") and ask the user which group(s) have progress to report, then drill into just those.
3. Wait for the user's answers before editing any file.
4. Once the user confirms which todos are done (and gives any result/context for them), move those items from `todo:` to `done:` in the diary frontmatter, update the narrative sections accordingly, and refresh `~/diaries/SUMMARY.md`.
5. Present the updated Task Summary using the standard output format.

Rules:

- Never assume a todo is done without explicit user confirmation.
- Never invent a result to put next to a moved todo — if the user says "done" without detail, either ask for the one-line result or move it with no extra text rather than fabricating one.
- If the user only answers about some items, leave the rest in `todo:` untouched.
