# AGENTS.md

How I want you to write code. Read this before writing anything.

---

## Architecture: Ports & Adapters

This is the default architecture. Every project follows it unless there's a documented reason not to.

### Structure

```
src/
  domain/          # Pure business logic. No framework imports. No I/O.
                    Functions, types, and rules that would still make sense
                    if you ran them on a desert island with no computer.
  adapters/
    inbound/       # Things that call the domain: HTTP handlers, CLI commands,
                    event listeners, scheduled jobs. Thin glue — parse input,
                    call domain, serialise output. No business logic here.
    outbound/      # Things the domain calls: databases, APIs, file systems,
                    email, queues. Each adapter implements a port interface
                    defined in domain/.
  ports/           # Interfaces the domain depends on. These are the contracts
                    that let you swap SQLite for Postgres, or fetch for axios,
                    without touching business logic.
  config/          # Environment config, wiring, composition root. This is the
                    only place that knows which adapters are plugged in.
```

### Rules

1. **Domain has zero imports from outside `domain/`.** No framework, no database driver, no HTTP library. If the domain can't compile without an external package, the boundary is wrong.
2. **Adapters depend on ports, not the other way round.** Ports are defined by what the domain needs, implemented by adapters. The domain never knows an adapter exists.
3. **The composition root (config/) wires everything together.** This is the only file that imports both domain and adapters. If you're importing an adapter inside domain code, stop and refactor.
4. **Every side effect goes through a port.** Database writes, API calls, console.log, file reads — all of it. If the domain needs to persist something, it calls a port function. The adapter decides how.
5. **Test the domain with in-memory adapters.** No test should need a database, HTTP server, or external service to run. If it does, the boundary is wrong.

---

## Functional Programming over OOP

Prefer FP. Use OOP only when the problem genuinely demands stateful objects with identity (and it rarely does).

- **Pure functions by default.** Same input, same output, no side effects. If a function reads `Date.now()`, generates a UUID, or queries a database, it's not pure — push that impurity to the boundary.
- **Immutable data.** Never mutate. Return new objects. Use `const` exclusively for bindings. Spread, don't assign.
- **Composition over inheritance.** No class hierarchies deeper than one level. Prefer `pipe`, `compose`, or simple function chains. If you're writing `extends`, you should probably be writing a function instead.
- **`map`/`filter`/`reduce` over loops.** Use array methods. If you're writing a `for` loop, there's almost certainly a declarative alternative.
- **Avoid `class` entirely unless there's a compelling reason.** Type definitions use `interface` or `type`. Factory functions return plain objects. No decorators, no abstract classes, no strategy patterns implemented as class hierarchies.
- **Railway-oriented error handling.** Return `Result<T, E>` types (or `{ ok: true, value } | { ok: false, error }`) rather than throwing. Exceptions are for truly exceptional situations — a missing database row is not exceptional, it's an expected case.

---

## SOLID — Always SRP

Apply all five principles, but **Single Responsibility is non-negotiable**.

- **S — Single Responsibility:** Every module, function, and file has exactly one reason to change. If a function does two things, split it. If a file handles parsing and formatting, split it. If you're unsure whether something violates SRP, it probably does.
- **O — Open/Closed:** Extend behaviour by adding code, not modifying existing code. Use composition and dependency injection. Strategy objects with a port interface, not `if/else` chains on type.
- **L — Liskov Substitution:** Subtypes must be fully substitutable. No overrides that narrow preconditions or widen postconditions.
- **I — Interface Segregation:** Prefer narrow, focused interfaces. No consumer should depend on methods it doesn't use. Split fat interfaces into smaller ones.
- **D — Dependency Inversion:** Depend on abstractions (port interfaces), not concretions (adapter implementations). Inject dependencies — never reach out for them.

---

## Clean Code

Robert C. Martin's principles, translated into concrete rules:

- **Functions under 25 lines.** If it's longer, extract. No exceptions.
- **Files under 250 lines.** If it's longer, it's doing too much.
- **React components under 100 lines.** Extract hooks and sub-components.
- **Intention-revealing names.** No `data`, `info`, `mgr`, `handler`, `util`, `helper`, `misc`. If you can't name it, you don't understand it. Spend more time on naming than feels reasonable.
- **No comments that repeat the code.** The code says what. Comments say why — decisions, constraints, trade-offs, things the code can't express.
- **No dead code.** If it's not called, delete it. Version control remembers everything. Don't comment it out "just in case."
- **No speculative abstractions.** No interfaces with one implementation. No factory patterns for a single use case. No config systems for values that never change. Add abstraction when you have three concrete examples that prove you need it, not before.
- **No unnecessary dependencies.** If the standard library or ten lines of your own code can do it, don't install a package. Every dependency justifies itself or gets removed.

