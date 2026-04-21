# 02 — Composition by Graph

> **TL;DR** — If an app is a function, and functions compose, then composing an app is composing functions. A **graph** is the natural picture of function composition: nodes are functions, edges are "this function's output becomes that function's input." Different graph shapes (DAG, state machine, tree, event graph) correspond to different *rules* for how the inner functions compose, but the underlying thing is always the same. Programmers have invented a hundred syntactic costumes around function composition (OOP, FP, reactive, event-driven, request/response, game loops). A visual composition environment strips the costumes off. If users must learn a costume-word to use Blueprint, Blueprint has failed.

---

## From premise to medium

The premise said: an app is a function.

A single function can be built from smaller functions. `f(x) = g(h(x))` is a function built from two others. This is composition, and it is how all non-trivial programs are made, in every language, in every paradigm, on every platform. The name changes (functions, methods, components, handlers, operators, stages, nodes, blocks) but the activity is the same: **take smaller pieces that each do one thing, and arrange them so the whole does the larger thing.**

Programmers do this activity constantly. They call it "programming."

What Blueprint is proposing is: **let the arrangement be a picture, not syntax.**

---

## A graph is a picture of composition

A function-composition picture looks like this:

```
         ┌────────┐
  in ─→  │   A    │ ─→ ┌────────┐
         └────────┘    │        │
                       │   C    │ ─→ out
         ┌────────┐    │        │
  in ─→  │   B    │ ─→ └────────┘
         └────────┘
```

Boxes are functions. Lines carry values between them. If you wrote this in code it would be something like `out = C(A(in), B(in))`. The picture and the expression say the same thing. They are interchangeable.

**A graph is just the picture, formalized.** Nodes are functions. Edges are data flowing between them. The composition *is* the graph; the graph *is* the composition. They are not two different things, one describing the other. They are the same thing in two representations.

This is why visual programming keeps showing up across unrelated domains:

- **Shader graphs** in 3D (Unreal, Unity, Blender, Houdini) — the node graph *is* the shader.
- **Audio patching** (Max/MSP, Pure Data, Reaktor) — the patch *is* the synthesizer.
- **Dataflow** (Apache Beam, Dagster, Airflow UI) — the pipeline diagram *is* the pipeline.
- **Automation** (Node-RED, n8n, Zapier, Make) — the flow *is* the integration.
- **Visual scripting** (Unreal Blueprint, Scratch, App Inventor) — the blocks *are* the program.
- **Scene graphs** (every 3D tool, every game engine) — the scene graph *is* the scene.

Every one of those systems is solving the same underlying problem (compose functions visually) for a specific kind of function composition. Each one stays in its niche because the domain-specific assumptions are baked in at the bottom. Blueprint is the attempt to do this generally, by keeping function composition at the bottom and letting everything domain-specific live in pluggable layers above.

---

## Why this is a coherent idea, not a gimmick

Visual programming has a reputation for being cute, toy-like, or niche. It is worth addressing that directly.

Visual programming has succeeded spectacularly in certain domains (scene graphs, shaders, audio, IoT wiring, motion graphics) and failed to generalize outside them. The failures have a common pattern, and so do the successes. Understanding both is worth doing, because Blueprint has to thread the needle.

### Where visual composition wins

The cases where graphs beat text — and often beat text by a huge margin — share several properties:

1. **The composition is dense.** Many small pieces fit together in non-obvious ways. Text linearizes this badly; pictures show it directly.
2. **The relationships are structural.** "This feeds into that" is a relationship text can only describe indirectly, while a picture shows it.
3. **The user's mental model is already spatial.** Audio signal chains, 3D scenes, hardware wiring — the real thing is already a picture in the user's head.
4. **Immediate visible feedback.** The user changes the graph and sees the effect right now.

When those four properties hold, visual composition is not just viable — it is *dominant*. Every professional in those domains uses visual tools. No amount of "well, you could do it in code" wins against the picture.

### Where visual composition loses

Visual composition has generally failed when:

1. **The composition tries to be all of the program,** including fine-grained syntax (individual integer arithmetic, string manipulation, loop bodies). At that grain size the boxes become a clumsier version of text.
2. **The graph gets too big to see.** Hundreds of nodes on one canvas, wire spaghetti everywhere, no navigation story.
3. **The structure is hidden inside node properties.** Boxes become opaque — all the real logic is in text fields on each node — and the graph becomes a skin over a config file.
4. **There is no way to zoom out.** Users can't collapse, reuse, or navigate; the tool only has one level.

Every failed visual-programming tool failed on at least one of those four. The successful ones succeeded because they addressed all four.

### What Blueprint must do

Blueprint's bet is that the winning properties can be preserved at the platform level, and the failure modes are all addressable architecturally:

- **Don't try to be all of the program.** Individual node bodies can be code, or fine-grained visual tools if that fits the domain. The graph is about **composition**, not **expressions**. (See [05-nodes.md](05-nodes.md).)
- **Stay navigable at scale.** This is what compounds are for. You live inside one compound at a time, like a function. (See [06-compounds.md](06-compounds.md).)
- **Make nodes inspectable.** Every node and compound should be seeable — its inputs, outputs, current values, what it does. Scene graphs work because you can see the scene. Blueprint must make compounds equally inspectable.
- **Zoom is first-class.** Compounds are the zoom mechanism. The editor lets you move up and down the compound hierarchy as a primary navigation.

