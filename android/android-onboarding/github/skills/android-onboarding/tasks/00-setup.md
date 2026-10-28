# Task: Setup
**Author:** Rajendhiran Easu

## Purpose
Establish run metadata and the module list that every later task
depends on.

## Steps
1. Get the current timestamp and confirm it matches what's already in
   progress.md's `output_file` (it should, since agent.md sets this on
   first creation of progress.md).
2. If this is a Git repo, record the short commit hash and branch name
   in progress.md (if not already recorded).
3. Read `settings.gradle` or `settings.gradle.kts` and list every
   included module, with its path.
4. For each module, open its `build.gradle(.kts)` and classify it:
   app module, feature module, core/shared module, data module, domain
   module, ui/design-system module, or test-only module. Use the
   `plugins {}` block and module name as evidence.
5. Note the Gradle plugin versions, Kotlin version, Compose version (if
   used), minSdk, and targetSdk from the root/module build files or
   `libs.versions.toml`.
6. Record total source file count (`find . -name "*.kt" -o -name "*.java" | wc -l`
   or equivalent) for later coverage reporting.

## Output
Append to draft under "## Section 0: Run Metadata" and
"## Section 2: Project Shape" (the module table and version info
specifically; entry points and build variants go in task 01).

## Progress note
Write the full module list into progress.md's "## Modules discovered"
section — this is the canonical list every later task should use.
Mark `00-setup` done.

## Self-check
- Every module from settings.gradle(.kts) appears in the list.
- Every module has a classification with a one-line reason.
- progress.md's module list matches exactly what's in the draft.
