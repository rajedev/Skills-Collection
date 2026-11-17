# Copilot usage dashboard: build prompt

Save this file in the project root (or as `.github/copilot-instructions.md`). Then tell Copilot: **"Follow COPILOT_BUILD_PROMPT.md"**. After that, `resume` continues where it stopped.

## Goal
Build a simple local dashboard of my Copilot usage in Android Studio on **macOS**, using the telemetry my company's bash script already collects. No Python, no frameworks, no CDN, no network calls. Output is plain `index.html` + **one** `app.js` + a small shell script.

## Rules
- Work in `copilot-dashboard/`. Keep the plan in `PLAN.md` (checklist + a "Resume point" section). Tick items as you go.
- Everything stays on this machine. Read the telemetry files only; never modify, disable or re-run the company script, and never send data anywhere.
- Don't invent fields. Build only from what the real data contains.
- If you cannot run terminal commands in this mode, give me the commands in Phase 0 to paste in Terminal and wait for my output.

## Phase 0: discover the telemetry (do this first, build nothing yet)
Run read-only checks and report the results:
1. `find ~ -maxdepth 4 \( -iname "*telemetry*" -o -iname "*copilot*" \) -not -path "*/Library/Caches/*" 2>/dev/null`
2. `ls -la ~ ~/Library/LaunchAgents /Library/LaunchAgents 2>/dev/null | grep -i -E "telemetry|copilot|monitor"` and `crontab -l`
3. `grep -i -E "telemetry|copilot" ~/.zshrc ~/.bash_profile ~/.zprofile 2>/dev/null`
4. Open the bash script found and tell me in plain words: where it writes (file or folder), what it records, how often it runs, and whether it also uploads anything. Don't print secrets or tokens.
5. Show the **last 5 records** of its output (mask names, emails, hostnames, tokens) and state the format: JSONL, CSV, JSON or plain log.
6. Also check Android Studio's log: `ls ~/Library/Logs/Google/AndroidStudio*/idea.log` (only a rough activity signal).

Then write the findings to `PLAN.md` under `## Findings` (path, format, fields, timestamp format) and **stop and ask me to confirm** before Phase 1.

## Phase 1: build (after I confirm)
Files:
- `refresh.sh`: bash only. Reads the telemetry path(s) from the Findings plus `../copilot-sessions.jsonl` if it exists, and writes `data.js` containing `window.DATA = ...`. For JSONL use `paste -sd, file | sed` to wrap it into an array. For CSV, wrap the raw text in a JS template string (escape backslashes and backticks) and parse it in `app.js`. No `jq` or Python.
- `index.html`: layout and CSS only. System fonts, light and dark mode, readable on a laptop and a phone.
- `app.js`: the only JS file. Loads `window.DATA`, draws charts as plain SVG, no libraries.
- `.gitignore` entries for `data.js` and any raw telemetry copy.

I use it with: `./refresh.sh && open index.html`.

Dashboard requirements:
- **Date filter:** presets (7 / 30 / 90 days, All) plus custom From / To. Every number and chart follows it.
- **Overview numbers:** total events, active days, events per day, busiest hour, and a comparison with the previous period of the same length.
- **Charts:** events per day, hour of day, day of week.
- **Breakdowns (only if the field exists):** event types, languages or file types, acceptance rate (accepted ÷ shown), latency (median and p95), models, modes.
- **Session analytics** from `copilot-sessions.jsonl`: sessions, credits used, credits per session, share done, and sessions and credits by task type, model and mode, plus credits per day.
- Friendly empty states, handle missing or malformed lines without crashing, treat unknown credits as unknown (not zero).

## Phase 2: session routine (every session after the build)
- **`resume`**: read `PLAN.md`, tell me in 3 lines where we stopped and what is next, then continue with the first unchecked item. Don't ask me to paste anything.
- **`wrap up`**: update `PLAN.md`, then ask me for the **model** (from the chat picker) and the **credits / premium requests used** (from the Copilot usage indicator), since you can't see them. Accept `?` for unknown. Append one line to `copilot-sessions.jsonl`, or print it if you can't edit files in this mode:

```
{"timestamp":"YYYY-MM-DD HH:MM:SS","event":"session","task_type":"feature","mode":"agent","model":"<from me>","credits":<number or null>,"minutes":<estimate>,"outcome":"done|partial|blocked","files_changed":<number>,"topic":"<max 6 words>"}
```

- `task_type`: one of `feature`, `bug`, `triage`, `refactor`, `test`, `review`, `docs`, `explore`, `other`.
- `mode`: one of `ask`, `edit`, `agent`, `plan`. Use my local date and time. Never guess credits or model names.

