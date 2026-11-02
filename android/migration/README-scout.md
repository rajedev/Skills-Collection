# Scout Agent (source-side scoping for feature migration)

A small GitHub Copilot agent that runs inside the **source project**. You give it a feature name, a short definition and a few rough key terms. It finds the likely scope and writes a pre-filled `initial.md`, which you review and hand to the Plan agent of the migration workflow.

It saves you from filling in the source-side fields of `initial.md` by hand. It does not replace `@plan`, which still runs the full discovery.

**Author:** Rajendhiran Easu

---

## 1. Where it fits

```
Source project                         Destination repo
--------------                         ----------------
/scout-feature  ->  initial.md  --->   cc-tasks/<task-name>/initial.md
/rescout-for-gaps   (you review,            |
                     edit, copy)            v
                                      @plan -> @migration -> @review
```

| Step | Who | Output |
|---|---|---|
| 1. Scope the feature | Scout (source project) | `initial.md` with a Gaps checklist and Scout notes |
| 2. Review and fix gaps | You, plus `/rescout-for-gaps` if needed | Updated `initial.md` |
| 3. Hand over | You copy it to the destination `cc-tasks/<task-name>/` | Ready for `@plan` |

---

## 2. What is in the package

| File | Goes to | Purpose |
|---|---|---|
| `scout.agent.md` | `.github/agents/` | The agent: steps, rules, template, re-scout mode |
| `scout-feature.prompt.md` | `.github/prompts/` | `/scout-feature`: first run |
| `rescout-for-gaps.prompt.md` | `.github/prompts/` | `/rescout-for-gaps`: rerun on open gaps with your insights |
| `README-scout.md` | anywhere | This guide |

Install the first three in the **source** repo.

---

## 3. One-time setup (source project)

1. Copy the three files into the source repo at the paths above. In Android Studio, use **Project** view to see hidden folders.
2. Keep them out of the source's commits: add to `.git/info/exclude`:
   ```text
   .github/agents/scout.agent.md
   .github/prompts/scout-feature.prompt.md
   .github/prompts/rescout-for-gaps.prompt.md
   cc-scout/
   ```
   If the source is a pinned read-only checkout, skip `cc-scout/`. The scout will print `initial.md` in chat and you save it yourself.
3. Check that `@scout`, `/scout-feature` and `/rescout-for-gaps` appear in Copilot chat. If they do not, your plugin version may not support custom agent or prompt files. The content can then be pasted as a normal prompt.

Keep filenames and the `name:` line in each file's frontmatter identical, or the slash command will not match.

---

## 4. How to run

### First run
1. Open the source project in Android Studio.
2. In Copilot chat, type `/scout-feature`.
3. Give three things:
   - **Feature name**
   - **Short definition** (one or two sentences)
   - **Rough key terms** (words that may appear in class, screen, string or route names)
4. The scout searches, picks the likely entry point, and writes `initial.md` (to `cc-scout/<task-name>/` or in chat).
5. Read `initial.md`. Check the entry point, modules, suggested out-of-scope items and the **Gaps** list.

### If there are gaps
1. Add your insights, for example "the entry point is `XyzSupportActivity`; ignore the chat widget".
2. Run `/rescout-for-gaps` with those insights.
3. The scout rechecks only the open gaps, updates `initial.md` in place (your edits are kept), ticks resolved gaps and replies with a short summary.
4. Repeat until you are happy. A few open gaps are fine, since `@plan` asks its own questions.

---

## 5. Two projects: where each step runs

The scout is installed and run **only in the source project**. The migration (destination) project receives one markdown file, not code.

### In the source project (Android Studio)
1. Copy the three files into the source repo: `scout.agent.md` to `.github/agents/`, and `scout-feature.prompt.md` and `rescout-for-gaps.prompt.md` to `.github/prompts/`.
2. Run `/scout-feature` with the feature name, a short definition and rough key terms.
3. Review the generated `initial.md`. If there are open gaps, add your insights and run `/rescout-for-gaps`.

### In the migration (destination) project
1. Delete the Scout notes block from `initial.md`.
2. Save the file as `cc-tasks/<task-name>/initial.md`. This is a tooling folder, not app code, so none of the variant source sets are touched.
3. Fill in the destination fields (path, variant, build commands) if the defaults do not cover them.
4. Run `@plan cc-tasks/<task-name>` to continue the main workflow.

