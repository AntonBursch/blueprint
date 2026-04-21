# 05 — Nodes

> **TL;DR** — A **node** is the smallest function in Blueprint. It has **input ports** (its parameter list), **output ports** (its return), and a **processor** (its function body). A node may be pure or stateful, synchronous or asynchronous, side-effecting or not. Nodes are the atoms of composition. They are grammar-agnostic in shape — every grammar has nodes — but the meaning of a node's behavior, and what its edges connect, depends on the grammar. Processors are **what nodes do**; nodes are the graph-visible handle to a processor, parameterized by config and wired into ports.

---

## A node is a function

Restating invariant 1 from [03-invariants.md](03-invariants.md):

> Every node is a function. Input-side → transformation → output-side.

A node has:

- **A name / id** — unique within its compound.
- **A processor reference** — which function body the node is running.
- **Config** — values that parameterize the processor (how this specific instance of the processor is tuned).
- **Input ports** — named slots where values arrive.
- **Output ports** — named slots where values leave.
- **Optional node-local state** — persisted between evaluations, for stateful processors.
- **Metadata** — label, position (for the editor), notes, anything else that doesn't affect behavior.

That's the whole anatomy. Everything a node does is captured by those fields.

### Analogy to a function in code

If you wrote a node in code it would look like:

```js
// The processor (function body):
function average({ values, window }, state) {
  const next = [...state.recent, values].slice(-window);
  return {
    out: { mean: mean(next) },
    nextState: { recent: next }
  };
}

// The node (an instance of the processor, placed in a compound):
{
  id: "avg-temperature",
  processor: "@blueprint/core/rolling-average",
  config: { window: 10 },
  inputs:  { values: "sensor-feed.reading" },
  outputs: { mean: "dashboard.display" }
}
```

The processor is defined once, anywhere in the ecosystem. The node is a placement of that processor into a specific graph with specific wiring. Many nodes in many graphs can share the same processor.

---

## Ports

Ports are named slots. Two kinds:

- **Input ports** receive values.
- **Output ports** produce values.

A port declares:
- **A name** unique within its direction on that node.
- **A kind / type** — what kind of value it carries (scalar, structured, event, stream, whatever the grammar supports).
- **Multiplicity** — single value, sequence, optional, required, etc. (Grammar-dependent.)
- **Default / fallback behavior** — what happens if no value arrives. (Grammar-dependent.)

Edges connect an output port on one node to an input port on another node. The connection is typed: ports must be compatible.

### Why names, not positions

Ports are named rather than positional because:
- Visual wiring is clearer when the port has a human-readable label.
- Processors can evolve (add optional ports, reorder, deprecate) without breaking existing graphs.
- Accessibility and generated tools rely on names.

Positional argument lists (first input, second input, …) would be more compact to serialize, but the cost in editor clarity and refactorability is too high. Named ports won.

### Port compatibility

Two ports are compatible if:
1. Their kinds / types are compatible (output kind is a subtype of, or equal to, the input kind).
2. Their multiplicity rules are compatible.
3. They belong to the same grammar (you can't wire a DAG output directly to an FSM state's entry without going through a compound boundary, because the edges mean different things).

Cross-grammar wiring happens **only through compound boundaries**. That is where grammar boundaries live; that is where grammar translation is legal. See [06-compounds.md](06-compounds.md).

---

## The processor is the function body

A **processor** is a reusable function body that nodes reference.

Think of it as the analogue of a function definition in code. Processors:

- Have a stable identity (`@namespace/package/name@version`).
- Define their input ports and output ports (the function signature).
- Define their config shape (how they can be parameterized).
- Contain the implementation — the actual code (or composition) that runs when the node fires.
- May declare whether they are **stateless** or **stateful**.
- May declare what grammars they are valid in (e.g., "DAG only," "FSM only," "any grammar that supports timed nodes").

A processor can be:

- **Built-in** — shipped with an engine or the core (e.g., `@blueprint/core/identity`, `@blueprint/core/rolling-average`).
- **From a package** — published by the community or enterprise teams.
- **A user-defined compound** — a compound itself can be used as a processor. When a compound is used as a processor, nodes referencing it are **instances** of that compound.

Processors are versioned and published the same way as compounds. The two are closely related: a **leaf processor** is an atomic function body (code); a **compound processor** is a function body expressed as a graph.

See the processor spec (`specs/` and the processors package) for the actual contract.

### Stateless vs. stateful processors

- **Stateless processors** are pure functions of their inputs and config. No memory between evaluations. The same inputs always produce the same outputs.
- **Stateful processors** carry **node-local state** — memory that persists across evaluations. Each time the node fires, it receives its previous state as an additional input and may return a new state alongside its outputs.

Both are nodes. Both honor invariant 1. Stateful processors simply have a richer "function body" that includes a state transition.

The state of a stateful node is:
- **Declared by the processor** — the processor defines the shape.
- **Owned by the engine** — the engine passes the previous state in and stores the new state after firing.
- **Snapshotable** — state can be serialized and restored, enabling pause/resume, time travel debugging, migration, etc.
- **Part of the node**, not the graph — two nodes using the same processor have independent state.

