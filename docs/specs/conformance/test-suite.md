# Test suite and test procedure

**Status:** Draft
**Version:** 0.1.0-draft
**Editor:** Anton Bursch

Part of the [Blueprint specification](../overview.md), version 0.1.0-draft.

> This document describes how an implementer produces the test
> results required by a conformance claim ([conformance.md](conformance.md)
> §8). It fixes the structure of a test run, the coverage
> obligation, the report format, and the governance of the corpus
> once a specification version is released. This document introduces
> no new normative requirements; it elaborates the testing
> obligations already stated in [conformance.md](conformance.md).

## Dependencies

- [conformance.md](conformance.md) — conformance classes, tiers, and claim shape
- [REQUIREMENTS.md](REQUIREMENTS.md) — the consolidated requirement index
- [../versioning.md](../versioning.md) — draft vs. released testing regimes

## 1. Two regimes

Testing against the specification happens in one of two regimes,
determined by whether the claimed version is a draft or a release.

- **Draft regime** — no normative corpus exists. The implementer
  self-tests against [REQUIREMENTS.md](REQUIREMENTS.md), producing a
  per-requirement report.
- **Released regime** — a normative corpus is published at
  `https://specs.blueprint.build/<version>/tests/`. The implementer
  runs the corpus and produces a pass/fail/skip report.

0.1.0-draft is a draft. Every conformance claim against 0.1.0-draft
uses the draft regime.

## 2. Draft regime: how to self-test

### 2.1 Build a test catalog from REQUIREMENTS.md

For the class and tier being claimed, enumerate every applicable
requirement from [REQUIREMENTS.md](REQUIREMENTS.md). Applicability
rules:

- **Engine-L1 claim** — every requirement categorized L1.
- **Engine-L2 claim** — every requirement categorized L1 or L2.
- **Engine-L3 claim** — every requirement categorized L1, L2, or L3.
- **Authoring-tool claim** — every requirement categorized *Authoring*.
- **Validator claim** — every requirement categorized *Validator*.

Cross-cutting requirements (canonical form, versioning, extensions)
apply to every class.

### 2.2 Write a test for each requirement

For each enumerated requirement, write a test that:

- exercises a Blueprint, input, or call path that depends on the
  requirement;
- records a pass if the implementation behaves as the requirement
  demands;
- records a fail otherwise.

Where a requirement is a prohibition (MUST NOT), the test
demonstrates the absence of the prohibited behavior — typically by
showing that an attempted violation is refused, diagnosed, or
otherwise prevented.

Where a requirement covers a contract between two implementations
(for example, snapshot portability in
[semantics/lifecycle.md](../semantics/lifecycle.md) R-10), the
implementer is responsible for documenting a methodology that
substitutes for cross-implementation testing in the absence of a
corpus. A common approach is to test portability within the
implementation's own snapshot round-trip and note in `evidence` that
cross-engine testing is deferred to the released-spec corpus.

### 2.3 Record results

Per [conformance.md](conformance.md) R-13, the `per_requirement`
array in the conformance claim MUST list every applicable
requirement with:

- `id` — the form `document#R-N`, for example
  `semantics/composition.md#R-21`;
- `status` — one of `"pass"`, `"fail"`, or `"not-applicable"`;
- `evidence` — a URI or short string. Acceptable forms of evidence:
  - a URI to a test file or test-run log;
  - a URI to an excerpt of the implementation's source that
    demonstrates the behavior;
  - a short prose rationale when the requirement is satisfied by
    design (for example, a type-system guarantee);
  - for `"not-applicable"`, a brief justification.

A claim MUST NOT report `"pass"` without evidence.

### 2.4 Publish

The conformance claim and its supporting evidence MUST be published
at durable URLs per [conformance.md](conformance.md) R-10.
Implementers SHOULD preserve the test artifacts for the lifetime of
the claim, so third parties can audit the results if the
specification introduces a verification mechanism later.

## 3. Released regime: how to run the corpus

When a specification version is released, the corpus is published
at `https://specs.blueprint.build/<version>/tests/`. Its structure
is not fixed in 0.1.0-draft beyond the following outline. A later
revision will finalize these details alongside the released
specification.

