# Android Onboarding Agent
**Author:** Rajendhiran Easu

## Purpose

Explore an Android project task by task and assemble one consolidated,
read-only onboarding document: `explored_<YYYY-MM-DD_HH-MM>.md`, written
to the project root. Designed to be resumable across many turns/runs so
it works on large codebases that cannot be fully explored in one pass.

This file is the orchestrator. It decides what runs next. The actual
"how" for each kind of analysis lives in `SKILL.md`, and the bounded
unit of work for each area lives in `tasks/*.md`.

## Hard rules

- **Read-only.** Never modify, delete, or generate application source
  code, build files, or resources.
- The only files this workflow may write are:
  - `.github/skills/android-onboarding/progress.md`
  - `draft_<timestamp>.md` (scratch file, project root)
  - `explored_<timestamp>.md` (final output, project root — written
    only once, at finalization)
- Run **exactly one task per invocation** (one sub-module pass counts
  as one task-invocation for task 09, see that task file). Do not chain
  multiple tasks in a single turn even if there is room — this is what
  keeps the run reviewable and safely resumable.
- Never overwrite an existing `explored_*.md`. Each full run produces a
  new, timestamped file.

## On every invocation

1. **Read state.** Look for
   `.github/skills/android-onboarding/progress.md`.
   - If it does not exist, create it using the template below. Generate
     the timestamp from the system clock right now (e.g. `date +%F_%H-%M`)
     and set `output_file: explored_<that timestamp>.md`. Also record
     the current commit hash (`git rev-parse --short HEAD`) if this is
     a Git repo, and the branch name.
   - If it exists, read it as-is. Do not regenerate the timestamp or
     module list from scratch — reuse what's recorded, except where a
     task's own instructions say to refresh something (e.g. module list
     at the start of a new full run).

2. **Pick the next task.** Scan the "Task status" list in progress.md
   top to bottom. Find the first task that is not `done`. If a task has
   per-module sub-status (currently only task 09), find the first
   undone module within it.
   - If every task is `done`, skip to **Finalization** below.

3. **Announce.** State in one short line which task (and sub-scope, if
   any) you're about to run.

4. **Run the task.** Open the matching file under `tasks/`. Follow its
   steps, using `SKILL.md` for search patterns, templates, and diagram
   conventions referenced by the task. Do not duplicate SKILL.md content
   from memory — read it.

5. **Self-check.** Every task file ends with a self-check list. Run it.
   Do not mark the task (or sub-module) done until it passes. If it
   fails, fix the output within the same invocation before finishing.

6. **Write output.** Append the task's output to
   `draft_<timestamp>.md` (create it if this is the first task), under
   a clearly marked heading matching the task's section name. Never
   overwrite earlier sections already in the draft.

7. **Update progress.md:**
   - Mark the task (or sub-module) `done`, with a timestamp.
   - Append any new entries to "Open questions" or "Gaps" that the task
     surfaced.
   - Update `last_updated`.

8. **Report and stop.** Tell the user, in 2–4 lines: what was done,
   2–3 notable findings, and what the next task will be. Do not
   continue automatically into the next task.

## Finalization (runs only when every task shows `done`)

1. Run `tasks/11-assembly.md`: merge every section of
   `draft_<timestamp>.md` into the final document structure (see
   SKILL.md's section list), de-duplicate any repeated content, verify
   every Mermaid block parses, verify every claim marked Confirmed has
   a `path:line` citation, and move the consolidated "Open questions"
   and "Gaps" lists to just after the run metadata at the top of the
   document.
2. Write the merged result to `explored_<timestamp>.md` in the project
   root (the exact filename recorded in progress.md's `output_file`).
3. Delete `draft_<timestamp>.md`.
4. Report to the user: final file path, top 3 gaps by priority, total
   count of open questions, and module/class coverage counts (e.g.
   "11/11 modules explored, 47 classes carded, 212 classes cataloged").

## Resuming after an interruption

If a task was marked `in-progress` but not `done` when the previous
invocation ended, treat it as not started: re-read the task file and
redo it from scratch. Tasks are scoped to be small and cheap enough
that redoing one costs little, and it's safer than trying to resume
mid-task from an unknown partial state.

## Starting a new full run

If the user explicitly asks to start a *new* exploration (not continue
the current one), and `progress.md` shows a fully `done` run already:
rename or archive the old `progress.md` (e.g. move its content into a
dated note, or simply note the prior run's output_file for reference),
then create a fresh `progress.md` with a new timestamp and re-discover
the module list from `settings.gradle(.kts)` rather than reusing the
old one.

## progress.md template (used when creating it fresh)

```markdown
# Onboarding exploration progress

output_file: explored_<YYYY-MM-DD_HH-MM>.md
started: <timestamp>
last_updated: <timestamp>
commit: <short hash, or "not a git repo">
branch: <branch name, or "n/a">

## Task status
- [ ] 00-setup — pending
- [ ] 01-project-shape — pending
- [ ] 02-architecture — pending
- [ ] 03-navigation — pending
- [ ] 04-features — pending
- [ ] 05-data-flow-api — pending
- [ ] 06-dependency-injection — pending
- [ ] 07-ui-approach — pending
- [ ] 08-libraries — pending
- [ ] 09-key-classes — pending (sub-status tracked per module once started)
- [ ] 10-legacy-and-gaps — pending
- [ ] 11-assembly — pending

## Modules discovered
(filled in by 00-setup)

## Running list of open questions
(none yet)

## Running list of gaps
(none yet)
```
