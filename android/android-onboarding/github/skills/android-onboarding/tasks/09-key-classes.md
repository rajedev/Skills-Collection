# Task: Key class deep-dive
**Author:** Rajendhiran Easu

## Depends on
00-setup (module list), 03-navigation, 04-features (for critical-path
cross-referencing)

## Scope — IMPORTANT
Process ONE module per invocation. This task almost always needs
multiple invocations to complete; that's expected. Check progress.md
for which modules under `09-key-classes` are already done, and pick
the next undone one.

## Steps, per module
1. Search the module for every role listed in SKILL.md's role taxonomy
   table. Confirm each match by reading the class, not just matching
   the name pattern.
2. Build a catalog table for this module: class, path, role, fan-in
   count (rough — number of distinct files referencing it), depth
   (card/brief/none).
3. Assign depth per SKILL.md's rule: card the top 8–12 by fan-in in
   this module, plus anything on a feature's critical path (check task
   04's output for this module's feature(s)); brief for the rest;
   none for generated code (list separately, don't catalog in detail).
4. Write full deep-dive cards (SKILL.md template) for everything
   assigned "card."
5. If this module has a State/Event/Reducer set, a Middleware/
   Interceptor/Command chain, or notable Listener usage, draw the
   relevant cluster diagram(s) from SKILL.md's diagram list.
6. Run the key-class cross-checks from SKILL.md against this module:
   unreachable states, unhandled events, listener leak risks,
   non-Application Context in long-lived objects, hidden-service
   Helpers/Utils.

## Output
Append to draft under "## Section 11: Key Classes" → subsection per
module, e.g. "### Module: feature-checkout". Include the catalog table,
the cards, any cluster diagrams, and the cross-check results for this
module.

## Progress note
Track per-module sub-status in progress.md:
```
- [ ] 09-key-classes — app: done, core-network: done, feature-login: in-progress
```
Only mark `09-key-classes` fully done once every module from the
canonical module list (from task 00) has a sub-status of done.

## Self-check
- Every card has a path:line citation.
- Every role assignment was confirmed by reading the class body, not
  guessed from its name.
- Cross-check findings (leaks, unhandled events, etc.) cite the
  specific class and location.
- Any cluster diagrams included are syntactically valid Mermaid.
- The catalog table accounts for every class the searches turned up —
  nothing silently dropped.
