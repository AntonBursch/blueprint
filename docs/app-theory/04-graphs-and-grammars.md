# 04 — Graphs and Grammars

> **TL;DR** — A **graph** is a set of nodes (functions) connected by edges (data paths). A **grammar** is the rule set governing how a particular graph composes — what edges mean, how the graph evaluates, what an input and output are at that level. Blueprint is grammar-agnostic at its core: the same compound mechanism hosts any grammar (DAG, FSM, tree, event, signal, behavior tree, and others). Different grammars exist because different compositional relationships need different rules, and forcing one grammar to be universal hurts everything. Grammars compose across compound boundaries.

---

## What a graph is

A graph in Blueprint consists of:

- **Nodes.** Each node is a function. It has input ports, output ports, and a processor (its function body). See [05-nodes.md](05-nodes.md).
- **Edges.** Each edge connects an output port of one node to an input port of another node. Edges carry values (or, in some grammars, control, events, or containment relationships — see below).
- **Ports.** Named input or output slots on a node. Edges connect ports. Ports declare the kind of value they accept or produce.
- **Metadata.** Identity, labels, positions (for the editor), configuration.

At the file-format level, a graph is just a data structure. See `specs/canonical-form.md` for the actual on-disk representation. This document is about the *concept*.

A graph is always **contained inside a compound** — the top-level app is a compound, and every subgraph is a compound too. There is no graph without a compound around it. See [06-compounds.md](06-compounds.md).

---

## What a grammar is

A **grammar** is the rule set that governs how a particular graph composes.

Different grammars define:

1. **What nodes mean** — what a node is in this grammar (a transformation, a state, a container, a subscriber, etc.).
2. **What edges mean** — what it means to connect two ports (value flow, state transition, parent-child, event subscription, reactive dependency, etc.).
3. **How evaluation works** — given the current state of the world, what happens next? (forward pass, state transition, parent-down propagation, message delivery, etc.).
4. **What inputs and outputs are at the graph's outer ports** — what does it mean when the graph receives an input from outside, and what does the graph's output mean to the outside?
5. **What constraints the graph must satisfy** — acyclicity, unique initial state, single parent, well-formed transitions, etc.

A grammar is paired with an **engine** that runs it (see [07-engines.md](07-engines.md)). The engine knows how to honor the grammar's rules.

### Example: the DAG grammar

- **Nodes** are transformations: input-values → output-values, optionally with node-local state.
- **Edges** are value flows from an upstream output port to a downstream input port.
- **Evaluation** is forward: when inputs arrive, the engine walks the graph in topological order and fires nodes whose inputs are ready.
- **Graph's outer ports** receive inputs from outside (adapter delivers) and produce outputs to outside (adapter consumes).
- **Constraints**: acyclic. Ports have declared types. Outputs flow only forward.

### Example: the FSM grammar (forthcoming)

- **Nodes** are states.
- **Edges** are transitions, guarded by event conditions and optional actions.
- **Evaluation**: exactly one state is active; when an event arrives, matching outgoing transitions are evaluated; on a match, the state changes and the action fires.
- **Graph's outer ports**: the graph emits events (state changes) and accepts events (external triggers).
- **Constraints**: one initial state. Transitions have guards. States can have entry/exit actions.

### Example: the tree grammar (for containment)

- **Nodes** are containers (panels, groups, widgets, scenes, whatever the tree represents).
- **Edges** are parent-child relationships.
- **Evaluation**: downward propagation (layout, theme, context) and upward bubbling (events).
- **Graph's outer ports**: the root is the container's "result"; its children are its composition.
- **Constraints**: each node has exactly one parent; no cycles.

### Example: the event / pub-sub grammar (forthcoming)

- **Nodes** are publishers, subscribers, or both.
- **Edges** are subscriptions (possibly implicit via topic names).
- **Evaluation**: when a node publishes, all subscribed nodes fire.
- **Graph's outer ports**: external events enter as publications; outputs leave as publications to external topics.
- **Constraints**: topics are named; subscription is explicit.

Other grammars are possible: **reactive signal**, **behavior tree**, **rule engine**, **petri net**, **stream/dataflow with time**, **workflow with approvals**, etc. Each is a specialist in some kind of composition.

Blueprint does not commit to a fixed list. New grammars can be added over time, and they can coexist, because **compounds let grammars nest.**

---

## Why grammars are plural

A single grammar cannot cover every kind of compositional relationship gracefully. Forcing one to fit all applications produces bad compositions:

- **DAG forced to model UI containment** — you end up with awkward explicit parent-pointer edges and no natural place for layout. Real UI frameworks use trees for a reason.
- **DAG forced to model workflow state** — you end up with explicit "current state" values on every node and guarded activation logic baked into every node. Real workflow engines use state machines for a reason.
- **FSM forced to model dataflow** — every value transformation becomes a state and every edge becomes an event; the state space explodes. Real dataflow systems use DAGs for a reason.
- **Tree forced to model events** — you end up either flattening the tree into a bus or duplicating event wiring across parents. Real event systems use pub-sub for a reason.

Each existing graph shape has succeeded in its domain because it fits the domain's natural compositional structure. Blueprint respects that by making the grammar a first-class pluggable concept, not a fixed choice.

The alternative — a single "universal" grammar that tries to be all things — either (a) picks one shape and forces others into awkward workarounds, or (b) tries to generalize to the point where it loses the specificity that makes any one grammar useful. Both outcomes are bad. Better to have multiple grammars and compose them.

---

## Why the function invariant still holds across grammars

