# Task: Project shape & newcomer guide
**Author:** Rajendhiran Easu

## Depends on
00-setup (module list)

## Steps
1. Identify entry points: the `Application` class (if any), the
   launcher `Activity` (intent filter with `MAIN`/`LAUNCHER`), and any
   deep link intent filters across `AndroidManifest.xml` files.
2. List exported components (Activities, Services, Receivers,
   ContentProviders marked `android:exported="true"`) with a note on
   why each is exported.
3. Identify build types and product flavors from the app module's
   `build.gradle(.kts)`. Note where variant-specific config comes from
   (`BuildConfig` fields, `local.properties`, environment variables,
   `gradle.properties`).
4. Work out build/run steps: required JDK version, how secrets/config
   are supplied locally (without reproducing actual secret values —
   describe the mechanism only), the command to run unit tests and
   instrumented tests.
5. Write a plain-language description of what the app does, based only
   on what the code shows (screens, strings, permissions requested) —
   not on assumptions about the product.
6. Draft a glossary of project-specific terms: recurring acronyms,
   domain vocabulary, and custom class-name suffixes you notice while
   reading (e.g. "Vm", "UseCase", "Repo", "Dto").

## Output
Append to draft under:
- "## Section 1: Project Overview & Newcomer Guide"
- "## Section 2: Project Shape" (entry points, build variants —
  module table already written by task 00)

Include a short "recommended reading order" list pointing to the later
sections in a sensible sequence for a newcomer (e.g. overview →
architecture → navigation → one feature end-to-end → data flow → key
classes).

## Self-check
- Entry points are backed by actual manifest/code citations.
- The app description makes no claims not grounded in code or
  resources (no marketing language, no guessed business purpose).
- The glossary only includes terms actually observed, not invented.