### Notes
- The two projects can be open in separate Android Studio windows. The scout only needs the source project open, and you copy the file across yourself.
- Nothing is written to the source project except the optional untracked `cc-scout/<task-name>/initial.md`. If the source is a pinned read-only checkout, let the scout print `initial.md` in chat and save it yourself.
- Only `initial.md` moves between the projects. Source code is copied later by `@migration`, never by the scout.

---

## 6. Use case: Help Center

**You type**
```text
/scout-feature
Feature name: Help Center
Definition: in-app FAQ and support articles with a contact-us screen
Key terms: faq, article, support, contact, help
```

**The scout finds** (illustrative)
- Search hits on `HelpCenterActivity`, `FaqAdapter`, `layout_help_*` and a `help_*` string group.
- `HelpCenterActivity` is the most likely entry point, launched by an Intent from the settings screen.
- The same adapter file also holds code for an unrelated "feedback" screen.

**`initial.md` it writes (shortened)**
```markdown
# Task: help-center

Status: Draft
Destination branch: migration/help-center
Source reference: 3f9c2ab

## 1. Source project directory path
/path/to/source-pinned
Source modules involved: app

## 3. Feature or functionality
Help Center: in-app FAQ and support articles with a contact-us screen
Entry point (if known): HelpCenterActivity
Launched in source via: Intent from SettingsFragment

## 4. Specific things
Known host-provided things: session token read in HelpCenterActivity

## Out of scope
Feedback screen code in FaqAdapter (suggested)

## Gaps (scout could not identify)
- [ ] Contact-us screen entry unclear (tried: searched "contact" in routes and manifest)
- [ ] Analytics tracker usage not found (tried: searched "help" in analytics classes)

## Scout notes (delete before use)
- Entry point chosen because: name matches and it is launched from Settings
- Other candidates: HelpCenterV2Fragment (unused?)
- Questions for the developer: is the contact-us screen part of this task?
```

**You reply** with an insight: "Contact-us is part of the task; `ContactUsFragment` is attached from `HelpCenterActivity`. Analytics lives in `HelpEvents`."

**Run `/rescout-for-gaps`.** The scout checks both items, ticks them with one-line results, adds `ContactUsFragment` to the entry notes, and replies:
```text
Resolved: 2. Still open: 0.
Changed: Entry point notes (added ContactUsFragment), Gaps ticked.
Question: none.
```

**Hand over.** Delete the Scout notes block, save the file as `cc-tasks/help-center/initial.md` in the destination repo, and run `@plan cc-tasks/help-center`.

---

## 7. What the scout does and does not do

| Does | Does not |
|---|---|
| Find the likely entry point and launch path | List every file, layout or resource |
| Note the modules involved | Map all dependencies or libraries |
| Suggest out-of-scope items | Decide scope for you |
| Record what it could not identify as Gaps | Guess to fill a blank |
| Note obvious host-provided things only | Hunt for them |

Rules in short:
- Read-only on the whole source project. The only write is `cc-scout/<task-name>/initial.md`.
- Everything it fills is a suggestion. Your insights win over its guesses.
- If the key terms match nothing useful, it asks for better terms instead of widening the search.
- It asks at most one batch of questions, and only if the entry point is unclear.

---

## 8. Tips and troubleshooting

| Problem | Try |
|---|---|
| Wrong entry point picked | Add the right class in your insights and run `/rescout-for-gaps` |
| No useful matches | Give more specific terms (screen names, string keys, route names) |
| Commands do not appear | Check file paths, filename vs `name:` match, and plugin support for custom agents |
| Cannot write `cc-scout/` | Let it print in chat and save the file yourself |
| Source commit hash is blank | No terminal available. Run `git rev-parse HEAD` yourself and paste it in |

Good key terms are the words a developer would grep for: screen names, adapter or ViewModel names, string key prefixes, route names. Three to six terms is plenty.

---

## 9. Adjust as your need

- Whether your Copilot plugin version loads custom `.agent.md` and `.prompt.md` files.
- Whether Copilot can run `git` commands in your setup (needed only for the source commit hash).
- No pilot run yet. Try it on one small feature and adjust `scout.agent.md` as needed.
