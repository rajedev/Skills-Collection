# Shared rules for the Plan, Migration and Review agents

This file is the single source of truth. Every agent reads it fully before doing anything.
If an instruction in chat conflicts with a rule here, stop and ask the developer.

Placeholders in angle brackets (`<flavor>`, `<source.ns>`, `<dest.ns>`, `<variant.pkg>`) are always resolved from the Gradle files and the project, never typed from memory.

---

## 1. Access boundaries

| Area | Access |
|---|---|
| Source project (all of it) | Read-only. The only place code and resources are taken from |
| Destination variant source sets (`src/<flavor>/java`, `src/<flavor>/res`, `src/<flavor>/AndroidManifest.xml`, variant test source sets) | Read and write |
| Destination `main` | Never scanned, reused or edited |
| `app/build.gradle` and other Gradle files | Only with developer approval, flavor-scoped where possible |
| `cc-tasks/<task-name>/` | Written by the agent whose role owns that output (see the agent files) |

Boundary and scope checks (what changed outside the variant) ignore `.github/` and `cc-tasks/`, which are tooling.

The variant is self-contained. If a migrated file needs something (including `BaseActivity`, `BaseFragment`, base custom views, base WebView classes), copy the needed members from the **source** into the variant. Never check whether `main` has an equivalent.

Wiring the feature into `main` (Fragment attach, Intent, Activity launch, `BuildConfig` condition) is the developer's job. The agents produce an entry contract (section 12).

## 2. Preflight (before any work)

Each agent runs these checks and reports the result in its first message. If a check fails, stop and tell the developer what to fix.

1. **Source path readable:** open one file from the source path in `initial.md` (for example its `app/build.gradle`). If the file cannot be read, stop. Do not guess its contents.
2. **Source reference:** if `initial.md` gives a branch, tag or commit, verify the source checkout is at that reference (`git -C <source> rev-parse HEAD`) when a terminal is available. If there is no terminal, ask the developer to confirm.
3. **Terminal available?** Note whether the agent can run commands. If not: Gradle builds, `cp` of binary resources and `git diff` become developer steps (see sections 5 and 10).
4. **Destination baseline:** the destination is on the branch named in `initial.md` with a baseline commit. Verify with git if possible, otherwise ask the developer to confirm. Review diffs against this baseline.
5. **Destination namespace and flavor:** read `namespace` from the destination `app/build.gradle` and confirm `<flavor>` exists in `productFlavors`.

## 3. Reading `initial.md`: hints and limits

- A value the developer fills in (entry point, launch path, source modules, must include) is a **starting point**, not a limit. Begin there, but still discover the whole feature with the traversal in section 4.
- **Hard limits** are only "Must exclude" and "Out of scope". Never bring those, even if the traversal reaches them. Log what was skipped and why.
- Settings are followed as given: Locales, Build commands, Source reference, Gradle pre-approval (covers only what is listed).
- A blank field is resolved from the defaults in `.github/copilot-instructions.md` or by discovery. Write every resolved or discovered value into `plan-output/findings.md`.
- Anything found beyond the developer's hints goes in a "Discovered beyond hints" section of `findings.md` (what, where, why the feature needs it) for review at plan approval.

## 4. Discovery procedure

The Plan agent builds the file list with this traversal, not by guessing.

1. Start at the entry point (Activity, Fragment, composable, route). If none is given, find it from the feature description.
2. Walk outward, adding each reference to a **visited list**:
   - ViewModels, repositories, use cases, managers
   - data sources, API interfaces, models, enums, constants
   - adapters, view holders, listeners, callbacks, items, content objects
   - custom views and the base classes the feature extends
   - Dagger pieces the feature uses (section 8)
   - analytics events and trackers
   - layouts and everything they reference (drawables, colors, dimens, strings, styles, fonts, anim)
   - manifest declarations and navigation entries
   - unit tests for every class on the list
3. From shared or multi-feature files, take only the members the feature needs (methods, constants, properties, nested classes, generics).
4. Stop when no unvisited references remain. Skip anything in "Must exclude" or "Out of scope".
5. Save the visited list in `plan-output/findings.md`.

## 5. What a feature can include (checklist for plan and review)

