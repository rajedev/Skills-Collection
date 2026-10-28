# Task: Features
**Author:** Rajendhiran Easu

## Depends on
00-setup (module list), 03-navigation (for cross-referencing screens)

## Scope
Process ONE feature module (or one clearly bounded feature package, if
the project isn't split into feature modules) per invocation. Check
progress.md for which features under this task are already done; pick
the next undone one. Treat this the same way as task 09's per-module
sub-scoping.

## Steps, per feature
1. Write the feature's purpose in one or two sentences and the user
   journey it supports, based on its screens and code — not assumed.
2. List its status: active, partial/in-progress, stub, or deprecated —
   with the evidence (e.g. unreferenced from navigation = likely dead;
   TODO-heavy = partial).
3. List the screens, ViewModels, UseCases, Repositories, and data
   sources it touches, with paths.
4. Draw a Mermaid `sequenceDiagram` from user action (e.g. button tap)
   through ViewModel, UseCase, Repository, to the data source and back
   to the UI state update.
5. Note feature flags, runtime permissions requested, and any
   platform-specific dependencies (e.g. camera, location, Bluetooth).

## Output
Append to draft under "## Section 6: Features" → one subsection per
feature, e.g. "### Feature: Checkout".

## Progress note
Track per-feature sub-status in progress.md under task 04, same
pattern as task 09:
```
- [ ] 04-features — login: done, profile: done, checkout: in-progress
```
Only mark `04-features` fully done when every discovered feature is
done.

## Self-check
- Every class referenced in the sequence diagram has a path citation
  in the feature's own subsection.
- Status (active/partial/stub/deprecated) is backed by a stated reason,
  not just asserted.
- The sequence diagram is syntactically valid Mermaid.
