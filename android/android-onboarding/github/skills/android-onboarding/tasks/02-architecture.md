# Task: Architecture
**Author:** Rajendhiran Easu

## Depends on
00-setup (module list)

## Steps
1. Determine the architectural pattern in use (MVVM, MVI, Clean
   Architecture, or a mix). Look for: ViewModel classes, UseCase/
   Interactor classes, Repository classes, and UiState/Intent/Effect
   style classes. If modules differ in approach, note that explicitly
   per module rather than picking one pattern for the whole project.
2. Build a layer diagram (UI → ViewModel → UseCase → Repository →
   data source) as a Mermaid `flowchart LR`, generalized across the
   project, and a separate module dependency graph (which modules
   depend on which, from each module's `build.gradle(.kts)`
   `dependencies {}` block).
3. Check for layering violations: does any UI-layer class import a
   data-layer class directly, bypassing ViewModel/UseCase? Report each
   instance found with a citation.
4. Identify the state/event flow mechanism(s) in use: StateFlow,
   SharedFlow, LiveData, Channel, or plain callbacks — and where each
   is used, if more than one coexists.
5. Note the threading model: use of `viewModelScope`, `lifecycleScope`,
   explicit `Dispatchers`, WorkManager for background work.

## Output
Append to draft under "## Section 4: Architecture" with both Mermaid
diagrams included.

## Self-check
- The module dependency graph matches what's actually declared in
  build files, not an assumed clean-architecture ideal.
- Every layering violation cited includes path:line.
- Mermaid diagrams are syntactically valid.
