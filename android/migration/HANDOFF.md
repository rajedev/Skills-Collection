# Handoff: Android feature migration agents (resume notes)

Paste or attach this at the start of a new session, together with the files of this package added to the project. The assistant has no memory of earlier sessions.

## Goal
Three GitHub Copilot agents (Plan, Migration, Review) for Android Studio that move ONE feature at a time from a standalone SOURCE project into a VARIANT (flavor source set) of a DESTINATION app, using lift and shift. Destination `main` belongs to another product and is never touched.

## Decisions already agreed (do not re-open unless I say so)
- Source project is read-only and lives OUTSIDE the destination repo (path given in `initial.md`; pinned checkout, chmod read-only). Fallback if Copilot cannot read outside the project: a snapshot folder or content root inside the workspace.
- Only writable code area: destination variant (`app/src/<flavor>/java`, `res`, `AndroidManifest.xml`, `src/test<Flavor>`). `main` is never scanned, reused or edited. The variant is self-contained (base classes are copied from the SOURCE).
- Variant is a flavor source set inside the `app` module. R, binding, BR, BuildConfig namespace comes from `namespace` in destination `app/build.gradle`; the source namespace may differ (several R packages possible).
- Lift and shift: copy as is, only members the feature needs. Whole files copied with `cp`; merged members copied exactly (text, no tidying). If a variant file exists, add only feature members, never edit existing ones. A large base class (BaseActivity, BaseFragment) brings only the methods the feature uses plus what they depend on.
- Package names, sub-package folders and file names stay AS IS. Nested classes stay inside their parent.
- Layouts: every Activity/Fragment/adapter/ViewHolder/custom view brings its layout (R.layout, setContentView, inflate, binding class name mapped to the layout file) plus qualified variants and includes. Binding classes are never copied (generated).
- Only alignment edits: R, binding, BR, BuildConfig references, class names in XML/manifest, imports. Anything else = "forced change", reported separately.
- Outside references are classified: copy / host-provided (session or user data, DI component creation, SDK init) / implicit (base-class setup). Host-provided and implicit ones are asked, not assumed. Developer can pre-declare them in `initial.md` ("Known host-provided things").
- Dagger is NOT special: same lift and shift; missing pieces copied from source; duplicates or graph issues go to the developer.
- Unit tests move with their classes; never delete or disable tests. Build + tests run before `Migrated`.
- Gradle edits always need approval (flavor-scoped, source versions, never bump), unless pre-approved in `initial.md` for listed items.
- Wiring the feature into `main` is the developer's job; agents output an entry contract.
- `initial.md` hints (entry point, launch path, modules, must include) are starting points, not limits. Only "Must exclude" and "Out of scope" are hard limits. Things found beyond hints are listed in `findings.md`.
- Three separate agents plus one shared `rules.md` (single source of truth). Status lifecycle: Draft, Plan Ready, Approved, In Migration, Blocked, Migrated, In Review, Needs Fix, Plan Revision, Done. Loop A = code gap (no re-approval), Loop B = plan gap (scoped questions, re-approval). Only the developer sets Approved and Done.
- Review agent diffs copied files against the source (needs a terminal) and rebuilds and retests independently.
- Namespace leftover check looks for generated-class references qualified with the source namespace (R, databinding, BR, BuildConfig, library R), NOT the plain namespace string, because copied package names may start with the namespace.
- Plan changes: prompt `@plan`, or edit files then ask `@plan` to re-read and sync; logged under "Plan edits".
- Some earlier draft instruction files generated elsewhere were reviewed and deliberately ignored; this design supersedes them.

## What exists
```
migration-agents/
  README.md                       the project reference: rules (mirror of rules.md), agent outlines, templates, setup, worked example
  HANDOFF.md                      this file
  .github/copilot-instructions.md always-on rules + <FILL_IN> defaults
  .github/agents/                 plan.agent.md  migration.agent.md  review.agent.md
  .github/prompts/                new-task.prompt.md (/new-task)  resume.prompt.md (/resume)
  cc-tasks/_shared/               rules.md  templates.md
```
Placement: copy `.github/` and `cc-tasks/` to the destination repo root. Per task: `cc-tasks/<task-name>/initial.md` (via /new-task or by hand); the agents create the rest. All examples use neutral names (xyz, alpha, Help Center).

## Sample projects
`test-projects.zip` (a separate download, not part of this package) has `source-app`, `destination-app` and `TEST-GUIDE.md` (23 planted test cases). They were written without being able to run Gradle, so they are not yet compiled; build both first. No pilot of the agents has been run yet.

## State
Everything agreed above is applied in these files. Nothing pending.

## Not verified (could not test in my environment)
1. Does the Android Studio Copilot plugin support custom `.agent.md` and `.prompt.md` files in `.github/`? Frontmatter uses only `name` and `description`; a `tools` field may be needed.
2. Can Copilot read the source path outside the open project, and run terminal commands (Gradle, `cp`, `diff`)? (README setup step 7)
3. No pilot run yet. Plan: run one small feature with Dagger, a data binding layout, multiple locales and a unit test, then tune `rules.md`.

## Still to fill in by me
Three `<FILL_IN>` defaults in `.github/copilot-instructions.md` (default source path, destination path, variant).

## Starter prompt for the new session
"I'm continuing the Android feature migration agents work. Read HANDOFF.md and the files in this package first. Do not change the agreed decisions. Today I want to discuss: <topic>."
