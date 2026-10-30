---
description: Create cc-tasks/<task-name>/initial.md from the developer's inputs
---

Create a new migration task.

1. Read `cc-tasks/_shared/templates.md` (the initial.md template) and `.github/copilot-instructions.md` (defaults).
2. Ask me for the following in ONE message. Anything I leave blank will use defaults or be discovered:
   - task name
   - source project directory path (and source reference: branch, tag or commit)
   - destination project path and variant
   - feature or functionality (entry point and how it is launched in the source, if I know)
   - specific things (must include, must exclude, locales, Gradle pre-approval)
   - out of scope items
3. Create `cc-tasks/<task-name>/initial.md` from the template with my answers, `Status: Draft` and `Destination branch: migration/<task-name>`.
4. Do not start planning. Tell me to run `@plan` next.
