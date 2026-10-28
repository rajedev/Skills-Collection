# Android Onboarding — Skill / Agent / Tasks package
**Author:** Rajendhiran Easu

This package drops into an Android project's repo root and gives
Copilot (in Android Studio or elsewhere) a resumable, read-only
workflow that explores the whole codebase and writes one consolidated
onboarding document: `explored_<date>_<time>.md`, in the project root.

## What's inside

```
.github/
  copilot-instructions.md              ← entry point; tells Copilot to follow agent.md
  skills/
    android-onboarding/
      SKILL.md                         ← reference knowledge: search patterns,
                                          templates, diagram conventions
      agent.md                         ← orchestrator: decides what task runs
                                          next, tracks state, finalizes the doc
      progress.md                      ← created automatically on first run;
                                          live state, safe to inspect anytime
      tasks/
        00-setup.md                    ← module discovery, run metadata
        01-project-shape.md            ← entry points, build variants, newcomer guide
        02-architecture.md             ← pattern, layer graph, module graph
        03-navigation.md               ← routes, deep links, nav flow
        04-features.md                 ← per-feature purpose/status/sequence diagram
        05-data-flow-api.md            ← endpoints, DTO mapping, caching, data flow
        06-dependency-injection.md     ← DI framework, modules, bindings graph
        07-ui-approach.md              ← Compose vs XML, static vs dynamic UI
        08-libraries.md                ← library catalog, in-house vs third-party
        09-key-classes.md              ← Fragments/State/Reducer/Listener/etc. deep-dive
        10-legacy-and-gaps.md          ← deprecated code, dead code, test gaps, TODOs
        11-assembly.md                 ← merges everything into the final .md
```

## Installation

