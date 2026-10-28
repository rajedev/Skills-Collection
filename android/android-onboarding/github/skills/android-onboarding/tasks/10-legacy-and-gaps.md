# Task: Legacy, unused code, gaps, and open questions
**Author:** Rajendhiran Easu

## Depends on
All prior tasks (00–09) — this task consolidates findings already
surfaced plus a few dedicated searches of its own.

## Steps
1. Search for deprecated APIs and patterns: `AsyncTask`, RxJava usage
   where Flow is used elsewhere in the project, `findViewById` in
   Compose-primary modules, `startActivityForResult`/`onActivityResult`
   (vs Activity Result API), old support-library (`android.support.*`)
   imports, LiveData used inconsistently alongside StateFlow.
2. Search for dead code: classes/functions with no references found
   anywhere, unused resources (`./gradlew lintDebug` output if
   available, or manual search), commented-out code blocks of
   meaningful size, unused feature flags.
3. For each migration-in-progress pattern found (e.g. XML → Compose,
   RxJava → Coroutines, LiveData → StateFlow), note the apparent target
   and, if estimable from the proportion of modules/files already
   migrated, a rough completion estimate — label this Inferred.
4. Consolidate gaps: test coverage (count test files per module vs
   source files), hardcoded keys/URLs/config found during earlier
   tasks, TODO/FIXME/HACK counts with the 5 most significant examples
   (by apparent impact, not just count), critical paths (e.g. payment,
   auth) with no visible error handling, and missing documentation.
   Assign each a priority: high, medium, or low, with a one-line reason.
5. Pull together every "open question" noted in progress.md across all
   prior tasks into one consolidated list, removing duplicates.
6. Write the "Limitations" section: what could not be determined and
   why (obfuscated/minified code, generated code not present in the
   repo, config only resolved at runtime, build variants not buildable
   in this environment).

## Output
Append to draft under:
- "## Section 12: Legacy, Unused Code, and Migration"
- "## Section 13: Gaps and Risks"
- "## Section 14: Open Questions"
- "## Section 16: Limitations"
(Section 15, Glossary, was already written in task 01 — leave as is.)

## Self-check
- Every gap has a location, a reason it matters, and a priority.
- Migration-completion estimates are explicitly labeled Inferred.
- The open-questions list has no duplicates and reflects what was
  actually logged in progress.md, not new unlogged questions invented
  now.
- The limitations section is honest about scope that wasn't covered,
  not just a formality.
