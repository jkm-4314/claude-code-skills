# claude-code-skills

A small collection of opinionated [Claude Code](https://docs.claude.com/claude-code) skills for project bootstrapping, persistent session memory, and peer code review.

These skills are designed to work as a system, but each one is independently useful — install only the ones you want.

## What's here

| Skill | What it does |
|---|---|
| [`bootstrap-project`](./bootstrap-project) | One-command setup for a new project: scaffolds session memory, ROADMAP, CLAUDE.md, copyright headers, and a commit-msg hook. Asks for your ownership/copyright preferences once and applies them. |
| [`init-project-memory`](./init-project-memory) | Lighter-weight version of the bootstrap — just creates the `.claude/` memory file structure (CONTEXT, DECISIONS, MISTAKES, PATTERNS, SESSION_LOG, etc.). Use this if you don't want the full bootstrap. |
| [`session-start`](./session-start) | Loads context, verifies the session log, creates a stub entry for today, optionally starts your dev environment with port-conflict detection. |
| [`session-update`](./session-update) | Mid-session checkpoint — captures decisions, WIP, dead-ends, and blockers into today's log entry **before `/clear` or handoff**. Multi-agent safe via a tracking marker. |
| [`session-end`](./session-end) | Finalizes today's session: reconciles against git log, prunes the log to keep the last 10 in full, updates ROADMAP / DECISIONS / MISTAKES, optionally shuts down the dev environment. |
| [`peer-review`](./peer-review) | Adversarial spec/code review loop between Claude (this skill) and OpenAI's Codex CLI (GPT-5.5). Iterates until both models agree the work is production-ready, with a 5-round circuit breaker. Requires the [Codex CLI](https://github.com/openai/codex). |

## How they fit together

```
  ┌─────────────────────┐
  │ bootstrap-project   │  one-time, per project
  │   ─ creates ─→      │
  └──────────┬──────────┘
             │
             ▼
  ┌─────────────────────────────────────────────────────────┐
  │ .claude/                                                 │
  │   CONTEXT.md  DECISIONS.md  PATTERNS.md  MISTAKES.md    │
  │   SESSION_LOG.md  SESSION_ARCHIVE.md  MEMORY.md         │
  │ CLAUDE.md   ROADMAP.md   .githooks/commit-msg            │
  └────────────────────────┬────────────────────────────────┘
                           │
            ┌──────────────┼──────────────┐
            ▼              ▼              ▼
       session-start → session-update → session-end
                     (run before /clear)
```

`peer-review` is independent — install it on its own if you don't want the rest.

## Installation

### Prerequisites
- [Claude Code](https://docs.claude.com/claude-code) installed
- For `peer-review` only: [Codex CLI](https://github.com/openai/codex) `>= 0.125.0` in `PATH`, authenticated against an OpenAI account that can call `gpt-5.5`

### Install all skills

**Linux / macOS:**

```bash
git clone https://github.com/jkm-4314/claude-code-skills.git
cd claude-code-skills
mkdir -p ~/.claude/skills
cp -r bootstrap-project init-project-memory session-start session-update session-end peer-review ~/.claude/skills/
```

**Windows (PowerShell):**

```powershell
git clone https://github.com/jkm-4314/claude-code-skills.git
cd claude-code-skills
$dst = "$env:USERPROFILE\.claude\skills"
New-Item -ItemType Directory -Force -Path $dst | Out-Null
Copy-Item -Recurse -Force bootstrap-project,init-project-memory,session-start,session-update,session-end,peer-review $dst
```

Then restart Claude Code (or start a fresh session) so the skills are discovered.

### Install one skill

Each skill is a self-contained `<name>/SKILL.md`. Just drop the directory into `~/.claude/skills/` (or `%USERPROFILE%\.claude\skills\` on Windows).

## Usage

Skills are invoked by phrase or slash-command. The session lifecycle in particular is meant to be triggered explicitly by you — none of these skills auto-fire on every conversation.

| Skill | Invocation phrases | Slash command |
|---|---|---|
| `bootstrap-project` | "bootstrap project", "setup project" | `/bootstrap-project` |
| `init-project-memory` | "init memory", "set up memory", "scaffold memory" | `/init-project-memory` |
| `session-start` | "start session", "new session", "begin session" | `/session-start` |
| `session-update` | "update session", "log progress", "checkpoint session" | `/session-update` |
| `session-end` | "end session", "wrap up", "let's stop here" | `/session-end` |
| `peer-review` | "review this spec", "review this code" (auto-triggers after spec/code completion) | `/peer-review spec` or `/peer-review code` |

## Customization

Each skill is a single readable Markdown file with inline templates and prompts — edit them directly to match your workflow:

- Don't want a commit-msg hook? Delete Step 6 from `bootstrap-project/SKILL.md`.
- Want different ownership profiles or copyright text? Edit Step 2 / Step 5 of `bootstrap-project/SKILL.md`.
- Want to swap Codex for a different review model? Edit the `codex exec` invocation in `peer-review/SKILL.md` Step 3.
- Want stricter / looser review prompts? Edit `SPEC_PROMPT` / `CODE_PROMPT` at the bottom of `peer-review/SKILL.md`.

## Design notes

- **Git-grounded.** Session-end reconciles its log against `git log` rather than trusting in-conversation memory alone. Same for session-update. The session log lives in version control and is meant to survive context clears.
- **No auto-firing.** Session skills only run when you ask. Working informally without a tracked session is fully supported.
- **Multi-agent safe.** Session-update uses a `<!-- last-update: <hash> -->` marker so multiple Claude instances working through the day don't double-count each other's work.
- **Fail-closed.** Peer-review treats ambiguous Codex output as `NO-GO`, validates refs against a safe charset before substituting into shell, and passes prompts via stdin (not interpolated argv).

## License

MIT — see [`LICENSE`](./LICENSE). Use, modify, fork, redistribute. No warranty.

## Contributing

Issues and PRs welcome. The skills are deliberately small and Markdown-only — keep changes focused on a single skill, and explain in the PR what behavior changes for users.
