# agents-md

My personal [AGENTS.md](https://agents.md/) — a coding philosophy file that works across all major AI coding agents.

## What's in here

A single `AGENTS.md` file that translates my coding principles into instructions an AI agent can actually follow:

- **Verification Protocol** — exact commands to run before declaring a task done (test, typecheck, lint, build). Trust but verify.
- **Agent Behaviour** — when to ask for help, when to stop, small steps, spec before code, bounded scope.
- **Ports & Adapters** — default architecture. Domain has zero external imports. Every side effect goes through a port.
- **FP over OOP** — pure functions, immutable data, composition over inheritance, Result types instead of exceptions.
- **SOLID (always SRP)** — every module has one reason to change. No exceptions.
- **Clean Code** — functions under 25 lines, files under 250, intention-revealing names, no dead code, no speculative abstractions.
- **Twelve Factor** — config in env vars, stateless processes, logs to stdout, dev/prod parity.
- **XP Practices** — test-first, refactor after green, root-cause every bug (Five Whys), small commits, small PRs, incremental design, pair on hard things.
- **Security** — never commit secrets, validate inputs, review dependencies, prevent XSS and SQL injection.
- **Known Pitfalls** — recurring mistakes accumulated over time. Add to this section when you learn something new.
- **Living Document** — update the file when you learn something new. Rules, not philosophy. Every line actionable and verifiable.

## Works everywhere

AGENTS.md is the single source of truth. Bridge files point every major coding agent at it:

| File | Tool | Method |
|---|---|---|
| `AGENTS.md` | Codex, Cursor, Windsurf, Cline, Copilot coding agent | Read natively |
| `CLAUDE.md` | Claude Code | `@AGENTS.md` import |
| `GEMINI.md` | Gemini CLI | `@AGENTS.md` import |
| `.cursor/rules/agents.mdc` | Cursor IDE | Reference to AGENTS.md |
| `.github/copilot-instructions.md` | GitHub Copilot | Reference to AGENTS.md |
| `.windsurfrules` | Windsurf | Reference to AGENTS.md |
| `.clinerules` | Cline | Reference to AGENTS.md |

### Why not symlinks?

Symlinks (`ln -s AGENTS.md CLAUDE.md`) work on macOS/Linux but break on Windows (require admin privileges). Import/reference files are cross-platform and let you add tool-specific instructions below the import if needed.

### Adding tool-specific rules

Each bridge file can include extra instructions for its tool. For example, `CLAUDE.md` could add:

```markdown
@AGENTS.md

## Claude-specific
- Use Plan Mode for multi-step changes
- After learning something non-trivial, update memory files
```

## Why AGENTS.md

`README.md` is for humans. `AGENTS.md` is for agents. This file sits at the root of any repo and gives coding agents project-specific instructions they can act on — not vague philosophy names, but concrete rules.

The [AGENTS.md standard](https://agents.md/) is stewarded by the Agentic AI Foundation under the Linux Foundation and supported by 60,000+ open-source projects and 30+ coding agent tools.

## How to use

1. Copy `AGENTS.md` to the root of your project
2. Copy the bridge files for the tools you use
3. Most coding agents will pick it up automatically
4. For Aider, add `read: AGENTS.md` to `.aider.conf.yml`
5. For Gemini CLI, set `"contextFileName": ["GEMINI.md", "AGENTS.md"]` in `.gemini/settings.json`

## Philosophy

> Subtract what doesn't matter. Keep what does. Don't lie about the difference.

Every line in this file is something an agent can check or enforce. No "write clean code" — instead: "functions under 25 lines, files under 250 lines." No "follow SOLID" — instead: "every module has one reason to change." Philosophy translated into rules.

## License

MIT — use it, fork it, adapt it to your own principles.