# Blueprint Specification — Authoring Roadmap

This document tracks the planned structure of the Blueprint specification and
the order in which it will be written. It is an **internal planning document**,
not part of the normative spec. It will evolve as the spec is written.

Status: Draft. Last updated 2026-04-19.

---

## 1. Target structure

The full Blueprint specification is organized into the following layers.

### 1.1 Core spec (normative)

These documents define conformance. Every one is referenced by `overview.md`.

#### `vocabulary/` — one document per term
- `blueprint.md` — the artifact
- `compound.md` — composition unit
- `channel.md` — communication primitive
- `state.md` — evolution over time
- `scope.md` — boundary and capability propagation
- `schema.md` — typed shapes
- `mark.md` — authorship trace
- `signature.md` — cryptographic proof
- `draft.md` — authoring state

#### `semantics/` — runtime behavior
- `composition.md` — instantiate, import, overlay rules
- `lifecycle.md` — scope begin/end, compound instantiation/destruction
- `channel-delivery.md` — ordering, backpressure, delivery guarantees
- `scope-propagation.md` — capability flow and name resolution
- `determinism.md` — what determinism guarantees and what it does not
- `authorship.md` — runtime behavior of marks and signatures

#### `schema/` — machine-readable definitions
- `blueprint.schema.json` — JSON Schema for Blueprint documents
- `mark.schema.json`
- `signature.schema.json`

#### Top-level normative documents
- `canonical-form.md` — byte-level canonical serialization (key ordering,
  number formatting, whitespace). Required for stable hashes and signatures.
- `versioning.md` — version declaration, evolution rules, migration semantics
- `extensions.md` — namespaced extension mechanism and registration

#### `conformance/`
- `conformance.md` — conformance classes (core, full, named subsets)
- `engine-contract.md` — runtime contract for engine implementers
- `tool-contract.md` — contract for authoring tools
- `test-suite.md` — description of the conformance test suite

### 1.2 Supporting material (informative)

These documents set context but do not define conformance.

#### `rationale/` — the *why* behind the spec
- `ambition.md` — port from `blueprint-sdk` (theory branch)
- `co-authorship.md` — humans and AI as peer authors
- `openapi-precedent.md` — what we copy, differ on, have no precedent for
- `prior-art.md` — port from `blueprint-sdk`
- `non-goals.md` — expansion of overview §9

#### `primers/` — audience-specific introductions
- `for-developers.md`
- `for-enterprise-architects.md`
- `for-ai-systems.md`
- `quick-start.md` — the "five minute Blueprint" experience

### 1.3 Examples

`examples/` — working Blueprint documents that validate against the schema.
- `hello-world.blueprint.json`
- `counter.blueprint.json`
- `dashboard.blueprint.json`
- (more to come)

---

## 2. Authoring order

Documents are written in **dependency order**, not alphabetical order. Each
document is written only after the documents it depends on exist in at least
draft form.

### Phase 1 — Vocabulary foundations
1. `vocabulary/schema.md` (no dependencies)
2. `vocabulary/scope.md` (no dependencies)
3. `vocabulary/state.md` (depends on scope, schema)
4. `vocabulary/channel.md` (depends on scope, schema)
5. `vocabulary/compound.md` (depends on all above)
6. `vocabulary/mark.md` (depends on compound, channel, state)
7. `vocabulary/signature.md` (depends on mark, canonical-form)
8. `vocabulary/draft.md` (depends on mark)
9. `vocabulary/blueprint.md` (depends on all above)

### Phase 2 — Semantics
1. `semantics/composition.md`
2. `semantics/lifecycle.md`
3. `semantics/scope-propagation.md`
4. `semantics/channel-delivery.md`
5. `semantics/determinism.md`
6. `semantics/authorship.md`

### Phase 3 — Formalization
1. `canonical-form.md`
2. `schema/blueprint.schema.json`, `mark.schema.json`, `signature.schema.json`
3. `versioning.md`
4. `extensions.md`

### Phase 4 — Conformance
1. `conformance/conformance.md`
2. `conformance/engine-contract.md`
3. `conformance/tool-contract.md`
4. `conformance/test-suite.md`

### Phase 5 — Rationale, primers, examples
1. Port rationale essays from `blueprint-sdk` (theory branch)
2. Write primers
3. Add worked examples

---

## 3. Working policy

### 3.1 Branch strategy
Normative specification work is drafted on a `drafts` branch (or per-document
feature branches) and merged to `main` only when a document reaches a
presentable state. This mirrors how IETF internet-drafts, W3C working drafts,
and the OpenAPI Initiative operate.

Rationale: the repository is public. A person arriving at `main` should see
coherent material, not half-formed thinking.

### 3.2 Status banners
Every spec document carries a status line near the top:
- *Draft* — work in progress, may change materially
- *Candidate* — stable enough for review, breaking changes unlikely
- *Stable* — part of a released specification version

### 3.3 Register
All normative documents use RFC 2119 / RFC 8174 conformance language (MUST,
SHOULD, MAY, etc.) with the conformance clause explicitly stated. Informative
documents use plain prose.

### 3.4 Commit convention
- Commits that change normative content are prefixed `spec:`.
- Commits that change informative content are prefixed `docs:`.
- Commits that change planning, build, or repo infrastructure are prefixed
  `chore:` or `ci:`.
- Customer names do not appear in commit messages.

### 3.5 What does not get published
The `blueprint-sdk` repository remains private as an R&D archive. Material
from it enters the public `blueprint` repository only after being rewritten
for a public audience.

---

## 4. Progress

Completed:
- `overview.md` — first canonical overview document.

In progress:
- (none)

Next:
- Scaffolding of directory structure and status placeholders.
- Port of rationale essays from `blueprint-sdk` into `rationale/`.

---

## 5. Open questions

Recorded here so they are not lost between sessions.

- Exact conformance classes: is "core" really distinguishable from "full," or
  is Blueprint better served by a single conformance class with optional
  features declared per-engine?
- Whether `examples/` belongs under `docs/specs/` or at the repository root.
- Whether `schema/*.json` should live under `docs/specs/schema/` or at a
  top-level `schema/` so validators can fetch it by stable URL.
- Whether to version the entire specification monolithically or per-document.
