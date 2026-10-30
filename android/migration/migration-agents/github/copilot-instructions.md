# Android feature migration: always-on rules

These rules apply only when the work is a feature migration (anything under `cc-tasks/`, or a request to move a feature from a source project into a destination variant). For all other work, ignore this file.

Before any migration work, read `cc-tasks/_shared/rules.md` fully. It is the single source of truth. Templates for output files are in `cc-tasks/_shared/templates.md`.

## Defaults (fill in once)

- Default source project path: `<FILL_IN>`
- Default destination project path: `<FILL_IN>`
- Default destination variant (flavor): `<FILL_IN>`
- Default build commands: `assemble<Flavor>Debug`, `test<Flavor>DebugUnitTest`

Blank fields in a task's `initial.md` resolve to these defaults. State which defaults were used.

## Non-negotiables

1. The source project is read-only. Never write, modify, delete or format anything there.
2. The only writable code area is the destination variant (`src/<flavor>/...`). Never scan, reuse or edit destination `main`.
3. Lift and shift: copy as is, only the members the feature needs. If a variant file exists, add only the feature's members and never remove or change existing ones.
4. Alignment edits only (package, `R`, binding, `BR`, `BuildConfig` references, class names in XML and manifest, imports). Anything else is a forced change, reported in its own section.
5. Never assume. Gradle changes, forced renames, duplicate classes or bindings, database items and anything needing a `main` change go to the developer. Ask all questions in one batch.
6. Move unit tests with their classes. Never delete or disable a test.
7. Wiring into `main` is the developer's job. Provide an entry contract.
8. Details the developer gives in `initial.md` are starting points. "Must exclude" and "Out of scope" are hard limits.
9. Track every task in `cc-tasks/<task-name>/planned_tasks.md` with the status header from the rules. Only the developer sets `Approved` and `Done`.

## Agents

- `@plan`: discovers the feature, asks questions, writes the plan. Writes only `.md` files under `cc-tasks/<task-name>/`.
- `@migration`: executes the approved plan in the variant, builds, tests, writes the migration output.
- `@review`: independently verifies plan, code, build and tests. Reports and routes; does not fix code.

Prompts: `/new-task` creates `initial.md`; `/resume` tells you which agent to run next.
