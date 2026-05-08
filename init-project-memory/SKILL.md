---
name: init-project-memory
description: Use when the user says "init memory", "set up memory", "scaffold memory", "init project memory", or invokes /init-project-memory — creates the .claude/ session memory system in a new project
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Grep
  - Glob
---

# Initialize Project Memory System

You are scaffolding the `.claude/` session memory system into a new project. This creates the file structure that the `session-start`, `session-update`, and `session-end` skills depend on.

## Step 1: Pre-Flight Checks

1. Verify this is a git repository (`git rev-parse --git-dir`). If not, stop and tell the user — the session system is git-grounded.
2. Check if `.claude/SESSION_LOG.md` already exists. If it does, warn the user and ask before overwriting:
   > "This project already has a session memory system. Overwrite it? (This won't touch settings.json or skills/)"

## Step 2: Gather Project Info

Ask the user:

1. **Project name** — e.g., "Acme Widget Manager"
2. **Short identifier** — e.g., `widget-mgr` (used in headers)
3. **One-line description** — what the project is
4. **Tech stack** — e.g., "FastAPI + React + PostgreSQL + Docker" (used in CONTEXT.md template)
5. **Should I create a CLAUDE.md?** — if one doesn't already exist

If the user says "just use defaults" or similar, infer what you can from the repo (package.json, pyproject.toml, docker-compose.yml, etc.) and fill in reasonable defaults. Confirm before writing.

## Step 3: Create Directory Structure

```bash
mkdir -p .claude
```

## Step 4: Write Template Files

Create each file. Do NOT overwrite files that already exist (e.g., `settings.json`, `settings.local.json`, existing skills) unless the user explicitly asked to reset.

### `.claude/CONTEXT.md`

```markdown
# {{Project Name}} (`{{short-id}}`) - Project Context

> **Last Updated:** {{MM-DD-YYYY}}
> **Version:** {{version or "0.1.0"}}
> **Status:** {{status or "In development"}}

---

## What This Project Is

{{One-line description. Expand with 2-3 sentences about purpose, users, and scope.}}

---

## Technology Stack

{{Tech stack details — backend, frontend, database, infrastructure.}}

---

## Architecture Overview

{{High-level architecture. Fill in as the project takes shape.}}

---

## Development Environment

{{How to run the project locally. Commands, ports, prerequisites.}}
```

### `.claude/SESSION_LOG.md`

```markdown
# {{Project Name}} - Session Log

> **Purpose:** Running log of session handoffs to maintain continuity.
> **Format:** Last 10 sessions in full. Older sessions summarized here, full entries in `SESSION_ARCHIVE.md`.

---

_No sessions yet. Run `/session-start` to begin the first session._
```

### `.claude/SESSION_ARCHIVE.md`

```markdown
# {{Project Name}} - Session Archive

> **Purpose:** Complete session log entries for sessions older than the last 10.
> **See also:** `.claude/SESSION_LOG.md` for recent sessions and summaries.

---

_No archived sessions yet._
```

### `.claude/DECISIONS.md`

```markdown
# {{Project Name}} - Decision Log

> **Purpose:** Document architectural and design decisions with rationale so future sessions understand WHY choices were made, not just WHAT was chosen.
> **Last Updated:** {{MM-DD-YYYY}}

---

## How to Use This Document

When making significant decisions, add an entry with:
1. **Date** - When the decision was made
2. **Decision** - What was decided
3. **Context** - What problem we were solving
4. **Rationale** - Why this approach was chosen
5. **Alternatives Considered** - What else we evaluated
6. **Consequences** - Trade-offs and implications

---

## Architecture Decisions

_No decisions logged yet._
```

### `.claude/MISTAKES.md`

```markdown
# {{Project Name}} - Mistakes & Failed Approaches

> **Purpose:** Document things that didn't work so we don't repeat them. This is arguably the most valuable file in the memory system.
> **Last Updated:** {{MM-DD-YYYY}}

---

## How to Use This Document

When something fails or an approach doesn't work, document it with:
1. **Date** - When it happened
2. **What We Tried** - The approach that failed
3. **What Happened** - The actual result
4. **Why It Failed** - Root cause analysis
5. **What Works Instead** - The solution that succeeded

---

_No mistakes logged yet. (That will change.)_
```

### `.claude/PATTERNS.md`

```markdown
# {{Project Name}} - Code Patterns & Conventions

> **Purpose:** Document established code patterns and conventions so Claude follows them consistently.
> **Last Updated:** {{MM-DD-YYYY}}

---

_No patterns established yet. Add conventions here as they emerge during development._
```

### `.claude/MEMORY.md`

