# 03 — The Invariants

> **TL;DR** — Five rules that must hold everywhere in Blueprint, at every layer, in every grammar, at every scale. If a proposed design breaks one of them, the design is wrong. **(1)** Every node is a function. **(2)** Every compound is a function. **(3)** The app itself is a function; adapters are its edges. **(4)** Grammars are calling conventions — they change *how* inner functions compose, never the function-in/function-out shape. **(5)** Inputs and outputs are open-ended; adapters introduce new edge kinds freely. These five keep the platform honest.

---

## Why invariants

The premise ("an app is a function") is the top-level truth claim. The pillars (graph, nodes, compounds, engines, adapters, stores) are the concrete things we build to make the premise real. The **invariants** are the rules that connect the two — the contract that every pillar must honor, so that the premise stays true as the platform grows.

Without invariants, the premise is just an aspiration. With invariants, every design decision has a check: *does this honor the invariant, or does it violate it?* If the invariant holds, the decision is safe. If it doesn't, the decision is wrong — regardless of how clever it is.

These invariants are deliberately few and deliberately boring. Their job is to be unambiguous and hold at every scale.

---

## Invariant 1 — Every node is a function

**A node takes inputs, does something, produces outputs.** That is all a node does. Ever.

- A node may be pure (stateless) or stateful.
- A node may be synchronous or asynchronous.
- A node may fire once, repeatedly, conditionally, or never.
- A node may have side effects; those effects are part of its "something happens."
- A node may have zero inputs (a source), zero outputs (a sink), or any number of each.

But: **the shape is always "input-side → transformation → output-side."**

### What this rules out

- Nodes that reach outside the platform to grab things on their own, without those things being inputs. (That is a hidden input; make it explicit or it will bite.)
- Nodes that emit outputs through some side-channel that isn't a declared output port. (That is a hidden output; same problem.)
- Nodes whose behavior depends on global graph state. (That is implicit input; makes composition impossible to reason about.)

### Why this matters

Because if every node is a function, then:
- Nodes are **composable** in the mathematical sense (one's output becomes another's input).
- Nodes are **inspectable** (given an input, you can see what output comes out).
- Nodes are **replaceable** (any node with the same input/output shape can stand in).
- Nodes are **analyzable** (static tooling can reason about the graph).

The moment a node starts reaching sideways or upward for data or side channels, all four of these properties break.

### User-facing phrasing

Users don't hear "function." They hear things like "block," "piece," "thing," "step" — whatever the editor's metaphor is. The invariant is about what the platform *guarantees*, not what the user *says*.

---

## Invariant 2 — Every compound is also a function

**A compound is a named, reusable graph-as-function.** It has input ports, output ports, and internal contents. From the outside, it is a black box with an input-side and an output-side. From the inside, it is another graph of nodes (possibly using a different grammar).

This is the **fractal invariant**. It says: the "function" abstraction holds at every level of nesting.

- A leaf node is a function.
- A compound of 5 nodes is a function.
- A compound of 200 nodes, using three grammars, with 40 nested compounds inside it, is still a function.
- The top-level app is a function.

Each level presents **input-side → transformation → output-side** to the level above. None of them need to know how they are composed internally.

### What this rules out

- Compounds that leak internal state directly to their callers.
- Compounds that only make sense inside a specific parent grammar.
- Compounds whose outputs depend on things other than their declared inputs and their durable state.
- Special "magic" compounds that break the function abstraction for convenience.

### Why this matters

Because this is how Blueprint stays tractable at scale. Real apps have thousands of moving parts. Nobody can see thousands of nodes at once. The answer is to **collapse regions into named compounds** and navigate the hierarchy, zooming in only where you currently care.

If compounds were not functions, zooming would break the abstraction — opening a compound would sometimes require understanding its parent's internals. With the function invariant, a compound is self-contained. You can open it, understand it, and fix it without needing to know where it is used.

This is the single invariant that makes compounds safely nestable. Without it, Blueprint apps would unscale the moment they got real.

### Related: why compounds can switch grammars

Because a compound is a function from the outside, its caller does not need to know what grammar it uses inside. A DAG parent can contain an FSM compound, which can contain a tree compound, which can contain a DAG compound, which can contain an event compound. Each is a function to its parent. Each uses whatever grammar fits its internal composition best.

See [04-graphs-and-grammars.md](04-graphs-and-grammars.md) and [06-compounds.md](06-compounds.md).

---

## Invariant 3 — The app itself is a function; adapters are its edges

