---
description: Read a task's status header and say which agent to run next
---

Resume a migration task.

1. Ask me for the task name if I did not give it.
2. Read the status header of `cc-tasks/<task-name>/planned_tasks.md` (or `initial.md` if the plan does not exist yet).
3. Report: status, iteration, blocked-from, open issues, and the last agent that updated it.
4. Tell me the next step and give me the exact prompt to run, using this table:

| Status | Next step |
|---|---|
| Draft | `@plan` (answer its question batch if pending) |
| Plan Ready | Developer review, then set `Approved` |
| Approved | `@migration` |
| In Migration or Blocked (from migration) | Answer the open question in `migration-output/gaps.md`, then `@migration` |
| Blocked (from Draft or Plan Revision) | Answer the pending questions, then `@plan` |
| Migrated | `@review` |
| In Review | `@review` (resume) |
| Needs Fix | Code gap: `@migration`. Plan gap: `@plan`. Review findings say which |
| Plan Revision | `@plan` |
| Done | Nothing. Wire the feature using the entry contract in `migration-output/summary.md` |

5. Do not change any files.
