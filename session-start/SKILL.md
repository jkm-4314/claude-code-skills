---
name: session-start
description: Use when the user says "start session", "new session", "begin session", or invokes /session-start
allowed-tools:
  - Bash
  - Read
  - Edit
  - Grep
  - Glob
---

# Session Start Protocol

You are initializing a work session. Follow these steps.

**Scope note:** Do not auto-fire this skill on every new Claude session. It runs only when the user explicitly invokes it. Working outside a formal session is fully supported — if the user doesn't invoke this skill, skip session-tracking entirely.

**Override note:** This is the generic global version. If this project has a project-level `session-start` skill (in `.claude/skills/session-start/`), that version runs instead of this one and may include project-specific app startup logic.

## Step 1: Load Context

Read these files in order (skip any that don't exist):

1. **`CLAUDE.md`** — Project conventions and session lifecycle (if present)
2. **`.claude/CONTEXT.md`** — Project architecture, tech stack, current state
3. **`.claude/SESSION_LOG.md`** — Recent sessions (top 2-3 entries are enough)
4. **`ROADMAP.md`** — Progress tracking (skim "Recent Updates" section)
5. **`PATTERNS.md`** or **`.claude/PATTERNS.md`** — Code conventions

Reference as needed:
- `.claude/DECISIONS.md` — Architectural decisions with rationale
- `.claude/MISTAKES.md` — Failed approaches (check before trying something new)

## Step 2: Verify Log is Current

Run `git log --oneline -5` and cross-reference against the most recent SESSION_LOG.md entry. Flag any commits not reflected in the log — they may need to be folded into today's entry at session end.

## Step 3: Check for Today's Session Entry

Read the top of `.claude/SESSION_LOG.md`.

**Dates use MM-DD-YYYY format** (e.g., `04-13-2026`, not `2026-04-13`).

**If the top entry's date matches today's date** (e.g., today is 04-13-2026 and the top entry starts with `## 04-13-2026 - Session NNN:`):
- Do NOT create a new entry. Today's session is a continuation of an existing entry (same-day multi-agent / context-cleared work appends to one entry per day).
- Note the existing session number for the Step 6 confirmation.
- Skip Step 4; continue to Step 5.

**If the top entry's date is before today:**
- Determine today's session number by incrementing the most recent session number.
- Continue to Step 4.

Note on legacy entries: older entries in SESSION_LOG.md may use YYYY-MM-DD format. Treat them as equivalent when matching dates, but always write new entries in MM-DD-YYYY.

## Step 4: Create Bare Stub Entry

Insert a new entry at the top of `.claude/SESSION_LOG.md` — after the header/front-matter (the `---` separator on line 6), before the previous top entry.

Use exactly this format:

```markdown
## MM-DD-YYYY - Session NNN: [In Progress]

**Focus:** _In progress — finalize at session end_

---

```

- `MM-DD-YYYY` — today's date (e.g., `04-13-2026`)
- `NNN` — previous top session number + 1

Do not pre-populate accomplishments, decisions, or files modified. The session-end skill fills these in at the end of the day against git history and user input.

## Step 5: Check Ports and Start the Application

If `CLAUDE.md` documents a dev environment startup procedure, check for port conflicts before starting services.

### 5a: Scan for Port Conflicts

Read `CLAUDE.md` to identify the ports the project uses (dev servers, databases, etc.). Then check which of those ports are already bound:

- **Linux/macOS:** `ss -tlnp 2>/dev/null || netstat -tlnp 2>/dev/null` — look for LISTEN entries on project ports
- **Windows (bash):** `netstat -ano | grep LISTENING` — filter for project ports; resolve PIDs with `tasklist /FI "PID eq <pid>"` if needed

### 5b: Resolve Conflicts

**If a project port is already in use:**

1. **Same service already running** (e.g., uvicorn already on 8000) — skip starting that service; note it in the Step 6 report.
2. **Different process on a project port** — report the conflict to the user with the PID and process name. Suggest either killing the conflicting process or starting the project service on an alternate port (e.g., `--port 8003`). Do not kill processes without user confirmation.

**If no conflicts** — proceed normally.

### 5c: Start Services

Follow the startup instructions from `CLAUDE.md` to verify the app is running and start it if needed, skipping any services already confirmed running in 5b.

If no startup instructions exist in `CLAUDE.md`, skip all of Step 5.

## Step 6: Confirm to the User

Report in one message:

- **Session:** `Session NNN started for MM-DD-YYYY` — or, if continuing: `Continuing today's Session NNN`
- **App status:** Report backend/frontend state if checked in Step 5; omit if Step 5 was skipped

Then: `Ready to work.`

## Quick Reference

| Condition | Action |
|-----------|--------|
| Top entry date == today | Acknowledge, continue existing entry |
| Top entry date < today | Create new bare stub (Session N+1) |
| User didn't invoke this skill | Don't run it — informal work is fine |
| CLAUDE.md has startup instructions | Check ports first, then follow them |
| No startup instructions | Skip app startup and port check |
| Port in use by same service | Skip starting that service, note in report |
| Port in use by different process | Report conflict, ask user before killing |
