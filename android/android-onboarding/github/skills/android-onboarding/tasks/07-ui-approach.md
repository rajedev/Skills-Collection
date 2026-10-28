# Task: UI approach
**Author:** Rajendhiran Easu

## Depends on
00-setup (module list)

## Steps
1. Determine whether the project uses Jetpack Compose, XML Views, or
   both. If both, identify which is the current target and which is
   being phased out (evidence: newer features in Compose, comments/
   TODOs mentioning migration, proportion of each across modules).
2. Distinguish static UI (fixed layouts/resources, compile-time
   content) from dynamic UI (server-driven content, remote config/
   feature-flag-driven UI, runtime-computed layouts).
3. Document theming: color system, typography, `MaterialTheme`
   customization or XML style/theme inheritance, dark mode support.
4. Document any shared design-system/component library module: what
   components it exposes, which modules consume it.
5. Note string/resource organization (per-module strings.xml vs
   centralized, string key naming conventions) and any accessibility
   handling observed (content descriptions, `semantics {}` blocks,
   TalkBack-specific code).

## Output
Append to draft under "## Section 9: UI Approach".

## Self-check
- The Compose-vs-XML split is quantified or at least illustrated with
  real examples from multiple modules, not a single file.
- Static/dynamic UI distinction is backed by actual code, not assumed.
- Claims about accessibility are based on code found, not its absence
  alone (absence should be reported as a gap in task 10, not asserted
  here as "no accessibility support" without checking).
