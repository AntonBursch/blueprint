# 09 — Stores

> **TL;DR** — A **store** is durable memory that outlives a single evaluation. Engine state is in-memory; node-local state is per-node and per-evaluation; store state is **persistent and often shared**. Stores hold user data (customers, orders, documents), cross-run engine state (snapshots), configuration, and anything else that must survive restarts. Stores are accessed through processors (typically via connections) or through the adapter (for snapshot/restore). Stores are a **separate pillar**, not part of the engine and not part of the graph — the graph composes logic; stores remember.

---

## Why stores are their own pillar

Memory in a Blueprint app lives at multiple scopes:

| Scope | Where | Example | Lifetime |
|---|---|---|---|
| Evaluation-local | The engine's working memory during one tick | Intermediate values on edges | One evaluation |
| Node-local | Per-node state the engine carries across evaluations | A rolling average's sample buffer | Across evaluations within a run |
| Session / run | The engine's full state during one run | The app's current state | Until the process exits |
| Durable | A store | User accounts, order history, learned preferences | Across restarts, possibly shared across processes |

The first three are engine concerns. The last one is what **stores** are for.

Stores are not just "databases." They are **the durable memory of the app** — whatever must survive a restart, a redeployment, or a cold start. That includes user data, shared state, learned models, audit logs, snapshots of engine state, or anything else that needs to persist.

---

## What a store is

A store is an abstraction over durable storage. Concretely, a store:

- Has **identity** (e.g., a named store in the app's declaration).
- Has a **kind** (key-value, relational, document, blob, append-only, etc.).
- Has a **schema or type declaration** (may be loose or strict, depending on the kind).
- Is accessed through **a connection or adapter** (the actual I/O to whatever backs the store — Postgres, SQLite, Redis, S3, filesystem, etc.).
- Is named and referenced by graphs, adapters, or processors that need it.

A store is not a specific database product. It is the platform-level notion of "durable memory in this app." Many stores may be backed by the same physical database; many apps may share a store; one app may have many stores.

---

## How stores interact with the rest of the platform

### With engines

Engines do not directly touch stores. Engines work in-memory, deterministically.

Cases where stores meet engines:

- **Engine snapshots.** Periodically, or at well-defined boundaries, the adapter snapshots the engine's state to a store. On restart, the adapter reads the snapshot from the store and rehydrates the engine. The engine itself does not know a store was involved — it just receives its initial state from the adapter.
- **Engine run logs / traces.** For observability, an adapter may record evaluation traces to a store.

Engines never read or write stores during evaluation. Determinism depends on this.

### With processors (nodes)

Most interaction with stores happens through processors. A node whose purpose is "write this value to the customer table" is a processor that, internally, uses a connection to reach the store and perform the write.

This keeps the engine unaware of I/O (invariant of [07-engines.md](07-engines.md)) while still letting the graph express operations that hit durable storage. The graph says "this node writes to the customer store"; the processor does the actual work through a connection.

### With adapters

Adapters interact with stores for:

- Snapshot/restore of engine state across runs.
- Cross-run adapter state (a task adapter may remember "last successful run time" in a store, for example).
- Environment-specific persistent concerns (session storage, rate-limit counters, etc.).

### With compounds

A compound may declare dependencies on stores: "this compound expects to be wired to a store that provides an `orders` table." Parameterizing compounds by store references lets the same compound be reused across different deployments backed by different actual stores.

### With graphs

Graphs don't directly reference stores, except through the nodes (processors) that use them. Store references are effectively parameters: the graph declares "I use a store called X"; the deployment binds X to a concrete backend.

---

## What stores are *not*

### Stores are not the engine's runtime state

The engine's in-memory state during a run is not in a store. It's in the engine. If the app crashes, the engine's state is lost — unless the adapter has been snapshotting to a store.

Mixing these up leads to engines that try to read/write stores during evaluation, which breaks determinism and turns every evaluation into an I/O-bound operation.

### Stores are not caches

A cache is a performance optimization for expensive reads. Caches can live in processors, in connections, in adapters, or in the engine's evaluation-local memory. Stores are *durable state*, not caches.

### Stores are not graphs

A store doesn't contain logic or composition; it just contains data. Logic lives in graphs and processors. Stores are the passive persistence layer.

### Stores are not the only way to persist

A compound's declared state, an adapter's lifecycle hooks, an engine's snapshot — all involve persistence in some way, but not all are "stores." A store is specifically the named, durable memory abstraction for app data.

---

## Stores and the multi-grammar story

Stores cross grammar boundaries cleanly. A DAG compound might write to a store; an FSM compound might read the same store; a tree compound might render data that came from a store query earlier.

This works because stores are outside the graph. They are durable state that any grammar-native operation (through a processor) can touch, as long as the graph has been wired to the store.

Nothing about stores is grammar-specific. Any grammar can have processors that read and write them.

---

## Stores and portability

Because stores are parameterizable, the same graph can run against different stores in different environments:

- In development: SQLite backing a set of stores.
- In production: Postgres for relational stores, S3 for blob stores, Redis for ephemeral shared state.
- In a demo: in-memory stores that reset on every run.

The graph is unchanged. The deployment decides which concrete backends are wired. This is analogous to how adapters work for edges: the graph is portable; the wiring to the outside world is deployment-specific.

---

## Stores are a separate concern for a reason

Stores could conceivably be rolled into other pillars:

- "Stores are a kind of connection." — Yes, implementation-wise, but conceptually stores are where data lives, not how we reach it. Conflating them obscures the abstraction.
- "Stores are a kind of adapter." — No, adapters drive the engine; stores are passive.
- "Stores are a kind of processor." — No, processors consume stores, but a store isn't a function body.
- "Stores should just be in the graph." — No, then the graph becomes sensitive to storage backend choices, defeating portability.

Stores are their own pillar because **where durable data lives** is fundamentally different from **how it is computed on**, **how the graph is evaluated**, or **how the app meets the world**. They deserve their own concept.

---

## Not deeply specified here

Unlike engines and adapters, stores are not central to the theoretical framing of what Blueprint *is*. They are a necessary support pillar — apps without durable memory are rare and limited — but the theory of Blueprint doesn't hinge on exactly how stores work.

The details of store shapes, contracts, connection protocols, and backends are in `specs/store-contract.md` and the store packages. This document just names stores as a pillar, describes their role at the theory level, and positions them relative to the other pillars.

---

## Summary

- A **store** is durable memory that outlives a single evaluation.
- Stores hold: user data, engine snapshots, cross-run adapter state, any other persistent concern.
- The graph and engine are memoryless beyond their in-memory state; stores are where persistence lives.
- Stores are accessed by **processors** (via connections) during normal operation, and by **adapters** for snapshot/restore.
- Engines never touch stores directly during evaluation — preserving determinism.
- Stores are **parameterizable**: same graph, different backends per environment.
- Stores are a **separate pillar** because "where durable data lives" is a distinct concern from the others.

Next: the full **taxonomy** — a single map showing where every concern lives, and the argument for why the separation is worth defending.
