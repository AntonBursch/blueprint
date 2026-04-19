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

Documents are written in an order designed to produce the best spec, not
merely the fastest draft. The guiding rule is:

> *Write the document whose absence is currently distorting the most other
> decisions.*

The order interleaves rationale, vocabulary, formalization, semantics, and
conformance. It is not strictly bottom-up: in several places a later-phase
document is written earlier because its absence would cause earlier documents
to be written incorrectly.

### Round 1 — Establish the frame
Rationale first, so that every normative decision has a frame to sit in.
Informative docs are ported or rewritten for public register.
1. `rationale/ambition.md` — port from `blueprint-sdk/theory`; the
   three-layer answer (portable, legible, co-authored).
2. `rationale/openapi-precedent.md` — pins down what Blueprint *is*.
3. `rationale/non-goals.md` — expands overview §9; exposes overreach early.
4. `rationale/co-authorship.md` — the theory behind marks and signatures.
   Must exist before `vocabulary/mark.md` can be written correctly.

### Round 2 — The keystone vocabulary doc
5. `vocabulary/compound.md` — written before the other vocabulary docs,
   not after. Compound is the heart of Blueprint; writing it first forces
   the hardest decisions (instantiation, parameter binding, visibility,
   nesting) to be made once, constraining the rest.

### Round 3 — Support structure around the keystone
Vocabulary, now bottom-up, with `canonical-form.md` promoted into this
round because it is a prerequisite for `signature.md`.
6. `vocabulary/scope.md`
7. `vocabulary/schema.md`
8. `vocabulary/channel.md`
9. `vocabulary/state.md`
10. `vocabulary/mark.md`
11. `canonical-form.md` — promoted out of "formalization" because
    `signature.md` and `versioning.md` both require it.
12. `vocabulary/signature.md`
13. `vocabulary/draft.md`
14. `vocabulary/blueprint.md` — the roll-up; last of the vocabulary.

### Round 4 — Machine-readable schemas
The JSON Schemas come *after* vocabulary prose (which determines fields)
and *before* semantics (which would otherwise reference fields that do not
yet exist).
15. `schema/blueprint.schema.json`
16. `schema/mark.schema.json`
17. `schema/signature.schema.json`

### Round 5 — Semantics
18. `semantics/composition.md` — the runtime keystone.
19. `semantics/lifecycle.md`
20. `semantics/scope-propagation.md`
21. `semantics/channel-delivery.md`
22. `semantics/determinism.md` — placed near the end because it is a
    cross-cutting clarification of what earlier documents do and do not
    promise.
23. `semantics/authorship.md`

### Round 6 — Versioning and extensions
Placed after semantics rather than with the other formalization work,
because breaking-change policy and extension-validation interaction both
depend on semantics being committed.
24. `versioning.md`
25. `extensions.md`

### Round 7 — Conformance
Conformance is last among normative work; it references everything.
26. `conformance/engine-contract.md`
27. `conformance/tool-contract.md`
28. `conformance/conformance.md` — resolves the open question of whether
    there is one conformance class or a core/full split.
29. `conformance/test-suite.md`

### Round 8 — Examples and primers
Examples first: if the smallest possible Blueprint is painful to write,
the spec is wrong and we want to find out now. Primers follow once the
examples exist to point at.
30. `examples/hello-world.blueprint.json`
31. `examples/counter.blueprint.json`
32. `examples/dashboard.blueprint.json`
33. `primers/quick-start.md`
34. `primers/for-developers.md`
35. `primers/for-enterprise-architects.md`
36. `primers/for-ai-systems.md`

### Round 9 — Closer
37. `rationale/prior-art.md` — written last on purpose. An honest
    comparison to prior art requires knowing exactly what Blueprint is.

### Review policy

**After every round, we pause and review.** No document from round N+1 is
begun until the completed documents of round N have been re-read with fresh
eyes and revised for any distortion the next round has exposed. If a
document in round N is discovered to be wrong, it is fixed before round
N+1 begins.

This is the single discipline that separates a spec that holds up from one
that accumulates contradictions.

---

## 3. Working policy

### 3.1 Branch and tag strategy
Specification work is committed directly to `main`. Per-round feature
branches are not used, because the two authors review against the working
tree and mid-round fixes to earlier rounds must stay easy.

At the completion and review of each round, `main` is tagged
`v0.1.0-draft-roundN` (e.g. `v0.1.0-draft-round1`). Tags provide milestone
markers, easy inter-round diffs, and a clean branch-off point if a
round-in-progress ever needs to be shared for external review.

Feature branches are created only when a round-in-progress is being shared
externally for review before it lands.

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
- Full directory scaffolding with placeholder files for every planned
  document.

In progress:
- (none)

Next:
- Round 1: port `rationale/ambition.md` from `blueprint-sdk`, then write
  `openapi-precedent.md`, `non-goals.md`, `co-authorship.md`.

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