**The root of the app is a top-level compound.** It has inputs coming in from somewhere and outputs going out to somewhere. The "somewhere" is what adapters are.

An **adapter** is what sits on the outermost edges of the root function and translates between the graph and the world:

- A task adapter says: "call the graph on a schedule; deliver the clock tick as an input; take the graph's outputs and write them wherever they need to go."
- A server adapter says: "call the graph on HTTP requests; deliver the request as an input; take the graph's output and return it as the HTTP response."
- A UI adapter says: "call the graph on user events; deliver those events as inputs; take the graph's outputs and render them to the screen."
- A device adapter says: "call the graph when sensors change; deliver sensor readings as inputs; take the graph's outputs and drive actuators."

The root function does not know and does not care which adapter is driving it. It just sees inputs arriving and produces outputs. This is why **the same graph can run in different edge configurations** (as a server, as a task, as a UI, etc.) by swapping adapters.

### What this rules out

- Graphs that bake "I am an HTTP server" or "I am a background job" or "I am a dashboard" into their structure. That is the adapter's job, not the graph's.
- Engines that know about HTTP, clocks, screens, or sensors. That is the adapter's job.
- Adapters that reach into graph internals to get data. Adapters only touch the declared edges.

### Why this matters

Because **portability across app shapes is the whole point.** If the root graph knows what kind of app it is, the platform is not a composition environment — it is a specific kind of app builder with a composition feature.

By keeping the graph agnostic and the adapter specialized, one composition can power many shapes of app. That is what makes Blueprint a platform rather than a tool.

### What counts as an edge

An edge of the root function is any point where the composed function needs to interact with the world. Common categories:

- **Time** (clock ticks, schedules, deadlines, frame rates).
- **User actions** (clicks, keypresses, gestures, voice).
- **Network I/O** (HTTP, WebSocket, MQTT, gRPC, etc.).
- **Persistent storage** (database reads/writes, file I/O).
- **Sensors / actuators** (physical devices).
- **External services** (third-party APIs).
- **Lifecycle** (mount, unmount, start, stop, suspend, resume).

Each of these is some adapter's responsibility. The root graph just declares ports that say "I expect this kind of input" and "I produce this kind of output"; the adapter connects them to the world.

---

## Invariant 4 — Grammars are calling conventions

**Different grammars specify different rules for how inner functions pass control and data among themselves.** They do *not* change the fundamental shape of each individual function (which is always input-side → transformation → output-side).

- A **DAG** grammar says: "inner functions are composed as forward dataflow; data flows along edges from upstream to downstream; no cycles."
- An **FSM** grammar says: "inner functions are composed as states and transitions; exactly one state is active at a time; events drive transitions."
- A **tree** grammar says: "inner functions are composed as parent-child containment; parents own children; changes propagate down."
- An **event / pub-sub** grammar says: "inner functions fire when topics are published; no strict topology."
- A **reactive signal** grammar says: "inner functions are composed as dependency-tracked signals; updates propagate automatically."
- A **behavior tree** grammar says: "inner functions are composed as hierarchical selection and sequencing."

Each grammar is a self-contained way of composing functions. Each has its own engine (see [07-engines.md](07-engines.md)). Each has its own visual language in the editor.

### What this rules out