- **Code:** Activities, Fragments, custom views, WebView and base classes, ViewModels, repositories, managers, models, nested classes, generics, adapters, view holders, listeners, analytics, utils, constants, Kotlin and Java
- **UI:** Compose and XML layouts, menus, navigation graphs, data binding variables and imports
- **Resources:** colors, dimens, strings (default and every `values-xx` in scope), styles, themes, fonts, drawables (all densities, night variants), images, anim, raw, xml
- **Manifest:** activities, services, receivers, providers, permissions, intent filters, meta-data
- **Dagger:** components, subcomponents, modules, provides, binds, scopes, qualifiers, inject targets
- **Tests:** unit tests, test resources, fakes, test utilities
- **Gradle (approval-gated):** dependencies, plugins, `buildFeatures`, `BuildConfig` fields, Proguard/R8 rules
- **Other:** feature flags and remote config keys, network layer pieces. Database entities, DAOs and migrations are always asked about first (section 11)

Binary resources (PNG, WebP, fonts, audio) cannot be reliably written as text. Copy them with a terminal `cp`. If no terminal is available, list them in `migration-output/gaps.md` for the developer to copy, and the Review agent verifies they arrived.

## 6. Merge rules (file already exists in the variant)

| Situation | Action |
|---|---|
| File does not exist | Create it (and its package) with only the needed members |
| File exists | Add only the feature's methods, constants, properties, params. Never edit or remove existing members |
| Existing override in a variant base class | Keep it. Add only what the feature needs |
| Feature needs behavior that conflicts with an existing variant member | Stop and ask |
| Same resource name already in the variant | Existing file: add only missing entries. Same name with a different value: stop and ask |
| Same layout name with different content | Stop and ask (view IDs and binding fields would change) |

Copy code as is. The only edits to copied code are alignment edits (section 7). Anything else is a forced change (section 13).

## 7. Namespace and import alignment

Destination `R`, binding, `BR` and `BuildConfig` come from the `namespace` in the destination `app/build.gradle`, not from the variant folder or package. The source may use several `R` packages (app and library modules).

1. Read the source namespace(s) and the destination `namespace`. Never guess.
2. Write a mapping table in `plan-output/findings.md`; the developer confirms it at plan approval.

| Item | Source package | Destination package |
|---|---|---|
| `R` | `<source.ns>.R` (and library `R`s) | `<dest.ns>.R` |
| Binding classes | `<source.ns>.databinding.*` | `<dest.ns>.databinding.*` |
| `BR` | `<source.ns>.BR` | `<dest.ns>.BR` |
| `BuildConfig` | `<source.ns>.BuildConfig` | `<dest.ns>.BuildConfig` |
| Feature classes | `<source.pkg>` | `<variant.pkg>` (keep names and structure unless forced) |

3. Apply the mapping to Kotlin, Java, tests, layout XML, manifest and nav graphs:
   - imports, including `import static`; replace wildcards with explicit imports
   - `<import type>` and `<variable type>` in data binding layouts
   - custom view tags and `android:name` for fragments and activities
   - manifest class names
   - inline fully qualified references (`com.source.R.id.x`) and reflection strings: flag them, do not silently edit
4. A file whose package equals the destination `namespace` needs no `R` import. Compare each file's new package before adding or removing imports.
5. With non-transitive R classes, library resources need the owning module's `R`. Resources that exist only in a source library module are copied into the variant `res`.
6. Optimize imports and remove wrong ones at the end.
7. After alignment, search the migrated files for the **source namespace as a literal string** in `.kt`, `.java`, `.xml` and the manifest. None may remain.

## 8. Dagger

Dagger is not a special case. It follows the same lift-and-shift rules.

- Find the feature's Dagger pieces in the source (components, subcomponents, modules, provides, binds, scopes, qualifiers, inject classes, factories, constants used by them) and copy them as is.
- Take only the members the feature needs. If the file exists in the variant, add only those members and leave everything else alone.
- Align package, `R`, binding and `BuildConfig` imports only.
- Build. If Dagger reports a missing binding, copy that provider from the source and log it.
- Read only the **first** Dagger error; later ones are usually cascades.
- If something cannot be fixed by copying or aligning (for example a duplicate binding), log it in `gaps.md` and ask.
- Missing `dagger`, `kapt` or `ksp` setup is a Gradle change and needs approval.

## 9. Gradle policy

- Compare source and destination `app/build.gradle`: dependencies, plugins, `buildFeatures` (view binding, data binding, compose), `BuildConfig` fields, `kapt`/`ksp`, Proguard.
- Always ask before editing. Show exactly what will be added and why. Pre-approval in `initial.md` covers only what is listed there.
- Prefer flavor-scoped configurations (`<flavor>Implementation`, `test<Flavor>Implementation`).
- Use the version the source uses. If the destination already has the library at a different version, do not bump it. Stop and ask.

