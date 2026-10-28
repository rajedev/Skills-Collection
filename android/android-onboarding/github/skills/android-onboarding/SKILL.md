---
name: android-onboarding
description: Reference knowledge base for exploring an Android codebase — search patterns, role taxonomy for key classes, card templates, section templates, and diagram conventions. This file is read by agent.md and by the task files under tasks/; it is not meant to be run on its own.
author: Rajendhiran Easu
---

# Android Onboarding — Reference

This file holds the knowledge the exploration tasks rely on. Task files
tell the agent *what to do and in what order*; this file tells it *how
to do each kind of analysis correctly*.

## Core principles (apply to every task)

- **Evidence first.** Every claim cites `path:line`, or at minimum `path`.
- **Label confidence.** Mark each claim **Confirmed** (seen directly in
  code) or **Inferred** (deduced; state the reasoning in one clause).
- **No invention.** If a search returns nothing, write "Not found" —
  never infer that something probably exists elsewhere. If unsure
  whether two classes are related, list them separately rather than
  asserting a relationship, and note the uncertainty as an open question.
- **Write for a newcomer** who knows Android generally but has never
  seen this specific project.
- **Redact secrets.** Replace API keys, tokens, and credentials with
  `<REDACTED>` — never copy them into the output file.
- **Read-only.** Nothing in this workflow edits, deletes, or generates
  application source code. The only writes are progress.md,
  draft_<timestamp>.md, and the final explored_<timestamp>.md.

## Role taxonomy for key classes

Used by task 09. Search the codebase for these patterns, then confirm
the role by actually reading the class — a class named `UserHelper`
that holds a Retrofit call is a service, not a helper; record its real
role, not just its name.

| Role | Search hints | What to capture |
|---|---|---|
| Fragment / DialogFragment | `: Fragment(`, `: DialogFragment(`, `: BottomSheetDialogFragment(` | Host Activity, arguments, binding, nav actions, lifecycle quirks |
| Activity | `: ComponentActivity(`, `: AppCompatActivity(` | Intent extras, launch mode, what it hosts |
| ComposeView / composables | `ComposeView`, `@Composable`, `AndroidView(` | Entry points, state hoisting, interop with XML |
| View / custom View | `: View(`, `: ViewGroup(`, `onDraw`, `onMeasure` | Custom attributes, drawing cost, usage sites |
| Adapter / ViewHolder | `ListAdapter`, `RecyclerView.Adapter`, `PagingDataAdapter`, `ViewHolder(` | Item types, DiffUtil, click callbacks |
| Component (DI) | `@Component`, `@Subcomponent`, `Component` | Scope, exposed types, parent component |
| State | `UiState`, `State`, `sealed class .*State`, `ScreenState` | Fields, defaults, producer |
| Event / Action / Intent | `Event`, `Action`, `Intent`, `sealed class .*Event` | Every variant, sender, handler |
| Effect | `Effect`, `SideEffect`, one-shot `Channel`/`SharedFlow` | Navigation/toast signals, one-shot behavior |
| Reducer | `reduce(`, `Reducer`, functions returning new State | Transition table: state + event → new state |
| Middleware / Interceptor | `Middleware`, `Interceptor`, `Chain`, `Pipeline` | Execution order, short-circuit points |
| Command | `Command`, `execute(`, `: Command` | What it does, issuer, undo support |
| Listener / Callback | `Listener`, `Callback`, `fun interface`, `on[A-Z]\w*:` params | Registrar, unregistration (leak risk) |
| Observer | `Observer`, `addObserver`, `collect`, `observe(` | Lifecycle owner, cleanup |
| Manager | `Manager` | Owned resource, lifecycle, singleton vs scoped |
| Helper / Util | `Helper`, `Util`, `Utils`, `Extensions.kt` | Real responsibility (often a hidden service), callers |
| Checker / Validator | `Checker`, `Validator`, `Verifier`, `Policy`, `Rule` | Rules enforced, failure output |
| Context holder | `Context` params, `applicationContext`, `ContextWrapper` | Activity vs Application context, leak risk |
| Repository / DataSource | `Repository`, `DataSource`, `Dao`, `Api`, `Service` | Link to task 05 (data flow / API) |
| Mapper / Converter | `Mapper`, `Converter`, `toDomain`, `toDto` | Field mapping, lossy conversions |
| Factory / Provider / Builder | `Factory`, `Provider`, `Builder` | What it builds, lifetime |
| Worker / Service / Receiver | `Worker`, `CoroutineWorker`, `Service`, `BroadcastReceiver`, `JobService` | Triggers, constraints, retries |

