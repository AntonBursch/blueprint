# App Theory

> **TL;DR** — A Blueprint app is a **function**: inputs come in, something happens, outputs go out. You compose that function by arranging smaller functions visually as a **graph**. The platform has five concrete pillars that make this real: **graphs** (the composition), **nodes** (the functions), **compounds** (reusable, fractal, grammar-switching functions), **engines** (evaluate composition), and **adapters** (connect the top-level function to the world). Grammars are the *rules* for how a given graph composes. Stores are durable memory. Each piece does one job. If you strip away the fancy words, Blueprint is "compose a function visually, then run it anywhere."

This folder documents **what a Blueprint app actually is** — the premises, invariants, and pillars that everything else in the platform rests on.

It is deliberately separate from `specs/`. Specs describe how specific parts of the platform *behave*. App theory describes *what the platform is, conceptually.* When the two disagree, app theory wins and the spec is the thing that must change.

---

## Who this is for

- **Anyone** trying to understand what Blueprint is, before they look at specs, engines, adapters, or the editor.
- **Contributors** deciding whether a proposed feature honors or violates the foundations.
- **Future me, after context loss.** This is the document I re-read to remember why anything is the way it is.

It is **not** for end users. End users never need to know what a "grammar" or an "engine" is. Everything here is platform-builder concern.

---

## Why this matters

Blueprint is making a bet about the near future of software: that app development stays healthy when **humans remain accountable** for the apps that get built, and that accountability is something a platform can structurally support rather than merely ask for.

AI can now produce enormous amounts of software very quickly. That is a tremendous amount of value if the resulting apps are understandable, inspectable, and owned by people — and a harder problem if they are not. The difference between those two outcomes is not whether AI helps build apps. It is whether the artifact a human ends up responsible for is something a human can actually see, reason about, and stand behind.

A composed function, drawn as a graph, is that kind of artifact. A person can look at it, understand it, change it, approve it, reject it, and sign their name to it. An AI can generate it, refine it, explain it, and hand it over. The human stays in the loop not as a formality, but as the one who decides what the app *is*.

There is a second thing this folder is quietly arguing for, almost as a by-product. Decades of work have tried to produce a universal visual language for app development — much of it beautiful inside its niche, difficult to generalize past it. The path described here (function composition at the bottom, pluggable grammars above, adapters at the edges) is our attempt at the general form, made newly practical because AI can sustain the authoring of dense composed graphs alongside people.

The intent, through every pillar in this folder:

- **Non-coders** get real leverage to build real apps.
- **Software engineers** are elevated to higher-value composition and judgement, not displaced.
- **Shipped apps** are ones humans understand and are willing to be accountable for.
- **AI** is freed to produce solutions at a rate and scale that genuinely improves things, because human accountability is part of the medium, not bolted on.

Everything documented here is in service of those four outcomes at once.

---

## Reading order

These documents are ordered so each one builds on the last. Read them in order the first time.

1. **[01-premise.md](01-premise.md)** — *"An app is a function."*
   The single premise the whole platform rests on. If you only read one document, read this one.

2. **[02-composition-by-graph.md](02-composition-by-graph.md)** — *Why graphs.*
   Once apps are functions, composing apps is composing functions. Graphs are pictures of function composition. This document connects the premise to the medium.

3. **[03-invariants.md](03-invariants.md)** — *The five rules that never bend.*
   The invariants that every design decision must honor. These are the contract between the premise and the implementation.

4. **[04-graphs-and-grammars.md](04-graphs-and-grammars.md)** — *Structure and rules.*
   What a graph is made of. What a grammar is. How grammars differ. Why grammars are plural.

5. **[05-nodes.md](05-nodes.md)** — *The smallest function.*
   A node is a function. Processors are node bodies. Ports are the parameter list.

6. **[06-compounds.md](06-compounds.md)** — *The fractal function.*
   A compound is a named, reusable graph-as-function. Compounds host different grammars. Compounds are how Blueprint stays navigable at scale.

7. **[07-engines.md](07-engines.md)** — *The composition evaluator.*
   What an engine is. What it owns. What it must never own. Why engines are plural. How engines compose across compound boundaries.

8. **[08-adapters.md](08-adapters.md)** — *The edge translator.*
   What an adapter is. Why adapters are where apps meet the world. Why adapters are plural. What adapters must never own.

9. **[09-stores.md](09-stores.md)** — *Durable memory.*
   State that outlives a single evaluation. Brief.

10. **[10-separation-of-concerns.md](10-separation-of-concerns.md)** — *The taxonomy.*
    The full map of what goes where. A table you can look up against. Why the separation is worth defending.

11. **[11-the-test.md](11-the-test.md)** — *The success criterion.*
    The single question we measure the platform against.

---

## The pillars in one diagram

```
                           the world
                  HTTP · clocks · screens · sensors · files · networks · humans
                  ─────────────────────────────────────────────────────────────
                                         │
                                         ▼
                               ┌─────────────────┐
                               │    adapters     │    ← edge translators
                               │  (one per edge) │      (task / server / ui / device / …)
                               └─────────────────┘
                                         │
                                         ▼
                          ┌──────────────────────────────┐
                          │         the graph            │    ← the composition
                          │   nodes · edges · compounds  │      (one or more grammars)
                          │                              │
                          │     [ run by engines ]       │    ← composition evaluators
                          │    (one engine per grammar)  │      (DAG / FSM / tree / event / …)
                          └──────────────────────────────┘
                                         │
                                         ▼
                                   ┌───────────┐
                                   │  stores   │    ← durable memory
                                   └───────────┘
```

Each ring does one job. Rings do not reach across. That is the entire architectural idea.

---

## The working taxonomy

A cheat sheet. Expanded in [10-separation-of-concerns.md](10-separation-of-concerns.md).

| Concern | Lives in |
|---|---|
| Graph structure (nodes, edges, ports) | **Graph** |
| How composition is evaluated | **Engine** (one per grammar) |
| What a single node does | **Processor** (a node's function body) |
| Node-local state between evaluations | Engine's state table, passed to the processor |
| When the graph runs | **Adapter** |
| Where inputs come from | **Adapter** |
| Where outputs go | **Adapter** |
| Durable / cross-run memory | **Store** |
| Network calls / external APIs | **Connection** (a processor or specialized adapter) |
| User-facing names & visual metaphor | **Editor** |
| Reusable composed units | **Compounds** |

---

## Non-goals of this folder

This folder deliberately does **not** cover:

- The file format for graphs (see `specs/canonical-form.md`).
- Specific processor libraries or engine implementations (see `specs/` and `engines/`).
- Adapter contracts at the API level (see `specs/engine-contract.md` and adapter repos).
- Editor UX (see editor docs elsewhere).
- Distribution, deployment, ejection, compilation (see `specs/` and roadmaps).
- Business positioning, pricing, go-to-market.

Those are all downstream of the theory. They must honor the theory. They do not define it.

---

## When to come back to this folder

Re-read one or more of these documents whenever:

- A design proposal feels clever but something about it feels off.
- The platform starts being discussed as if it were a specific app type (a dashboard tool, a pipeline runner, a workflow engine).
- Somebody proposes pushing a grammar-specific mechanism into the general platform.
- Somebody suggests users need to learn a new technical term to do a basic thing.
- A feature would require an exception to one of the five invariants.
- A new pillar or category is being proposed (stop; check the theory first).

If a design survives these documents, it is probably honest. If it does not, it is probably off.
