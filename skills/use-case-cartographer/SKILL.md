---
name: use-case-cartographer
description: >
  Map and describe the use-cases of a codebase (especially hexagonal /
  ports-and-adapters backends — Java, Kotlin, TypeScript, or anything with
  inbound-port interfaces) as a catalogue plus Mermaid sequence diagrams
  derived from verified facts. Use this whenever the user asks to "describe
  the use-cases", "diagram this flow", "what does this service do", "map
  the architecture", "show me who calls whom", "walk me through this
  endpoint", wants sequence diagrams of an application flow, is onboarding
  into an unfamiliar backend, or asks for a catalogue of commands/queries
  with their entry points — even if they don't say the words "use case".
  Prefer this over general architecture explanations or static UML/class
  diagrams when the question is about runtime behaviour of application-layer
  use-cases; do not use it for refactoring, test-writing, or API-spec
  generation.
---

# Use-Case Cartographer

Describe a system's use-cases honestly. The one rule that makes this skill
work: **facts come from code, the description only interprets them.** Never
add a participant, call, or component that you did not verify in source.
A missing interaction is a small flaw; an invented one poisons the reader's
trust in everything else you say.

## The workflow

### 1. Build the catalogue (deterministic, no guessing)

Enumerate the application's use-cases from the code's own structure before
describing anything:

- **Hexagonal / ports-and-adapters (preferred signal):** one use-case per
  inbound-port interface. The naming convention varies by repo — `*UseCase`
  interfaces in a `port/in` / `application.port.in` package are the common
  Java shape, but you may find `*UseCase`/`*InputPort`/handler interfaces
  elsewhere. Spend the first minute detecting the repo's own convention
  (search for the port package or the `*UseCase` suffix), then apply it
  mechanically. Find each implementation by searching for
  `implements <Name>` / `: <Name>`, and its entry points by finding which
  in-adapters (REST controllers, messaging consumers, CLI handlers)
  reference the interface name.
- **Classify command vs query** from the name: names starting with
  List/Get/Find/Search or ending in Queries/View expose data (queries);
  everything else mutates state (commands). Say this is a heuristic.
- **Other architectures:** fall back to the framework's own units —
  Spring `@RestController` handlers, gRPC RPCs, message consumers, CLI
  commands. One public entry point = one use-case.

Present the catalogue as a table: name, kind, implementing service, entry
points. An entry with no in-adapter reference is worth flagging — it may
be dead code.

### 2. Extract the facts for the chosen use-case

For one use-case, collect everything a description may legitimately
contain, by reading the actual sources:

- The port-in method signatures (returns, name, args).
- The implementing service's constructor-injected fields (or equivalent
  DI), classified: **out-ports** (inbound-facing interfaces the repo
  declares in its outbound port package, e.g. `port/out`), **collaborating
  services**, **other** (mappers, support types — usually diagram noise,
  drop them).
- The verified method menu of each out-port it uses — this is what lets
  you label calls *and* returns without inventing names.
- Domain types touched (from imports) and outbox/event ports.
- HTTP mappings of the REST entry point (`@GetMapping` etc. plus the
  class-level `@RequestMapping` prefix) or the messaging topic.

### 3. Fix the participant list first

Derive the participant whitelist *before* writing any prose or diagram:
entry points + the service + out-ports + collaborating services + domain
types. Drop exception/error types — failures never interact; they are
diagram noise. **Only whitelisted names may appear anywhere in the output.**

### 4. Describe and diagram

- Start with a coarse, honest skeleton: who is involved, verified call
  directions, no invented internal ordering. It is fine for it to be
  coarse — say so.
- Then render the readable Mermaid `sequenceDiagram`, constrained to the
  whitelist, with real method names from the fact sheet, and dotted return
  arrows labelled with the verified return types. Void calls may omit
  returns.
- **Completeness:** the service must interact with every out-port,
  delegate to every collaborating service, and publish every listed event.
  A diagram showing only the entry call is a failed answer.
- Follow the diagram with a short prose description: what the use-case is
  for, the flow in words, and any entry points not covered by the diagram.

If you notice yourself omitting a verified interaction (this is the most
common failure mode — entry-call-only diagrams), re-check the fact sheet
and add it; never fill gaps from imagination.

## Output format

```markdown
## <UseCaseName> (<command|query>)

One-paragraph intent of the use-case.

```mermaid
sequenceDiagram
    participant <Entry>
    ...
```

**Flow:** 2–4 sentences narrating the diagram in words.
**Entry points:** VERB /path, consumer class, ...
**Facts:** port-in methods, out-ports, events, domain types.
```

## Honest-by-construction tips

- Prefer scripted extraction over eyeballing when the codebase is large;
  regexes over interface/field/mapping declarations get the catalogue in
  seconds and never hallucinate.
- Show the skeleton diagram alongside the rendered one when you can — the
  deterministic floor stays useful even if a rendering pass fails.
- If a fact cannot be verified (dynamic dispatch, reflection, config-wired
  beans), say "unverified" rather than guessing.
- Method-level nuance stays in the IDE; this skill maps type-level
  structure on purpose. Coarse and correct beats detailed and invented.
