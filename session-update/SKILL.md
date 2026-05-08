---
name: session-update
description: Use when the user says "update session", "log progress", "checkpoint session", or invokes /session-update — especially before /clear or handoff
allowed-tools:
  - Bash
  - Read
  - Edit
  - Grep
  - Glob
---

# Session Update (Checkpoint)

You are capturing work-in-progress into today's SESSION_LOG.md entry **before context is cleared or handed off**. The goal: preserve what THIS Claude instance knows — decisions, reasoning, blockers, dead-end approaches — before it's lost. Git alone can't recover any of that.

Primary use case: run this right before `/clear`. Secondary: mid-session checkpoint after a chunk of work.

## Step 1: Verify Today's Entry Exists

Read `.claude/SESSION_LOG.md`. Find the top entry.

**If top entry's date == today** (MM-DD-YYYY format; legacy YYYY-MM-DD is equivalent): continue to Step 2.

**If no today entry:** tell the user:

> "No session-start stub exists for today. Run `/session-start` first to create today's entry, then re-run session-update."

Stop. Do not create the entry here — that's session-start's job.

## Step 2: Determine Scope of "New Work"

Look inside today's entry for a tracking marker of the form:

```html
<!-- last-update: <git-hash> -->
```

- **If present:** new work = commits since `<git-hash>` + current uncommitted changes + this conversation's work not yet in the entry.
- **If absent:** new work = commits since today's start that aren't obviously reflected in the entry + current uncommitted changes + this conversation's work.

Collect inputs:

```bash
# Current HEAD (for the new marker)
git rev-parse --short HEAD

# Commits since last marker (if marker exists):
git log --oneline --all <marker-hash>..HEAD

# Or commits since today's start (if no marker):
git log --oneline --all --since="6am"

# Uncommitted state
git status --short
git diff --stat
```

Also mine **this conversation's context** for things git won't show:
- Decisions made and *why* (especially alternatives rejected)
- Approaches tried that didn't work (dead ends)
- Blockers discovered
- WIP reasoning — what was about to happen next

## Step 3: Draft Section Additions

Draft bullets/rows for these four sections (merge into whatever already exists in the entry — don't replace):

### `### What Was Accomplished`
- Numbered item per distinct accomplishment since last update
- 1 line prose + short rationale if the "why" isn't obvious from the code

### `### Key Decisions`
- Decisions not encoded in commits (why X, not Y)
- Tradeoffs considered, options rejected

### `### Files Modified`
- Rows for files touched since last update (use `git diff --name-only <marker>..HEAD` + uncommitted)

### `### Notes for Next Session`
- **WIP:** uncommitted work — current state, what was about to happen next
- **Blockers:** issues discovered but not yet resolved
- **Dead ends:** approaches tried that didn't work, so the next Claude doesn't retry them

## Step 4: Handle Uncommitted Work

If `git diff --stat` shows changes, summarize them to the user:

> "Uncommitted changes:
>  - `path/to/file.py` (+12 / -3)
>  - ...
>
> Capture as WIP in Notes for Next Session?"

- **Yes:** add a WIP bullet describing the state and intent.
- **No:** skip (user is mid-refactor and doesn't want noise).

Never commit the changes from this skill — WIP captures are descriptive only.

## Step 5: Present Draft and Confirm

Show the user the drafted additions, grouped by section. Ask:

> "Apply these updates to Session NNN? You can edit, add anything I missed, or accept as-is."

Wait for confirmation or edits. If they edit, incorporate and re-confirm.

## Step 6: Merge Into Existing Entry

Read today's entry. Apply the confirmed additions using these rules:

**DO NOT touch:**
- The `## MM-DD-YYYY - Session NNN: [In Progress]` title line
- The `**Focus:** _In progress — finalize at session end_` line

Those are session-end's job to finalize.

**DO:**
- Insert or update the tracking marker directly under the Focus line:
  ```markdown
  **Focus:** _In progress — finalize at session end_

  <!-- last-update: <new-HEAD-hash> -->
  ```
- For existing sections: append new items/rows (continue numbered lists from the last number).
- For missing sections: create them with the new content.
- Preserve the trailing `---` separator before the next session entry.

Example — entry after first update:

```markdown
## 04-13-2026 - Session 115: [In Progress]

**Focus:** _In progress — finalize at session end_

<!-- last-update: a1b2c3d -->

### What Was Accomplished

1. **Thing one** — one-line description

### Key Decisions
- Decision — rationale

### Files Modified

| File | Changes |
|------|---------|
| `api/app/foo.py` | Added bar handler |

### Notes for Next Session
- **WIP:** About to refactor `baz()` — paused here.

---
```

## Step 7: Confirm

Tell the user:

> "Session NNN updated through commit `<hash>`. Captured N accomplishments / M decisions / K files / [WIP: yes|no]. Safe to `/clear`."

## Quick Reference

| Situation | Action |
|-----------|--------|
| No today entry in SESSION_LOG.md | Tell user to run `/session-start` first |
| `<!-- last-update: hash -->` present | Scope new work to commits after that hash |
| No marker | Scope new work to today's commits + conversation |
| Uncommitted changes exist | Offer to capture as WIP |
| Called right before `/clear` | Primary use case — prioritize decisions + WIP + dead ends |
| Multi-agent day | Each Claude instance updates → marker advances → no double-counting |
