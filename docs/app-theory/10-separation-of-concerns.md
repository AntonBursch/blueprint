# 10 — Separation of Concerns

> **TL;DR** — Each pillar does **one job**. **The graph** *is* the composition. **Nodes / processors** are the leaf functions being composed. **Compounds** are reusable, fractal, grammar-switching functions. **Grammars** are composition rules. **Engines** evaluate composition per grammar. **Adapters** connect the composed function to the world. **Stores** are durable memory. **The editor** is how humans manipulate all of this. The separation is not aesthetic — it is the *only* reason the platform is **portable across app shapes** (same graph, different adapters → different apps) and **scalable in complexity** (compounds keep big apps navigable). Break the separation and Blueprint collapses into a specific-shape-of-app builder. Keep it and Blueprint is a general composition environment.

---

## The master taxonomy

For any concern you encounter while designing, implementing, or discussing Blueprint, find it in this table and put it in the right place. When a new concern comes up, ask which pillar it belongs to. If it doesn't fit any existing pillar cleanly, you are either (a) misreading the concern, or (b) discovering a missing pillar (rare; look twice).

| Concern | Where it lives | Why |
|---|---|---|
| Graph structure: nodes, edges, ports, compound references | **Graph** | The graph is the composition. Structure *is* the artifact. |
| What a single node does (its function body) | **Processor** | Processors are the reusable function bodies; nodes are placements. |
| Node-local state between evaluations | **Engine's state table** (per node), passed to the processor | Isolating state in the engine keeps processors replaceable and state snapshottable. |
| How a graph is evaluated (firing rules, propagation, transitions) | **Engine** (per grammar) | Evaluation is grammar-specific and is the engine's single job. |
| Composition rules (what's legal in this kind of graph) | **Grammar** | Rules are declared as grammar; engines implement them. |
| When the graph runs | **Adapter** | The graph is a function; deciding when to call it is world-facing. |
| Where inputs come from | **Adapter** | The adapter owns the edge between world and graph. |
| Where outputs go | **Adapter** | Same reason — outbound edge. |
| Lifecycle (start, stop, pause, resume, crash recovery) | **Adapter** | Lifecycle is environment-specific. |
| Retry, backoff, rate limit, throttle, dedupe, batch | **Adapter** (usually) | Environment-specific policy belongs at the edge. |
| Call an external API | **Processor** using a **Connection** | A node inside the graph wants to reach the outside; that is a composed operation. |
| Durable / cross-run memory | **Store** | Persistence that outlives a single evaluation. |
| Ephemeral shared state | **Store** (if cross-process) or engine state (if in-process) | Based on whether it must survive a restart. |
| Engine state snapshot / restore | **Adapter** orchestrates, **Store** persists | Adapter decides when; store is where the bytes live. |
| User-facing names and visual metaphor | **Editor** | Grammar words never leak to users. |
| Drag/drop, wiring UI, inspection, previews | **Editor** | The editor is the composition medium for humans. |
| Validation of graph correctness | **Grammar / engine contract** | Graph rules are declared by the grammar; engines enforce. |
| Reusable composed units | **Compounds** | Compounds are the reusable function abstraction. |
| Distribution of composition units | **Compound packages** (with identity and version) | Compounds are first-class artifacts. |
| Grammar dispatch across compound boundaries | **Runtime dispatcher** | A small piece of the core runtime; grammar-agnostic. |
| Networking, sockets, connection pools | **Adapter** or **Connection** | Adapters for edge-driving; connections for processor-driven I/O. |
| Rendering to screen, audio out, haptics | **UI adapter** | Output-to-world is adapter territory. |
| Sensor reads, actuator writes | **Device adapter** | Input/output-to-physical-world is adapter territory. |
| Scheduling, cron, timers | **Task adapter** | Time-based triggering is adapter territory. |
| Authentication / authorization at boundaries | **Adapter** | Auth is environment-specific. |
| Authorization of in-graph actions (business rules) | **Nodes in the graph** | Policy about what a user can do *with the app* is part of the app's composition. |
| Metrics, traces, audit logs | **Adapter** (for edges), **Engine** (for internal evaluation) | Observability has two natural places. |
| Packaging, deployment, hosting | Out of scope here | Covered by deployment specs. |
| Theme, styling, layout | **Tree grammar** + specific processors + **UI adapter** | Composition of presentation is grammar + processors; rendering is adapter. |
| Form of the app on disk (file format) | **Canonical form** (spec) | The serialization of graphs, compounds, and metadata. |

---

## A useful diagram

```
                                   the world
  HTTP • clocks • screens • sensors • files • networks • users • services • devices
  ────────────────────────────────────────────────────────────────────────────────
                                        │
                                        ▼
                  ┌──────────────────────────────────────────┐
                  │                 adapters                 │
                  │  task • server • ui • device • component │   ← edge translators
                  │  messaging • watcher • ... (per edge)    │
                  └──────────────────────────────────────────┘
                                        │   drives
                                        ▼
                  ┌──────────────────────────────────────────┐
                  │             runtime dispatcher           │   ← grammar-agnostic
                  │   (routes compounds to matching engines) │      core
                  └──────────────────────────────────────────┘
                                        │   dispatches
                                        ▼
         ┌──────────────┬──────────────┬──────────────┬──────────────┐
         │  DAG engine  │  FSM engine  │  tree engine │ event engine │   ← one per
         └──────────────┴──────────────┴──────────────┴──────────────┘     grammar
                                        │
                                        ▼
                  ┌──────────────────────────────────────────┐
                  │                  the graph               │
                  │ nodes • edges • compounds (any grammar)  │   ← the composition
                  │         (referencing processors)         │      (the app)
                  └──────────────────────────────────────────┘
                                        │   uses (via connections)
                                        ▼
                  ┌──────────────────────────────────────────┐
                  │                  stores                  │   ← durable memory
                  │      user data • snapshots • config      │
                  └──────────────────────────────────────────┘
```

Read it top-down: **the world** touches the app only through **adapters**; adapters drive the runtime; the runtime dispatches each compound to the right **engine** per grammar; engines evaluate **the graph** (the composition); nodes in the graph use **processors** (possibly using **connections**) to read and write **stores**.

Each ring is single-purpose. Each ring communicates with the next through a narrow, documented interface. That is the architectural idea.

---

## The two claims this separation makes possible

Two big claims rest on this separation. Both are only achievable with clean boundaries.

### Claim 1: Portability across app shapes

**The same composed function can run as different shapes of app by swapping adapters.**

- A graph wrapped in a task adapter is a scheduled collector.
- The same graph wrapped in a server adapter is an API.
- The same graph wrapped in a UI adapter is an interactive tool.
- The same graph wrapped in a device adapter is an edge device controller.

This is only possible if the graph does not know what kind of app it is. Which in turn is only possible if:

- The engine is **I/O-free** (doesn't read from the world).
- The engine is **clock-free** (doesn't know the time of its own accord).
- The engine is **environment-agnostic** (doesn't assume anything about where it is running).

Any violation breaks portability. Keep the separation, and one graph can power many apps.

### Claim 2: Scalability of complexity

**Compounds keep large apps navigable.**

- Users work inside one compound at a time.
- Compounds collapse complex regions into single boxes with a clean interface.
- Compounds nest arbitrarily without the abstraction leaking.
- Compounds are reusable and distributable.

This is only possible if **the function invariant holds at every level of nesting** — which requires that engines, graphs, and the core platform respect compound boundaries rigorously.

Any violation breaks the nesting story. Users end up with flat, unworkable graphs the moment their app gets real. Keep the separation, and Blueprint scales to real-sized apps.

Both claims depend on the same invariants. Both fail together if the separation fails.

---

## Common temptations (and why to resist)

Real design conversations drift. Here are the recurring temptations and the reasoning for resisting each.

### "Let's let the engine read from an external source for convenience."

*No.* The engine must stay I/O-free. Convenience here destroys determinism, replay, snapshot, and portability. If the graph needs data from the outside, a node with a processor (using a connection) brings it in through a declared input. The engine stays in its lane.

### "Let's bake HTTP / a clock / a screen into the engine."

*No.* That welds the engine to one kind of app. The three-app portability argument collapses immediately. Adapters handle these; engines never do.

### "Let's add a DAG-specific optimization to the core."

*No.* That optimization belongs in the DAG engine's implementation. The core graph model stays grammar-neutral. If the optimization is genuinely useful across grammars, generalize it; if not, keep it local.

### "Let's make compounds leak a little state to their parents — it's easier."

*No.* That breaks invariant 2, which is carrying the weight of every scalability claim. Once one compound leaks, all compounds become untrustworthy abstractions, and navigability collapses.

### "Let's make adapters smarter — put a little business logic there so we don't have to add another node."

*No.* Business logic in adapters violates the premise (an app is a function composed in the graph). It also means swapping adapters changes the app's behavior, which defeats portability. Business logic belongs in the graph.

### "Let's expose 'DAG' and 'FSM' as concepts the user has to learn."

*No.* Grammar words are plumbing. Users see natural labels. Leaking grammar terminology to users means the abstraction the platform offers is already broken.

### "Let's have one universal engine handle all grammars to reduce complexity."

*No.* That is a mush. Different grammars have different natural semantics; a universal engine either favors one and contorts the others, or generalizes to the point of doing nothing well. Plural specialist engines, a small dispatcher, and clean compound boundaries give a simpler total system.

### "Let's combine stores and connections — they both do I/O."

*No.* Stores are *where* durable data lives; connections are *how* to reach services. Conflating them produces a worse abstraction. Keep them distinct and let processors use connections to reach stores.

### "Let's let processors reach sideways into ambient state."

*No.* Ambient state breaks the function invariant. Whatever the processor needs should arrive as an input, come from a store accessed through a declared connection, or be handled by an adapter.

---

## What to do when two concerns feel like they could go in either of two places

Sometimes a concern really does have a natural home in more than one place, and you have to choose. A few heuristics:

1. **Which side needs to change more often?** If the concern is environment-specific and changes when the deployment target changes, it's an adapter. If it's about the app's logic and changes when the app's features change, it's in the graph.

2. **Which side does it need to know about?** If the concern needs to know about HTTP, clocks, screens, sensors, it's an adapter. If it only reasons about composed values and business meaning, it's in the graph.

3. **Would the concern be identical across every app?** If yes, it belongs to the platform (engine, core, runtime). If no, it belongs to an app-specific layer (graph, compound, adapter).

4. **Does putting it here violate an invariant?** If yes, move it. If no, it's probably fine.

5. **Which placement makes testing easier?** If placing the concern in the engine makes every test require an HTTP mock, move it to an adapter. If placing it in the adapter makes unit tests for the logic require spinning up a server, move it into the graph.

These aren't rules; they're checks. When in doubt, pick the placement that best preserves the invariants.

---

## The separation is how we defend the premise

The premise — "an app is a function; compose it visually" — can only be honored if the architecture preserves that shape end-to-end. Every piece in the separation is doing one thing in service of that premise:

- **Graph / nodes / compounds** are the composition itself.
- **Grammars** are the different ways composition can be organized.
- **Engines** evaluate composition, cleanly, without worldly contamination.
- **Adapters** connect the composition to the world without polluting it.
- **Stores** keep data alive without entering the composition.
- **The editor** lets humans do the composing.

Each pillar stays narrow. Together they make the premise operational.

A violation of the separation is not a stylistic issue. It is an attack on the premise. If the engine starts owning I/O, the app is no longer a clean function. If compounds leak state, the fractal abstraction fails. If adapters start computing business logic, the graph is no longer the composition. If grammar words reach users, the "no costume" promise is broken.

Defending the separation is defending the premise. That is why this is the longest document in the folder.

---

## Summary

- Each pillar does one job; together they implement the premise.
- The taxonomy tells you where any concern belongs.
- Two big claims — **portability across app shapes** and **scalability of complexity** — depend entirely on honoring the separation.
- Common temptations erode the separation; resist them.
- When in doubt between two placements, the one that best preserves the invariants wins.

Next: the single test we measure the platform against.
