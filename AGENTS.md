# AGENTS.md

How I want you to write code. Read this before writing anything.

---

## Verification Protocol

Before declaring a task done, run these in order. Fix all failures before proceeding to the next step:

1. `bun test` — all tests pass
2. `tsc --noEmit` — no type errors
3. `bun run lint` — no lint errors
4. `bun run build` — production build succeeds

Never skip a step. Never mark a task complete with failing checks. If a check doesn't exist yet, create it. Trust but verify — agent-generated code is not correct by default. Tests prove it.

---

## Agent Behaviour

- **Ask for help when uncertain.** If you don't know the answer, say so. Flag uncertainty. Pretending certainty causes downstream defects.
- **Stop when blocked.** If you've tried two approaches and neither works, stop and explain the problem. Don't keep guessing.
- **Small steps.** One change, verify, then the next. Don't make ten changes and hope they all work.
- **Spec before code.** Write or ask for a specification before implementing. Vague prompts produce vague code.
- **Don't run unbounded.** If a task is growing beyond scope, stop and report. Scope creep is a defect.

---

## Architecture: Ports & Adapters

Default architecture. Every project follows it unless there's a documented reason not to.

```
src/
 domain/     # Pure business logic. No framework imports. No I/O.
 adapters/
  inbound/   # HTTP handlers, CLI commands, event listeners. Thin glue.
  outbound/  # Databases, APIs, file systems, email. Implements port interfaces.
 ports/      # Interfaces the domain depends on. Contracts for swapping implementations.
 config/     # Environment config, wiring, composition root. The only place that knows which adapters are plugged in.
```

1. **Domain has zero imports from outside `domain/`.** No framework, no database driver, no HTTP library. If the domain can't compile without an external package, the boundary is wrong.
2. **Adapters depend on ports, not the other way round.** The domain never knows an adapter exists.
3. **The composition root wires everything.** This is the only file that imports both domain and adapters.
4. **Every side effect goes through a port.** Database writes, API calls, file reads — all of it.
5. **Test the domain with in-memory adapters.** No test should need a database or external service.

---

## Functional Programming over OOP

Prefer FP. Use OOP only when the problem genuinely demands stateful objects with identity (rarely).

- **Pure functions by default.** Same input, same output, no side effects. Push impurity to the boundary.
- **Immutable data.** Never mutate. Return new objects. Spread, don't assign.
- **Composition over inheritance.** No class hierarchies deeper than one level. Prefer `pipe`, `compose`, or function chains.
- **`map`/`filter`/`reduce` over loops.** If you're writing a `for` loop, there's a declarative alternative.
- **Avoid `class` unless there's a compelling reason.** Use `interface` or `type`. Factory functions return plain objects.
- **Railway-oriented error handling.** Return `Result<T, E>` types rather than throwing. Exceptions are for truly exceptional situations.

---

## SOLID — Always SRP

Apply all five, but **Single Responsibility is non-negotiable**.

- **S:** Every module, function, and file has exactly one reason to change. If it does two things, split it.
- **O:** Extend by adding code, not modifying existing code. Composition and dependency injection.
- **L:** Subtypes must be fully substitutable. No overrides that narrow preconditions.
- **I:** Prefer narrow, focused interfaces. No consumer depends on methods it doesn't use.
- **D:** Depend on abstractions (port interfaces), not concretions. Inject dependencies.

---

## Clean Code

- **Functions under 25 lines.** If longer, extract. No exceptions.
- **Files under 250 lines.** If longer, it's doing too much.
- **React components under 100 lines.** Extract hooks and sub-components.
- **Intention-revealing names.** No `data`, `info`, `mgr`, `handler`, `util`, `helper`, `misc`.
- **No comments that repeat the code.** Comments say why — decisions, constraints, trade-offs.
- **No dead code.** If it's not called, delete it. Version control remembers everything.
- **No speculative abstractions.** No interfaces with one implementation. No factories for a single use case. Add abstraction when you have three concrete examples, not before.
- **No unnecessary dependencies.** If the standard library or ten lines of code can do it, don't install a package.

---

## Twelve Factor

