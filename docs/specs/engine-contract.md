# Engine Contract

Version: 0.1 (draft)
Scope: blueprint-sandbox/02-reverse-engineer

An **engine** is software that, given a graph conforming to some **grammar**, executes that graph according to the grammar's semantics. This document defines how engines are packaged, declared, discovered, and invoked.

---

## Principle: one kind of engine

There is no distinction between "built-in" and "third-party" engines. Every engine — including the reference engines that ship with Blueprint — plays by the rules in this document. No privileged path.

Reference engines ship as packages like `@blueprint/engine-dag`. Third-party engines ship as packages like `@acme/fast-dag`. Blueprint treats them identically at resolution time.

---

## Identifiers

**Grammar ID** — a URI plus a semver version.

    blueprint.io/grammars/dag@1.0
    acme.com/grammars/hsm@2.1

**Engine ID** — the package's canonical name in its language's ecosystem.

    @blueprint/engine-dag             (npm)
    blueprint-engine-dag              (pypi)
    blueprint-engine-dag              (crates)
    Blueprint.Engine.Dag              (nuget)
    dev.blueprint:engine-dag          (maven)

An engine declares which grammars it implements. An app declares which grammar its graph uses. Blueprint resolves the pair at load time.

---

## Manifest

Every engine package contains a `blueprint.json` at its root:

```json
{
  "kind": "engine",
  "name": "@blueprint/engine-dag",
  "version": "0.1.0",
  "language": "js",
  "implements": [
    { "grammar": "blueprint.io/grammars/dag", "range": "^1.0" }
  ],
  "capabilities": ["tick", "inspect"],
  "entrypoint": "./index.js",
  "contract-version": "0.1"
}
```

Fields:

- `kind` — always `"engine"` for this spec.
- `name` — package name in the language's ecosystem.
- `version` — semver of the engine package.
- `language` — runtime this engine requires: one of `js`, `python`, `rust`, `go`, `csharp`, `java`, `kotlin`, `dart`.
- `implements` — one or more `{grammar, range}` pairs. A single engine may implement multiple grammars.
- `capabilities` — features beyond the minimum lifecycle. Known values are reserved (see Capabilities below). Vendors may add `x-*` prefixed entries.
- `entrypoint` — language-appropriate pointer to the loader.
- `contract-version` — which version of this spec the engine claims to honor. Blueprint refuses to load mismatched major versions.

---

## Lifecycle

Every engine implements the same four verbs. The *shape* of the interface differs per language; the *meaning* is identical.

```
load(graph, context)      → session
tick(session, input?)     → output | events | nothing
inspect(session)          → snapshot
teardown(session)         → void
```

- **load** — validate the graph against the declared grammar, construct runtime state, return an opaque session handle.
- **tick** — advance the session by one unit of work. "One unit" is grammar-defined (DAG: one node fired; FSM: one transition evaluated; BT: one tick of the root).
- **inspect** — return current session state for debugging and observability. Shape is grammar-defined but must be serializable.
- **teardown** — release resources. Must be idempotent.

These four are the minimum contract. Any engine that does not implement all four is not a conforming engine.

---

## Capabilities (optional)

Beyond the minimum lifecycle, engines may declare optional capabilities:

- `pause` / `resume` — cooperative suspension of a session.
- `snapshot` / `restore` — persistable session state.
- `replay` — deterministic re-execution from a snapshot plus input log.
- `hot-reload` — swap the graph definition without destroying session state.
- `nondeterministic` — this engine does not promise determinism given identical inputs.

Declared capabilities are a contract. If an engine lists `snapshot`, Blueprint and third-party tools may rely on it. Undeclared capabilities must not be assumed.

Vendors may introduce experimental capabilities prefixed with `x-`. These carry no cross-implementation guarantees.

---

## Language bindings

Each supported language defines the concrete interface that realizes the lifecycle. The semantic contract is language-free; bindings are thin.

**JavaScript / TypeScript**

```js
// index.js — default export is a factory
export default {
  load(graph, ctx) { /* ... */ },
  tick(session, input) { /* ... */ },
  inspect(session) { /* ... */ },
  teardown(session) { /* ... */ }
};
```

**Python**

```python
# A class implementing blueprint.Engine
class Engine(blueprint.Engine):
    def load(self, graph, ctx): ...
    def tick(self, session, input=None): ...
    def inspect(self, session): ...
    def teardown(self, session): ...
```

**Rust**

```rust
pub trait Engine {
    type Session;
    fn load(&self, graph: Graph, ctx: Context) -> Self::Session;
    fn tick(&self, session: &mut Self::Session, input: Option<Input>) -> TickResult;
    fn inspect(&self, session: &Self::Session) -> Snapshot;
    fn teardown(&self, session: Self::Session);
}
```

Other languages follow the same pattern. Each binding is a short appendix; the semantic contract above is the source of truth.

---

## Discovery and resolution

When Blueprint loads an app:

1. Read the app's graph. Note its `$grammar` field (e.g. `blueprint.io/grammars/dag@1.0`).
2. Walk installed packages in the chosen language. Select packages whose `blueprint.json` has `kind == "engine"` and whose `implements` covers the requested grammar at a compatible version.
3. If multiple candidates remain, pick by: explicit lockfile entry → app-declared preference → highest-version match.
4. If zero match, fail loudly, naming the grammar and the installed candidates that were inspected.

The SDK provides a language-appropriate resolver helper (`@blueprint/resolver` in JS, `blueprint.resolver` in Python, etc.). The resolver has no knowledge of which engines are "official." It treats all engine packages identically.

---

## Responsibilities

An engine **must**:

- Refuse to load a graph that does not validate against its declared grammar.
- Be deterministic within a session given the same input sequence, unless it declares the `nondeterministic` capability.
- Clean up on `teardown`.
- Document any grammar-defined behaviors it specializes, restricts, or extends.

An engine **must not**:

- Mutate the input graph.
- Exhibit capabilities not declared in its manifest.
- Share state across sessions.

---

## Non-goals

- Engines do not persist data. That is the responsibility of **stores**.
- Engines do not speak to external systems. That is the responsibility of **connections**.
- Engines do not render UI. That is the responsibility of **adapters**.
- Engines do not know about hosts. An engine runs wherever its language runs; the host concern belongs to adapters and servers.

---

## Compatibility

- `implements[].range` uses semver.
- Grammar versions use semver. A grammar's `2.0` release may break compatibility; engines must re-declare support.
- `contract-version` is bumped only when the lifecycle shape itself changes. Additive capabilities do not require a contract bump.

---

## Open questions (for future drafts)

- Exact schema of the `TickResult` / events returned from `tick`.
- Error model: structured error codes vs. language-native exceptions vs. result types.
- Transport: in-process calls only, or cross-process / cross-language via a shared wire protocol?
- Multi-grammar engines: resolution rules when one package implements several grammars.
- Conformance test suite: who authors it, where it lives, how engines opt in.

This spec locks the *shape* of the contract. The details above are the first areas to firm up before engines ship to production.
