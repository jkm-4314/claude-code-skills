# peer-review

> 📦 Also part of [claude-code-skills](https://github.com/jkm-4314/claude-code-skills) — a small collection of opinionated Claude Code skills (project bootstrap, persistent session memory, and this peer review). Clone the repo for the full set.

A [Claude Code](https://docs.claude.com/claude-code) skill that orchestrates **iterative adversarial review** of specs or code between Claude and OpenAI's Codex (GPT-5.5) until both models independently agree the work is production-ready.

## Why

Claude and Codex catch different classes of issues. Running them as adversarial reviewers — each given the same artifact and the other's feedback — surfaces problems that either model would miss alone: missed edge cases, security gaps, ambiguous requirements, regressions introduced by remediation. The loop terminates when both models reach `GO`, or after 5 rounds (Claude's position prevails to avoid blocking work indefinitely).

## What it does

- **Spec mode:** Reviews a specification document for completeness, security, stability, correctness, implementability, and testability.
- **Code mode:** Reviews a git diff (feature branch vs. default branch, or uncommitted changes) for security, correctness, stability, performance, maintainability, and test coverage.
- Iterates: Codex returns `GO` / `NO-GO` with severity-tagged findings, Claude remediates (or disputes), and the next round begins.
- Logs disagreements to Claude Code's project auto-memory if the loop hits the 5-round circuit breaker.

## Installation

Requires:
- Claude Code
- [Codex CLI](https://github.com/openai/codex) `>= 0.125.0` in `PATH`, authenticated for `gpt-5.5`

Install:

```bash
# Linux / macOS
mkdir -p ~/.claude/skills/peer-review
curl -o ~/.claude/skills/peer-review/SKILL.md \
  https://gist.githubusercontent.com/jkm-4314/58e046af18a836c52680d8cf083fec05/raw/SKILL.md
```

```powershell
# Windows
$dst = "$env:USERPROFILE\.claude\skills\peer-review"
New-Item -ItemType Directory -Force -Path $dst | Out-Null
Invoke-WebRequest -OutFile "$dst\SKILL.md" `
  -Uri "https://gist.githubusercontent.com/jkm-4314/58e046af18a836c52680d8cf083fec05/raw/SKILL.md"
```

Then restart Claude Code (or start a fresh session) so the skill is discovered.

## Usage

Explicit invocation:

```
/peer-review spec
/peer-review code
```

Automatic invocation: the skill triggers after spec writing or code completion in production projects unless the user opts out (e.g., "skip review", "no review").

## Safety

- Codex runs with `-s read-only` and `--ignore-user-config` — it can read the project but cannot write or shell out.
- Prompts are passed via stdin, never interpolated into shell commands.
- Refs are validated against `[a-zA-Z0-9_./-]` before substitution.
- The skill warns and stops if it detects likely secrets in the diff (`API_KEY=`, `-----BEGIN`, `password:` patterns).
- Temp review logs live in `$TMPDIR` and are deleted on every exit path.

## Customization

The model, reasoning effort, and review prompts are inline in `SKILL.md` — edit them to swap models, adjust rigor, or change review criteria. The 5-round circuit breaker is also adjustable.

## License

MIT — use, modify, share. No warranty.
