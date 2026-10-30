# Templates for task files

Use these structures as is. Replace angle-bracket placeholders.

---

## initial.md (developer writes; `.github/prompts/new-task.prompt.md` can create it)

```markdown
# Task: <task-name>

Status: Draft
Destination branch: migration/<task-name>
Source reference: <branch / tag / commit>   (blank = current checkout)

## 1. Source project directory path
<path>   (blank = default)
Source modules involved: <app, :lib-x, ...>   (blank = agent finds them)

## 2. Destination project path and variant
<path> / variant: <flavor>   (blank = default)
Build commands: assemble<Flavor>Debug, test<Flavor>DebugUnitTest   (blank = defaults)

## 3. Feature or functionality
<what the feature is, screens involved>
Entry point (if known): <class>
Launched in source via: <Intent / Fragment attach / route>   (optional)

## 4. Specific things
<quirks, special notes>
Must include: <classes, layouts, resources the agent might miss>
Must exclude: <anything that must not move>
Locales: all   (or list)
Gradle pre-approval: no   (or: flavor-scoped dependency additions at source versions)

## Out of scope
<other features living in the same files>

## References (optional)
<ticket, design, notes>
```

---

## planned_tasks.md

```markdown
Status: Plan Ready
Iteration: 1
Blocked from: -
Last updated by: plan agent
Open issues: 0

## Iteration 1

### Phase 1: Gradle approvals
- [ ] <item>

### Phase 2: Base and shared classes (variant copies)
- [ ] <source file> -> <destination file> (<members>)

### Phase 3: Feature code (UI, ViewModel, data, adapters, managers, analytics)
- [ ] ...

### Phase 4: Dagger pieces
- [ ] ...

### Phase 5: Resources (layouts, strings incl. locales, colors, dimens, styles, fonts, drawables, anim)
- [ ] ...

### Phase 6: Manifest and navigation
- [ ] ...

### Phase 7: Namespace and import alignment pass
- [ ] Apply mapping table to code, tests, XML, manifest
- [ ] Search for source namespace literal: none left

### Phase 8: Unit tests
- [ ] ...

### Phase 9: Build and test
- [ ] assemble<Flavor>Debug
- [ ] test<Flavor>DebugUnitTest

### Phase 10: Entry contract and summary
- [ ] migration-output/summary.md, file-map.md, gaps.md written
```

---

## plan-output/summary.md

```markdown
# Plan summary: <task-name> (Iteration N)
- Feature:
- Source / destination / variant:
- Number of files: copy X, merge Y, create Z
- Gradle changes needed:
- Open risks:
- Next step: developer approval
```

## plan-output/findings.md

```markdown
# Findings: <task-name>

## Resolved inputs (defaults and discovered values)
## Namespace mapping (confirm at approval)
| Item | Source package | Destination package |
|---|---|---|

## Visited list (traversal)
## Discovered beyond hints
| What | Where | Why the feature needs it |
|---|---|---|

## Skipped (Must exclude / Out of scope)
| What | Why |
|---|---|

## Gradle comparison
## Questions and answers
## Plan edits
| Date | Change | Reason |
|---|---|---|

## Iteration notes
```

## plan-output/prompt.md

```markdown
# Migration contract: <task-name> (Iteration N)

Rules: follow cc-tasks/_shared/rules.md. Source is read-only. Write only to the variant.

| # | Source path | Destination path (variant) | Action (copy / merge / create) | Members to bring | Alignment needed | Notes |
|---|---|---|---|---|---|---|

## Binary resources (copy with cp, or list for the developer)
## Gradle items (need approval unless pre-approved)
## Unit tests
## Namespace mapping reference
```

---

## migration-output/summary.md

```markdown
# Migration summary: <task-name> (Iteration N)

## Goal accomplished
## Files: copied X, merged Y, created Z, skipped W
## Build and test results
- assemble<Flavor>Debug: PASS / FAIL
- test<Flavor>DebugUnitTest: PASS / FAIL (n run, n failed)

## Entry contract
- Entry class (fully qualified):
- Launch paths:
- Required extras / arguments / newInstance signature:
- Manifest entries added:
- Host must provide:

## Forced changes
## Open gaps (see gaps.md)
```

## migration-output/file-map.md

```markdown
| Source path | Destination path | Status (copied / merged / created / skipped) | Members brought | Reason / notes |
|---|---|---|---|---|
```

## migration-output/gaps.md

```markdown
## Open questions for the developer
| # | Question | Options | Blocking |
|---|---|---|---|

## Mechanical fixes made (imports, namespace, missing item copied from source)
| # | File | Fix | Reason |
|---|---|---|---|

## Forced changes
| File | Change | Reason |
|---|---|---|
```

---

## review-output/checklist.md

```markdown
# Review checklist: <task-name> (Iteration N)

- [ ] Preflight done (source readable, baseline, terminal availability)
- [ ] Plan items all mapped in file-map.md
- [ ] Feature files complete vs source (code, res, locales, densities, night, manifest)
- [ ] Every "Must include" item present
- [ ] Nothing from "Must exclude" / "Out of scope" migrated
- [ ] Items discovered beyond hints were listed in the plan
- [ ] No source namespace left (kt, java, xml, manifest, tests)
- [ ] Imports resolve to variant / destination namespaces (none to main classes)
- [ ] Existing variant members unchanged
- [ ] No changes outside the variant (except approved Gradle)
- [ ] Gradle edits approved, no version bumps
- [ ] assemble<Flavor>Debug: PASS / FAIL
- [ ] test<Flavor>DebugUnitTest: PASS / FAIL (n run, n failed)
- [ ] Lint: PASS / FAIL / not run
- [ ] Entry contract matches code
- [ ] Forced changes documented
- [ ] planned_tasks.md matches reality

## Issues
| # | Description | File | Type (code gap / plan gap) | Iteration | Status |
|---|---|---|---|---|---|

## Verdict
Done proposed / Needs Fix / Blocked
```

## review-output/findings.md

```markdown
# Review findings: <task-name> (Iteration N)
## Summary
## Evidence (build output highlights, commands run)
## Issues in detail
## Recommended loop (A: code gap / B: plan gap)
```