### 3.1 Corpus organization

Expected corpus organization (subject to finalization at release):

- one directory per class (`engine/`, `authoring-tool/`, `validator/`);
- within `engine/`, one directory per tier (`l1/`, `l2/`, `l3/`);
- test files named after the requirement identifier they cover, for
  example `composition-R-21.blueprint.json` with a companion
  `composition-R-21.expected.json`.

### 3.2 Running against an engine

An engine test typically has three files:

1. **input Blueprint** — the document to load;
2. **input trace** — the sequence of external inputs to deliver
   after loading;
3. **expected observable surface** — the expected state
   projections, channel outputs, and diagnostics per
   [semantics/determinism.md](../semantics/determinism.md) §2.

The engine loads the input, applies the trace, and emits its
observable surface. A comparison harness (also published with the
corpus) checks observable equivalence per
[semantics/determinism.md](../semantics/determinism.md) R-1.

### 3.3 Running against an authoring tool

An authoring-tool test typically provides:

- **input spec** — a structured description of a Blueprint to
  produce (high-level, tool-agnostic);
- **expected output Blueprint** — the canonical-form document the
  tool should emit, or a schema the output must match, depending on
  the test's focus.

Canonical-form tests compare byte-for-byte.

### 3.4 Running against a validator

A validator test provides:

- **input Blueprint** — a document;
- **expected verdict** — valid, invalid (with a specific set of
  reported errors), or signature-related (verified, unverified,
  failed).

A validator that reports a different verdict fails the test.

### 3.5 Reporting

Released-regime results are reported per
[conformance.md](conformance.md) R-14: `corpus` URI, pass/fail/skip
counts, and a `details` URI to the full results. The corpus MAY
publish a reference reporter; implementers are not required to use
it.

## 4. Coverage obligation

Whether in draft or released regime, the coverage obligation is the
same.

- Every requirement applicable to the claimed class and tier MUST be
  addressed.
- A requirement MAY be marked `"not-applicable"` only if the
  justification is visible in the claim. Common legitimate N/A
  cases:
  - draft-only tier requirements when claiming against a released
    version that has supplanted them;
  - requirements on features the implementation does not expose
    (e.g., a validator that declines signature verification marks
    the signature-verification R's N/A with the reason "validator
    does not perform signature verification");
  - requirements on extensions the implementation does not
    recognize.

An implementer who marks requirements N/A liberally will erode the
credibility of the claim. Self-certification rests on the integrity
of the test record.

## 5. Governance of the corpus

Until 0.1.0 is released, no normative corpus exists. Governance
rules for the corpus will be published with the released
specification. Expected properties, recorded here as intent:

- The corpus is versioned alongside the specification: corpus for
  `0.1.0` is distinct from corpus for `0.1.1`, and both are
  preserved.
- The corpus is open to community contributions, but normative
  additions (tests that constrain what counts as passing) require
  editor approval.
- The corpus itself is signed
  ([vocabulary/signature.md](../vocabulary/signature.md)) to prevent
  tampering. Conformance claims against the released regime
  reference the signed corpus by its hash and version.

0.1.0-draft does not make these governance points normative. They
are listed so implementers building tooling now can anticipate the
shape of the released regime.

## 6. Relationship to determinism

A test that compares observable surfaces
([semantics/determinism.md](../semantics/determinism.md) §2) works
only if the two surfaces being compared were produced from the same
input trace (§4 of that document). Draft-regime self-tests
typically run the engine twice on the same input trace and verify
observable equivalence between runs; released-regime corpus tests
compare against expected outputs recorded in the corpus.

Tests that depend on cross-engine determinism (such as a portable
snapshot produced by engine A restored by engine B) require a
second engine to be meaningful. In the draft regime, this is a
documentation-and-rationale test; in the released regime, the
corpus provides reference snapshots that any engine can restore.

## 7. Scope

This document describes procedure, not new requirements. The only
normative content governing testing lives in
[conformance.md](conformance.md) §8 (R-13, R-14, R-15).

---

*End of test-suite.md.*
