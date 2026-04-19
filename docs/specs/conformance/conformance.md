# Conformance

**Status:** Draft
**Version:** 0.1.0-draft
**Editor:** Anton Bursch

Part of the [Blueprint specification](../overview.md), version 0.1.0-draft.

> This document defines what it means for an implementation to conform
> to the Blueprint specification. It establishes three conformance
> classes (engine, authoring tool, validator), defines three tiers
> within the engine class, specifies what a conformance claim must
> contain and how it is published, and fixes the relationship between
> conformance, extensions, and specification versions. The companion
> documents [test-suite.md](test-suite.md) describe how to test; the
> companion document [REQUIREMENTS.md](REQUIREMENTS.md) enumerates
> every normative requirement in the specification.

## Conformance language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this
document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119)
and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when,
they appear in all capitals.

## Dependencies

- [../versioning.md](../versioning.md) — defines the spec version an implementation conforms to
- [../extensions.md](../extensions.md) — defines the extension mechanism an implementation must support
- [../canonical-form.md](../canonical-form.md) — required of all classes
- [../semantics/*.md](../semantics/) — engine obligations
- [../vocabulary/*.md](../vocabulary/) — authoring-tool and validator obligations
- [REQUIREMENTS.md](REQUIREMENTS.md) — consolidated index of every R-numbered requirement

## 1. What conformance covers

Conformance to this specification is defined relative to the
normative requirements it contains.

**R-1.** A conformance claim covers only requirements expressed using
**MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, or **SHALL NOT**
([RFC 2119](https://www.rfc-editor.org/rfc/rfc2119);
[RFC 8174](https://www.rfc-editor.org/rfc/rfc8174)). Recommendations
expressed with **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**,
or **OPTIONAL** are non-binding; not following a recommendation is
not non-conformance.

**R-2.** A conformance claim is always made against a specific
specification version ([../versioning.md](../versioning.md)). An
implementation MAY hold conformance claims for multiple versions
simultaneously; each claim is independent.

## 2. Conformance classes

Three classes of implementation are recognized for conformance in
0.1.0-draft.

### 2.1 Engine

An **engine** implements the runtime: it loads a Blueprint document,
executes it, and exposes state and outputs per the semantics
documents. Engines are the implementations that actually *run*
Blueprints.

### 2.2 Authoring tool

An **authoring tool** produces Blueprint documents. It writes the
canonical form, produces marks, produces signatures when required,
and enforces the conditions under which a Blueprint is released.

A tool that *modifies* an existing Blueprint is an authoring tool;
every modification is an authoring act.

### 2.3 Validator

A **validator** reads a Blueprint document and reports whether it is
well-formed and whether its signatures verify, without executing it.
A validator is a read-only subset of an engine's load phase.

### 2.4 Deferred classes

Conformance for connectors, primitive compounds, and stores is not
formalized in 0.1.0-draft. Those classes will be addressed once the
connection registry and primitive registry are specified. Until
then, a connector or primitive implementation MAY describe itself
as "engine-compatible" without making a conformance claim under this
document.

## 3. Engine tiers

Engine conformance is tiered. A claim of engine conformance MUST
identify one of three tiers.

### 3.1 Engine-L1 (Core)

An **Engine-L1** implementation satisfies:

- Load sequence through composition
  ([semantics/composition.md](../semantics/composition.md) §6), up to
  and including step 7.
- Full lifecycle: scope begin, run, scope end, stop
  ([semantics/lifecycle.md](../semantics/lifecycle.md) §§1–5, §9).
- Channel delivery for all three modes
  ([semantics/channel-delivery.md](../semantics/channel-delivery.md)).
- Scope propagation
  ([semantics/scope-propagation.md](../semantics/scope-propagation.md)).
- Determinism
  ([semantics/determinism.md](../semantics/determinism.md)) — structural
  determinism and replay equivalence.
- Schema validation of loaded Blueprints
  ([schema/README.md](../schema/README.md)).
- Canonical-form correctness for any Blueprint the engine emits
  ([canonical-form.md](../canonical-form.md)).
- Extension handling for unknown extensions
  ([extensions.md](../extensions.md) R-9, R-10, R-11).
- Versioning refusal rules
  ([versioning.md](../versioning.md) R-12, R-13, R-14).

An Engine-L1 implementation MAY skip snapshot/restore
([semantics/lifecycle.md](../semantics/lifecycle.md) §6) and MAY skip
signature verification
([semantics/authorship.md](../semantics/authorship.md) §2). Engines
that skip signature verification MUST document the skip per
[authorship.md](../semantics/authorship.md) R-3.

Engine-L1 is the minimum runtime. It is suitable for development,
short-lived applications, and environments where durability and
authenticity are not required.

### 3.2 Engine-L2 (Durable)

An **Engine-L2** implementation satisfies all of Engine-L1 and in
addition:

- Snapshot and restore
  ([semantics/lifecycle.md](../semantics/lifecycle.md) §6), including
  the portability invariant (R-10).
- Safe-point enforcement (R-11).
- Address stability across snapshot/restore (R-13).
- In-flight message preservation for reliable delivery modes
  ([semantics/channel-delivery.md](../semantics/channel-delivery.md)
  R-13).

Engine-L2 is suitable for applications whose state must survive
restarts or migrate between engine instances.

### 3.3 Engine-L3 (Verified)

An **Engine-L3** implementation satisfies all of Engine-L2 and in
addition:

- Signature verification enabled by default
  ([semantics/authorship.md](../semantics/authorship.md) §2).
- Full enforcement of release status
  ([semantics/authorship.md](../semantics/authorship.md) R-4, R-5).
- Revocation handling per engine-documented policy
  (R-14, R-15, R-16).
- Support for at least `Ed25519`
  ([vocabulary/signature.md](../vocabulary/signature.md) R-3).
- Re-verification on restore
  ([semantics/authorship.md](../semantics/authorship.md) R-12).

Engine-L3 is suitable for production deployment in environments
where Blueprint authorship must be authenticated — typically
regulated or multi-author settings.

### 3.4 Tier subset relation

Every Engine-L3 is an Engine-L2. Every Engine-L2 is an Engine-L1.
An engine claiming a higher tier implicitly claims the lower tiers.
An engine MUST claim the highest tier it satisfies; it MUST NOT
claim a tier whose requirements it does not meet.

**R-3.** An engine's tier claim MUST accurately reflect its
implemented requirement set. An engine that implements tier-N
requirements partially MUST claim tier N−1 (or no conformance).

## 4. Authoring-tool conformance

Authoring-tool conformance is flat: a tool either conforms or it
does not. An authoring tool conforms by satisfying:

- Canonical-form production per
  [canonical-form.md](../canonical-form.md).
- Mark production per [vocabulary/mark.md](../vocabulary/mark.md),
  including author-kind rules and mark immutability.
- Signature production per
  [vocabulary/signature.md](../vocabulary/signature.md), including
  algorithm support and key-binding discipline.
- Release-condition production per
  [vocabulary/draft.md](../vocabulary/draft.md), including all three
  release conditions.
- Schema validity of every document produced
  ([schema/README.md](../schema/README.md)).
- Version-declaration rules per [../versioning.md](../versioning.md),
  including `-draft` suffix handling.
- Extension-authoring rules per [../extensions.md](../extensions.md),
  including preservation on read-modify-write (R-9, R-17).

**R-4.** An authoring tool claiming conformance MUST produce only
Blueprint documents that validate against the schema bundle at the
claimed specification version. A tool that emits documents failing
schema validation at that version MUST NOT claim conformance.

## 5. Validator conformance

Validator conformance is flat. A validator conforms by satisfying:

- Schema validation per the schema bundle at the claimed version.
- Canonical-form checking per [canonical-form.md](../canonical-form.md).
- Signature verification per
  [vocabulary/signature.md](../vocabulary/signature.md) §5 when the
  validator performs signature checks.
- Extension preservation and opacity
  ([extensions.md](../extensions.md) R-9, R-10, R-11).
- Release-condition checking per
  [vocabulary/draft.md](../vocabulary/draft.md) §5 when the validator
  reports release status.
- Faithful reporting: every well-formedness error MUST be reported
  per the spec; a valid document MUST NOT be reported as invalid.

**R-5.** A validator MUST NOT execute a Blueprint. A tool that
executes (begins any scope, runs any compound) is an engine, not a
validator; it MUST make an engine-class claim.

**R-6.** A validator MAY decline to perform signature verification,
but it MUST clearly report whether verification was performed and
MUST NOT report a non-verified signature as verified.

## 6. Extensions and conformance

Conformance is orthogonal to extension support. The extension
mechanism ([extensions.md](../extensions.md)) defines what engines
MUST, MUST NOT, and MAY do with extensions; satisfying those rules
is already covered by the class-specific conformance requirements
(§§3–5).

**R-7.** An implementation's recognition of specific extensions does
NOT expand its conformance claim. The claim covers only the standard
requirements at the claimed class and tier.

**R-8.** If an implementation requires a Blueprint to carry a
specific extension to be accepted (per
[extensions.md](../extensions.md) R-15), that requirement is a
deployment constraint and MUST be documented in the conformance
claim (§7.2). It does not narrow the implementation's conformance
class.

## 7. The conformance claim

A conformance claim is a machine-readable assertion by an
implementer that their implementation satisfies the requirements of
a class (and, for engines, a tier) at a specific specification
version.

### 7.1 Required shape

**R-9.** A conformance claim MUST be a JSON object with at minimum
the following members:

- `implementation` — an object identifying the implementation:
  - `name` (string, required);
  - `version` (string, required);
  - `vendor` (string, required).
- `spec_version` — a string matching the spec version grammar
  ([versioning.md](../versioning.md) §2.1).
- `class` — a string, one of `"engine"`, `"authoring-tool"`,
  `"validator"`.
- `tier` — for `class: "engine"`, a string, one of `"L1"`, `"L2"`,
  `"L3"`. MUST NOT be present for other classes.
- `test_results` — an object summarizing testing per §8.
- `attestation` — an object:
  - `claimed_by` (string, required — name of the person or entity
    making the claim);
  - `date` (string, RFC 3339 date, required);
  - optionally `signature` (object matching
    [vocabulary/signature.md](../vocabulary/signature.md)'s signature
    structure, signing the rest of the claim).
- `recognized_extensions` — an array of objects (may be empty):
  each with `key` (string) and `documentation` (URI).
- `required_extensions` — an array of extension keys (strings, may
  be empty) this implementation requires Blueprints to carry.

### 7.2 Required documentation

**R-10.** An implementation claiming conformance MUST publish its
conformance claim at a durable URL reachable from the
implementation's documentation.

**R-11.** An implementation MUST link its conformance claim from any
marketing or technical material that asserts conformance to
Blueprint. A conformance claim made only in prose, without the
machine-readable object, does not satisfy this requirement.

### 7.3 Claim freshness

A conformance claim is a statement about the implementation version
named in `implementation.version`. A new implementation version
requires a new claim; implementers MUST NOT roll a claim forward
across versions.

**R-12.** A conformance claim applies to exactly one implementation
version. An implementer issuing a new implementation version MUST
issue a new conformance claim or explicitly cease to claim
conformance.

## 8. Testing obligations

### 8.1 Draft-spec testing

For a **draft specification version** (identifier ending in
`-draft`), no normative test corpus is published. Implementations
claiming conformance to a draft version MUST self-test against
[REQUIREMENTS.md](REQUIREMENTS.md), producing results that
identify every tested requirement and whether it passed.

**R-13.** For a conformance claim against a draft version,
`test_results` MUST contain:

- `corpus` — the string `"self-test"`;
- `per_requirement` — an array of objects, each with `id` (the
  R-number in the form `document#R-N`), `status` (`"pass"`,
  `"fail"`, or `"not-applicable"`), and `evidence` (a URI or short
  string pointing to a test, log, or rationale).

A claim with missing requirements is incomplete; an implementer
MUST either report every applicable requirement or not claim
conformance.

### 8.2 Released-spec testing

For a **released specification version** (identifier without
`-draft`), a normative test corpus is published at
`https://specs.blueprint.build/<version>/tests/` as part of the
release process ([versioning.md](../versioning.md) §5).

**R-14.** For a conformance claim against a released version,
`test_results` MUST contain:

- `corpus` — a URI identifying the exact corpus version used;
- `passed`, `failed`, `skipped` — counts;
- `details` — a URI to a machine-readable results report.

A claim against a released version MUST pass every applicable test
in the corpus. Skipped tests MUST be justified (e.g., a test for
an extension the implementation does not recognize, which is
legitimately N/A).

### 8.3 Self-certification

Conformance in 0.1.0-draft is self-certified. The implementer runs
the tests, produces the claim, and stakes their reputation on the
result. No third-party verification is defined in 0.1.0-draft.

**R-15.** An implementer making a conformance claim MUST have
actually performed the testing the claim reports. A claim that
reports tests the implementer did not run is a false claim.

A later revision MAY introduce third-party verification; its
absence in 0.1.0-draft is deliberate.

## 9. Loss of conformance

An implementation loses conformance when:

- a required component of its claimed class is removed or broken;
- a defect is discovered that causes the implementation to fail a
  previously-passing requirement;
- the spec version it claimed is retracted
  ([versioning.md](../versioning.md) §2.4 forbids retraction for
  released versions, so this applies only to drafts).

**R-16.** An implementer who discovers that an issued conformance
claim is no longer accurate MUST retract or update the claim at the
claim's published URL within a reasonable period from discovery.

## 10. Normative requirements summary

- **R-1** Conformance covers only MUST / MUST NOT / REQUIRED / SHALL / SHALL NOT.
- **R-2** Claims are per-version and independent.
- **R-3** Engine tier claim must accurately reflect implementation.
- **R-4** Authoring tools produce only schema-valid documents.
- **R-5** Validators do not execute; execution makes it an engine.
- **R-6** Validators may decline signature verification but must report honestly.
- **R-7** Extension recognition does not expand the conformance claim.
- **R-8** Required-extension constraints are documented in the claim.
- **R-9** Conformance claim JSON shape.
- **R-10** Claim published at a durable URL.
- **R-11** Conformance assertions in marketing link the machine-readable claim.
- **R-12** Claims apply to exactly one implementation version.
- **R-13** Draft-spec test_results format.
- **R-14** Released-spec test_results format.
- **R-15** Claimed tests must have been actually performed.
- **R-16** Inaccurate claims must be retracted or updated.

## 11. Open questions

- Third-party verification: deferred to a later revision.
- A conformance-claim JSON Schema: deferred; the prose shape in §7.1
  is normative but not yet schema'd. A later revision MAY publish
  `conformance-claim.schema.json`.
- Interoperability testing between engines (same Blueprint, same
  input trace, compare observable surfaces per
  [semantics/determinism.md](../semantics/determinism.md)): deferred
  until the test corpus exists to anchor comparisons.

---

*End of conformance.md.*