Whether Blueprint pulls this off is a product and implementation question, not a theoretical one. Theoretically, function composition is visualizable. Practically, keeping it visualizable at scale is the craft.

---

## The grammars

One thing complicates the simple "graph = composition" picture: **not all composition works the same way.**

Consider:

- In a **dataflow** composition, functions transform values that flow through them. "This output feeds that input" is forward and deterministic.
- In a **state machine**, the composition is about transitions: "when we are in state A and event X happens, go to state B." The graph has cycles and different semantics on its edges.
- In a **containment tree**, the composition is "this thing is inside that thing" — a parent-child relationship, not a flow.
- In an **event / pub-sub** composition, functions fire when things are announced, and the graph is not strictly evaluated from start to end but reacts as messages arrive.
- In a **reactive signal** composition, values flow continuously and updates propagate automatically when their dependencies change.

These are **different kinds of composition**. They each have a natural graph shape. They each have their own rules for how the graph "runs." They are all valid; they are all useful; no single one is the "right" way to compose functions.

A **grammar** in Blueprint is the name for one of these composition-rule sets. A grammar says:

- What kinds of nodes are allowed.
- What edges mean in this grammar.
- How the graph evaluates (forward flow, state-and-transition, parent-child, fire-and-subscribe, continuous propagation, etc.).
- What counts as an "input" and "output" at this grammar's level.

Different grammars correspond to different ways of thinking about composition. A real app often benefits from more than one:

- An outer containment structure (pages, panels — a tree).
- Data flowing through computations (dataflow — a DAG).
- Multi-step processes (workflows, form stages — a state machine).
- Reactions to external events (notifications, alerts — pub-sub).

Blueprint supports multiple grammars and lets them **nest inside each other** through compounds. See [04-graphs-and-grammars.md](04-graphs-and-grammars.md) and [06-compounds.md](06-compounds.md) for how.

The key point: **the underlying premise (an app is a function, compose functions to build apps) does not change across grammars.** Only the rules for *how* the composition works change. The function-in/function-out shape is invariant.

---

## The costume argument

Programming looks like a hundred different activities because programmers have invented a hundred **syntactic costumes** around function composition:

- **Object-oriented programming** → composition expressed as methods on objects and inheritance.
- **Functional programming** → composition expressed as function application, currying, higher-order functions.
- **Reactive programming** → composition expressed as streams and operators.
- **Event-driven programming** → composition expressed as event handlers and emit calls.
- **Request/response / MVC** → composition expressed as routes, controllers, services.
- **Game loops** → composition expressed as update/render methods on entities.
- **Coroutines / async** → composition expressed as awaits and yields.
- **Effects / algebraic effects** → composition expressed as handlers and performs.

Each costume has strengths. Each has partisans. Each has produced working software. But they are all dressing on the same animal: **something takes stuff and makes stuff, and we arrange many such things so the whole arrangement does the larger thing.**

The costume obscures the thing. Users who cannot write code are not lacking some mysterious programming gene. They are excluded because the costume is what we present to them when we say "programming," and costumes are hard to learn.

**A visual composition environment strips the costume off.** You drag a function, draw a line to another function, and the arrangement is what happens. No classes, no monads, no handlers, no subscribers, no routes — unless you choose to use them, and even then only because they fit a specific problem, not because the medium requires them.

This is the promise of Blueprint: **composition without syntax.** Not "programming without concepts." Not "apps without understanding." Just: the composition, visible and manipulable, without having to learn a costume to express it.

---

## The user-facing corollary

If users must learn the words "DAG," "FSM," "reactive," "tree," "pub-sub," "compound," "grammar," "processor," "engine," or "adapter" to use Blueprint, **Blueprint has failed.**

These words are the plumber's names for things in the pipes. Users should never see them. Users see things like:

- "Pages and panels" (what the platform calls a *containment tree*).
- "Data flow" or "Calculation" (what the platform calls a *DAG*).
- "Workflow" or "Steps" (what the platform calls an *FSM*).
- "When this, then that" (what the platform calls a *pub-sub* or *event* grammar).
- "Section" or "Reusable piece" (what the platform calls a *compound*).

The grammar is the calling convention underneath. The label is what the human sees. The editor is responsible for the translation — see the invariants document and the editor docs (forthcoming).

This is the one-way valve: **grammar words can never leak into user-facing UI.** If they ever appear, that is a platform bug, not a user bug.

---

## Summary

- An app is a function; composing an app is composing functions.
- A graph is the natural picture of function composition.
- Nodes are functions. Edges are values flowing between them.
- Different "kinds of composition" correspond to different **grammars**.
- Grammars are the *rules* for how a graph composes; the underlying premise is invariant.
- Visual composition wins when it is dense, structural, spatial, and immediately visible — and loses when it tries to be syntax, spaghettifies, hides logic, or can't zoom.
- Programmers have built many **syntactic costumes** around function composition. Blueprint strips the costumes off.
- **Users must never need to learn the costume-words.** The editor presents natural, domain-appropriate labels; the grammar words live in the platform, not the UI.

From here, we document the **invariants** that any implementation of this idea must honor.
