# SESSION_COMMANDS.md
# HardcoreAI — Agency Session Management Commands
# Version: 1.0 | Created: 2026-03-27
#
# These three slash commands standardize session open, close, and status
# across all agency projects. Reference this file from CLAUDE.md and
# .cursorrules in every project repo.
#
# To trigger: type !!closed, !!pickup, or !!status in any Claude Code session.

---

## !!closed — Close the Session

**Purpose:** Wrap up the current session. Extract what happened, write it to
the project files, commit everything, and confirm capture before closing.

**When to use:** At the end of any working session, before closing the terminal.

---

### Execution Steps

**Step 1 — Review the session**

Scan the full conversation history for this session and extract:
- What was accomplished (concrete outputs: files created, decisions made, code written)
- What decisions were made (anything that would affect future sessions)
- What was explicitly left undefined or deferred
- Any blockers or risks surfaced
- The single most important next action

If the session was short or produced no meaningful output, note that explicitly
and skip to Step 4 (commit any file changes that exist, even if minor).

**Step 2 — Update ACTIVE_PROJECT.md**

Read the current ACTIVE_PROJECT.md. Then update:
- "Where We Are" / current state section — reflect what changed this session
- "Immediate Next Step" — update to the next action identified above
- "Build Sequencing" — mark any completed items as [DONE]
- "Open Items" — add any new items surfaced; check off any resolved
- "Session Notes" — append a new entry in this format:

```
**[TODAY'S DATE]:** [2-4 sentence summary of what happened, what was decided,
and what comes next.]
```

If ACTIVE_PROJECT.md does not exist: create it with the session summary as
the seed content. Use the template structure from company-context/ACTIVE_PROJECT.md.

**Step 3 — Update DECISIONS.md**

For each significant decision made this session, append a new row to the
decisions table at the top of the existing entries:

```
| [TODAY'S DATE] | [Scope] | [Decision] | [Rationale] |
```

"Significant" means: a person joining this project tomorrow would want to know
this was decided. Trivial implementation choices don't qualify.

If no significant decisions were made, skip this step — do not add empty rows.

If DECISIONS.md does not exist: create it with the standard header and the
first entry being the session summary decision.

**Step 4 — Bubble up to company-level ACTIVE_PROJECT.md**

After updating the project files, sync the current project state to the company-wide dashboard:

1. Extract from the updated project `ACTIVE_PROJECT.md`:
   - Current phase
   - Immediate next action
   - Any blockers or risks

2. Open `C:\Users\rowem\projects\company-context\ACTIVE_PROJECT.md`

3. Find the projects table. Locate the row for the current project (match by project name):
   - If the row exists: update the **Phase**, **Last Worked**, **Next Action**, and **Blockers** columns with today's date and the values extracted above
   - If no row exists: add a new row for this project with all four columns populated

4. Save the file.

If `company-context/ACTIVE_PROJECT.md` cannot be found or contains no projects table,
note it in the confirmation output and skip this step — do not fail the close.

**Step 5 — Commit and push**

**Project repo:**
```
git add -A
git commit -m "session: [one-line summary of what this session produced]"
git push
```

If git push fails:
- Check if a remote exists (`git remote -v`)
- If no remote: note it to the user and confirm the commit was made locally
- If auth failure: commit locally, report the push failure with the exact error,
  and ask the user to push manually

If there are no changes to commit (working tree clean): skip the commit,
note that no file changes occurred this session.

**Company-context repo:**
```
cd C:\Users\rowem\projects\company-context
git add ACTIVE_PROJECT.md
git commit -m "sync: [project-name] session update [TODAY'S DATE]"
git push
```

If git push fails for company-context: note it in the confirmation output
and ask the user to push manually. Do not block the rest of the close.

**Step 6 — Confirm capture**

Output a brief confirmation in this format:

```
Session closed. Here's what was captured:

✓ Accomplished: [bullet list]
✓ Decisions logged: [count, or "none"]
✓ ACTIVE_PROJECT.md: updated
✓ Company dashboard: updated (or "skipped — [reason]")
✓ Committed: [commit hash or "no changes"]
✓ Pushed: [yes / no — reason if no]
✓ Company-context committed: [commit hash or "no changes / skipped"]
✓ Company-context pushed: [yes / no — reason if no]

Next session starts with: [the single next action]
```

---

## !!pickup — Start the Session

**Purpose:** Orient quickly at the start of a new session. Read the project
state, deliver a standup-style briefing, and ask what to work on today.

**When to use:** At the start of any working session, before doing anything else.

**Note — Claude chat (non-Claude Code):** When running !!pickup outside of Claude Code
(e.g., in Claude.ai chat), the working directory is not automatically set. You must
specify the project path explicitly:

