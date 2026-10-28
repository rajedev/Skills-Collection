# Task: Data flow & API integration
**Author:** Rajendhiran Easu

## Depends on
00-setup (module list), 04-features (to link endpoints to features)

## Steps
1. Identify the networking stack: Retrofit, Ktor, OkHttp directly,
   GraphQL client, or a mix.
2. Find base URLs per build variant/flavor (redact any embedded keys
   or tokens as `<REDACTED>`, but the URL pattern itself is fine to
   include).
3. Build a table of every API interface/endpoint: method, path,
   request DTO, response DTO, and which feature(s) consume it
   (cross-reference task 04's output where available).
4. Trace the mapping chain: DTO → domain model → UI model. Note where
   each mapper lives.
5. Document caching and persistence: Room, DataStore, in-memory caches,
   HTTP-level caching. Note offline behavior, pagination approach,
   auth token handling (where tokens are stored and attached),
   interceptors, retry/backoff logic, and how errors are mapped to
   UI-facing states.
6. Build a Mermaid `flowchart` of data flow across layers: UI →
   Repository → (network | local DB) → back to UI, including where
   caching sits in that path.

## Output
Append to draft under "## Section 7: Data Flow & API Integration",
including the endpoint table and the Mermaid flowchart.

## Self-check
- No actual secret values appear in the output — only patterns/
  mechanisms.
- Every endpoint in the table has a path:line citation to its
  interface definition.
- The mapping chain is traced with real mapper locations, not assumed.
- The Mermaid diagram is syntactically valid.
