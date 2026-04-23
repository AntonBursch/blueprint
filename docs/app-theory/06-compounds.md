# 06 — Compounds

> **TL;DR** — A **compound** is a named, reusable function whose body is another graph. It presents **input ports and output ports** to the outside world, just like a node. Inside, it contains nodes, edges, and (optionally) further compounds, using some grammar. Compounds are the single most important mechanism in Blueprint. They are **how apps stay navigable at scale** (they are the zoom), **how grammars nest** (a compound's internal grammar can differ from its parent's), **how pieces are reused** (compounds are distributable packages with stable identity), and **how the fractal function invariant holds** (every level of nesting is a function).

---

## A compound is a fractal function

Invariant 2 from [03-invariants.md](03-invariants.md):

> Every compound is also a function. Input-side → transformation → output-side.

This is the invariant that makes compounds safe to nest arbitrarily. It is worth stating again because everything else in this document is a consequence of it.

- A compound with **2 nodes** inside is a function.
- A compound with **200 nodes** inside is a function.
- A compound with **2 grammars** inside is a function.
- A compound with **20 nested compounds** inside (each possibly using different grammars) is a function.
- A compound at **depth 15** of the compound hierarchy is a function.

Each level, from the outside, looks like: input ports in, transformation happens, output ports out. The caller doesn't know or care what's inside. That is the whole mechanism that keeps compounds safe to stack.

---

## Anatomy of a compound

A compound has:

- **Identity.** A stable name (e.g., `@namespace/package/compound-name`) and version. Compounds can be distributed, depended on, and versioned like any other software artifact.
- **Outer ports.** Named, typed inputs and outputs — the function signature from the outside.
- **Config schema.** How this compound can be parameterized when instantiated. (Optional.)
- **Inner grammar.** Which grammar governs the inside. Declared explicitly.
- **Inner graph.** The nodes, edges, and nested compounds inside. Must be valid under the declared grammar.
- **Metadata.** Description, documentation, author, icon, category, whatever helps the editor display and users understand it.
- **Optional boundary contracts.** Pre/post conditions, type declarations, lifecycle hooks — grammar-dependent.

That is the whole anatomy. A compound is just a graph with a signature, an identity, and a declared grammar.

---

## Compounds in relation to nodes

**A node is a placement. A compound is a definition.** Nodes reference processors. Processors can be either leaf processors (atomic code) or compounds.

When a compound is used as a processor:

