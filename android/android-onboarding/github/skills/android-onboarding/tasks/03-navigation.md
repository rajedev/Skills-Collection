# Task: Navigation
**Author:** Rajendhiran Easu

## Depends on
00-setup (module list)

## Steps
1. Identify the navigation system: Navigation Compose (`NavHost`,
   `NavController`), Fragment-based Navigation Component (nav graph
   XML), plain Activity intents, or a custom router/coordinator.
2. List every route/destination with its arguments (and types), any
   deep links defined, and bottom-nav or tab-nav structures.
3. Note back-stack behavior where it's non-default (e.g.
   `popUpTo`, `launchSingleTop`, custom back handling).
4. Build a Mermaid `flowchart` of screen-to-screen navigation, grouped
   by feature where the graph gets large.

## Output
Append to draft under "## Section 5: Navigation".

## Self-check
- Every destination listed is backed by an actual route/fragment/
  activity declaration, cited.
- Deep links listed match actual intent-filter or nav-graph entries.
- The Mermaid flowchart is syntactically valid and not missing any
  destination found in the search.
