# 07 — Engines

> **TL;DR** — An **engine** is the thing that evaluates a graph. It is a *reducer over a graph*: given the current state, an input, and a clock value, it produces outputs, a new state, and an evaluation record. Each engine implements **one grammar**, and nothing more. Engines are strictly clock-free, I/O-free, network-free, and environment-agnostic — they only evaluate composition. The outside world is handled by adapters. Engines are **plural** (one per grammar) and interchangeable (any two engines honoring the same grammar contract are substitutable). They compose **across compound boundaries**: a parent engine hands an inner compound to the engine for its inner grammar, and neither engine knows anything about the other's internals.

---

## What an engine is, precisely

An engine is a **reducer**. Given:

- A **graph** — nodes, edges, compounds, all valid under some grammar.
- The **current state** — whatever per-node or per-graph state the grammar maintains.
- An **input** — new values arriving at outer ports.
- A **`now` value** — the current time, as provided by whatever is driving the engine.

It produces:

- **Output values** — on the graph's outer output ports, and anywhere else the grammar says outputs can appear.
- **A new state** — the updated per-node / per-graph state after this evaluation.
- **An evaluation record** — what fired, in what order, what changed, for debugging and observability.

That is all the engine does. It does not start itself. It does not schedule itself. It does not decide where inputs came from or where outputs should go. **Something else calls it**, hands it the inputs, and does something with the outputs. That "something else" is the adapter.

### The engine's question

The engine's job is to answer one question, repeatedly:

> *Given this graph and this state, and these inputs right now at this time, what are the outputs and the new state?*

That is a **pure question about composition**. It has no dependencies on HTTP, clocks, screens, files, sensors, or any other real-world concern. Give it the same inputs and state, it produces the same outputs and new state. Engines are **deterministic given their inputs**, which is what makes them testable, replayable, and snapshottable.

### The engine's non-questions

The engine does **not** answer:

- *When* should the graph run? (Adapter.)
- *Where* do the inputs come from? (Adapter.)
- *Where* do the outputs go? (Adapter.)
- *How long* should we wait for something? (Adapter.)
- *How* do we persist things across restarts? (Store / adapter.)
- *What* does the screen look like? (UI adapter.)
- *Who* is the user? (Adapter / store.)
- *How* do we retry on failure? (Adapter, usually.)

If an engine starts answering any of those, it has grown beyond its role and will start conflicting with adapters. That is the warning sign.

---

## One grammar per engine

Each engine implements exactly one grammar.

- The **DAG engine** implements the DAG grammar: forward dataflow, acyclic, topological evaluation, (optionally) node-local state and conditional emission.
- The **FSM engine** (forthcoming) implements the FSM grammar: one active state, transitions driven by events, entry/exit actions.
- The **tree engine** implements the tree grammar for containment: parent-down propagation, child-up bubbling, layout.
- The **event engine** implements the pub-sub grammar: topic-based publication and subscription.
- The **reactive signal engine** (if we build one) implements that grammar.
- And so on.

Each engine is small and focused. It knows one way to compose, and it does that one thing extremely well. Engines are not a shared runtime — each is its own implementation of one grammar's semantics.

This is intentional. A monolithic "universal engine" trying to handle all grammars would be complex, slow, hard to reason about, and hard to evolve. Specialist engines are simpler, faster, easier to test, and cleanly versionable.

---

## Engines are interchangeable within a grammar

Two engines that implement the same grammar correctly are **substitutable**. A graph using grammar `dag@1.0` should run identically on any engine that implements `dag@1.0`, regardless of the engine's internals.

This is the **engine contract**: implement the grammar faithfully, and you're a valid engine. How you implement it — optimizations, scheduling strategies, concurrency model, memory layout — is your business.

Why this matters:

- **Multiple implementations can exist.** A JavaScript DAG engine for the browser; a Rust DAG engine for embedded; a Java DAG engine for enterprise JVM deployment. All can run the same graph.
- **Optimizations can compete.** One engine can prioritize throughput; another can prioritize memory; another can prioritize startup time. Users pick.
- **Engines can evolve.** An engine implementation can change drastically across versions, as long as the grammar contract is honored.

The engine contract for each grammar lives in `specs/engine-contract.md` and per-grammar engine specs. Those specs describe the behavior any valid engine must provide.

### Test for "this engine stayed in its lane"

If you can swap the engine for another engine implementing the same grammar **without touching the graph or the adapter**, the engine is pure.

If swapping breaks something, the engine has grown outward — it is making assumptions about its environment that the grammar does not require. That is a leak. Fix it by moving those assumptions into the adapter.

---

## What an engine owns

An engine owns:

1. **Graph evaluation.** Honoring the grammar's firing rules, composition semantics, and constraints.
2. **Per-node state.** Storing and passing node-local state for stateful processors.
3. **Scheduling within an evaluation.** In what order do nodes fire during a single tick / step / propagation?
4. **Error propagation within the graph.** How errors flow to error ports or fail the evaluation.
5. **Invalidation tracking.** Which nodes need to re-fire when inputs change (grammar-dependent optimization).
6. **Evaluation records / tracing.** For debugging and observability.
7. **Dispatch across compound boundaries.** When evaluating a compound using a different grammar, hand the compound to the engine for that grammar.

Engines do not have to implement all of these identically; some are grammar-specific (invalidation tracking only makes sense for grammars where it applies).

---

## What an engine must *never* own

An engine must never own:

1. **Real-world clock.** The clock / `now` value is supplied by whoever is driving the engine (almost always an adapter). The engine treats `now` as an opaque value it uses for time-sensitive processors (if the grammar has any). Engines never call `Date.now()` or similar.
2. **I/O.** No file reads, no HTTP calls, no database queries, no screen rendering. All I/O is the adapter's or a store's.
3. **Scheduling the engine itself.** The engine does not decide when it runs. It runs when called.
4. **Persistence.** Engine state is in-memory. If it needs to survive a restart, the adapter snapshots it to a store and restores it on startup.
5. **User input.** The engine does not know what a "user" is. It sees inputs arriving at ports.
6. **Networking.** The engine doesn't talk to any network. If a node's processor needs to call an external API, that processor uses an adapter or a connection — the engine just sees a processor firing and returning a value (possibly asynchronously).
7. **Rendering.** Engines don't render. Rendering is a UI adapter concern.

When a processor needs to do something the engine cannot do (call an API, render, read a sensor), the processor is actually a wrapper that invokes an adapter or a connection. The engine just sees "this processor is firing asynchronously and will return a value"; it has no idea what is happening inside.

---

## Engines compose across compound boundaries

This is the most important mechanism for multi-grammar apps.

A compound declares its inner grammar. When the engine for the parent grammar encounters a compound, it:

1. Sees the compound as a node with input and output ports.
2. Forwards input values to the compound's outer ports.
3. **Hands the compound to the engine for its inner grammar.**
4. Receives output values back.
5. Propagates those outputs to downstream nodes in the parent graph.

The parent engine doesn't know or care what happens inside. It just sees "function called, inputs in, outputs out." The inner engine evaluates the compound according to its own grammar, returns results, and the parent engine resumes.

This is exactly how it must work for **invariant 2** (every compound is a function) and **invariant 4** (grammars are calling conventions) to hold together. The compound boundary is where grammar dispatch happens, cleanly.

### Practical shape

The core runtime provides a **dispatcher** that:

- Knows which engine implements which grammar.
- Given a compound instance, resolves its inner grammar and routes evaluation to the corresponding engine.
- Marshals inputs and outputs across the boundary.

Engines don't need to know about each other directly. They each interact with the dispatcher, which hands them the next compound to evaluate. The dispatcher is grammar-agnostic and lives in the core runtime.

This is how a DAG app can contain FSM compounds can contain tree compounds, and everything just works.

---

## State, snapshots, and replay

Because engines are deterministic given their inputs, state, and `now`, they can be:

- **Snapshotted.** The engine's state can be serialized and restored. The adapter captures snapshots periodically or at well-defined boundaries.
- **Replayed.** Given a recorded sequence of inputs and `now` values, the engine will reproduce the same outputs and state. This is the basis for time-travel debugging, testing, and bug reproduction.
- **Migrated.** Engine state from one version can be migrated to another, if the grammar's state schema supports versioning.

These capabilities are not free; the grammar's engine contract must explicitly support them. But the determinism constraint is what makes them possible at all.

---

## Performance concerns (and what belongs where)

Engines are performance-sensitive. A DAG engine evaluating a graph with thousands of nodes per second must be fast. Some ways engines legitimately optimize:

- **Topological caching.** Precompute firing order.
- **Dirty tracking.** Only re-fire nodes whose inputs changed.
- **Dispatch optimization.** Avoid indirection for hot paths.
- **Parallelism.** Fire independent nodes concurrently (per grammar rules).
- **Incremental evaluation.** Only evaluate the affected subgraph on an update.

All of these are implementation concerns. They do not change the grammar. They are how a specific engine chooses to honor its contract. Two engines implementing the same grammar can make completely different performance trade-offs.

Performance concerns that belong **outside the engine**:

- **I/O batching.** Adapter or processor concern.
- **Network caching.** Connection concern.
- **Rendering throttling.** UI adapter concern.
- **Persistence batching.** Store concern.

Don't push those into the engine just because it's convenient. Each layer has its own performance story.

---

## Engine versioning

Engines declare which version of which grammar they implement.

- A graph declares the grammar version it was authored against.
- An engine declares the grammar versions it supports.
- The runtime checks compatibility at graph-load time.

Grammars evolve. Engines evolve. The contract system keeps them coordinated.

See the versioning spec (`specs/versioning.md`) for the actual rules.

---

## What an engine is *not*

- An engine is **not a framework**. It doesn't provide opinions on how apps are structured outside the graph.
- An engine is **not a runtime environment**. It runs inside one (Node, browser, JVM, CLR, whatever), but it doesn't define that environment.
- An engine is **not a scheduler**. It runs when called. Something else decides when.
- An engine is **not a database**. It has in-memory state; stores are elsewhere.
- An engine is **not user-facing**. Users don't know or care which engine is running their app. Authors choose grammars; runtimes choose engines.

---

## Summary

- An **engine** is a reducer over a graph — evaluates composition, nothing else.
- Each engine implements **exactly one grammar**.
- Engines are **interchangeable** within a grammar — two engines for the same grammar are substitutable.
- Engines own: graph evaluation, node-local state, scheduling within an evaluation, invalidation, error propagation, traces, compound dispatch.
- Engines never own: real-world clock, I/O, scheduling themselves, persistence, user input, networking, rendering.
- Engines **compose across compound boundaries** via a runtime dispatcher, so multi-grammar apps are natural.
- Engines are **deterministic given their inputs**, which is what makes snapshot, replay, and testing possible.

Next: **adapters** — the edge translators that wake engines up and connect them to the world.
