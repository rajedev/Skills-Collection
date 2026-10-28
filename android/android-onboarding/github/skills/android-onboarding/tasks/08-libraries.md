# Task: Libraries
**Author:** Rajendhiran Easu

## Depends on
00-setup (module list)

## Steps
1. Read `gradle/libs.versions.toml` (or each module's declared
   dependencies if no version catalog is used).
2. For each library, note its purpose (one short phrase) and which
   modules use it.
3. Mark each as in-house/proprietary (internal company library) vs
   third-party, and separate AndroidX/Jetpack libraries from other
   third-party ones.
4. Flag:
   - Outdated versions (compare against what you know of current
     stable releases, but mark this comparison as Inferred since it
     depends on when the catalog was last checked).
   - Duplicates — e.g. two HTTP clients, two image-loading libraries,
     two DI frameworks coexisting.
   - Apparently unused dependencies (declared but no import found
     anywhere in the module that declares them).

## Output
Append to draft under "## Section 10: Libraries".

## Self-check
- Every library's "used by" list is based on an actual import search,
  not assumed from the dependency declaration alone.
- Outdated-version flags are labeled Inferred.
- Duplicate-library findings cite both libraries and at least one
  usage site for each.