## 10. Build and test gates

| Who | When | What |
|---|---|---|
| Migration agent | Before setting `Migrated` | `assemble<Flavor>Debug` and `test<Flavor>DebugUnitTest` (or the commands given in `initial.md`). Fix mechanical errors and log them. Everything else goes to `gaps.md` |
| Review agent | Independently | Rebuild, rerun tests, run lint if available |

- **Mechanical (agent fixes and logs):** wrong namespace, missing import, unresolved `R` or binding, a missing class or resource that exists in the source.
- **Everything else goes to the developer:** duplicate class or binding, behavior differences, test failures caused by real differences, anything needing a `main` change.
- Never delete, skip or disable a test to get a green build.
- If no terminal is available, write the exact commands into the checklist. The developer runs them and pastes the output back. Work continues from there.

## 11. Must-ask list

1. Any Gradle or dependency change (unless pre-approved in `initial.md`)
2. Any forced rename (class, file, resource) beyond alignment
3. Duplicate class or duplicate binding errors
4. Same layout or resource name in the variant with different content
5. Database entities, DAOs, migrations
6. Anything that would require changing `main`
7. Version conflicts
8. Anything the traversal reaches that looks like a different feature rather than part of this one (ask before including it)
9. Any assumption about behavior that cannot be confirmed from code

Ask all clarifying questions in one batch, not one at a time. Never assume.

## 12. Entry contract

Written by the Migration agent in `migration-output/summary.md`, verified by the Review agent:

- Entry class, fully qualified name in the variant
- How it is launched: Intent action or component, required extras, Fragment arguments, `newInstance` signature (list every launch path found, the one in `initial.md` first)
- Manifest entries added in the variant
- What the feature expects from the host: container view ID, theme, init calls, callbacks to implement, Application-level requirements
- Note for the developer: a string-based Intent contract is the safest way to wire from `main` when other flavors do not contain the class. A direct class reference needs matching stubs in those flavors.

## 13. Forced changes

Every output file set ends with a **Forced changes** section for anything beyond plain alignment: renames, signature changes, added parameters, stubbed members. Each entry lists file, change and reason. If there are none, write "None".

## 14. Status lifecycle and iterations

Statuses: `Draft`, `Plan Ready`, `Approved`, `In Migration`, `Blocked`, `Migrated`, `In Review`, `Needs Fix`, `Plan Revision`, `Done`.

| Transition | Set by |
|---|---|
| Draft to Plan Ready | Plan agent |
| Plan Ready to Approved | Developer only |
| Approved to In Migration, In Migration to Migrated | Migration agent |
| Migrated to In Review, In Review to Needs Fix | Review agent |
| Needs Fix to Plan Revision, Plan Revision to Plan Ready | Plan agent |
| Any to Blocked | Any agent (record `Blocked from: <status>`; resume there when answered) |
| In Review to Done | Review agent proposes, developer sets |

Header at the top of `planned_tasks.md`, updated by every agent:

```
Status: <status>
Iteration: <n>
Blocked from: <status or ->
Last updated by: <agent>
Open issues: <n> (<x> code gap, <y> plan gap)
```

Rules:
1. Never rewrite history. Add `Iteration N` sections to `planned_tasks.md`. Completed items stay checked unless review proves them wrong, then uncheck with a note.
2. **Loop A (code gap):** the plan was right, execution missed it. No new approval. Migration redoes only the failed items.
3. **Loop B (plan gap):** the plan missed something. Plan agent revises, developer re-approves, Migration does only new or changed items. `prompt.md` lists only new or changed rows for that iteration.
4. On a revision, the Plan agent asks questions only about the new gap. It re-asks an earlier decision only if the gap touches it. It does a full re-plan if the traversal boundary was wrong or the same plan gap appears twice.
5. Same issue still open after two iterations: set `Blocked` and ask the developer.
6. The review agent tags every issue as code gap or plan gap with the iteration number. The developer can override the tag.
7. **Changing the plan:** if the developer edits plan files by hand, re-read all plan files, sync `planned_tasks.md`, `prompt.md`, `findings.md` and `summary.md`, and list inconsistencies. Log every plan change under "Plan edits" in `findings.md` with the reason. A plan edit after `Approved` and before migration starts sets the status back to `Plan Ready`. If migration is running, stop (`Blocked`) and handle it as a plan revision.

## 15. Output file formats

Templates for every output file are in `cc-tasks/_shared/templates.md`. Use them as is.
