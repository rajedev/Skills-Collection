# Task: Dependency injection
**Author:** Rajendhiran Easu

## Depends on
00-setup (module list)

## Steps
1. Identify the DI framework: Hilt, Koin, Dagger (manual, not Hilt), or
   no framework (manual service locators / singletons).
2. List DI modules, what each provides, and the scope used (Singleton,
   ViewModelScoped, ActivityRetained, custom scopes).
3. Build a Mermaid graph (`flowchart LR` or `classDiagram`, whichever
   renders the bindings more clearly) of major bindings and their
   dependencies — focus on the bindings with the most fan-out/fan-in,
   not an exhaustive graph of every provided type if the project is
   large.
4. Search for manual service locators or static singletons that bypass
   the DI framework (e.g. `companion object { val instance = ... }`
   holding shared mutable state). Report each with a citation and a
   note on why it bypasses DI.

## Output
Append to draft under "## Section 8: Dependency Injection".

## Self-check
- Every module/binding listed has a path citation.
- The Mermaid graph is syntactically valid.
- Manual-singleton findings are backed by actual code, not assumed
  from class naming alone.