1. Copy the `.github/` folder from this package into the root of your
   Android project repo (merge with your existing `.github/` folder if
   you already have one — don't overwrite unrelated files).
2. Commit it. It's meant to live in the repo so the whole team can use
   it and so the playbook improves over time like any other code.

## Running it (Copilot / Android Studio, agent mode)

Open Copilot Chat in agent mode and say:

> Continue the onboarding exploration.

- First time: this creates `progress.md`, discovers your modules (task 00), and runs the first task.
- Each message after that runs exactly one more task (or one more
  module/feature sub-pass for tasks 04 and 09), reports a short
  summary, and stops.
- Keep saying "continue the onboarding exploration" until the agent
  reports it has finalized the document.
- You can stop at any point and resume later, even in a new chat
  session — state lives in `progress.md`, not in chat history.

## What you get at the end

One file in your project root:

```
explored_2026-10-28_14-30.md
```

containing (as sections): run metadata, a top-of-document summary of
gaps and open questions, project overview/newcomer guide, project
shape, architecture (with Mermaid diagrams), navigation (Mermaid),
features (one subsection each, with sequence diagrams), data flow & API
integration (Mermaid), dependency injection (Mermaid), UI approach,
libraries, a full key-class catalog with deep-dive cards and cluster
diagrams for Fragments/ComposeViews/Adapters/State/Event/Reducer/
Middleware/Command/Listener/Manager/Helper/Checker/Context usage,
legacy & gaps, open questions, a glossary, and stated limitations.

## Guarantees

- **Read-only.** The workflow never edits, deletes, or generates
  application source code. It only ever writes `progress.md`,
  a scratch `draft_<timestamp>.md`, and the final
  `explored_<timestamp>.md` — all enforced explicitly in
  `copilot-instructions.md` and `agent.md`.
- **Never overwrites a previous result.** Each full run gets its own
  timestamped filename, so you can keep snapshots over time and diff
  them if you want to see how the codebase evolved.
- **Evidence-based.** Every task enforces path:line citations and a
  Confirmed/Inferred label on every claim; nothing is invented when a
  search comes up empty.

## Choosing a model

This workflow was designed assuming a Sonnet-class model at medium-or-
higher reasoning effort. Each task is intentionally scoped small (one
module, one feature, one section) so it also works on lighter/faster
models — if you're using a smaller model, consider further lowering
the "card" threshold in `tasks/09-key-classes.md` (currently top 8–12
per module) so each invocation stays well within its context budget.

## Customizing

- **Filename format:** change the `date +%F_%H-%M` pattern referenced
  in `agent.md` if you want a different timestamp format.
- **Section set:** add/remove sections by editing SKILL.md's section
  template list and adding/removing a matching task file; update the
  task-status checklist template in `agent.md` to match.
- **.gitignore:** if you don't want `explored_*.md` snapshots committed
  to version control, add `explored_*.md` and
  `.github/skills/android-onboarding/progress.md` to `.gitignore`.
  Otherwise, committing them lets the whole team share the same
  up-to-date onboarding doc.

## Prompt to select and run it

### First run (starts fresh):

Use the android-onboarding agent at .github/skills/android-onboarding/agent.md. Read progress.md if it exists, otherwise
create it. Run exactly one task — the next task that is not marked done — following its instructions and the reference
material in SKILL.md. Do not modify any source code. Report what you did and what’s next.

Every subsequent turn (shorter, once Copilot has the instructions file loaded):

### Continue the onboarding exploration.

That’s it for normal use — copilot-instructions.md is what makes the short version work, since it already tells Copilot
to follow agent.md whenever you say something like that. The longer prompt above is just a fallback in case your Copilot
setup doesn’t pick up .github/copilot-instructions.md automatically (some IDE integrations require the instructions file
to be explicitly referenced once per session).

### One more useful one for later:

To force a brand-new full run (ignoring a previous completed one):

Start a new onboarding exploration run. Archive the current progress.md and begin fresh, re-discovering the module list
from settings.gradle.

Good question to sort out before you drop this in — overwriting an existing `copilot-instructions.md` would silently
break whatever workflows are already there. Here's how to merge safely, for each case.

## How to Apply to the existing codebased already have the agentic setup Eg: GHCP (will update Claude, Cursor later)
## Case 1: `.github/copilot-instructions.md` already exists

**Don't replace it — append to it.** Open the existing file and add this block at the end (or wherever instructions get
appended in that file's convention):

```markdown
## Android onboarding exploration

When asked to explore, onboard, or generate a project overview of this
codebase, follow the agent defined at:

    .github/skills/android-onboarding/agent.md

Rules that always apply to this workflow:

- Read `.github/skills/android-onboarding/progress.md` first (create it
  from the template in `agent.md` if it does not exist).
- Run exactly ONE task per invocation, as listed in
  `.github/skills/android-onboarding/tasks/`.
- Never modify application source code. The only files this workflow
  may write are `progress.md`, `draft_<timestamp>.md`, and
  `explored_<timestamp>.md` in the project root.
- Use `.github/skills/android-onboarding/SKILL.md` as the reference.
```

Then delete the standalone `copilot-instructions.md` from the zip (don't copy it in) — only merge its *content* into the
existing file.

## Case 2: `.github/skills/` already has other skills in it

No conflict — just copy the `android-onboarding/` subfolder in alongside whatever's already there:

```
.github/skills/
  existing-skill-a/
  existing-skill-b/
  android-onboarding/     ← just added, no overwrite needed
    SKILL.md
    agent.md
    progress.md
    tasks/
```

Each skill folder is self-contained, so this is additive by nature — nothing to merge, just drop it in.

## Case 3: There's already an "agents" convention in the repo (e.g. `.github/agents/`)

This is the one that actually needs a decision, since it means the repo has an established pattern I didn't match. Two
options:

- **Conform to their pattern** — move `agent.md` to wherever their other agents live (e.g.
  `.github/agents/android-onboarding.md`), and update its internal references to `SKILL.md`/`tasks/` to point back into
  `.github/skills/android-onboarding/` with the adjusted relative (or absolute repo-root) path.
- **Keep it self-contained** — leave `agent.md` where it is, inside the skill's own folder, and just add one line to the
  existing agents index/registry (if there is one) pointing to it.

I don't know which your repo already does, so tell me and I'll restructure the exact paths and re-zip rather than you
doing find-and-replace by hand.

## Case 4: `progress.md` naming collision

If the existing setup already uses a file called `progress.md` for something else at the repo root, note that this one
lives scoped inside `.github/skills/android-onboarding/progress.md` specifically to avoid exactly that collision — so
this should be safe as-is unless the existing convention also nests a `progress.md` at that same nested path.

---

If you paste me (or describe) what your existing `copilot-instructions.md` and skills/agents folder currently look like,
I'll give you the exact merged version instead of general guidance — safer than you hand-splicing it.