- Grammars that define what a *function* is differently. Every grammar composes functions. The function concept is invariant.
- Grammar-specific mechanisms that leak into the general platform. (Example: the DAG engine's dirty-tracking optimization must not show up in the core graph type; it is a DAG-engine implementation detail.)
- A "universal grammar" that tries to be all grammars at once. Different compositions need different rules; conflating them produces a mush that does none well.

### Why grammars are plural

Because different relationships want different graph shapes, and forcing everything into one shape hurts everything:

- A UI's widget tree is **not** a DAG. Trying to make it one forces awkward constructs for children and containment.
- A workflow is **not** a DAG. Trying to make it one forces awkward reification of "current state."
- A dataflow pipeline is **not** an FSM. Trying to make it one forces awkward fake "states" for every computation step.
- An event bus is **not** a DAG. Trying to make it one forces either wire-spaghetti or hidden indirection.

Each composition kind deserves its own natural expression. Grammars provide that, while the function invariant keeps them all interoperable at compound boundaries.

### Why grammars must compose

Because a real app mixes kinds of composition:

- The outer app is a *containment* (pages → panels → widgets → …).
- Inside a widget, logic might be *dataflow* (inputs feed computations feed display).
- Inside a logic step, control might be a *workflow* (draft → submitted → approved).
- On top of everything, *events* flow (alerts, notifications, user actions).

The mechanism that makes this safe is **compounds**. A compound with grammar X can live inside a compound with grammar Y, because both present the same function interface. See [06-compounds.md](06-compounds.md).

---

## Invariant 5 — Inputs and outputs are open-ended

**Anything can be an input. Anything can be an output. The platform must not privilege any particular kind.**

Examples of what can be an input:
- Clock ticks, schedules, time expressions.
- HTTP requests, WebSocket messages, MQTT publications.
- User actions: mouse, keyboard, touch, gestures, voice.
- Sensor readings from real or virtual devices.
- Database changes, file changes, queue messages.
- Lifecycle events: mount, unmount, visibility, focus, resume.
- Outputs from other apps or systems.
- Nothing at all (a pure source that produces on its own cadence).

Examples of what can be an output:
- Pixels, rendered DOM, native widgets.
- Rows in a database, files on disk, queue messages.
- HTTP responses, WebSocket broadcasts, gRPC responses.
- Motor commands, lights, servos, speakers.
- Emails, SMS, push notifications, logs, telemetry.
- Nothing at all (`void` — a pure side effect).

### What this rules out

- Engines that hardcode input or output assumptions (e.g., "the graph always reads from HTTP").
- Grammar specs that assume a particular input cadence (e.g., "ticks are always clock-based").
- Core platform types that enumerate input or output kinds exhaustively. The platform must accept new edge kinds being added over time.

### Why this matters

Because the space of apps is the space of possible edge configurations. If the platform decides up-front what the edges can be, it decides what apps can exist. The whole point is to not decide that.

### How this plays out

- **The engine** takes "inputs" as opaque values arriving on ports. It does not know whether a value came from a sensor or an HTTP request.
- **The adapter** knows everything about its edge kind (how to wait for requests, how to schedule, how to render) and translates to and from the engine's opaque values.
- **New adapters** can be added without touching engines, grammars, or existing graphs. A new edge kind = a new adapter.

This is why Blueprint can grow to cover new app shapes just by adding adapters, not by modifying the core.

---

## How to use these invariants

When evaluating any proposed design — a new feature, a new primitive, a new subsystem, a new optimization, a new concept — ask these questions, in order:

1. **Does it preserve "every node is a function"?**
   - If the proposal involves a node reaching sideways or upward, or having an undeclared input/output, stop. That violates invariant 1.

2. **Does it preserve "every compound is a function"?**
   - If the proposal requires a compound to expose its internals to its parent, stop. That violates invariant 2.

3. **Does it keep the graph agnostic to edge configuration?**
   - If the proposal bakes HTTP, a clock, a screen, a database, or any other specific edge into the graph or engine, stop. That violates invariant 3 — it belongs in an adapter.

4. **Does it respect the grammar boundary?**
   - If the proposal adds grammar-specific mechanism (e.g., DAG topology optimization) to the general platform, stop. That belongs inside the relevant grammar's engine, not the platform core.

5. **Does it leave input/output kinds open?**
   - If the proposal requires a specific kind of input or output to exist at the platform level, stop. That belongs in adapters.

A design that passes all five is probably safe. A design that violates any one of them is probably wrong.

Invariants override convenience, cleverness, elegance, and pragmatism. They are the check against drift. Everything else is negotiable; these are not.

---

## What the invariants deliberately do *not* say

The invariants are tight on **shape** and silent on **implementation**. They do not dictate:

- How graphs are serialized. (File format is a spec, not an invariant.)
- How many grammars exist, or which ones. (The first is DAG; more will follow as needed.)
- How engines are written (language, runtime, style). (There can be many implementations of one grammar's engine; the contract is what matters.)
- How adapters are written. (Each platform will have its own.)
- How the editor presents anything. (UX is a separate discipline.)
- What processors ship out of the box. (Node libraries are downstream.)
- How Blueprint apps are deployed, compiled, packaged, or ejected. (Those are downstream concerns.)

This is deliberate. The invariants must survive every one of those decisions, and the only way that is possible is if they say nothing about any of them.

---

## Summary

1. **Every node is a function.** Inputs in, transformation, outputs out.
2. **Every compound is a function.** Same shape, at every nesting depth.
3. **The app itself is a function; adapters are its edges.** Edges live in adapters, not in the graph or engine.
4. **Grammars are calling conventions.** The shape of function-in/function-out is invariant across grammars.
5. **Inputs and outputs are open-ended.** New edge kinds come via adapters; the core does not enumerate.

These five are the contract between the premise and the implementation. Every other document in this folder is what happens when you take these seriously and build from there.