This is the subtle but essential point.

A grammar changes the *rules* of composition. It does not change the *shape* of what is being composed. Every grammar still composes **functions**:

- A DAG composes functions via forward dataflow.
- An FSM composes functions via state-and-transition, where each state's *entry/exit actions* are functions and *transition guards* are functions.
- A tree composes functions via parent-child containment, where each node is a function that takes its context and produces its rendered output (or its resolved layout, etc.).
- An event graph composes functions where publishers and subscribers are both functions; the composition is who fires when.
- A behavior tree composes functions as hierarchical selectors and sequences of leaf-action functions.

In every grammar, the leaf is a function. Composition rules vary. The thing being composed does not.

This is **invariant 4** from [03-invariants.md](03-invariants.md) and it is what lets grammars interoperate through compounds.

---

## How grammars nest

Compounds are the seams where grammars meet. Two key mechanics:

### 1. A compound declares its internal grammar

When you create a compound, you pick the grammar it uses internally. That choice governs:
- What nodes are allowed inside.
- What edges mean inside.
- Which engine runs it.

A compound with grammar X can be placed inside a parent with grammar Y. The parent sees the compound as a function — it has input ports and output ports, like any node. The parent doesn't know or care what happens inside.

### 2. Each grammar defines its own external function interface

A grammar specifies what a compound's **outer ports** look like to the parent. For a DAG compound, outer ports are input/output ports carrying values. For an FSM compound, outer ports are typically "events accepted" and "events emitted" plus the current state as a readable value. For a tree compound, outer ports might be the root container as a renderable output.

In every case, the outer interface is **function-shaped**: inputs, outputs. The parent grammar sees a function. The inner grammar's rules apply only inside the compound boundary.

### Illustrative nesting

```
[Root compound, grammar = tree]                          ← containment
  └── [Panel compound, grammar = tree]
       ├── [Widget compound, grammar = DAG]              ← dataflow inside a widget
       │    ├── [API call node]
       │    ├── [Transform node]
       │    ├── [Chart rendering node]
       │    └── [Submit flow compound, grammar = FSM]    ← workflow inside a dataflow step
       │         ├── [Draft state]
       │         ├── [Submitting state]
       │         ├── [Submitted state]
       │         └── [Error state]
       └── [Notifications compound, grammar = event]     ← pub-sub inside a panel
            ├── [Alert subscriber]
            └── [Log subscriber]
```

Each compound uses the grammar that naturally fits what it is composing. Each compound is a function to its parent. The platform runs them together by dispatching each compound to its matching engine; compound boundaries are where the dispatch happens.

---

## What grammars are *not*

To keep the concept clean, a few things worth explicitly excluding:

- **Grammars are not languages.** They don't have syntax in the programming-language sense. They are structural rule sets over a graph.
- **Grammars are not "paradigms" in the vague hand-wavy sense.** They are concrete rule sets that engines implement.
- **Grammars are not user-facing.** Users don't pick "DAG" or "FSM." They pick something like "Data Flow," "Workflow," "Layout," "When/Then." The editor translates. See [03-invariants.md](03-invariants.md) and the editor docs.
- **Grammars are not type systems.** Ports carry typed values, and that type system is shared across grammars. The grammar is about *composition rules*, not *value types*.
- **Grammars are not runtimes.** The engine is the runtime. A grammar is a specification; an engine is an implementation of that specification.

---

## When to reach for a new grammar

A new grammar makes sense when:

- A class of compositional relationship keeps coming up that fits awkwardly into every existing grammar.
- Users in a domain are forced to encode natural structure as workarounds (flags on nodes, hidden edges, imperative runtime logic) because no grammar supports it directly.
- The cost of adding the grammar (spec + engine + editor support) is less than the accumulated cost of workarounds.

A new grammar does **not** make sense when:

- The relationship can be naturally expressed as a compound in an existing grammar.
- The proposed grammar is just a flavor of an existing one.
- The motivation is convenience for one specific app rather than a compositional pattern.

Adding a grammar is expensive (new engine, new editor affordances, new teaching burden). Adding one carelessly is worse than doing nothing. The bar should be high.

---

## Graph identity and versioning

A graph does not exist in isolation. It has identity:

- **Compound identity.** Every compound has a stable name (e.g., `@namespace/package/name`) and version. Compounds can be distributed as packages.
- **Grammar identity.** Every grammar has a stable identifier (e.g., `grammars/dag`, `grammars/fsm`).
- **Engine identity.** Engines declare which grammar (and which version of it) they implement.

At runtime, a graph says: "I am a compound `foo/bar@1.2.0`, my grammar is `dag@1.0`, please dispatch me to an engine implementing that grammar." The platform's runtime uses that declaration to wire it up. This is how multi-grammar apps find their engines and how grammar evolution stays safe.

Full detail is in the specs (`specs/versioning.md`, `specs/vocabulary/compound.md`). This document just names that identity exists and matters.

---

## Summary

- A **graph** is nodes + edges + ports + metadata. It is always inside a compound.
- A **grammar** is the rule set governing how that graph composes.
- Different grammars exist because different compositional relationships need different rules.
- Every grammar still composes **functions** — the function invariant holds across grammars.
- **Compounds** are the seams where grammars nest. A compound with grammar X can live inside a parent with grammar Y.
- Grammars are **platform-builder concerns**; users never see grammar names, only natural labels for what each grammar represents.
- Adding a new grammar is a significant commitment; the bar is high.

Next: the smallest unit of composition — the node — and the function bodies (processors) that live inside them.
