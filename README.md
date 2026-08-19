# agents-md

My personal [AGENTS.md](https://agents.md/) — a coding philosophy file that works across all major AI coding agents (Codex, Jules, Aider, Cursor, Windsurf, Gemini CLI, and 30+ more).

## What's in here

A single `AGENTS.md` file that translates my coding principles into instructions an AI agent can actually follow:

- **Ports & Adapters** — default architecture. Domain has zero external imports. Every side effect goes through a port.
- **FP over OOP** — pure functions, immutable data, composition over inheritance, Result types instead of exceptions.
- **SOLID (always SRP)** — every module has one reason to change. No exceptions.
- **Clean Code** — functions under 25 lines, files under 250, intention-revealing names, no dead code, no speculative abstractions.
- **Twelve Factor** — config in env vars, stateless processes, logs to stdout, dev/prod parity.
- **XP** — test-first, refactor after green, small commits, small PRs.

## Why AGENTS.md

`README.md` is for humans. `AGENTS.md` is for agents. This file sits at the root of any repo and gives coding agents project-specific instructions they can act on — not vague philosophy names, but concrete rules.

The [AGENTS.md standard](https://agents.md/) is stewarded by the Agentic AI Foundation under the Linux Foundation and supported by 30+ coding agent tools.

## How to use

1. Copy `AGENTS.md` to the root of your project
2. Most coding agents will pick it up automatically
3. For Aider, add `read: AGENTS.md` to `.aider.conf.yml`
4. For Gemini CLI, set `context.fileName: "AGENTS.md"` in `.gemini/settings.json`

## Philosophy

> Subtract what doesn't matter. Keep what does. Don't lie about the difference.

Every line in this file is something an agent can check or enforce. No "write clean code" — instead: "functions under 25 lines, files under 250 lines." No "follow SOLID" — instead: "every module has one reason to change." Philosophy translated into rules.

## License

MIT — use it, fork it, adapt it to your own principles.