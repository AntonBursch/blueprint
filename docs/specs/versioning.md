# Versioning

**Status:** Draft
**Version:** 0.1.0-draft
**Editor:** Anton Bursch

Part of the [Blueprint specification](overview.md), version 0.1.0-draft.

> This document defines how the Blueprint specification evolves, how a
> Blueprint artifact declares the specification version it targets,
> what "compatible" means between engine and artifact, and how
> releases are cut. Two things are versioned here: the specification
> itself, and the Blueprint document that targets it. Primitive,
> connector, and store versioning lives elsewhere and is out of scope
> for this document.

## Conformance language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this
document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119)
and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when,
they appear in all capitals.

## Dependencies

- [vocabulary/blueprint.md](vocabulary/blueprint.md) — the `blueprint` field and its `-draft` rule
- [vocabulary/draft.md](vocabulary/draft.md) — draft vs. released at the document level
- [semantics/composition.md](semantics/composition.md) §6 — the load sequence that consults this document
- [canonical-form.md](canonical-form.md) — release artifacts include the schema at a stable URL

## 1. What is versioned

Two independently versioned things:

1. **The Blueprint specification** — the set of documents in this
   directory. Currently `0.1.0-draft`.
2. **The Blueprint document** — an individual Blueprint artifact,
   which carries a `blueprint` field declaring the specification
   version it targets ([vocabulary/blueprint.md](vocabulary/blueprint.md) §3).

Primitive compounds, connectors, stores, and engine implementations
are versioned by the registries and projects that produce them. This
specification does not govern their version schemes.

## 2. Specification version identifier

### 2.1 Format

