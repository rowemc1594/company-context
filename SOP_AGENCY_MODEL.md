# SOP_AGENCY_MODEL.md
# HardcoreAI — Agency Operating Model
# Version: 1.0 | Created: 2026-04-04
# Owner: Mike Rowe

---

## 1. The System in One Sentence

Git-based context files keep all AI tools synchronized on every project, session commands enforce consistent open/close ritual, and a shared company-context repo provides the cross-project brain — so every Claude session starts oriented and every session ends with its work captured.

---

## 2. Tool Stack

### Claude.ai (Chat)
**What it's for:** Long-form thinking, strategy, brainstorming, writing, and deep research. Best for open-ended exploration before you know what you're building.

**When to use it:** Early-stage discovery, content creation, research synthesis, one-off questions that don't need project context.

**How it sees files:** Via Claude Projects. Each project has a system prompt and uploaded knowledge files. Files uploaded to a Claude Project are available in every conversation in that project.

**Limitation:** Does not read from the filesystem. Cannot see git history. Cannot commit or push. Context is limited to what you've uploaded to the Project.

---

### Claude Code (CLI)
**What it's for:** Hands-on project work — writing and editing code, managing files, running git commands, maintaining ACTIVE_PROJECT.md and DECISIONS.md, and executing session commands (!!pickup, !!closed, !!status).

**When to use it:** Any working session on a project repo. Code writing, spec drafting, context file updates, commits, and pushes all happen here.

**How it sees files:** Direct filesystem access. Reads any file path you point it to. Runs in the working directory of the project.

**Limitation:** Has no memory between sessions unless you've run !!closed at the end of the previous session (which writes state to ACTIVE_PROJECT.md).

---

### Cursor (IDE)
**What it's for:** Code-heavy sessions where AI-assisted editing inside an IDE is better than command-line. Same projects as Claude Code — complementary, not separate.

**When to use it:** When the work is primarily code editing, refactoring, or debugging inside a full IDE context.

**How it sees files:** Via .cursorrules in the project root. Reads the current workspace. Respects the ground truth hierarchy defined in .cursorrules.

**Limitation:** Does not have access to company-context or cross-project state unless you point it there explicitly.

---

### Claude Cowork (Scheduled Tasks / Background Agents)
**What it's for:** Recurring tasks, background synthesis, and work that doesn't need your real-time attention — weekly reviews, periodic summarization, status generation.

**When to use it:** When you want something to run on a schedule or in the background while you work on something else.

