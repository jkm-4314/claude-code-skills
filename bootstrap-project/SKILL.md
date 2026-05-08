---
name: bootstrap-project
description: Use when setting up a new project with the full opinionated workflow — session memory, commit hooks, copyright headers, ROADMAP, and CLAUDE.md. Invoke with "bootstrap project", "setup project", "new project setup", or /bootstrap-project
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Grep
  - Glob
---

# Bootstrap Project

You are setting up a new project with an opinionated standard workflow. This creates the full project infrastructure: session memory files, commit hooks, copyright headers, ROADMAP, and CLAUDE.md.

## Step 1: Pre-Flight Checks

1. Verify this is a git repository (`git rev-parse --git-dir`). If not, ask the user if you should run `git init`.
2. Check which of the expected files already exist:
   - `.claude/SESSION_LOG.md`
   - `.claude/CONTEXT.md`
   - `CLAUDE.md`
   - `ROADMAP.md`
   - `.githooks/commit-msg`
3. If any exist, warn the user and ask before overwriting.

## Step 2: Gather Project Info

Ask the user these questions. If they say "just use defaults" or similar, infer from the repo (`package.json`, `pyproject.toml`, `docker-compose.yml`, etc.) and confirm before writing.

1. **Project name** — e.g., "Acme Widget Manager"
2. **Short identifier** — e.g., `widget-mgr`
3. **One-line description** — what the project is
4. **Tech stack** — e.g., "FastAPI + React + PostgreSQL + Docker"
5. **Project ownership** — ask the user to pick one of their configured ownership profiles, or to define a new one inline. An ownership profile is a `(label, copyright-text | none)` pair. Examples:
   - `Personal` → single-line copyright with the user's legal name
   - `Company` → multi-line copyright with the company name and a confidentiality notice
   - `Client / Government / Open-source` → no copyright header (use this for work-for-hire, public-domain, or permissively-licensed projects)

   If the user has not previously declared their profiles, ask now: "What ownership profiles should I support? For each, give me a label and the copyright text — or `none` for no header." Save the answers for future invocations (e.g., write them to `~/.claude/bootstrap-project.config.md` or note them in the global CLAUDE.md).
6. **Dev environment startup** (optional) — commands to start backend/frontend, ports used

## Step 3: Create `.claude/` Memory Files

Create the `.claude/` directory if it doesn't exist. For each file below, skip if it already exists (unless the user approved overwriting in Step 1).

### `.claude/CONTEXT.md`

```markdown
# {{Project Name}} (`{{short-id}}`) - Project Context

> **Last Updated:** {{MM-DD-YYYY}}
> **Version:** 0.1.0
> **Status:** In development

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

{{How to run the project locally. Commands, ports, prerequisites. Populated from Step 2 answers if provided.}}
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

Populate with the copyright header convention based on the ownership answer from Step 2. See Step 5 for the copyright content to insert.

```markdown
# {{Project Name}} - Code Patterns & Conventions

> **Purpose:** Document established code patterns and conventions so Claude follows them consistently.
> **Last Updated:** {{MM-DD-YYYY}}

---

{{Insert copyright header section from Step 5 here}}

---

_Add additional conventions here as they emerge during development._
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

---

**Created:** {{MM-DD-YYYY}}
**Project:** {{Project Name}} (`{{short-id}}`)
```

## Step 4: Create `ROADMAP.md`

Create in the project root (skip if it already exists):

```markdown
# {{Project Name}} — Roadmap

> **Last Updated:** {{MM-DD-YYYY}}

---

## Current Focus

_Define the current focus area._

---

## Planned Work

### Phase 1: [Name]
- [ ] Task 1
- [ ] Task 2

---

## Recently Completed

_Nothing yet._

---

## Blockers

_None._

---

## Deferred / Won't Do