## Checklist (copy into PLAN.md and tick as you go)
- [ ] Phase 0: telemetry script, output path, format and sample records found
- [ ] Findings written to PLAN.md and confirmed by me
- [ ] `refresh.sh` produces `data.js` from the real files
- [ ] `index.html` + `app.js` render with date filter
- [ ] Overview numbers and three time charts work
- [ ] Breakdowns match the fields that actually exist
- [ ] Session log read, session analytics shown
- [ ] Tested with an empty range and a malformed line
- [ ] `.gitignore` updated, README line on how to run
- [ ] Resume point updated

---

# Add-on: Sapience work-day dashboard (my own data, read-only)

Goal: a "Work day" view of **my own** Sapience numbers (time, productivity, idle, apps, meetings) in the same `copilot-dashboard/`, with the same date filter. It must work **on its own** (without Copilot data) and also side by side with Copilot usage when both exist. Same stack: `index.html` + the single `app.js` + bash `refresh.sh`, no Python, no libraries, no network.

## Ground rules (apply to every step)
- Read-only. Never modify, stop, restart or work around the Sapience agent.
- Use only data I am allowed to take for myself: an official export / download / report, or a file Sapience writes for me to use. Do **not** read the agent's internal databases or caches, intercept its network traffic, reuse its login or tokens, or script its web portal.
- Everything stays on this machine and out of git (`.gitignore` the raw files and `data.js`).
- Do not decide the policy question yourself. I confirm it in S0.

## Phase S0: is it available and allowed? (build nothing yet)
1. Ask me to check, and tell you the answers:
   - Does Sapience Buddy or my Sapience web page offer an **export, download or report** for my own data? In which formats (CSV, XLSX, PDF)?
   - Does my company policy or IT allow me to save that data on this Mac and use it in a local tool? (If unsure, I will ask IT or my manager first.)
2. If there is no export, or I am not sure it is allowed, **stop**. Offer only these fallbacks: a small CSV I type in by hand, or screenshots I give you.
3. If both answers are yes, ask me to save the export into `copilot-dashboard/sapience/`, then continue.

## Phase S1: inspect what I saved (read-only)
1. List `copilot-dashboard/sapience/`. For each file show the header and 5 rows with names, emails, machine names and tokens masked.
2. State the format and every field you found: dates, time zone, units (minutes or hours), and which of these exist: total, active, idle, productive / unproductive / neutral, first and last activity, app or website names, categories, meetings, breaks, goals.
3. Write them under `## Findings: Sapience` in `PLAN.md`, flag anything unclear (missing days, odd units), and **wait for my confirmation**.

## Phase S2: build (only after I confirm)
- `refresh.sh`: also wrap `sapience/*` into `window.SAPIENCE` in `data.js` (same no-Python approach; CSV wrapped as text and parsed in `app.js`). Skip missing or malformed rows without crashing.
- `app.js` (still the only JS file): add a **Work day** tab. The date filter is shared. Show only what the data supports:
  - **Time:** logged, active and idle hours per day; average per active day; first activity and last activity; weekday and hour patterns.
  - **Productivity:** productive / unproductive / neutral split, daily trend, change versus the previous period of equal length, progress against a weekly target if the export has one.
  - **Apps and sites:** top apps and top websites by time, category split, and each app's share of the day.
  - **Meetings:** count, hours, share of the working day, busiest meeting days, and focus time left (active hours minus meeting hours) if both exist.
  - **Idle and breaks:** idle share, longest idle gaps, break pattern.
- **Combined view** (only when both datasets have data in the range): Copilot sessions and credits next to active hours per day, credits per active hour, and Copilot-heavy days versus meeting-heavy days.
- Friendly empty states, unknown values shown as unknown (never zero), consistent units everywhere.
- Add a clear "last data date" label so I can see when I last refreshed.

Run: `./refresh.sh && open index.html`

## Checklist additions
- [ ] S0: export option exists, and I confirmed it is allowed on this Mac
- [ ] S1: sample files inspected, fields and units written to PLAN.md, confirmed by me
- [ ] S2: `window.SAPIENCE` loaded by `refresh.sh`
- [ ] Work day tab works on its own (no Copilot data)
- [ ] Time, productivity, apps and meetings sections show only fields that exist
- [ ] Combined Copilot + work-day view works when both exist
- [ ] Empty range and malformed rows tested
- [ ] Raw Sapience files and `data.js` in `.gitignore`
- [ ] Resume point updated
