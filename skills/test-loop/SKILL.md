---
name: test-loop
description: >
  Select the narrowest test feedback loop for the change at hand — which
  tests to run while iterating, and when to run the full suite. Use this
  whenever you are about to run tests during development, when deciding
  between unit / integration / e2e / architecture loops, when test runs
  feel slow or an agent avoids running them, when setting up or reviewing
  a categorized test suite, or when the user asks about test categories,
  opt-in suites, or "which tests should I run". Do not use it for how to
  write tests (that is the tdd skill) or for CI pipeline configuration.
---

# Test Feedback Loop

The loop is the edit → test cycle inside a change. The rule that governs it: **run the narrowest loop that can fail for the right reason**. A loop that takes minutes gets skipped by everyone — including agents — and a skipped loop is worse than no loop, because it produces false confidence. The full suite runs exactly **once**, at final verification. Never mid-change.

## Category taxonomy

Every test class carries **exactly one** category tag. The categories are a pattern, not mandates — adapt names to the stack, but keep the boundaries sharp:

| Category | Meaning | Hallmark |
|----------|---------|----------|
| `fast` | Pure process: no Spring/app context, no containers, no broker, no network. In-memory fakes allowed. | Runs in milliseconds; safe to run on every edit |
| `integration` | Real infrastructure edges: app context, relational database, HTTP surface | Asserts the seams fast tests fake |
| `e2e` | Full stack including brokers/messaging, complete workflows | Locks acceptance; expensive, run on demand |
| `architecture` | ArchUnit / taxonomy / dependency-rule enforcement | Guards structure, not behavior |

Where the stack allows, enforce the "exactly one category" contract with a **meta-test** that fails the build on a missing or duplicated tag. Convention without enforcement erodes within weeks.

**Opt-in, not opt-out.** A bare test command with no filters runs everything. Loops are selected explicitly via category filters and module/target scoping. This is what makes the fast loop possible: you cannot opt out of what is on by default.

**Cost warning.** If the default test invocation boots containers or waits on a broker, agents and humans will stop running tests. Make the `fast` loop cheap first; everything else follows.

## Choose the loop by what changed

Map the change's location to the narrowest loop that exercises it — for hexagonal codebases, work inward-first:

1. **Domain/application core change** → the module's own `fast` loop, often a single test class or the architecture suite.
2. **Contract change** (API/schema/event shape) → the adapter's unit loop for the contract, **plus one** matching `integration` test — no more.
3. **Iteration inside a module whose build runs expensive codegen** → skip codegen for the inner loop if generated sources persist between runs; re-enable it before final verification.
4. **Persistence/infra adapter change** → the `integration` loop for that adapter.
5. **Workflow/acceptance** → `e2e`, on demand only, never as a regression filter for routine edits.
6. **Final verification** → the **single** full-suite run (all categories, all modules). One run, at the end, before handing off.

Anti-patterns:

- **Full-reactor reflex**: running everything after every edit. Slow, noisy failures, and it hides which test actually caught the regression.
- **Wrong-reason red**: running an `e2e` suite to iterate on pure domain logic — when it fails you learn nothing about the logic, only about the plumbing.
- **Suite drift**: adding tests without a category, so "everything" and "the fast loop" silently diverge.

## Per-project contract

This skill owns the **procedure**; the project owns the **map**. Before selecting a loop, read the project's test loop map — usually a table in `AGENTS.md` (or equivalent agent docs) declaring:

- the category names in use and what each excludes
- the concrete commands per loop (filters, module scoping, cost-saving flags)
- the change-location → loop mapping for its module layout

If the project has no test loop map, say so and offer to draft one (categories, commands, mapping) from its build layout — then record it where the project keeps agent docs. Do not guess commands; a wrong loop selection wastes more time than a slow one.