_Nothing deferred yet._
```

## Step 5: Set Up Copyright Headers

Based on the ownership profile chosen in Step 2, configure copyright headers. **Skip this entire step if the chosen profile has no copyright text** (e.g., open-source, public-domain, work-for-hire).

### 5a: Define the Copyright Notice

Use the copyright text associated with the chosen ownership profile from Step 2. Two common shapes:

**Single-line (typical for personal projects):**

```
Copyright (c) {{year}} {{Copyright Holder Name}}. All rights reserved.
```

**Multi-line (typical for proprietary commercial work):**

```
Copyright (c) {{year}} {{Company Legal Name}}. All rights reserved.
PROPRIETARY AND CONFIDENTIAL. Unauthorized use, copying, or distribution
of this material is strictly prohibited.
```

`{{year}}` is **always the current calendar year at the time the file is created or modified** — not a fixed value from project setup. When creating new files in future years, use that year. When making substantial modifications to an existing file in a new year, update the year.

### 5b: Write the Copyright Section into `.claude/PATTERNS.md`

Insert this section into PATTERNS.md (after the header, before any other content):

```markdown
## Copyright Headers

**Project ownership:** {{ownership profile label from Step 2}}

Every source code file, script, and documentation file must include the copyright header as the first content in the file (after shebangs in scripts).

**Year:** Always use the current calendar year when creating or substantially modifying a file — do not copy a stale year from these examples.

### Comment Syntax by File Type

| File Types | Comment Syntax |
|---|---|
| `.py`, `.sh`, `.bash`, `.rb`, `.yml`, `.yaml`, `.toml`, `.r`, `.R` | `#` per line |
| `.js`, `.ts`, `.tsx`, `.jsx`, `.go`, `.rs`, `.c`, `.cpp`, `.h`, `.java`, `.cs`, `.swift`, `.kt` | `//` per line |
| `.css`, `.scss`, `.less` | `/* ... */` block |
| `.html`, `.xml`, `.svg`, `.vue` | `<!-- ... -->` block |
| `.sql` | `--` per line |
| `.md`, `.mdx` | `<!-- ... -->` block at top of file |
| `.ps1`, `.psm1` | `#` per line |

### Notice Text

{{Insert the appropriate single-line or multi-line notice from 5a}}

### Examples

**Python / Shell / YAML:**
```
# Copyright (c) {{year}} ...
# (additional lines for multi-line notices)
```

**JavaScript / TypeScript / Go / Rust:**
```
// Copyright (c) {{year}} ...
// (additional lines for multi-line notices)
```

**CSS:**
```
/* Copyright (c) {{year}} ...
   (additional lines for multi-line notices) */
```

**HTML / Markdown:**
```
<!-- Copyright (c) {{year}} ...
     (additional lines for multi-line notices) -->
```

**SQL:**
```
-- Copyright (c) {{year}} ...
-- (additional lines for multi-line notices)
```
```

### 5c: Add Existing Files Reminder

After writing PATTERNS.md, tell the user:

> "Copyright headers are configured for new files. Want me to add headers to existing source files now?"

If yes, scan for source files in the repo and add the appropriate header to each. Skip files that already have a copyright header (grep for `Copyright` in the first 5 lines). Skip binary files, lockfiles, and generated files.

If no, move on.

## Step 6: Install commit-msg Hook

Create `.githooks/commit-msg`:

```bash
#!/usr/bin/env bash
# Strip Anthropic co-author attributions from commit messages.
if grep -q 'Co-Authored-By:.*anthropic\.com' "$1"; then
  sed -i.bak '/Co-Authored-By:.*anthropic\.com/d' "$1"
  rm -f "$1.bak"
fi
```

Then run:

```bash
chmod +x .githooks/commit-msg
git config core.hooksPath .githooks
```

If `.githooks/` already exists with other hooks, merge — don't overwrite.

## Step 7: Create or Update `CLAUDE.md`

If `CLAUDE.md` doesn't exist, create it. If it does exist, merge the sections below — don't overwrite existing content.

```markdown
# Claude's Master Reference Document

> **Purpose:** Documentation index and project conventions for Claude.
> **Last Updated:** {{MM-DD-YYYY}}

---

## Session Lifecycle

Session tracking is handled by three skills, all triggered explicitly by the user:

- **`session-start`** — Invoke with "start session", "new session", or `/session-start`. Loads context files, verifies the session log is current, creates a bare stub entry in `.claude/SESSION_LOG.md` if today doesn't already have one, and confirms readiness.
- **`session-update`** — Invoke with "update session", "log progress", "checkpoint session", or `/session-update`, **especially right before `/clear` or handoff**. Captures accomplishments, decisions, WIP, and dead-ends into today's entry while the working Claude still has rich context.
- **`session-end`** — Invoke with "end session", "wrap up", "let's stop here", or `/session-end`. Reconciles against git log, finalizes today's entry, updates housekeeping files, prunes the oldest entry to `SESSION_ARCHIVE.md` when the log exceeds 10 full entries, and asks whether to shut down the application.

**Working outside a formal session is fully supported** — if you don't invoke session-start, no session tracking happens.

---

## Memory System Files

```
.claude/
├── CONTEXT.md          # Project architecture and tech stack
├── DECISIONS.md        # Decision log with rationale (WHY)
├── PATTERNS.md         # Code patterns, conventions, copyright headers
├── MISTAKES.md         # Failed approaches (don't repeat)
├── SESSION_LOG.md      # Last 10 sessions (full) + older summaries
├── SESSION_ARCHIVE.md  # Complete older session entries
└── MEMORY.md           # System documentation
```

---

## Quick Context

**Project:** {{Project Name}}
**Ownership:** {{ownership profile label}}
**Status:** In development
**Tech Stack:** {{tech stack}}

---

## Strict Requirements

1. **Do not suggest session end** — Do not ever suggest that it is time to wrap up or end the session.
2. **No Anthropic attributions** — Do not add `Co-Authored-By` lines referencing `@anthropic.com` in commit messages. The commit-msg hook strips them, but don't add them in the first place.
3. **Copyright headers** — {{If the ownership profile has copyright text: "All new source files, scripts, and documentation must include the copyright header defined in `.claude/PATTERNS.md`." | If the profile has no copyright: "No copyright headers (per project ownership)."}}
4. **Commit message format** — `type(scope): description` where type is `feat`, `fix`, `test`, `docs`, `refactor`, `chore`.

---

{{If dev environment startup info was provided in Step 2:}}
## Development Environment

### Startup
{{startup commands, ports}}

### Shutdown
{{shutdown commands}}

---

_Add project-specific conventions, best practices, and red flags below as they emerge._
```

## Step 8: Verification

Run through this checklist and report results to the user:

| Item | Check |
|------|-------|
| `.claude/CONTEXT.md` | Exists and populated |
| `.claude/SESSION_LOG.md` | Exists |
| `.claude/SESSION_ARCHIVE.md` | Exists |
| `.claude/DECISIONS.md` | Exists |
| `.claude/MISTAKES.md` | Exists |
| `.claude/PATTERNS.md` | Exists, copyright section present (unless ownership profile has no copyright) |
| `.claude/MEMORY.md` | Exists |
| `ROADMAP.md` | Exists in project root |
| `CLAUDE.md` | Exists with session lifecycle section |
| `.githooks/commit-msg` | Exists and executable |
| `core.hooksPath` | Set to `.githooks` |

Report:

> "Project bootstrapped: {{Project Name}} ({{ownership}})
> - {{N}} memory files created
> - ROADMAP.md created
> - CLAUDE.md {{created | updated}}
> - commit-msg hook installed
> - {{Copyright headers configured for [owner] | No copyright (per ownership profile)}}
>
> Run `/session-start` to begin the first tracked session."

## Quick Reference

| Situation | Action |
|-----------|--------|
| Not a git repo | Offer to `git init` |
| Files already exist | Warn, ask before overwriting |
| User says "just use defaults" | Infer from repo files, confirm |
| Ownership profile has no copyright | Skip copyright headers entirely |
| Single-name owner profile | Single-line copyright with the configured holder name |
| Company / proprietary profile | Multi-line copyright with confidentiality notice |
| Existing source files need headers | Offer to add after setup |
| `.githooks/` has existing hooks | Merge, don't overwrite |
| Dev environment info not provided | Skip startup/shutdown in CLAUDE.md |
