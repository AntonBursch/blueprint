# Blueprint Reframed

## Thesis

Blueprint should be treated as a **bidirectional composition and debugging layer over real code**, not as a custom UI runtime.

In this model:

- engineers continue building applications in normal frameworks and languages
- Blueprint projects those applications into a graph representation
- an LLM composes application structure through that graph
- Blueprint deterministically translates graph changes back into source code
- the real application preview continues to run through the native framework toolchain
- debugging happens against the real runtime, with Blueprint providing a synchronized graph projection

The result is a single system where **code, graph, LLM composition, runtime preview, and debugging remain aligned**.

## Why This Direction Is Better

The original instinct behind Blueprint was correct: applications have a graph structure that is hard to see in raw imperative code. The reframed model sharpens that insight.

Instead of trying to own rendering, runtime, and composition all at once, Blueprint becomes the layer that:

- makes application structure visible
- gives the LLM a safe constrained medium for composition
- preserves normal developer workflows
- keeps the final application as native source code
- enables a graph-native debugging experience on top of native debuggers

This makes Blueprint easier to adopt, easier to trust, and easier to scale across frameworks and languages.

## Product Definition

Blueprint is a **synchronized composition system** with five projections of the same application:

1. **Code**
   Human-authored and generated source files in the application's native language.

2. **Graph**
   A structured semantic representation of the app that can be inspected, edited, composed, diffed, and audited.

3. **LLM Conversation**
   A constrained planning surface where the LLM proposes and applies graph changes instead of emitting arbitrary production code.

4. **Running App**
   The actual application running through its normal dev server, runtime, framework, and bundler.

5. **Debug View**
   A synchronized execution projection that maps runtime code locations and state back onto graph nodes and edges.

Blueprint is not replacing the runtime. Blueprint is making the runtime inspectable and composable.

## The Two Core Requirements

The entire approach depends on two things working reliably.

### 1. Code <-> Graph Round-Trip

Blueprint must be able to:

- import source code into a graph
- preserve stable identity for imported symbols
- let humans and LLMs compose and modify the graph
- deterministically emit source code from the graph
- preserve application behavior across the round-trip

This is the primary trust contract.

If developers cannot move between code and graph without losing meaning, Blueprint becomes a side tool instead of a core platform.

### 2. Parallel Debugging

Blueprint must be able to:

- run and debug the real application code using native debuggers
- map runtime locations back to graph nodes and edges
- let engineers move between code stepping and graph stepping
- support graph-aware breakpoints, inspection, and flow highlighting

This is what makes the graph feel real instead of decorative.

## High-Level Workflow

### Engineer Workflow

An engineer works almost exactly as they do today.

1. Open a normal workspace.
2. Run the normal app preview or dev server.
3. Turn on Blueprint.
4. Blueprint imports project structure into a graph projection.
5. The engineer composes application changes through conversation with the LLM.
6. Blueprint translates approved graph changes into deterministic source code updates.
7. The dev server hot reloads the actual app.
8. The engineer can inspect the result as:
   - source code
   - graph structure
   - running UI
   - debugger state

Blueprint becomes an augmentation layer over normal development, not a replacement IDE or replacement runtime.

### Non-Engineer Workflow

A non-engineer uses a more guided editor.

1. They never need to work directly in source code.
2. They compose the app through graph-first tools and an LLM conversation.
3. Blueprint still produces the same underlying graph and deterministic source code.
4. Publish and update flows remain stable because the same compiler and runtime model are used.

This means Blueprint can support both code-first and graph-first users without splitting into two fundamentally different products.

## What the LLM Should and Should Not Do

The LLM should be used in two very different modes.

### Mode A: Reusable Code Creation

The LLM can help engineers create reusable building blocks such as:

- components
- hooks
- services
- adapters
- transforms
- data access helpers
- domain logic

This is normal code generation and refactoring, under engineer review.

### Mode B: App Composition

The LLM should not freeform-generate the final assembled application.

Instead, it should:

- inspect the graph
- propose graph edits
- connect approved nodes
- configure nodes through constrained metadata
- rely on deterministic compilation for final source emission

This is the key separation.

**Reusable code may be LLM-authored under supervision. Final app assembly should be graph-composed and compiler-translated.**

That separation is what makes the system auditable.

## Editable Code vs Read-Only Dependencies

Blueprint should distinguish between two sources of code.

### Editable Workspace Code

This is project-owned source code.

Capabilities:

- import to graph
- compose in graph
- emit back to source
- round-trip support
- normal debugging support

### Read-Only Dependency Code

This is external published code, such as packages from `node_modules`.

Capabilities:

- inspect type surfaces and exported APIs
- represent APIs as graph nodes
- use those nodes in app composition
- emit imports and usage sites
- never rewrite dependency internals

This boundary is correct and necessary.

