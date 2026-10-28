# Copilot instructions for this repository
**Author:** Rajendhiran Easu

When asked to explore, onboard, or generate a project overview of this
codebase, follow the agent defined at:

    .github/skills/android-onboarding/agent.md

Rules that always apply to this workflow:

- Read `.github/skills/android-onboarding/progress.md` first (create it
  from the template in `agent.md` if it does not exist).
- Run exactly ONE task per invocation, as listed in
  `.github/skills/android-onboarding/tasks/`.
- Never modify application source code. The only files this workflow
  may write are:
  - `.github/skills/android-onboarding/progress.md`
  - `draft_<timestamp>.md` (scratch file, project root)
  - `explored_<timestamp>.md` (final output, project root)
- Use `.github/skills/android-onboarding/SKILL.md` as the reference for
  search patterns, templates, and diagram conventions. Do not duplicate
  its content inline; just follow it.

To start or continue an exploration, the user will say something like
"continue the onboarding exploration" or "run the next onboarding task."
Respond by following agent.md exactly.