```
!!pickup C:\Users\rowem\projects\[project-folder]
```

Claude Code sessions infer the path from the working directory automatically.

---

### Execution Steps

**Step 1 — Read project state**

Read the following files in order:
1. `ACTIVE_PROJECT.md` — full file
2. `CONTEXT.md` — full file (or PRODUCT.md if CONTEXT.md doesn't exist)
3. `DECISIONS.md` — last 5 entries only (most recent decisions)

If any file doesn't exist:
- ACTIVE_PROJECT.md missing: note it, offer to create it, continue with available context
- CONTEXT.md missing: look for PRODUCT.md as fallback; if neither exists, ask the user
  to describe the project before continuing
- DECISIONS.md missing: note it, continue without decision history

**Step 2 — Deliver the standup briefing**

Output in this exact format:

```
!!pickup — [PROJECT NAME]
Last updated: [date from ACTIVE_PROJECT.md]

LAST SESSION
[2-3 sentences summarizing what the last session accomplished, from session notes]

CURRENT STATE
[2-3 sentences on where the project stands right now]

WHAT'S LOCKED
[bullet list of 3-5 key locked decisions or artifacts — most relevant to active work]

BLOCKERS / RISKS
[bullet list, or "None identified" if clean]

NEXT ACTION
[The single most important thing to do today, from ACTIVE_PROJECT.md]

RECENT DECISIONS
[Last 3 decisions from DECISIONS.md, one line each]
```

Keep the briefing tight — the goal is orientation in under 60 seconds of reading.

**Step 3 — Ask what to work on**

After the briefing, ask:

> What do you want to work on today? I can start on [NEXT ACTION], or take a
> different direction if you have something else in mind.

Then wait for the user's answer before doing anything else.

---

## !!status — Company Dashboard

**Purpose:** Get a cross-project view of everything active at HardcoreAI.
Shows all project states, what needs attention, and a focus recommendation.

**When to use:** Weekly review, when context-switching between projects, or
when deciding what to prioritize.

---

### Execution Steps

**Step 1 — Find all active projects**

Read `ACTIVE_PROJECT.md` from the company-context repo (company-level state).

Then look for project repos in the known locations:
- `C:\Users\rowem\projects\` — scan for directories containing ACTIVE_PROJECT.md
- Read ACTIVE_PROJECT.md from each one found

If a project directory exists but has no ACTIVE_PROJECT.md: flag it as
"untracked — no context file found."

If no projects are found beyond company-context itself: report that and
offer to help set up context files for any projects the user names.

**Step 2 — Build the dashboard**

Output in this format:

```
!!status — HardcoreAI Dashboard
[TODAY'S DATE]

COMPANY STATE
[1-2 sentences on overall agency status from company-context/ACTIVE_PROJECT.md]

─────────────────────────────────────────
PROJECT SNAPSHOTS
─────────────────────────────────────────

[PROJECT NAME] — [Phase]
Status: [current state, 1 sentence]
Next action: [next action from that project's ACTIVE_PROJECT.md]
Last updated: [date]
Attention needed: [yes/no — yes if last update > 7 days ago, or if blockers exist]

[repeat for each project found]

─────────────────────────────────────────
NEEDS ATTENTION
─────────────────────────────────────────
[List any projects flagged above, with the specific reason]
[Or: "Nothing critical — all projects have recent activity and clear next actions"]

─────────────────────────────────────────
FOCUS RECOMMENDATION
─────────────────────────────────────────
Today: [single recommended project and action, with brief rationale]
```

**Step 3 — Edge cases**

- Project not updated in > 14 days: flag as "stale — may need re-orientation"
- Project with no next action defined: flag as "next action undefined"
- Projects with conflicting priorities: surface the conflict, don't silently pick one
- If only one project exists: still run the full format — useful for establishing the habit

---

## Notes for Implementation

**File path assumption:** These commands assume the working directory is the
active project repo. The company-context repo is expected at
`C:\Users\rowem\projects\company-context`.

**Date handling:** Use the actual current date (available from system context)
for all date fields. Never use placeholder dates.

**Tone:** All three commands should be brief, direct, and actionable. No filler.
No "Great! Let me help you with that." Just execute and report.

**Session overlap:** If !!pickup detects that the last !!closed was not run
(i.e., the last session notes show work but no commit was recorded for that
session), surface it: "Looks like the last session wasn't closed. Want me to
run !!closed for it now before we start?"

**Git branch awareness:** !!closed commits to whatever branch is currently
checked out. It does not create branches or open PRs — that's a separate
workflow decision. If the user is on a feature branch, the commit goes there.
