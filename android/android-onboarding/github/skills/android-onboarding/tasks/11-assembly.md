# Task: Final assembly
**Author:** Rajendhiran Easu

## Depends on
Every other task must be `done` (including all per-module/per-feature
sub-scopes of tasks 04 and 09) before this task runs. agent.md checks
this before invoking it.

## Steps
1. Read `draft_<timestamp>.md` in full.
2. Reorder/merge sections into the final structure (per SKILL.md's
   section template list, Sections 0–16). Task output was appended in
   roughly this order already, but check for:
   - Any section written out of order.
   - Duplicated content (e.g. a class accidentally carded in both
     task 04's feature pass and task 09's module pass) — keep the more
     detailed version, remove the duplicate, and add a cross-reference
     instead.
3. Verify every Mermaid code block parses (balanced brackets, valid
   diagram type keyword, no syntax errors). Fix or remove any that
   don't, noting the removal in Limitations.
4. Verify every claim marked **Confirmed** has a `path:line` or at
   least `path` citation. Any that don't, either find the citation or
   relabel as **Inferred**.
5. Build the consolidated "Top gaps and open questions" summary and
   place it immediately after Section 0 (Run Metadata), before Section
   1. This should be short — the top 5 gaps by priority and a count of
   total open questions, each as a one-line pointer to the full detail
   later in the document.
6. Confirm no secret values are present anywhere in the document
   (search for common patterns: `api_key`, `Bearer `, long base64-like
   strings near words like `secret`/`token`/`password`).
7. Write the final merged document to `explored_<timestamp>.md` in the
   project root, using the exact filename from progress.md's
   `output_file`.
8. Delete `draft_<timestamp>.md`.

## Self-check
- The final file has all 16 sections (or an explicit "Not applicable"
  note for any that don't apply to this project).
- Every Mermaid block in the final file is valid.
- No Confirmed claim lacks a citation.
- No secret-looking values remain.
- The top-of-document summary accurately reflects what's in the full
  Gaps/Open Questions sections later in the file.
- draft_<timestamp>.md no longer exists after this task completes.