Node-local state is **not** the same as durable store state. Durable, cross-run, shared data belongs in a **store** (see [09-stores.md](09-stores.md)). Node-local state is for the processor's internal memory.

---

## Config vs. inputs

Both config and inputs feed a node, but they are different:

- **Config** is part of the node's definition. It is known at graph-authoring time. It does not change per evaluation. It parameterizes the processor (e.g., "window size = 10," "endpoint = /api/users," "threshold = 0.5").
- **Inputs** arrive at runtime, through edges, from upstream nodes or from the graph's outer ports. They change each time the node fires.

Roughly: **config is what the node is**. **Inputs are what the node is acting on right now.**

Confusing the two causes the same problems as confusing a function's definition with its arguments. Keep them separate.

Some values can reasonably be either config or an input depending on use case. A "threshold" is config if it never changes during a run; it is an input if another node computes it dynamically. The graph author picks.

---

## Node cadence — when a node fires

When a node fires is a grammar-dependent question:

- **In a DAG**, a node fires when its inputs are ready. "Ready" is engine-specific (all inputs present, any input changed, adapter-provided tick, etc.). Stateful nodes may fire even with unchanged inputs if they emit on a schedule.
- **In an FSM**, nodes are states; "firing" is entering or exiting a state or evaluating a guarded transition.
- **In a tree**, nodes fire as part of layout / render passes propagated from their parent.
- **In an event graph**, nodes fire when their subscribed topics publish.
- **In a reactive signal graph**, nodes fire when any of their dependencies change value.

The grammar's engine is responsible for deciding when. The node's processor just declares *what to do when it fires.*

This separation — "what" in the processor, "when" in the engine — is a consequence of invariant 4 (grammars are calling conventions). The processor is invariant across grammars where it applies; the firing rule belongs to the grammar.

---

## Side effects

Nodes can have side effects. Emitting to a store, calling an external API, writing a file, rendering to the screen — all valid.

Constraints on side effects:

1. **Side effects must flow through an output port, a store, or an adapter-provided channel** — not through ambient globals.
2. **Side effects should be declared** by the processor metadata (for tooling to reason about them).
3. **Engines may guarantee ordering** of side effects within an evaluation (grammar-dependent).

Side effects are not evil. They are how an app affects the world. But they should be traceable through the graph — you should be able to look at a node and know what effects it emits, not hunt through invisible globals.

---

## Errors

When a node's processor fails — throws, rejects, returns a malformed value — the engine handles it according to the grammar's error policy:

- Catch and emit an error on an error port (if the node declares one).
- Fail the whole evaluation.
- Log and continue.
- Retry.
- Propagate to a parent compound's error port.

Error handling is a grammar concern. Individual grammars spec it out (see each grammar's engine contract). What is invariant:

- Errors are values. They can flow through ports. They can be handled by downstream nodes.
- Errors do not leak the platform's implementation details to the user.
- Errors are observable in the editor during development.

---

## Nodes and the editor

In the editor, a node is rendered as a box with ports on its sides. Its processor identity and config are shown in some form of inspector panel. Its inputs and outputs are drawn as wires.

A node may also render a **preview** — a small live rendering of what it is currently producing (a value, a chart, a widget, a sparkline). This matters hugely for usability. See the discussion of "immediate visible feedback" in [02-composition-by-graph.md](02-composition-by-graph.md). Scene graphs work because you can see the scene; Blueprint nodes should be equally inspectable.

Editor details are beyond the scope of this document. The essential theory point: the node is the unit of visible manipulation. Drag a node; drop it on the canvas; wire its ports. That is the gesture of composition.

---

## What nodes are *not*

To keep the concept clean:

- Nodes are not classes or objects. They don't have methods. They have a single function body (the processor), possibly with state.
- Nodes are not threads, processes, or services. How they're scheduled is the engine's and adapter's business. The node is just a function.
- Nodes are not always atomic. A compound used as a processor is a perfectly valid node body — see [06-compounds.md](06-compounds.md). "Atomic node" and "compound node" are just two ways a node can be implemented.
- Nodes are not user-visible names. The user sees "Rolling Average" or "HTTP Request" or "Submit Form." They don't see "node," "processor," "port." The editor translates.

---

## Summary

- A **node** is a function in the graph.
- A node has **input ports**, **output ports**, **config**, and a **processor** (function body).
- A **processor** is the reusable function body; it has stable identity and version.
- Processors can be **stateless** (pure) or **stateful** (with node-local state).
- **Ports** are named, typed slots; edges connect ports.
- **When** a node fires depends on the grammar; **what** it does when it fires is the processor.
- Side effects must flow through declared channels, never ambient globals.
- Nodes are the visible, draggable atoms of composition; what the user sees and arranges.

Next: **compounds** — reusable, fractal, grammar-switching functions built from graphs of nodes.