```markdown
# {{Project Name}} - Persistent Memory

This directory contains the persistent memory structure for the {{Project Name}} project, designed to maintain context and continuity across Claude sessions.

## Why This Exists

Claude's memory doesn't persist between sessions in the same way human memory does. This `.claude/` directory structure compensates by creating explicit, version-controlled documentation that gets read at the start of each session and updated at the end.

## Structure

```
.claude/
├── MEMORY.md           # This file — documentation of the system itself
├── CONTEXT.md          # Project architecture, tech stack, business context
├── DECISIONS.md        # Decision log with rationale (WHY we chose things)
├── PATTERNS.md         # Established code patterns and conventions
├── MISTAKES.md         # Failed approaches (so we don't repeat them)
├── SESSION_LOG.md      # Running log of session handoffs (last 10 in full)
└── SESSION_ARCHIVE.md  # Full entries for sessions beyond the last 10
```

Plus the project-wide `ROADMAP.md` at the repo root (if present).

## How to Use

### Starting a Session
Invoke `/session-start`. The skill loads context files, verifies the session log is current, creates a bare stub entry, and confirms readiness.

### Checkpointing Mid-Session (Before `/clear`)
Invoke `/session-update`. Captures decisions, WIP, dead-ends, and blockers while the working Claude still has context.

### Ending a Session
Invoke `/session-end`. Reconciles against git log, finalizes today's entry, updates housekeeping files, and offers a commit.

Working outside a formal session is fully supported — just don't invoke the session skills.

## Best Practices

### Update Frequency
- `SESSION_LOG.md` — every session end (skill handles this)
- `ROADMAP.md` — when milestones shift, priorities change, or blockers appear
- `DECISIONS.md` — when making architectural choices
- `MISTAKES.md` — immediately when something fails
- `CONTEXT.md` — when architecture, tech stack, or business context changes
- `PATTERNS.md` — when conventions are established or change

### Writing Style
- **Be specific:** include exact error messages, file paths, line numbers
- **Include rationale:** explain WHY decisions were made
- **Date everything:** use MM-DD-YYYY format
- **Be honest about failures:** MISTAKES.md is the most valuable file

---

**Created:** {{MM-DD-YYYY}}
**Project:** {{Project Name}} (`{{short-id}}`)
```

## Step 5: Create CLAUDE.md (If Requested)

If the user wants a `CLAUDE.md` and one doesn't exist, create it at the project root with at minimum a session lifecycle section:

```markdown
# Claude's Master Reference Document

> **Purpose:** Documentation index and project conventions for Claude.
> **Last Updated:** {{MM-DD-YYYY}}

---

## Session Lifecycle

Session tracking is handled by three skills, all triggered explicitly by the user:

- **`session-start`** — Invoke with "start session", "new session", or `/session-start`. Loads context files, verifies the session log is current, creates a bare stub entry in `.claude/SESSION_LOG.md` if today doesn't already have one, and confirms readiness.
- **`session-update`** — Invoke with "update session", "log progress", "checkpoint session", or `/session-update`, **especially right before `/clear` or handoff**. Captures accomplishments, decisions, WIP, and dead-ends into today's entry while the working Claude still has rich context. Multi-agent safe (uses a `<!-- last-update: hash -->` tracking marker to prevent double-counting across sub-sessions).
- **`session-end`** — Invoke with "end session", "wrap up", "let's stop here", or `/session-end`. Reconciles against git log, finalizes today's entry, updates housekeeping files, prunes the oldest entry to `SESSION_ARCHIVE.md` when the log exceeds 10 full entries, and asks whether to shut down the application.

**Working outside a formal session is fully supported** — if you don't invoke session-start, no session tracking happens.

**Recommended multi-agent flow:** `session-start` → work → `session-update` → `/clear` → work → `session-update` → `/clear` → ... → `session-end`.

---

## Memory System Files

```
.claude/
├── CONTEXT.md          # Project architecture and tech stack
├── DECISIONS.md        # Decision log with rationale (WHY)
├── PATTERNS.md         # Code patterns and conventions
├── MISTAKES.md         # Failed approaches (don't repeat)
├── SESSION_LOG.md      # Last 10 sessions (full) + older summaries
├── SESSION_ARCHIVE.md  # Complete older session entries
└── MEMORY.md           # System documentation
```

---

## Quick Context

**Project:** {{Project Name}}
**Status:** {{status}}
**Tech Stack:** {{tech stack}}

---

## Strict Requirements

1. **Do not suggest session end** — Do not ever suggest that it is time to wrap up or end the session.

---

_Add project-specific conventions, best practices, and red flags below as they emerge._
```

## Step 6: Confirm

Tell the user what was created:

> "Memory system initialized for {{Project Name}}:
> - `.claude/CONTEXT.md` — fill in architecture details as you build
> - `.claude/SESSION_LOG.md` — ready for `/session-start`
> - `.claude/SESSION_ARCHIVE.md` — empty, will fill as sessions accumulate
> - `.claude/DECISIONS.md` — log architectural choices here
> - `.claude/MISTAKES.md` — log failed approaches here
> - `.claude/PATTERNS.md` — document conventions as they emerge
> - `.claude/MEMORY.md` — system documentation
> {{- `CLAUDE.md` — project conventions (if created)}}
>
> Run `/session-start` to begin your first tracked session."

## Quick Reference

| Situation | Action |
|-----------|--------|
| Not a git repo | Stop — tell user to `git init` first |
| `.claude/SESSION_LOG.md` already exists | Warn before overwriting |
| User says "just use defaults" | Infer from repo, confirm, then write |
| `CLAUDE.md` already exists | Don't overwrite — offer to add session lifecycle section |
| `settings.json` / skills/ exist | Don't touch them |