- Each node referencing that compound is an **instance** of it.
- The instance has its own config values (if the compound has a config schema).
- The instance has its own state (each instance's internal state is independent).
- The instance appears in its parent graph as a single box, with the compound's declared input and output ports visible on its edges.

Opening the instance drills into the compound's internal graph. The editor usually represents this as a zoom-in or a split panel.

Because a compound can be used like any other processor, **compounds are first-class building blocks**. A user can assemble a library of compounds and compose apps from them the same way they compose apps from atomic nodes. This is the economic leverage of the platform — domain experts ship compounds; users drag them together.

---

## The six jobs a compound does

Compounds earn their central place by doing six things at once:

### 1. Reuse

A compound can be placed many times in the same graph or across different graphs. It is a unit of reusable composition — the way functions are in code.

### 2. Scoping and hiding

Things inside a compound are hidden from the outside. The parent sees only the outer ports. Internal nodes, internal state, and internal complexity are encapsulated.

This is what makes large apps tractable. Users opening a compound to edit it see only that compound's internals; they don't have to hold the whole app in their head.

### 3. Distribution

Compounds are packages. They have identity and version; they can be published, depended on, and shared. Enterprise teams ship internal compound packages; the ecosystem ships public ones. Apps import compounds the same way code imports libraries.

This is the mechanism for a Blueprint ecosystem to exist. Without compound distribution, everyone starts from scratch. With it, a shared body of well-made compounds accumulates over time.

### 4. Grammar switching

A compound's inner grammar can differ from its parent's. This is the seam where grammars nest:

- A DAG compound can contain an FSM compound.
- An FSM compound can contain a tree compound.
- A tree compound can contain a DAG compound.
- Any mixture, at any depth.

Because each compound presents the same function interface (input-side → transformation → output-side), grammars can nest freely. The parent's grammar doesn't need to know the child's internal grammar.

### 5. Lazy loading and distribution of runtime

Because compounds have stable identity and a declared interface, the runtime can **load them lazily**. The engine doesn't need every compound's internals in memory to run the outer graph; it can fetch and initialize each compound on first use.

This has two consequences that matter at scale:
- **Startup is fast.** Apps don't pay for compounds that aren't yet exercised.
- **Compounds can run in different processes, machines, or runtimes.** A compound could be a local function, a remote service, or a worker on a different machine — all invoked through the same function interface. The platform can place compounds wherever they belong.

This is how Blueprint supports distributed and hybrid deployment without changing the graph.

### 6. Debugging and observability boundaries

Compounds are natural places to hang debugging affordances:
- Breakpoints at compound boundaries.
- Tracing of values flowing across outer ports.
- Snapshots of compound state.
- Error boundaries (a failure inside a compound can be contained).
- Performance metrics attributed per compound.

Because a compound is a function with a clean boundary, it is a natural unit of observability. Tooling can reason about compounds the way IDEs reason about functions.

---

## Compounds and the function invariant

The function invariant is the mechanism that makes the six jobs above possible.

- **Reuse** works because a function can be invoked from many places.
- **Scoping** works because a function's internals are private.
- **Distribution** works because a function has a name, version, and signature.
- **Grammar switching** works because a function's external shape is grammar-independent.
- **Lazy loading** works because a function can be instantiated on demand.
- **Debugging boundaries** work because a function has a clear input/output interface to observe.

Break the invariant (let compounds leak state, let parents reach into children, let compounds depend on their usage context) and every one of the six jobs breaks. That is why invariant 2 is non-negotiable: it is carrying the weight of the whole platform.

---

## Compounds as parameterized templates

Compounds can be **parameterized** via config. The config shape is declared on the compound and filled in when an instance is placed.

Parameterization turns compounds into **templates**. Examples:

- A "rolling average" compound parameterized by window size.
- A "form stage" FSM compound parameterized by field list and validation rules.
- A "dashboard card" compound parameterized by title, data source, and chart type.
- A "CRUD list" compound parameterized by entity schema.

Parameterization is what separates a compound library from a repository of copy-paste snippets. Parameterized compounds are the reusable assets; un-parameterized compounds are one-off pieces.

The config shape is declared with the compound; the editor uses the declaration to render config inputs when the user places an instance.

---

## Identity and versioning

Every compound has stable identity: `@namespace/package/compound-name`, plus a version.

Why versioning matters:

- Graphs can pin a specific version.
- Breaking changes to a compound require a version bump; existing graphs continue to work.
- Migrations can be authored as a compound evolves.
- Enterprise teams can standardize on specific versions of internal compounds.

The compound format spec (in `specs/`) covers the concrete versioning rules. This document just names that versioning exists and is central.

---

## Compounds and the editor

Compounds are the core unit of **navigation** in the editor.

The user works **inside one compound at a time**, the way a developer works inside one function at a time. The compound under edit fills the canvas. Nested compounds appear as boxes with ports; opening one drills in.

Some editor idioms that fall out of this:

- **Breadcrumbs.** Show where in the compound hierarchy you are.
- **Split view / tabs.** Edit one compound while viewing another.
- **Map view.** Show the compound hierarchy as a tree or radial map; navigate to any compound.
- **Outline.** Show the nodes and nested compounds of the current compound as a list.
- **Search.** "Find all compounds that use this store." "Find every instance of this compound."

At scale, users spend most of their time in the map view, in search results, or inside a single focused compound. The canvas is not a single giant scrollable surface; it is one compound at a time.

This is the same navigation pattern that keeps filesystems, codebases, and 3D scene graphs tractable at scale. Compounds give Blueprint that pattern.

Editor specifics are beyond this document. The essential theory point: **compounds are the zoom**, and the zoom is what keeps Blueprint usable as apps grow.

---

## Anti-patterns

Some things compounds should **not** be used for:

### Compounds as namespaces

Using compounds purely to group related nodes visually, with no meaningful input/output interface, turns compounds into folders. That is fine for small things but defeats most of the six jobs. If a compound has no inputs or outputs, ask whether it really should be a compound or just visual grouping in the editor.

### Compounds as god objects

Compounds that have dozens of input ports and dozens of output ports, and try to do everything, are not providing function-shaped abstraction. They are just hiding a mess. Break them apart.

### Compounds with hidden side effects

Compounds that reach outside their declared interface (mutating stores directly without their parent knowing, calling external services without declaring it) break the function abstraction. All effects should flow through declared ports (outputs) or go through explicit store connections the parent can see.

### Circular compound usage

A compound referencing itself directly would be infinite recursion at graph-construction time. Recursion in composition, if needed at all, needs an explicit recursive grammar (which is a language design decision, not a compound usage pattern). Compound graphs are generally DAGs of compound references.

---

## Related: groups and templates (authoring-time conveniences)

The anti-patterns above all share a shape: users reaching for compounds to satisfy a need that isn't function abstraction. Two lighter-weight authoring constructs absorb those needs without disturbing the compound pillar:

- **Groups** — visual regions on the canvas (a labeled box around a cluster of nodes). Groups are pure authoring-time metadata. They have no identity, no ports, no version, and do not exist at runtime; deleting a group leaves the nodes untouched. This is the right answer to "I just want these organized on the canvas."
- **Templates** — named, reusable *snippets* of nodes that the editor can stamp out. Each stamp is **independent** after insertion (the nodes are copied, not referenced), so editing the template never reaches back to prior stamps. Templates have no runtime presence either; they are an editor/library convenience for patterns the author keeps re-assembling.

Neither is a compound. Neither affects engines, adapters, or the canonical runtime graph. They form a natural ladder with compounds:

| Construct | Identity | Ports | Updates propagate | Runtime artifact |
|---|---|---|---|---|
| **Group** | no | none | — | none |
| **Template** | template itself has identity; stamps don't | none | no (detached on stamp) | none |
| **Compound** | yes, versioned | yes | yes | yes |

Supporting groups and templates *protects* the compound abstraction — it gives authors the right-sized construct for grouping and repetition, so they stop pressuring compounds into doing jobs compounds shouldn't do. Conversion up the ladder (group → compound via "Extract Compound"; compound → inline via "Inline Compound"; group → template via "Save as Template") is a first-class editor affordance, not a theory concern.

These are editor/tooling concepts, not app-theory pillars, which is why they are named here as *related* and left to specs and the editor to detail.

---

## Summary

- A **compound** is a named, reusable graph wrapped as a function.
- A compound has identity, version, outer ports, an inner grammar, and an inner graph.
- Compounds are **first-class building blocks** — nodes reference compounds the same way they reference atomic processors.
- Compounds do **six jobs at once**: reuse, scoping, distribution, grammar-switching, lazy loading, debugging boundaries.
- The **function invariant is what makes compounds safe to nest**; break it and every benefit collapses.
- Compounds are **parameterized templates** via config.
- Compounds are the **zoom** in the editor; users navigate by hierarchy, not by scrolling.
- Without compounds, Blueprint doesn't scale. They are not a convenience feature; they are structural.

Next: the **engine** — the thing that actually evaluates a graph.