**R-1.** A specification version identifier MUST match the
[Semantic Versioning 2.0.0](https://semver.org/spec/v2.0.0.html)
grammar: `MAJOR.MINOR.PATCH` with an optional pre-release suffix.

### 2.2 Semantics of each component

**R-2.** The three numeric components have the following normative
meanings:

- **Major.** Increments for any change that a conforming engine at
  the previous major cannot accept. This includes: removal of any
  normative requirement, change in the meaning of an existing
  requirement, and **addition of any new `MUST` requirement** that an
  engine at the previous major would not satisfy.
- **Minor.** Increments for strictly additive changes: new
  vocabulary, new delivery modes, new mark actions, new optional
  fields, new `SHOULD` or `MAY` recommendations. An engine
  implementing a lower minor of the same major MAY fail to load a
  Blueprint that uses a feature from a higher minor; it MUST still
  recognize the artifact as well-formed at the version it declares.
- **Patch.** Increments for editorial changes only: typo fixes,
  clarifications, non-normative prose. Patch increments MUST NOT
  change any requirement number, any schema, or any observable
  behavior.

The rule that a new `MUST` is a major change is deliberate: a new
`MUST` means an engine that was conforming yesterday is not
conforming today unless it implements the new requirement.

### 2.3 Pre-release suffix: `-draft`

A specification version carrying the suffix `-draft` (for example
`0.1.0-draft`, `0.2.0-draft`) is a **draft specification**. Draft
specifications MAY change without a version bump; readers MUST NOT
treat any content of a draft specification as final.

**R-3.** A draft specification MUST NOT be used as the target version
for a Blueprint deployed to production, consistent with
[vocabulary/blueprint.md](vocabulary/blueprint.md) R-7.

**R-4.** Draft specifications are NOT ordered with respect to each
other. `0.1.0-draft` and `0.2.0-draft` are both drafts; neither
subsumes the other, and an engine that supports one MUST NOT
automatically accept the other.

**R-5.** When a draft specification is released, the `-draft` suffix
is removed. The resulting version (for example `0.1.0`) is the
released form; the draft history leading to it is not itself a
release.

### 2.4 Stability of released versions

**R-6.** Once released, a specification version MUST NOT be modified
in a way that changes any requirement. Editorial patches are
permitted only via patch increments (§2.2).

## 3. Blueprint document version declaration

### 3.1 The `blueprint` field

A Blueprint document declares the specification version it targets
via the required `blueprint` field, whose shape and semantics are
fixed in [vocabulary/blueprint.md](vocabulary/blueprint.md) §3. This
document adds the following rule:

**R-7.** The value of the `blueprint` field MUST match a specification
version identifier (§2.1) exactly, including any `-draft` suffix.

**R-8.** A Blueprint whose `blueprint` field carries a `-draft`
suffix is a **draft-targeted Blueprint**. An engine running in a
production mode MUST refuse it (per R-3 and
[vocabulary/blueprint.md](vocabulary/blueprint.md) R-7). Engines
running in development or validation modes MAY accept it.

### 3.2 The document is the fixed point

**R-9.** Once written, a Blueprint targets exactly one specification
version. The target is fixed by the `blueprint` field and is part of
the document's canonical form
([canonical-form.md](canonical-form.md)). Changing the target value
produces a new document (new canonical form, new hash, new
signature).

## 4. Engine compatibility

### 4.1 Declared support set

**R-10.** An engine MUST declare, in its engine documentation, the
**set of specification versions** it supports. The declaration is a
set, not a range; an engine MAY support `0.1.0` and `0.3.0` without
supporting `0.2.0`, and MUST say so explicitly.

**R-11.** An engine's supported-version declaration MUST identify
each supported version by its full identifier including any
pre-release suffix. `0.1.0` and `0.1.0-draft` are distinct.

### 4.2 Why a set, not a range

A range (for example "supports `>=0.1.0, <0.3.0`") would commit an
engine to accept every minor version in between, including ones that
introduced features the engine has not implemented. A minor version
is additive for *Blueprints*, not for *engines*: a Blueprint that
declares `0.2.0` may use a feature added in `0.2.0` that the engine
has not built. The set model keeps the engine's declaration honest.

### 4.3 Loading decision

**R-12.** An engine MUST refuse to load a Blueprint whose target
version ([R-7]) is not in the engine's declared support set. Refusal
MUST produce a diagnostic identifying:

- the Blueprint's target version;
- the engine's declared support set;
- the reason for refusal.

**R-13.** There is no "best-effort" loading at this level. An engine
MUST NOT partially load, silently downgrade, or ignore unknown
required fields introduced by a newer version. (Unknown fields in
the extension namespace are governed separately by the extensions
document.)

### 4.4 Forward compatibility

**R-14.** A Blueprint targeting a version *newer* than anything in
the engine's support set MUST be refused (R-12). This is the forward
boundary.

### 4.5 Backward compatibility within a major

**R-15.** Within a single major version, every minor bump MUST be
strictly additive: a Blueprint that was well-formed at version
`X.Y.Z` MUST remain well-formed at version `X.(Y+1).0` and every
later minor. Engines supporting a higher minor therefore accept
lower-minor Blueprints *if those Blueprints are re-declared at the
higher minor*; the `blueprint` field is the fixed-point (R-9), so
the Blueprint's author, not the engine, chooses which minor a
document targets.

A corollary: an engine that supports `0.3.0` does not automatically
support `0.1.0` or `0.2.0`. Authors keep Blueprints targeting the
version their engines declare.

### 4.6 Across major boundaries

**R-16.** Major version bumps are breaking. The spec text
introducing a new major version MUST include a **migration note**
enumerating every behavioral change that could affect a Blueprint
valid at the previous major. There is no automatic migration path.

## 5. Release process

Releasing a specification version is the parallel at the spec level
of releasing a Blueprint at the document level
([vocabulary/draft.md](vocabulary/draft.md) §5). The editor (the
holder of the specification's identity) releases a version by
performing all of the following:

1. Removing the `-draft` suffix from the version identifier in
   every document that carries it.
2. Tagging the specification repository with a `vMAJOR.MINOR.PATCH`
   annotated tag at the release commit.
3. Publishing the JSON Schema bundle at a stable URL rooted at
   `https://specs.blueprint.build/MAJOR.MINOR.PATCH/schema/`, such
   that every `$id` in the bundle resolves.
4. Recording the release in the specification repository's
   `CHANGELOG` (or equivalent), identifying the changes against the
   previous release at a normative level of detail (which
   requirements added, which changed, which removed).

**R-17.** An engine's supported-version declaration (R-10) MUST NOT
list a version whose schema bundle is not resolvable at the stable
URL in step 3. A version that has not been published at that URL is
not released.

**R-18.** Pre-release draft versions (§2.3) MAY be tagged
(`vMAJOR.MINOR.PATCH-draft`) and MAY publish schema bundles at the
stable URL for review purposes. An engine MAY list them in its
support set; a production engine MUST NOT (R-3, R-8).

## 6. Migration

Within a major (minor bumps), no migration is needed: a
Blueprint targeting a lower minor remains well-formed when re-read
by the same spec's definition of well-formedness.

Across majors, migration is a rewrite: the author produces a new
Blueprint document targeting the new major, with the migration note
(§R-16) as the guide. Engines do not perform automatic cross-major
migration. Tools MAY offer migration assistance; this specification
does not require it and does not define its shape.

**R-19.** Engines MUST NOT silently migrate a Blueprint from one
major to another. A Blueprint's target version is the author's
choice, and a change of major is a new authoring act.

## 7. Relationship to other versioned objects

Primitive compounds, connectors, and stores carry their own
versions, managed by their respective registries. Imports
([semantics/composition.md](semantics/composition.md) §2) reference
artifacts by identifiers that MAY include versions; the `pin`
sub-field of an import declaration pins an import to a specific
artifact version. This document does not govern those schemes.

A Blueprint targeting spec version `X.Y.Z` MAY import artifacts at
any primitive/connector/store version the registries support. The
spec version and artifact versions are orthogonal.

## 8. Normative requirements summary

- **R-1** Spec version matches SemVer 2.0.0 grammar.
- **R-2** Major = breaking (incl. new `MUST`); minor = additive; patch = editorial.
- **R-3** Draft spec MUST NOT be used in production.
- **R-4** Drafts are not ordered against each other.
- **R-5** Release removes the `-draft` suffix.
- **R-6** Released versions are immutable except via patch increments.
- **R-7** `blueprint` field matches the version grammar exactly.
- **R-8** Draft-targeted Blueprints refused by production engines.
- **R-9** Target version is fixed at authoring; change creates a new document.
- **R-10** Engines declare a SET of supported versions.
- **R-11** Declaration identifies full identifiers, including pre-release.
- **R-12** Engines refuse Blueprints outside their set, with diagnostic.
- **R-13** No partial or best-effort loading across versions.
- **R-14** Forward boundary: newer-than-set is refused.
- **R-15** Minor bumps are strictly additive.
- **R-16** Major bumps require a migration note in the spec.
- **R-17** Unpublished schema ⇒ not released; engines must not declare support.
- **R-18** Draft versions MAY be tagged and published for review.
- **R-19** Engines MUST NOT auto-migrate across majors.

## 9. Open questions

- Whether a deprecation cycle (mark-then-remove across major
  versions) belongs in a later revision. 0.1.0-draft does not define
  one; all removals are simply "major bump with migration note."
- Whether a spec-level release also requires a signature (parallel
  to `draft.md` §5). 0.1.0-draft does not; the tag and stable
  schema URL are the release evidence.

---

*End of versioning.md.*