- **Config in environment.** All environment-specific values come from env vars. Never hard-code config. Never commit secrets.
- **Backing services as attached resources.** Treat Postgres, Redis, S3 as interchangeable resources referenced by URL/env var.
- **Stateless processes.** No in-memory session state. No local disk state that must survive a restart.
- **Dev/prod parity.** Same runtime version, same database engine. Avoid environment-specific code paths.
- **Logs as event streams.** Log to stdout/stderr as structured events. Never write to log files.
- **Disposability.** Processes start fast and shut down gracefully on SIGTERM.
- **Build, release, run separation.** Build artefacts are immutable. Config is injected at runtime.

---

## XP Practices

- **Test-first.** Write the test before the implementation. Red, green, refactor. Every time.
- **Refactor after green.** Once the test passes, clean up. Don't skip this step.
- **Root-cause every bug.** Five Whys. Write a system-level test that would have caught it. Don't patch symptoms.
- **Small commits.** Each commit is one coherent change that builds, passes tests, and makes sense on its own.
- **Small PRs.** Target under 400 lines changed. Split: preparatory refactor first, then the feature.
- **Incremental design.** Design in the light of experience. Defer complexity until it's necessary. The most effective time to invest in design is after the first feature works, not before.
- **Pair on hard things.** If a change touches core domain logic or crosses boundaries, talk it through before coding.

---

## Git

- **Conventional Commits:** `type(scope): description` — e.g. `feat(api): add rate limiting to public endpoints`.
- **Commit body explains why, not what.** The diff shows what.
- **Never bundle unrelated changes.** Formatting, dependency updates, and feature work are separate commits.

---

## Runtime & Tools

- **Bun over Node.** `bun` not `node`, `bun test` not `jest`, `Bun.serve()` not `express`, `bun:sqlite` not `better-sqlite3`.
- **TypeScript strict mode.** No `any` without a comment explaining why. No `@ts-ignore` without a comment.
- **ESM.** Generate ESM-compatible TypeScript. No CommonJS.

---

## Testing

- **Domain tests are fast and in-memory.** No database, no HTTP server, no external service.
- **Adapter tests use real implementations** against real services or realistic fakes. Integration tests prove the adapter works.
- **Every change gets a test.** If you changed code, add or update a test. No exceptions.
- **Test behaviour, not implementation.** Test the output, not the internal calls.
- **Coverage is a regression signal.** Don't delete or weaken tests to make them pass. If PR coverage drops, that's a warning.

---

## Error Handling

- **Errors are not optional.** Handle them. Don't swallow them. Don't `console.log` and move on.
- **Use Result types in the domain.** Return `{ ok: true, value }` or `{ ok: false, error }`. Reserve exceptions for truly exceptional situations.
- **Adapters translate external errors into domain errors.** The domain never sees a Postgres error code.
- **If you're unsure, say so.** Don't guess and present it as fact.

---

## Security

- Never commit secrets, keys, tokens, or credentials. Use environment variables.
- Validate all inputs at the boundary. Never trust user input.
- Review new dependencies for vulnerabilities before adding them. Justify every dependency.
- Sanitize output to prevent XSS. Parameterise queries to prevent SQL injection.
- No hardcoded URLs, IPs, or credentials in source code.
- Don't make external calls or exfiltrate data beyond what the task requires.

---

## What not to do

- Don't build features I didn't ask for. No scope creep. No "improvements" that weren't requested.
- Don't add files, layers, wrappers, or configs that aren't needed right now. The default is to add — resist it.
- Don't over-engineer. No framework when a function will do. No architecture for scale that doesn't exist yet.
- Don't suppress errors to make tests pass.
- Don't write comments that describe what you wish the code did instead of what it actually does.
- Don't reach for a library, pattern, or framework just because it's popular. Earn the answer by thinking.

---

## Known Pitfalls

- Deleting or weakening tests to make them pass — worse than no tests.
- Suppressing errors to make the build green — hides real problems.
- Adding dependencies without justification — every dependency is attack surface.
- Refactoring working code without a test covering the behaviour — you can't verify you didn't break it.
- Making changes outside the scope of the task — scope creep compounds.

---

## The filter

If you're ever unsure what to do, run it through this:

1. **Does this actually matter?** If yes, make it as good as it can be. If no, remove it.
2. **Is this honest?** Does the code do what it says? Does the test test what matters? Does the name mean what it says?
3. **Does it feel inevitable?** If you can imagine an equally good alternative, keep working until you can't.

Subtract what doesn't matter. Keep what does. Don't lie about the difference.

---

## Living document

Update this file when you learn something new. Add recurring mistakes to "Known Pitfalls". Add new conventions as rules, not philosophy. Keep every line actionable and verifiable.