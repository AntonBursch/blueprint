# Blueprint Original vs Reframed

## Purpose

This document translates original Blueprint architecture concepts into the Blueprint Reframed model.

It is designed to answer one practical question:

How does each original concept carry forward into a web-first, new-app-first implementation where Blueprint is a synchronized composition layer over real code?

## Three-Column Mapping

| Original Blueprint Concept | Reframed Equivalent | Immediate Phase 1 Consequence (Web + New Apps) |
| --- | --- | --- |
| App Definition as executable runtime artifact | Graph as canonical composition model, projected to source code | Treat graph as source of composition truth and emit deterministic TypeScript from it |
| Engine executes app graph directly | Compiler pipeline (import -> validate -> compose -> emit) is the core | Prioritize importer/emitter/validator contracts over runtime execution engine development |
| Adapter renders native UI from graph runtime | Framework-native app runs as usual; Blueprint integrates with existing dev tooling | Keep React/Next/Vite runtime unchanged; Blueprint writes generated source into workspace |
| Server stores and serves app definitions | Coordination/governance plane for graph versions, approvals, audit logs | Start local-first; design data model so hosted collaboration can be added without refactor |
| Widgets as runtime primitives | Reusable code artifacts projected as graph-capable nodes | Ingest workspace components/functions/services into graph with stable symbol IDs |
| Processors as graph execution units | Typed composition capabilities compiled to framework-native code | Define node interfaces and deterministic emit rules for supported node families |
| Stores/connections are runtime contracts | Data access and integration are represented in graph, emitted as code wiring | Model connection usage as nodes; emit imports/config/calls, not custom runtime plumbing |
| Channels orchestrate distributed runtime flow | Graph edges represent composition semantics; runtime behavior is emitted natively | Focus on semantic edge contracts and code generation patterns for events/data flow |
| Compounds as reusable graph modules | Compounds as first-class architecture boundaries for composition, ownership, debugging | Build compound metadata now (identity, IO, ownership, provenance, policy, debug scope) |
| AI composes graph safely | Same, with stricter separation between planning and output | LLM can propose graph edits only; compiler is authoritative for final emitted code |
| Multi-platform runtime abstraction first | Web-first proof, then language/platform adapters | Deliver TS+React/Next reference stack before adding non-web/non-TS adapters |
| Full platform ambition from day one | Phased expansion after trust in two pillars | Gate expansion on proven round-trip fidelity and parallel debugging quality |
| Visual builder as central product surface | Dual surface: engineer code-first + consumer graph-first over same core | Build one shared core with two UX layers, not separate systems |
| Runtime inspectability | Native debugging + graph projection | Implement file/line/symbol <-> node mapping and graph highlight on debugger pause |
| Portability through runtime contracts | Portability through shared graph contracts + per-language adapters | Keep core schema stable; isolate importer/emitter/debugger specifics per language |
| Declarative safety model | Deterministic compilation and explicit diagnostics model | Fail fast on unsupported constructs; no silent drops, no hidden behavior |
| Extension ecosystem | Blueprint-friendly libraries with read-only dependency surfaces | Treat third-party packages as invocation-only node catalogs (typed, non-editable internals) |
| App as graph for humans and AI | Same, but grounded in existing codebases and toolchains | Keep native source ownership while making architecture visible and editable as graph |

## What Must Not Change Across the Reframe

1. AI should operate on constrained graph structure, not freeform production code output.
2. Determinism should remain the trust anchor for reviewability, reproducibility, and auditability.
3. Compounds should remain the primary scaling unit for large systems.
4. Human authority over reusable implementation code should remain explicit.

## What Must Change Immediately

1. Blueprint should stop centering custom runtime/rendering as the first implementation target.
2. Code <-> graph round-trip should become a first-class contract.
3. Parallel code and graph debugging should become a first-class contract.
4. Dependency handling should be explicit: workspace editable, third-party read-only internals.

## Phase 1 Build Priorities Derived From This Mapping

1. TypeScript importer for supported top-level constructs.
2. Deterministic TypeScript emitter with stable file outputs.
3. Graph validation with hard errors for unsupported/unsafe states.
4. Compound metadata model wired into graph schema early.
5. Source/node mapping index for code navigation and debug projection.
6. Minimal LLM action surface that applies graph operations only.
7. Native React/Next/Vite hot reload integration through generated source writes.

## How to Use This Document

Use this document as a translation reference when making roadmap or architecture decisions.

If a proposal introduces work that does not clearly map to a row in this table, it should be treated as optional until Phase 1 goals are proven.

## Summary

Blueprint Reframed is not a rejection of original Blueprint.

It is a change in implementation center:

- from runtime ownership to synchronized composition and debugging
- from broad-first execution to focused trust-first execution
- from hidden architecture in code to explicit architecture as graph, with deterministic source output

This mapping keeps original intent while improving adoption realism.