### Depth assignment
- **Card:** top 8–12 classes by fan-in per module, plus any class on a
  feature's critical path (cross-reference task 04's output).
- **Brief:** one or two sentences in the catalog table. Simple data
  holders and small helpers.
- **None:** generated code. List unused classes under task 10
  (legacy and gaps) instead.

State explicitly which classes were not carded and why.

## Deep-dive card template

```
#### ClassName
- Role: (from the taxonomy above) and why classified that way
- Location: path:line
- Responsibility: one or two sentences
- Collaborators: dependencies in, dependents out
- Inputs / outputs: for events, commands, reducers, middleware — types consumed/produced
- Lifecycle and scope: where created, how long it lives, DI scope
- Threading: dispatcher or thread, any main-thread work
- Side effects: I/O, navigation, analytics, persistence
- Gotchas: leak risks, ordering dependencies, nullable traps, deprecated usage
- Confidence: Confirmed or Inferred, with reason
```

## Cluster diagrams (task 09)

Group related key classes and draw:
- **State + Event + Reducer + Effect** → `stateDiagram-v2`, plus a table
  of each event and its handler.
- **Fragment / Activity / Adapter / ViewHolder / ComposeView** →
  `classDiagram` showing hosting and ownership.
- **Middleware / Interceptor / Command chains** → `flowchart LR` showing
  execution order and short-circuit points.
- **Listener / Callback** → `sequenceDiagram` showing registration,
  trigger, and unregistration.
- **Manager / Helper / Checker / Context holders** → `classDiagram`
  showing ownership and callers.

Always validate Mermaid syntax before writing a diagram block.

## Key-class cross-checks (task 09)

- Every State variant is produced by some ViewModel or reducer. Report
  unreachable variants.
- Every Event has a handler. Report unhandled events.
- Every Listener has a matching unregister, or is lifecycle-aware.
  Report leak risks.
- Every Context held by a long-lived object (singleton, Manager,
  companion object) is an Application context. Report violations.
- Helpers/Utils that hold state, make network calls, or have more than
  three collaborators are flagged as hidden services.

## Section templates (used across tasks, assembled by task 11)

1. **Run metadata** — timestamp, commit hash, branch, module count,
   file count, explorer model, scope.
2. **Project overview & newcomer guide** — what the app does, build/run
   steps, reading order, glossary.
3. **Project shape** — modules, build types/flavors, config sources,
   entry points, SDK/Kotlin/Compose versions.
4. **Architecture** — pattern, layer diagram, module graph (Mermaid),
   state/event flow, threading model.
5. **Navigation** — system in use, routes/arguments/deep links,
   Mermaid flowchart.
6. **Features** — one subsection per feature: purpose, status, screens/
   VM/UseCase/Repo, sequence diagram, flags/permissions.
7. **Data flow & API integration** — networking stack, endpoint table,
   DTO→domain→UI mapping, caching/offline/pagination/auth, Mermaid
   flowchart.
8. **Dependency injection** — framework, modules, bindings, scopes,
   Mermaid graph, manual service locators found.
9. **UI approach** — Compose vs XML (and migration status), static vs
   dynamic UI, theming/design system.
10. **Libraries** — from the version catalog; purpose, consuming
    modules, in-house vs third-party, outdated/duplicate/unused flags.
11. **Key classes** — catalog, cards, cluster diagrams, cross-checks
    (see above).
12. **Legacy, unused code, and migration** — deprecated patterns, dead
    code, in-progress migrations with target and completion estimate.
13. **Gaps and risks** — test coverage gaps, hardcoded config, TODO/
    FIXME/HACK counts with top examples, missing error handling,
    missing docs. Each finding: location, why it matters, priority.
14. **Open questions** — what only the team can answer.
15. **Glossary of symbols and conventions** — naming suffixes, package
    layout, custom annotations.
16. **Limitations** — what could not be determined and why.

## Diagram conventions, summary

| Content | Mermaid type |
|---|---|
| Module/layer graph | `flowchart LR` |
| Navigation graph | `flowchart` |
| State/Event/Reducer | `stateDiagram-v2` |
| Fragment/Adapter/ViewHolder relationships | `classDiagram` |
| Middleware/Command chains | `flowchart LR` |
| Listener lifecycle | `sequenceDiagram` |
| DI bindings graph | `flowchart LR` or `classDiagram` |
| Feature sequence (user action → data) | `sequenceDiagram` |

Always validate syntax (matching brackets, correct arrow types, no
reserved words as node IDs) before writing any diagram block into the
draft.
