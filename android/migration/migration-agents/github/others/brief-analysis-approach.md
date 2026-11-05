# Analysis brief: <FILL>

## Purpose and limits
Analyse the migration options and write one reviewable file.
This is an ANALYSIS task. Do not edit any code, resource, Gradle or manifest file.
Write only to: cc-tasks/<FILL>/analysis/approach-analysis.md

## Context
- Source project (read-only): <source path>, tag <tag>
- Destination project (contains main): <destination path>
- Destination main (the other product's code, read-only except for the feature-related edits listed below): <destination path>/app/src/main
- Destination variant (the only writable code area for the features): <destination path>/app/src/<flavor>
- Destination variant tests: <destination path>/app/src/test<Flavor>
- Destination Gradle: <destination path>/app/build.gradle
- Features: <FILL>Fragment (with its own feature code) and <OTHER_SUB_FEATURE_OR_FUNCTIONALITY>
- Both apps have BaseActivity, BaseFragment, a WebView base, inflater-based activities, and Dagger.
- Destination main may be edited ONLY for changes related to these two features: call sites, DI wiring, manifest entries, and members of base classes these features need. main must still compile in every flavor.

## Injected types the features use (fill in)
| Type | Kind (repository / session / analytics / config / other) | Used by |
|---|---|---|
| | | |

## Part 1: Base classes and diagrams
For each of BaseActivity, BaseFragment, the WebView base, and the inflater base:
1. Locate the source and main versions (fully qualified name, package, file path).
2. Compare: superclass chain; lifecycle overrides (onCreate, onResume, onDestroy, onBackPressed) and what each does; theme, toolbar, window and status-bar setup; inflation or ViewBinding setup; WebView client, WebChromeClient, settings, JavaScript interfaces, cookies, cache; deeplink or intent handling; feature flag or BuildConfig reads; analytics or screen-tracking calls.
3. Mark each difference as needed by the feature, not needed, or unknown.
4. Recommend one approach and list the others:
   - A. Reuse main's class as is (no edit, no copy)
   - B. Add only the needed members to main's class (affects every feature using it)
   - C. Variant subclass of main's class, adding only the needed behavior
   - D. Copy the source class into the variant under the same package (duplicate class risk)
   - E. Copy the source class under a different package (forced rename)
5. For each approach: files that change, exact file movements, benefits, risks, regression exposure in main, and whether flavors without the feature still compile.
6. Read main's versions of BaseActivity, BaseFragment, the WebView base and the inflater base from <destination path>/app/src/main, and compare them with the source versions.

Diagrams (Mermaid, about 12 nodes maximum each), in approach-analysis.md:
- D1: current source app, screen flow and DI graph (which component provides what, which Fragment or Activity consumes it, where session and user data come from)
- D2: current destination main, for the classes the features touch (BaseActivity, BaseFragment, AppComponent and its modules, how main launches features)
- D3: one flowchart per option (A to E, and DI options a to c in Part 3) showing feature classes, DI providers, host-provided values, and arrows for launch, data and DI
- D4: data flow for the recommended option, from host (session, user id, flags) to feature, with the delivery mechanism on each arrow
- D5: flavor view for the recommended option: what compiles into the feature flavor and what compiles into the others, with the stub or entry point each other flavor uses

Diagram rules: label each arrow with what flows along it; mark host-provided values and duplicate-name risks with a note; use only class names found in source or main; write "unknown" where unsure.

## Part 2: Behavior checks
For each feature, compare source against main's base classes for:
- UI rendering: theme, styles, toolbar, layout inflation, qualified layouts (land, sw600dp), night mode
- WebView: settings, client, JS interface, file and content access, back navigation, error handling
- Deeplinks: source intent filters, which Activity receives them, how main's manifest merges with the variant manifest, whether main's NavGraph or router intercepts them
- Feature flags and BuildConfig: where each flag is read, and whether it exists in the variant flavor
Report each as: same in both / differs (how) / depends on host.

## Part 3: DI ownership
For every provider, binding, module, scope, qualifier and @Inject target the features use:
1. Classify as copy, host-provided or implicit.
2. For each placement, state what main must change and what happens in flavors without these classes:
   - (a) variant module included by main's component only in the feature flavor
   - (b) subcomponent created from main's component
   - (c) a separate component owned by the feature
3. Flag every duplicate binding, duplicate scope, or qualifier clash with main.

## Part 4: Host-provided data
For each value (session, user id, token, flags): where main gets it, and compare delivery options: Intent extra, Fragment argument, DI binding provided by main, or another option. Recommend one per value.

## Part 5: Entry contract for main
Compare for launching <FILL_FEATURE> and <OTHER_SUB_FEATURE_OR_FUNCTIONALITY> from main:
- string-based Intent (no compile-time reference)
- flavor-specific interface with a no-op stub in other flavors
- optional injection (Optional or Provider that may be absent)
Recommend one and show the exact main call site.

## Part 6: File movement plan (per recommended approach)
Table: source path, destination path, action (copy / merge / create / edit main), reason. Include manifest entries, layouts and qualified variants, assets, and tests.

## Part 7: Benefits and bug risks
For the recommended approach and the main alternatives:
- maintenance and future-feature benefits
- bug risks: duplicate classes, silent wrong base behavior, missing manifest entries, missing DI bindings in some flavors, WebView regressions, deeplink conflicts
- how to detect each (build, unit test, device test)

## Part 8: Open questions
Ask all clarifying questions in one batch at the end. Do not assume answers.

## Output format
Write approach-analysis.md with sections in the order above. Use tables for comparisons. End with a one-paragraph recommendation and the single decision I must make first.