---
name: session-end
description: Use when the user says "end session", "wrap up", "let's stop here", or invokes /session-end
allowed-tools:
  - Bash
  - Read
  - Edit
  - Grep
  - Glob
---

# Session End Handoff

You are finalizing a work session to preserve context for future sessions. Follow these steps.

**Override note:** This is the generic global version. If this project has a project-level `session-end` skill (in `.claude/skills/session-end/`), that version runs instead of this one and may include project-specific app shutdown logic.

## Step 1: Git-Grounded Reconciliation

Run:

```bash
git log --since="6am" --oneline --all
git diff --stat
```

Cross-reference commits against this Claude session's own work. Flag any commits that THIS session didn't make (other agents, manual commits, or earlier context-cleared sub-sessions). For unfamiliar commits, briefly read the changed files so you can describe them accurately.

## Step 2: Detect Today's Session Entry

Read the top of `.claude/SESSION_LOG.md`.

**If the top entry's date matches today's date** (stub from session-start, or an already-finalized entry being appended to):
- Continue to Step 3 to finalize / augment it.

**If no entry exists for today** (user worked informally without running session-start):
- Ask: "No session-start entry exists for today. Options:
  1. Create a retroactive entry for today's work
  2. Skip log creation — just run housekeeping (ROADMAP / DECISIONS / MISTAKES / pruning)
  3. Do nothing"
- If they pick (1): create a new entry at the top with today's date and next session number, then continue to Step 3.
- If they pick (2): jump to Step 5.
- If they pick (3): stop.

## Step 3: Finalize Today's Entry

Check today's entry for a `<!-- last-update: <hash> -->` marker. Its presence signals that `session-update` has been run during the day and the entry already has substantive content — in that case, session-end is a lightweight finalization. Its absence means you're reconstructing from scratch against git history.

### Path A: Entry has been updated during the day (marker present OR sections populated)

Most content is already captured. Focus on:

1. "What title best describes today's session?" (replaces `[In Progress]`)
2. "One-line focus for the session?"
3. Quick cross-check against git reconciliation:
   - Compare the commits from Step 1 against `### Files Modified` and `### What Was Accomplished`. Anything missing? Ask the user briefly.
4. "Any final decisions, blockers, or notes to add?"
5. Remove the `<!-- last-update: ... -->` marker — the session is being finalized, marker no longer needed.

### Path B: Entry is a bare stub (no marker, no section content)

Full reconstruction. Prompt the user:

1. "What title best describes today's session?" (replaces `[In Progress]`)
2. "One-line focus for the session?"
3. Walk through the git reconciliation output with them:
   - "These commits happened today: [list]. Are all of these reflected in the entry? Anything missing?"
   - Probe for work from other agents or context-cleared sub-sessions that isn't documented.
4. "Any key decisions or discoveries to capture?"
5. "Any blockers or notes for next session?"

> **Tip for future sessions:** if this felt like reconstruction, suggest running `/session-update` before `/clear` next time — it captures decisions and WIP while the working Claude still has context.

Expand the entry to the full format. **Dates use MM-DD-YYYY format** (e.g., `04-13-2026`):

```markdown
## MM-DD-YYYY - Session NNN: [Descriptive Title]

**Focus:** [One-line description of session focus]

### What Was Accomplished

1. **[Feature/Fix Name]** — [Description of what was done]
2. **[Feature/Fix Name]** — [Description]

### Key Decisions
- [Decision] — [Why this approach was chosen]

### Files Modified

| File | Changes |
|------|---------|
| `path/to/file.py` | [Brief description] |

### Notes for Next Session
- [Anything the next session needs to know]

---
```

## Step 4: Prune Oldest Entry (Size Management)

Count the full entries (not one-line summaries) in `.claude/SESSION_LOG.md`.

**If there are more than 10 full entries:**

1. Read the oldest full entry (bottom of the full-entry section).
2. Append its complete text to `.claude/SESSION_ARCHIVE.md`.
3. In `.claude/SESSION_LOG.md`, replace the full entry with a one-line summary (MM-DD-YYYY format):
   ```
   ## MM-DD-YYYY - Session NNN: [Title]
   ```

Note: older archived entries may use YYYY-MM-DD; treat them as equivalent for matching, but always write new summaries in MM-DD-YYYY.

This keeps SESSION_LOG.md scannable while preserving full history in the archive.

## Step 5: Update ROADMAP.md (If Applicable)

Check if any of these apply:
- Major feature completed → move to "Recently Completed"
- Metrics changed significantly → update numbers
- New milestone reached → document
- Completed items → move to `COMPLETED_FEATURES.md`
- New blockers discovered → add to Blockers section
- Priorities changed → reorganize

Skip if no meaningful changes.

## Step 6: Update DECISIONS.md (If Applicable)

If architectural decisions were made this session, add to `.claude/DECISIONS.md`:
- What decision was made
- Why this approach was chosen
- What alternatives were considered

Determine if a formal ADR is warranted.

## Step 7: Update MISTAKES.md (If Applicable)

If something didn't work as expected, add to `.claude/MISTAKES.md`:
- What was attempted
- Why it failed
- What to try instead

## Step 8: Final Verification

Run `git status` to show any uncommitted documentation changes. Ask the user if they want to commit:

```bash
git add .claude/ ROADMAP.md
git commit -m "[session] Session NNN wrap-up"
```

## Step 9: Shut Down Application (Optional)

If the application is running (dev servers, Docker containers, etc.), ask the user:

**"Shut down the application, or leave it running?"**

If `CLAUDE.md` documents shutdown commands, follow those. Otherwise, skip this step.

## Quick Reference

| Priority | File | When |
|----------|------|------|
| 1 | `.claude/SESSION_LOG.md` | Every session-end |
| 1 | Prune → `SESSION_ARCHIVE.md` | When >10 full entries |
| 2 | `ROADMAP.md` | When milestones / metrics / priorities change |
| 3 | `.claude/DECISIONS.md` | When architectural decisions made |
| 3 | `.claude/MISTAKES.md` | When something failed |
| — | Application shutdown | Ask if app is running; defer to CLAUDE.md for commands |

**Key change from older versions:** Session-end no longer creates entries from scratch — that's session-start's job. Session-end finalizes / appends to today's existing entry, or (on request) creates one retroactively. When `session-update` has been run during the day, finalization is quick — just title, focus, gap-check.