**How it sees files:** Via the filesystem MCP server configured in `claude_desktop_config.json`. Can read from `C:\Users\rowem\projects\` when properly configured.

**Limitation:** Requires explicit MCP configuration. Not suitable for interactive back-and-forth work.

---

### Cross-Tool Synchronization Rule

All four tools stay synchronized through **git-based context files**. No native cross-tool memory exists. The synchronization protocol is:

1. Every project has `ACTIVE_PROJECT.md` and `DECISIONS.md` — the source of truth for current state
2. `!!closed` writes to those files and commits — this is the sync event
3. Any tool reading those files at session start gets the same picture
4. `company-context` repo serves as the company-wide brain — readable by all tools

---

## 3. Project Structure

### Company-Level Files
Location: `C:\Users\rowem\projects\company-context\`

| File | Purpose |
|------|---------|
| `CONTEXT.md` | HardcoreAI identity, services, tech stack preferences, brand voice, coding conventions |
| `ACTIVE_PROJECT.md` | Company-level state: active projects table, this week's focus, open items |
| `DECISIONS.md` | Company-wide decision log — append-only, never delete entries |
| `SESSION_COMMANDS.md` | Full definitions for !!pickup, !!closed, !!status — referenced from all project CLAUDE.md files |
| `BOOTSTRAP.md` | Bootstrap prompt for spinning up a new project context layer from scratch |
| `SOP_AGENCY_MODEL.md` | This file — operating model reference |

---

### Project-Level Files
Location: `C:\Users\rowem\projects\[project-name]\`

| File | Purpose |
|------|---------|
| `PRODUCT.md` | Product vision, behavioral rules, locked architecture decisions, tech stack — the constitution |
| `ACTIVE_PROJECT.md` | Current state, immediate next step, build sequencing, open questions, session notes log |
| `DECISIONS.md` | Project-specific decision log — append-only, most recent entries at top |
| `CONTEXT.md` | Full project orientation: vision, problem, differentiators, stage model, file index |
| `CLAUDE.md` | Claude Code session instructions: ground truth hierarchy, behavioral rules, artifact lifecycle, checklists |
| `.cursorrules` | Cursor-specific context: same hierarchy and rules, IDE-oriented format |
| `.gitignore` | Always includes `.claude/` (worktrees), `node_modules/`, `.env`, `.env.local` |

---

### Ground Truth Hierarchy (ProdIDE model — apply to all projects)

When any two sources conflict, this order governs. Lower sources never override higher ones.

1. `DECISIONS.md` — most recent entry wins for any resolved decision
2. `PRODUCT.md` — product identity, behavioral rules, locked architecture
3. Journey Master files (or equivalent spec docs)
4. Role POD files (or equivalent implementation specs)
5. Everything else — prototypes, templates, session notes

---

## 4. Session Commands

Full definitions: `C:\Users\rowem\projects\company-context\SESSION_COMMANDS.md`

### !!pickup — Start of every session

**When:** Before doing anything else in a working session.

**What it does:**
1. Reads `ACTIVE_PROJECT.md` (full), `CONTEXT.md` (full), last 5 entries in `DECISIONS.md`
2. Delivers a standup-style briefing: last session summary, current state, what's locked, blockers, next action, recent decisions
3. Asks what you want to work on today

**Why it matters:** Without pickup, every session starts from zero and risks re-doing work or missing locked decisions.

---

### !!closed — End of every session

**When:** Before closing the terminal. No exceptions.

**What it does:**
1. Scans the session conversation and extracts: accomplishments, decisions, deferred items, blockers, next action
2. Updates `ACTIVE_PROJECT.md` — current state, next step, build sequencing, session notes entry
3. Updates `DECISIONS.md` — appends any significant decisions made this session
4. Runs `git add -A && git commit -m "session: [summary]" && git push`
5. Outputs a confirmation with captured items and commit hash

**Why it matters:** This is the sync event. If you close without !!closed, the next session starts with stale state and the work done this session is not captured.

**Rule: Never close without running !!closed.**

---

### !!status — Weekly review or context switch

**When:** Monday morning, before switching from one project to another, or when you've lost track of the company-wide picture.

**What it does:**
1. Reads `ACTIVE_PROJECT.md` from `company-context`
2. Scans `C:\Users\rowem\projects\` for all directories containing `ACTIVE_PROJECT.md`
3. Reads each one and assembles a company-wide dashboard
4. Flags projects with no recent activity (>7 days), undefined next actions, or known blockers
5. Outputs a focus recommendation — one project, one action, brief rationale

---

## 5. Daily Workflow

### Morning Startup

1. Open Claude Code in the project directory you're working on
2. Run `!!pickup`
3. Read the standup briefing — confirm the next action matches your intent
4. Tell Claude Code what you want to work on today
5. Work

If you're not sure which project to open, run `!!status` from any project directory to get the company dashboard first.

---

### Working a Project

- Let session commands handle orientation — don't re-read all the context files manually
- When Claude Code detects a decision being made, it will log it to `DECISIONS.md` — let it
- If working in Cursor: the `.cursorrules` file provides the same orientation the IDE needs
- If you need Claude.ai (chat): open the project's Claude Project and use the system prompt and knowledge files there
- Never move an artifact from Draft → Approved or Approved → Locked without explicitly confirming it

---

### Session End

1. When you're wrapping up, run `!!closed`
2. Review the confirmation output — check that accomplishments, decisions, and next action are captured correctly
3. If something is missing from the summary, correct it before closing
4. Close the terminal only after !!closed confirms the push

---

### Starting a New Project

1. Create the project directory
2. Initialize a git repo and create a remote on GitHub
3. Run the bootstrap prompt from `company-context/BOOTSTRAP.md` in Claude Code
4. Review the generated context files — correct anything that doesn't match the actual project
5. Commit and push the initial context layer
6. Add the project to the `ACTIVE_PROJECT.md` table in `company-context`
7. Run `!!closed` to capture the setup session

---

## 6. Git Discipline

### The Simple Rule

**Pull before you work. Push when you close.**

- `git pull origin main` at the start of any session where others might have pushed (or where you've worked from another machine)
- `!!closed` handles the push at end of session — don't push manually unless you have a specific reason

---

### Branch Behavior

- **Main branch is the working branch** for solo project work. Feature branches are for significant new work streams that aren't ready to merge.
- **Claude Code uses worktrees** for some operations — these live under `.claude/worktrees/` which is gitignored and doesn't affect your working tree
- **!!closed commits to whatever branch is checked out** — it does not create branches or open PRs automatically
- **PRs are for review gates**, not required for solo commits to main

---

### Merge to Main

When a feature branch is ready:
1. Open a PR on GitHub (or use `gh pr create` if `gh` CLI is installed)
2. Review the diff — confirm nothing unexpected is included
3. Merge and pull down to main locally: `git checkout main && git pull origin main`

---

### Recovery Process

| Situation | Recovery |
|-----------|---------|
| Forgot to run !!closed last session | Run !!closed now — it will reconstruct from the session history. Context may be incomplete but better than nothing. |
| Pushed to wrong branch | Check out the correct branch, cherry-pick or re-apply the commit, push. Don't force-push main. |
| ACTIVE_PROJECT.md is stale or missing | Read DECISIONS.md and git log to reconstruct state. Create a new ACTIVE_PROJECT.md from that history. |
| Merge conflict in context files | Both versions represent real work — merge manually, keeping both session notes entries. Don't discard either side. |
| Stale worktrees cluttering git status | Run `git worktree prune` to clean up stale references. If that fails, use `git worktree list` to identify and manually remove. |

---

## 7. Failure Points and Fixes

| Failure | Symptom | Fix |
|---------|---------|-----|
| Session closed without !!closed | Next session starts with stale ACTIVE_PROJECT.md; work from last session not captured | Run !!closed at the start of the new session before doing new work. Claude Code will reconstruct from conversation history. |
| claude_desktop_config.json reverts on restart | Trusted folders or MCP servers disappear after Claude Desktop restart | Edit the file while Claude Desktop is fully closed. Claude Desktop overwrites the file on shutdown. |
| Claude Code slash command conflict | /pickup, /closed, /status don't work as expected | Use !! prefix (!!pickup, !!closed, !!status) — / prefix is reserved for Claude Code built-in commands. |
| Stale worktree blocking git commands | Git reports a locked or inaccessible worktree | Run `git worktree prune`. If a specific worktree needs removal: `git worktree remove [path]`. Physical directory deletion is a last resort. |
| Context drift between tools | Cursor or Claude.ai is working from outdated spec | Re-upload CONTEXT.md / PRODUCT.md to the Claude Project. In Cursor, .cursorrules reads from the repo — pull latest. |
| AI making decisions without PM confirmation | Locked artifact changed without explicit instruction | Never instruct Claude Code to "fix" a conflict silently. Always surface the conflict and present two paths: align to locked decision, or change-control the locked artifact. |
| New session, wrong project orientation | Claude Code has context from a different project | Run !!pickup from the correct project directory. Or manually read ACTIVE_PROJECT.md and confirm the working directory. |
| Decisions made but not logged | Future session is missing context; drift accumulates | Run !!closed even for short sessions — it will log any decisions made. If a decision was made in a session without !!closed, add it manually to DECISIONS.md. |
| gh CLI not found | `gh` commands fail | gh CLI is not installed by default. Install from https://cli.github.com or use the GitHub web UI for PR creation. |
| BOOTSTRAP.md is a placeholder | New project setup requires manual work | Paste the actual bootstrap prompt content into BOOTSTRAP.md. Until then, set up new projects by hand using the existing context files as templates. |

---

## 8. Claude Projects Setup

### One Project per Client/Product

Each active project gets its own Claude Project in Claude.ai. The Claude Project is the chat interface's window into the project — it supplements (not replaces) the git-based context layer.

---

### System Prompt Template

Use this template as the starting point for every project's Claude Project system prompt:

```
You are working on [PROJECT NAME] with Mike Rowe at HardcoreAI.