When developers are building an application, they are usually not editing third-party libraries. They need typed access to those capabilities, not ownership of their internals.

Blueprint should therefore support:

- full round-trip for workspace-owned code
- invocation-only or interface-only nodes for third-party code

## Native Runtime Preview

One of the most important implications of this model is that Blueprint does not need to own UI rendering.

For example, in a React application:

- React still renders the UI
- Vite/Next still run the app
- the app still hot reloads normally
- Blueprint writes or updates source files deterministically
- the real app preview updates through the normal framework loop

So the user can see the application in three synchronized ways:

1. as code
2. as graph
3. as a running application

This dramatically reduces product complexity and keeps developers inside familiar workflows.

## Parallel Debugging Model

Parallel debugging is the second pillar of the system.

### Runtime Truth

The real debugger attaches to the actual running code.

The runtime remains the source of truth.

### Graph Projection

Blueprint maintains a mapping between:

- generated file locations
- imported source symbols
- graph node IDs
- graph edge IDs where applicable

At a minimum, this allows:

- selecting a node and jumping to code
- selecting code and jumping to the node
- highlighting the active node when the debugger pauses
- setting graph breakpoints that translate to code breakpoints

This is analogous to debugging React components even though the JavaScript runtime is what is actually executing.

Blueprint would provide a graph-level execution lens over native debugging infrastructure.

## Canonical Representation

To make the system stable, Blueprint needs a canonical representation.

The correct choice is:

**The graph is the canonical semantic model. Source code and runtime views are projections of that model.**

That does not mean code becomes secondary. It means the graph is the medium that:

- the LLM can safely operate on
- the compiler can deterministically emit from
- the debugger can map into
- both engineers and non-engineers can understand

The graph must therefore preserve:

- stable symbol identity
- source provenance
- ownership metadata
- imports and exports
- type signatures
- supported runtime semantics
- mapping information for debugging and code navigation

## Deterministic Compilation

A core property of Blueprint is that final app code should be generated by deterministic code, not by the LLM.

That means:

- same graph + same compiler version + same node library = same emitted code
- generated source is diffable and reviewable
- behavior changes can be traced to graph changes
- audits are possible because the transformation is explicit and repeatable

This is the main reason the approach is trustworthy at scale.

## Recommended Architecture

### Shared Core

These parts should be shared across all languages:

- graph domain model
- composition engine
- validation framework
- provenance model
- debug mapping model
- compiler orchestration

### Per-Language Adapters

These parts should be implemented per language:

- importer: source -> graph
- emitter: graph -> source
- language-specific validation rules
- language-specific formatting
- debugger integration specifics

This architecture allows Blueprint to scale to TypeScript, Python, C#, Java, Go, Rust, and other languages without rebuilding the platform every time.

## Design Principles

### 1. Real Code Stays Real

Blueprint should never trap users in a proprietary runtime model.

### 2. The Graph Must Be Trustworthy

If the graph hides behavior, loses identity, or silently drops semantics, the whole product fails.

### 3. Deterministic Compiler, Not Opaque LLM Output

The LLM can plan. The compiler must produce.

### 4. Native Tooling Is an Asset

Existing frameworks, dev servers, bundlers, and debuggers should be reused whenever possible.

### 5. One System, Two Surfaces

Engineers and non-engineers should use different UX layers over the same graph and compiler, not different underlying systems.

### 6. Read-Only Dependencies Stay Read-Only

Third-party libraries can become nodes, but not compiler-owned source.

## Practical Boundary of the First Version

The first real version of this model should be intentionally narrow.

Recommended first target:

- TypeScript only
- workspace source only
- top-level variables, functions, classes, and imports
- read-only package API surfaces
- graph composition for a constrained subset of app structure
- deterministic file emission into stable generated paths
- code <-> graph navigation
- basic debugger location mapping

The first version does not need to solve every language or every construct. It needs to establish trust in the two core pillars:

- round-trip
- parallel debugging

## Product Interpretation

Blueprint, in this reframed model, is best understood as:

- a code-aware graph compiler
- a safe composition layer for LLM-assisted app building
- a synchronized graph/code/runtime debugger surface
- a bridge between engineer workflows and consumer-friendly app composition

That is why Blueprint makes more sense this way.

The system becomes easier to explain:

- code defines reusable building blocks
- graph defines application composition
- LLM operates on graph structure
- compiler emits native source code
- dev servers run the real app
- debuggers attach to the real runtime
- Blueprint keeps every view synchronized

## Summary

If this direction is taken seriously, Blueprint becomes:

**the synchronized layer between human intent, LLM planning, graph composition, deterministic code generation, native runtime preview, and native debugging.**

That is a cleaner architecture, a stronger developer story, a safer AI story, and a more plausible path to enterprise trust than treating Blueprint as a custom runtime-centered system.