---

## Twelve Factor

Apply these factors to all backend and infrastructure code:

- **Config in environment.** All environment-specific values (API keys, URLs, feature flags) come from env vars. Never hard-code config. Never commit secrets.
- **Backing services as attached resources.** Treat Postgres, Redis, S3, and any external service as interchangeable resources referenced by URL/env var — not as local, tightly coupled dependencies.
- **Stateless processes.** No in-memory session state. No local disk state that must survive a restart. Use the database, cache, or object storage for anything that persists.
- **Dev/prod parity.** Keep local, dev, and prod environments as similar as possible. Same runtime version, same database engine. Avoid environment-specific code paths.
- **Logs as event streams.** Log to stdout/stderr as structured events. Never write to log files or manage log rotation in application code.
- **Disposability.** Processes start fast and shut down gracefully on SIGTERM. Finish in-flight requests before exiting.
- **Build, release, run separation.** Build artefacts are immutable. Config is injected at runtime via env vars, not baked into builds.

---

## XP Practices

Extreme Programming, interpreted as concrete development habits:

- **Test-first.** Write the test before the implementation. Red, green, refactor. Every time.
- **Refactor after green.** Once the test passes, clean up. Don't skip this step. The code you wrote to make the test pass is probably not the code you want to keep.
- **Small commits.** Each commit is one coherent change that builds, passes tests, and makes sense on its own. Refactoring and feature work are separate commits.
- **Small PRs.** Target under 400 lines changed. If it's growing large, split it: preparatory refactor first, then the feature on top.
- **Pair on hard things.** If a change touches core domain logic or crosses architectural boundaries, talk it through before coding.

---

## Git

- **Conventional Commits:** `type(scope): description` — e.g. `feat(api): add rate limiting to public endpoints`.
- **Commit body explains why, not what.** The diff shows what.
- **Never bundle unrelated changes.** Formatting, dependency updates, and feature work are separate commits.

---

## Runtime & Tools

- **Bun over Node.** Use Bun for runtime, package management, and testing. `bun` not `node`, `bun test` not `jest`, `Bun.serve()` not `express`, `bun:sqlite` not `better-sqlite3`.
- **TypeScript strict mode.** No `any` without a comment explaining why. No `@ts-ignore` without a comment explaining why.
- **ESM.** Generate ESM-compatible TypeScript. No CommonJS.

---

## Testing

- **Domain tests are fast and in-memory.** No database, no HTTP server, no external service. Pure functions are easy to test — test them thoroughly.
- **Adapter tests use real implementations against real services (or realistic fakes).** Integration tests prove the adapter works, not that the mock returns what you told it to.
- **Every change gets a test.** If you changed code, add or update a test. No exceptions.
- **Test behaviour, not implementation.** If you're testing that a function calls `map` three times, you're testing the wrong thing. Test the output.

---

## Error Handling

- **Errors are not optional.** Handle them. Don't swallow them. Don't `console.log` and move on. Don't return silent defaults when something genuinely went wrong.
- **Use Result types in the domain.** Return `{ ok: true, value }` or `{ ok: false, error }` rather than throwing. Reserve exceptions for truly exceptional situations (out of memory, corrupt state).
- **Adapters translate external errors into domain errors.** A database connection failure becomes a domain error type. The domain never sees a Postgres error code.
- **If you're unsure, say so.** Don't guess and present it as fact. Flag uncertainty. Uncertainty is fine. Pretending certainty isn't.

---

## What not to do

- Don't build features I didn't ask for. No scope creep. No "improvements" that weren't requested.
- Don't add files, layers, wrappers, or configs that aren't needed right now. The default is to add — resist it.
- Don't over-engineer. No framework when a function will do. No plugin system when a config file will do. No architecture for scale that doesn't exist yet.
- Don't suppress errors to make tests pass.
- Don't write comments that describe what you wish the code did instead of what it actually does.
- Don't reach for a library, pattern, or framework just because it's popular. Ask whether it's actually the right tool. Earn the answer by thinking, not defaulting.

---

## The filter

If you're ever unsure what to do, run it through this:

1. **Does this line/module/dependency/abstraction actually matter?** If yes, make it as good as it can be. If no, remove it.
2. **Is this honest?** Does the code do what it says? Does the test test what matters? Does the name mean what it says?
3. **Does it feel inevitable?** If you can imagine an equally good alternative, keep working until you can't.

Subtract what doesn't matter. Keep what does. Don't lie about the difference.