CONTEXT:
[Paste the "What Is This Project" section from CONTEXT.md — 3-5 sentences]

YOUR ROLE:
[Describe what you want this Claude Project used for — research, writing, strategy, review, etc.]

GROUND TRUTH:
The locked decisions and product rules for this project are in the knowledge files uploaded to this Project. When in doubt, defer to those files over your own inferences.

BEHAVIORAL RULES:
- Never auto-lock or auto-promote artifacts
- Surface assumptions before acting on them
- If something contradicts a locked decision, flag it before proceeding
- Context is scoped to this project — do not draw from outside context

BRAND VOICE (HardcoreAI):
Direct, confident, expert but not arrogant. Practical over theoretical. Business outcomes over technology for its own sake.
```

---

### Knowledge File Guidance

Upload these files to the Claude Project and keep them current:

| File | When to update |
|------|---------------|
| `PRODUCT.md` or `CONTEXT.md` | When a locked decision changes or the product vision evolves |
| `ACTIVE_PROJECT.md` | At the start of a significant new phase |
| Journey Master files | When a Journey Master is first locked; update if change-controlled |
| Role POD files | When individual PODs are locked |

**Do not upload:**
- `DECISIONS.md` — too long and changes too frequently; summarize relevant decisions in the system prompt instead
- Raw session notes — noise, not signal
- Prototype code — the Claude Project is for product thinking, not code review

**Limitation:** Claude Projects have no git integration. Files go stale after you update the repo. Maintain a habit of re-uploading when major milestones complete.

---

## 9. What HardcoreAI Is

### Owner
**Mike Rowe** — 20+ years in software (QA → PM → product leadership). Domain expertise in fintech, healthcare, HSA/benefits, CRM, and blockchain. Most recently leading AI integration into the PDLC at HealthEquity.

### Mission
Solo AI-powered agency. Two tracks running in parallel.

---

### Track 1 — Software Products
Build and sell AI-native tools. The flagship is **ProductIDE**.

**ProductIDE** is an AI-native workspace for product managers — the governance and intelligence layer between raw PM thinking and engineering-ready requirements. It is not a document tool, a chat interface, or a roadmap builder. Its job is to take a PM from scattered research and half-formed ideas to a locked, governed, engineering-ready Product Requirement Prompt (PRP) — without losing the thinking that happened along the way.

Core differentiators:
- **Ambient Capture (P0):** AI detects crystallization moments in PM thinking and surfaces capture prompts before the insight is lost
- **Strict artifact lifecycle:** Draft → Approved → Locked. Locked artifacts are immutable — change control required to update
- **Context scoping:** AI only reasons from sources the PM has explicitly brought into the project
- **PRP output:** Structured markdown + JSON formatted for direct engineering AI consumption — not human-readable PRDs
- **Six-stage model:** Research → Idea → Initiative → Epic → Feature → PRP Ready. Each stage has stage-specific AI behavior

Current status: Four Journey Masters written and locked. v6 prototype covers Journeys 1 and 2. UX POD for Journey 3 is the immediate next deliverable.

---

### Track 2 — Consulting
AI workflow audits, agent stack setup, and AI PDLC integration for small to mid-size companies. Target clients: companies that need to operationalize AI but don't have internal capacity to design the workflows.

---

### Tech Stack Defaults
- **Frontend:** React / TypeScript
- **Backend:** Node.js / TypeScript
- **Database:** PostgreSQL
- **Hosting:** Vercel (frontend) / Railway (backend)
- **Payments:** Stripe
- **AI:** Anthropic Claude (primary) — claude-sonnet-4-6 for production, streaming enabled

---

### Coding Conventions
- Comments explain why, not what
- Simple over clever
- TypeScript preferred over JavaScript
- Tests alongside every feature
- Error handling always included

---
*This document reflects the HardcoreAI operating model as of 2026-04-04. Update when the model materially changes.*
