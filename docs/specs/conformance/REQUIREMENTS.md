# REQUIREMENTS index

**Status:** Draft
**Version:** 0.1.0-draft
**Editor:** Anton Bursch

Part of the [Blueprint specification](../overview.md), version 0.1.0-draft.

> This document is the consolidated index of every normative
> requirement in the Blueprint specification. Each requirement
> appears exactly once, identified by its source document and
> requirement number, with a tier/class mapping for conformance
> testing. A conformance claim against 0.1.0-draft tests against
> this index ([conformance.md](conformance.md) R-13,
> [test-suite.md](test-suite.md) §2.1).
>
> This document introduces no new requirements. It reflects and
> cross-references requirements defined elsewhere.

## Dependencies

- [conformance.md](conformance.md) — what "testing" means
- [test-suite.md](test-suite.md) — how to test
- every specification document — the sources

## 1. Index conventions

### 1.1 Identifier form

Every requirement is identified in the form
`<source-document>#R-<N>`, for example
`semantics/composition.md#R-21`. The requirement's full text is at
the anchor in the source document.

### 1.2 Tier/class column

Each requirement is tagged with one or more of:

- **L1** — required at Engine-L1 (Core).
- **L2** — required at Engine-L2 (Durable); implies L1.
- **L3** — required at Engine-L3 (Verified); implies L2.
- **AUTH** — required for Authoring-tool conformance.
- **VAL** — required for Validator conformance.
- **X** — cross-cutting; required at every class/tier that loads or
  processes Blueprints.

Multiple tags mean the requirement applies in multiple contexts.
For engine rows, the highest applicable tier is listed; lower tiers
inherit per [conformance.md](conformance.md) §3.4.

### 1.3 Coverage

The 13 source documents listed below contain every R-numbered
normative requirement in 0.1.0-draft. 212 requirements total.
Foundational vocabulary documents (`compound.md`, `scope.md`,
`channel.md`, `schema.md`, `state.md`) define terms used by these
requirements but do not themselves carry independently-numbered
requirements; their normative content is applied through the
requirements in the semantics documents that cite them.

### 1.4 Summary source

Summaries in this index are lifted from each source document's
"Normative requirements summary" section. Where a source document
pre-dates the summary-section convention, the summary here cites
the document section that states the requirement.

## 2. Cross-cutting: canonical form

**Source:** [../canonical-form.md](../canonical-form.md)

| ID | Class | Summary |
|---|---|---|
| `canonical-form.md#R-1` | X | Conforming impls MUST implement RFC 8785 JCS canonicalization. |
| `canonical-form.md#R-2` | X | Conforming impls MUST NOT transform the document beyond JCS (no re-ordering beyond key sort, no whitespace beyond JCS, no field synthesis). |
| `canonical-form.md#R-3` | X | Hashing MUST be SHA-256 over the canonical bytes. |
| `canonical-form.md#R-4` | X | Sub-object canonicalization applies the same JCS rules to the sub-object (for mark-target and signature hashing). |
| `canonical-form.md#R-5` | X | Extensions are part of canonical form; MUST NOT be stripped before hashing. |
| `canonical-form.md#R-6` | X | Engines MUST NOT add fields during save that were not present on load. |
| `canonical-form.md#R-7` | X | Two documents with the same `id` but different canonical bytes are distinct; registries MUST detect this. |

## 3. Cross-cutting: versioning

**Source:** [../versioning.md](../versioning.md)

| ID | Class | Summary |
|---|---|---|
| `versioning.md#R-1` | X | Spec version matches SemVer 2.0.0 grammar. |
| `versioning.md#R-2` | X | Major = breaking (incl. new `MUST`); minor = additive; patch = editorial. |
| `versioning.md#R-3` | L1, AUTH | Draft spec MUST NOT be used in production. |
| `versioning.md#R-4` | X | Drafts are not ordered against each other. |
| `versioning.md#R-5` | X | Release removes the `-draft` suffix. |
| `versioning.md#R-6` | X | Released versions are immutable except via patch increments. |
| `versioning.md#R-7` | AUTH | `blueprint` field matches the version grammar exactly. |
| `versioning.md#R-8` | L1 | Draft-targeted Blueprints refused by production engines. |
| `versioning.md#R-9` | AUTH | Target version is fixed at authoring; change creates a new document. |
| `versioning.md#R-10` | L1, VAL | Engines declare a SET of supported versions. |
| `versioning.md#R-11` | L1, VAL | Declaration identifies full identifiers, including pre-release. |
| `versioning.md#R-12` | L1, VAL | Engines refuse Blueprints outside their set, with diagnostic. |
| `versioning.md#R-13` | L1 | No partial or best-effort loading across versions. |
| `versioning.md#R-14` | L1 | Forward boundary: newer-than-set is refused. |
| `versioning.md#R-15` | X | Minor bumps are strictly additive. |
| `versioning.md#R-16` | X | Major bumps require a migration note in the spec. |
| `versioning.md#R-17` | L1 | Unpublished schema ⇒ not released; engines must not declare support. |
| `versioning.md#R-18` | L1 | Draft versions MAY be tagged and published for review. |
| `versioning.md#R-19` | L1 | Engines MUST NOT auto-migrate across majors. |

## 4. Cross-cutting: extensions

**Source:** [../extensions.md](../extensions.md)

| ID | Class | Summary |
|---|---|---|
| `extensions.md#R-1` | X | `x-` and `ext.` identical at engine level. |
| `extensions.md#R-2` | X | Extension keys are case-sensitive. |
| `extensions.md#R-3` | X | Reverse-DNS in `ext.` is RECOMMENDED, not REQUIRED. |
| `extensions.md#R-4` | X | Extensions MAY appear on any object. |
| `extensions.md#R-5` | AUTH | Extensions MUST NOT replace required standard fields. |
| `extensions.md#R-6` | L1, AUTH | Extensions MUST NOT change the meaning of standard fields. |
| `extensions.md#R-7` | AUTH | Extensions on arrays are disallowed (arrays aren't objects). |
| `extensions.md#R-8` | X | Standard field names MUST NOT begin with `x-` or `ext.`. |
| `extensions.md#R-9` | L1, AUTH, VAL | Impls preserve unknown extensions round-trip. |
| `extensions.md#R-10` | L1, VAL | Engines MUST NOT alter behavior on unknown extensions. |
| `extensions.md#R-11` | L1, VAL | Engines MUST NOT refuse for unknown extensions alone. |
| `extensions.md#R-12` | L1 | Recognized extensions MUST be documented. |
| `extensions.md#R-13` | L1 | Recognized behavior MUST NOT violate standard requirements. |
| `extensions.md#R-14` | L1 | Shared keys require agreed semantics, or distinct keys. |
| `extensions.md#R-15` | L1 | Required-extension deployment constraint is engine-scoped. |
| `extensions.md#R-16` | X | Extensions participate in canonical form and signatures. |
| `extensions.md#R-17` | L1, AUTH | No stripping extensions before hashing or signing. |
| `extensions.md#R-18` | L1 | Engines MUST NOT add extensions to Blueprints they didn't author. |
| `extensions.md#R-19` | L1, VAL | Unknown extensions on marks preserved and opaque. |
| `extensions.md#R-20` | L1, VAL | Unknown extensions on signatures preserved and opaque. |
| `extensions.md#R-21` | L3, AUTH | Extensions on marks and signatures ARE signature-covered. |
| `extensions.md#R-22` | L1 | Imports with unknown extensions MUST NOT be refused. |
| `extensions.md#R-23` | L1 | Recognized-extension handling in imports is engine-policy. |
| `extensions.md#R-24` | X | New standard fields never collide with extension prefixes. |
| `extensions.md#R-25` | X | Standardizing an extension-space feature uses a new name. |
| `extensions.md#R-26` | L1, VAL | Extension-related refusal produces structured diagnostic. |

## 5. Vocabulary: blueprint document

**Source:** [../vocabulary/blueprint.md](../vocabulary/blueprint.md)

Blueprint document shape and validity. Summaries derived from
[blueprint.md §12](../vocabulary/blueprint.md#12-normative-requirements-summary).

| ID | Class | Summary |
|---|---|---|
| `vocabulary/blueprint.md#R-1` | X | Blueprints are JSON objects. |
| `vocabulary/blueprint.md#R-2` | X | Encoding is valid UTF-8 JSON per RFC 8259. |
| `vocabulary/blueprint.md#R-3` | L1, VAL | Unrecognized non-extension top-level fields rejected. |
| `vocabulary/blueprint.md#R-4` | L1, VAL | Extension top-level fields preserved; no behavior attributed. |
| `vocabulary/blueprint.md#R-5` | L1 | `blueprint` field checked before any other field. |
| `vocabulary/blueprint.md#R-6` | L1 | Unsupported versions rejected with clear diagnostic. |
| `vocabulary/blueprint.md#R-7` | L1 | `-draft` versions MUST NOT be deployed to production. |
| `vocabulary/blueprint.md#R-8` | AUTH, VAL | `id` is a syntactically valid URI per RFC 3986. |
| `vocabulary/blueprint.md#R-9` | X | Same-`id` different-canonical-form is a conflict; registries detect. |
| `vocabulary/blueprint.md#R-10` | X | `id` is independent of spec version. |
| `vocabulary/blueprint.md#R-11` | AUTH, VAL | `compounds` non-empty. |
| `vocabulary/blueprint.md#R-12` | AUTH, VAL | Compound names unique within a Blueprint. |
| `vocabulary/blueprint.md#R-13` | AUTH | Compound names SHOULD follow identifier rules; escape mark targets correctly. |
| `vocabulary/blueprint.md#R-14` | L1, VAL | `entry` names a composed compound in `compounds`. |
| `vocabulary/blueprint.md#R-15` | AUTH, VAL | Entry compound has no parameters. |
| `vocabulary/blueprint.md#R-16` | AUTH, VAL | Exactly one `entry`. |
| `vocabulary/blueprint.md#R-17` | L1, VAL | Top-level schemas conform to schema.md; validated at load. |
| `vocabulary/blueprint.md#R-18` | AUTH, VAL | Each import declares a unique non-empty alias. |
| `vocabulary/blueprint.md#R-19` | L1 | All imports resolve at load time; failure fails the load. |
| `vocabulary/blueprint.md#R-20` | L1 | Imports carry version pins; pin mismatch fails the load. |
| `vocabulary/blueprint.md#R-21` | L1, AUTH | Marks conform to mark.md; array order preserved. |
| `vocabulary/blueprint.md#R-22` | L1, AUTH | Signatures conform to signature.md; references resolve. |
| `vocabulary/blueprint.md#R-23` | X | `status`, when present, is `"draft"` or `"released"`. |
| `vocabulary/blueprint.md#R-24` | L3, AUTH | `"released"` requires all draft.md §5 conditions. |
| `vocabulary/blueprint.md#R-25` | L1 | Engines do not interpret `metadata` semantically. |
| `vocabulary/blueprint.md#R-26` | AUTH | Unknown metadata members do not affect conformance. |
| `vocabulary/blueprint.md#R-27` | L1, VAL | Diagnostics distinguish well-formed from valid. |
| `vocabulary/blueprint.md#R-28` | L1 | Engines execute only valid Blueprints. |

## 6. Vocabulary: draft and release

**Source:** [../vocabulary/draft.md](../vocabulary/draft.md)

Draft/released lifecycle and the three release conditions.

| ID | Class | Summary |
|---|---|---|
| `vocabulary/draft.md#R-1` | AUTH | Default status is `"draft"` when unspecified. |
| `vocabulary/draft.md#R-2` | L3 | Release requires `status: "released"`. |
| `vocabulary/draft.md#R-3` | L3, AUTH | Release requires a release mark targeting `"blueprint"`. |
| `vocabulary/draft.md#R-4` | L3, AUTH | Release requires a valid signature over the release mark. |
| `vocabulary/draft.md#R-5` | L3 | All three conditions together — status + mark + signature — constitute release. |
| `vocabulary/draft.md#R-6` | L3 | Any one condition missing means not-released. |
| `vocabulary/draft.md#R-7` | L3 | Revocation of the release mark produces a new document (new canonical form). |
| `vocabulary/draft.md#R-8` | L3 | A released Blueprint is immutable; modifications produce new documents. |
| `vocabulary/draft.md#R-9` | L3 | Engines in production policy MUST NOT execute draft Blueprints. |
| `vocabulary/draft.md#R-10` | AUTH | Authoring tools MAY produce draft Blueprints freely. |
| `vocabulary/draft.md#R-11` | L3 | Released status MUST be verified at every load, not cached. |
| `vocabulary/draft.md#R-12` | L3 | Engines report status with a diagnostic distinguishing all failure modes. |

## 7. Vocabulary: marks

**Source:** [../vocabulary/mark.md](../vocabulary/mark.md)

Marks record authorship events. 6 actions, 3 author kinds.

| ID | Class | Summary |
|---|---|---|
| `vocabulary/mark.md#R-1` | L1, AUTH | Marks reside in the top-level `marks` array. |
| `vocabulary/mark.md#R-2` | L1, AUTH | Marks MUST NOT be modified after being written. |
| `vocabulary/mark.md#R-3` | L1, AUTH | Engines accept both `id` forms (UUID, sha256 content hash). |
| `vocabulary/mark.md#R-4` | AUTH | Authoring tools set `kind` explicitly on every author. |
| `vocabulary/mark.md#R-5` | AUTH | `kind: machine` requires the `machine` sub-object; forbids `tool`. |
| `vocabulary/mark.md#R-6` | AUTH | `kind: tool` requires the `tool` sub-object; forbids `machine`. |
| `vocabulary/mark.md#R-7` | AUTH | `kind: human` forbids both `machine` and `tool` sub-objects. |
| `vocabulary/mark.md#R-8` | AUTH | `action` is one of the 6 enumerated values. |
| `vocabulary/mark.md#R-9` | AUTH | `target` identifies blueprint, an instance path, or a compound name. |
| `vocabulary/mark.md#R-10` | AUTH | `timestamp` is RFC 3339 date-time. |
| `vocabulary/mark.md#R-11` | AUTH | `parents` (when present) references existing marks. |
| `vocabulary/mark.md#R-12` | L1 | Engines preserve mark order and do not re-order marks. |
| `vocabulary/mark.md#R-13` | AUTH | `release` marks target `"blueprint"`, not a compound. |
| `vocabulary/mark.md#R-14` | L1 | Marks array is read-only at runtime (also authorship.md R-6). |
| `vocabulary/mark.md#R-15` | AUTH | Mark `id`, if sha256 content hash, hashes the mark's canonical form minus `id`. |
| `vocabulary/mark.md#R-16` | L1, AUTH | Duplicate mark ids in a single `marks` array are rejected. |

## 8. Vocabulary: signatures

**Source:** [../vocabulary/signature.md](../vocabulary/signature.md)

Signatures cryptographically attest marks.

| ID | Class | Summary |
|---|---|---|
| `vocabulary/signature.md#R-1` | AUTH | `signature` field is base64url-encoded. |
| `vocabulary/signature.md#R-2` | AUTH | `public_key` field is a format recognized by the algorithm. |
| `vocabulary/signature.md#R-3` | L3 | Conforming engines MUST support `Ed25519`. |
| `vocabulary/signature.md#R-4` | L3 | Engines MAY support `ECDSA-P-256-SHA-256` and `RSA-PSS-SHA-256`. |
| `vocabulary/signature.md#R-5` | L3 | Unknown-algorithm signatures produce a distinct diagnostic, not a generic failure. |
| `vocabulary/signature.md#R-6` | AUTH | Extensions MAY define additional algorithms under `x-` / `ext.` namespace. |
| `vocabulary/signature.md#R-7` | L3, AUTH | Signatures whose `mark` refers to a non-mark (non-`"blueprint"`) element are rejected. |
| `vocabulary/signature.md#R-8` | L3 | Engines implement verification steps 1–4 exactly as specified. |
| `vocabulary/signature.md#R-9` | L3 | A signature passing steps 1–4 is *cryptographically* valid. |
| `vocabulary/signature.md#R-10` | L3 | Machine-author signatures treated as identity-bearing when key-binding resolves. |
| `vocabulary/signature.md#R-11` | L3 | Engines MUST NOT require machine signatures to be identity-bound beyond what the scheme supports. |
| `vocabulary/signature.md#R-12` | L3 | Rotated/revoked machine keys are handled per the identity scheme. |
| `vocabulary/signature.md#R-13` | L3 | Engines MUST NOT interpret `key_binding` semantically beyond identity lookup. |
| `vocabulary/signature.md#R-14` | L3, AUTH | Tools presenting signature status check both cryptographic validity and identity binding. |
| `vocabulary/signature.md#R-15` | L3 | Unsupported schemes MUST NOT be treated as identity-bound. |
| `vocabulary/signature.md#R-16` | L3, AUTH | `"released"` Blueprint has at least one valid signature over a `release` mark. |
| `vocabulary/signature.md#R-17` | L3 | After a release is signed, the `status` field is fixed at that mark. |

## 9. Semantics: composition

**Source:** [../semantics/composition.md](../semantics/composition.md)

22 requirements. Keystone of Round 5.

| ID | Class | Summary |
|---|---|---|
| `semantics/composition.md#R-1` | L1 | Composition is a function: same inputs → same instance tree. |
| `semantics/composition.md#R-2` | L1 | Instantiation produces a child scope with parameter bindings. |
| `semantics/composition.md#R-3` | L1 | Parameter values bound at instantiation site per §3. |
| `semantics/composition.md#R-4` | L1 | Imports resolve to canonical-form-pinned external compounds. |
| `semantics/composition.md#R-5` | L1 | Imported compounds resolve names against their own declarations. |
| `semantics/composition.md#R-6` | L1 | Overlay is a distinct compound at composition; no partial merging. |
| `semantics/composition.md#R-7` | L1, AUTH | Parameter bindings are literals or references only — no string-embedded logic. |
| `semantics/composition.md#R-8` | L1 | Engines MUST reject a binding value containing expression-language text. |
| `semantics/composition.md#R-9` | L1 | Parameter-to-parameter refs use `params.<name>` against the enclosing compound. |
| `semantics/composition.md#R-10` | L1 | Parameter refs resolve at load; no late binding. |
| `semantics/composition.md#R-11` | L1 | Parameter binding is validated at load against the parameter schema. |
| `semantics/composition.md#R-12` | L1 | Name resolution: local compounds → qualified imports → core primitives. |
| `semantics/composition.md#R-13` | L1 | Name-resolution fallback is strict; no cross-namespace fallback. |
| `semantics/composition.md#R-14` | L1 | Instantiation cycles (direct/transitive) MUST be detected and rejected. |
| `semantics/composition.md#R-15` | L1 | Overlay is not itself a cycle. |
| `semantics/composition.md#R-16` | L1 | Cycles through imports are detected like local cycles. |
| `semantics/composition.md#R-17` | L1 | Load sequence has 7 ordered steps. |
| `semantics/composition.md#R-18` | L1 | Failure at any load step halts the load; no partial loading. |
| `semantics/composition.md#R-19` | L1 | Instance addresses are slash paths using local names. |
| `semantics/composition.md#R-20` | L1 | JSON-Pointer (RFC 6901 §3) escaping applies to path segments. |
| `semantics/composition.md#R-21` | L1 | Instance addresses stable; MUST NOT be reused within a run. |
| `semantics/composition.md#R-22` | L1 | Composition diagnostics are deterministic given the same inputs. |

## 10. Semantics: lifecycle

**Source:** [../semantics/lifecycle.md](../semantics/lifecycle.md)

14 requirements. Three phases + snapshot/restore.

| ID | Class | Summary |
|---|---|---|
| `semantics/lifecycle.md#R-1` | L1 | Run requires a successful load. |
| `semantics/lifecycle.md#R-2` | L1 | Begin is atomic: steps 1–4 before observability. |
| `semantics/lifecycle.md#R-3` | L1 | State is fully initialized before inputs flow. |
| `semantics/lifecycle.md#R-4` | L1 | Begin propagates depth-first. |
| `semantics/lifecycle.md#R-5` | L1 | No extra-rule mutation during run. |
| `semantics/lifecycle.md#R-6` | L1 | Children end before parents. |
| `semantics/lifecycle.md#R-7` | L1 | Instance addresses MUST NOT be reused. |
| `semantics/lifecycle.md#R-8` | L1 | Scope end is observable. |
| `semantics/lifecycle.md#R-9` | L1 | Stop is deterministic; no orphans. |
| `semantics/lifecycle.md#R-10` | L2 | Snapshot/restore portability invariant. |
| `semantics/lifecycle.md#R-11` | L2 | Snapshots at safe points only. |
| `semantics/lifecycle.md#R-12` | L2 | Restore validates state; no partial restore. |
| `semantics/lifecycle.md#R-13` | L2 | Addresses stable across snapshot/restore. |
| `semantics/lifecycle.md#R-14` | L2 | Snapshots MUST NOT require engine-specific metadata. |

## 11. Semantics: scope propagation

**Source:** [../semantics/scope-propagation.md](../semantics/scope-propagation.md)

10 requirements.

| ID | Class | Summary |
|---|---|---|
| `semantics/scope-propagation.md#R-1` | L1 | No late name resolution at runtime. |
| `semantics/scope-propagation.md#R-2` | L1 | Unresolved names fail at load. |
| `semantics/scope-propagation.md#R-3` | L1 | Default capability inheritance is full. |
| `semantics/scope-propagation.md#R-4` | L1 | Inheritance is snapshot-at-begin; later acquisitions do not propagate. |
| `semantics/scope-propagation.md#R-5` | L1 | Restrictions enforced at inheritance point. |
| `semantics/scope-propagation.md#R-6` | L1 | Restrictions cannot widen; engines reject attempts. |
| `semantics/scope-propagation.md#R-7` | L1 | No upward capability flow. |
| `semantics/scope-propagation.md#R-8` | L2 | Snapshots do not carry raw capability values. |
| `semantics/scope-propagation.md#R-9` | L1 | Imported instances inherit from their instantiating scope. |
| `semantics/scope-propagation.md#R-10` | L1 | Capability failures diagnosed, not silently degraded. |

## 12. Semantics: channel delivery

**Source:** [../semantics/channel-delivery.md](../semantics/channel-delivery.md)

18 requirements.

| ID | Class | Summary |
|---|---|---|
| `semantics/channel-delivery.md#R-1` | L1, AUTH | Default delivery mode is `asynchronous-ordered`. |
| `semantics/channel-delivery.md#R-2` | L1 | Engines unable to meet the mode refuse the Blueprint. |
| `semantics/channel-delivery.md#R-3` | L1 | Synchronous channels are intra-engine. |
| `semantics/channel-delivery.md#R-4` | L1 | No silent downgrade at deploy time. |
| `semantics/channel-delivery.md#R-5` | L1 | Backpressure events are diagnosed. |
| `semantics/channel-delivery.md#R-6` | L1 | Buffer size is operator-configurable, not authored. |
| `semantics/channel-delivery.md#R-7` | L1 | Per-producer order preserved. |
| `semantics/channel-delivery.md#R-8` | L1 | Fan-in interleaving preserves per-producer order. |
| `semantics/channel-delivery.md#R-9` | L1 | Interleaving deterministic on replay. |
| `semantics/channel-delivery.md#R-10` | L1 | No cross-channel ordering guarantee. |
| `semantics/channel-delivery.md#R-11` | L1 | Closed producer endpoint accepts no new messages. |
| `semantics/channel-delivery.md#R-12` | L1 | Closed consumer endpoint drops further messages; drops diagnosed. |
| `semantics/channel-delivery.md#R-13` | L2 | In-flight delivery survives snapshot for reliable modes. |
| `semantics/channel-delivery.md#R-14` | L1 | Validation failures reject-diagnose-continue. |
| `semantics/channel-delivery.md#R-15` | L1 | No implicit error channel synthesized by engines. |
| `semantics/channel-delivery.md#R-16` | L1 | Validation elision requires proven type safety. |
| `semantics/channel-delivery.md#R-17` | L1 | Wire encoding preserves validation. |
| `semantics/channel-delivery.md#R-18` | L1 | Cross-process honors mode or refuses to establish. |

## 13. Semantics: determinism

**Source:** [../semantics/determinism.md](../semantics/determinism.md)

7 requirements.

| ID | Class | Summary |
|---|---|---|
| `semantics/determinism.md#R-1` | L1 | Observable equivalence definition. |
| `semantics/determinism.md#R-2` | L1 | Structural determinism of the instance tree. |
| `semantics/determinism.md#R-3` | L1 | Load-time diagnostics deterministic. |
| `semantics/determinism.md#R-4` | L1 | Replay equivalence given input trace. |
| `semantics/determinism.md#R-5` | L1 | No non-determinism outside the closed list. |
| `semantics/determinism.md#R-6` | L1 | Primitives use enumerated sources only. |
| `semantics/determinism.md#R-7` | L1 | No silent bit-exact provision by engines. |

## 14. Semantics: authorship (runtime)

**Source:** [../semantics/authorship.md](../semantics/authorship.md)

16 requirements.

| ID | Class | Summary |
|---|---|---|
| `semantics/authorship.md#R-1` | L3 | Engines MUST document signature-verification policy. |
| `semantics/authorship.md#R-2` | L3 | When enabled, run verification on every signature. |
| `semantics/authorship.md#R-3` | L1 | Disabled verification MUST be reported. |
| `semantics/authorship.md#R-4` | L3 | Released status requires all three draft.md §5 conditions. |
| `semantics/authorship.md#R-5` | L3 | Production engines SHOULD refuse drafts (NB: SHOULD, not testable for conformance). |
| `semantics/authorship.md#R-6` | L1 | Engines MUST NOT mutate the `marks` array at runtime. |
| `semantics/authorship.md#R-7` | L1 | Engines MUST NOT synthesize marks from runtime events. |
| `semantics/authorship.md#R-8` | L1 | Observability surface MUST expose marks and signature status. |
| `semantics/authorship.md#R-9` | L1 | Additional projections MAY be provided (NB: MAY, not testable). |
| `semantics/authorship.md#R-10` | L1 | Secrets MUST NOT be observable. |
| `semantics/authorship.md#R-11` | L2 | Snapshots preserve marks and signatures exactly. |
| `semantics/authorship.md#R-12` | L3 | Restore re-runs signature verification. |
| `semantics/authorship.md#R-13` | L1 | Engines MUST NOT sign at runtime. |
| `semantics/authorship.md#R-14` | L3 | Policy MUST describe revocation handling. |
| `semantics/authorship.md#R-15` | L3 | Load MUST NOT treat revoked release marks as effective. |
| `semantics/authorship.md#R-16` | L3 | Late revocation MUST NOT be masked on the observability surface. |

## 15. Conformance

**Source:** [conformance.md](conformance.md)

16 requirements. These govern the conformance claim itself.

| ID | Class | Summary |
|---|---|---|
| `conformance.md#R-1` | X | Claim covers only MUST / MUST NOT / REQUIRED / SHALL / SHALL NOT. |
| `conformance.md#R-2` | X | Claims are per-version and independent. |
| `conformance.md#R-3` | L1 | Engine tier claim must accurately reflect implementation. |
| `conformance.md#R-4` | AUTH | Authoring tools produce only schema-valid documents. |
| `conformance.md#R-5` | VAL | Validators do not execute. |
| `conformance.md#R-6` | VAL | Validators may decline signature verification but must report honestly. |
| `conformance.md#R-7` | X | Extension recognition does not expand the claim. |
| `conformance.md#R-8` | X | Required-extension constraint documented in the claim. |
| `conformance.md#R-9` | X | Conformance claim JSON shape. |
| `conformance.md#R-10` | X | Claim published at a durable URL. |
| `conformance.md#R-11` | X | Conformance assertions in marketing link the machine-readable claim. |
| `conformance.md#R-12` | X | Claims apply to exactly one implementation version. |
| `conformance.md#R-13` | X | Draft-spec `test_results` format. |
| `conformance.md#R-14` | X | Released-spec `test_results` format. |
| `conformance.md#R-15` | X | Claimed tests must have been actually performed. |
| `conformance.md#R-16` | X | Inaccurate claims must be retracted or updated. |

## 16. Tier/class rollup

Counts by class/tier across the 212 normatively-numbered
requirements in §§2–14 plus the 16 meta-requirements from §15.

| Class/Tier | Count | Notes |
|---|---|---|
| L1 (Engine Core) | ~125 | Load, run, stop, composition, scope, channels, determinism, mark read-only at runtime, structural rules |
| L2 (Engine Durable) | +9 | Snapshot/restore additions from lifecycle.md §6, scope-propagation.md R-8, channel-delivery.md R-13, authorship.md R-11 |
| L3 (Engine Verified) | +26 | Signature verification, release enforcement, revocation (authorship.md §2+§7, signature.md, draft.md) |
| AUTH (Authoring tool) | ~55 | Canonical form, mark production, signature production, release conditions, schema conformance, version declaration, extension authoring |
| VAL (Validator) | ~40 | Schema, canonical form, structural validity, signature verification (when performed), faithful reporting |
| X (Cross-cutting) | 52 | Canonical form (7), versioning (19), extensions (26) apply to every class that processes Blueprints |

Counts are approximate for the composite tags (L1 + AUTH, etc.);
the requirement-by-requirement mapping in §§2–14 is the
authoritative source.

## 17. How to use this index

### 17.1 For implementers

1. Identify your claimed class and tier
   ([conformance.md](conformance.md) §§2–3).
2. Extract the applicable rows from §§2–15 (this index).
3. Write a test for each applicable requirement per
   [test-suite.md](test-suite.md) §2.
4. Populate `per_requirement` in your conformance claim per
   [conformance.md](conformance.md) R-13.

### 17.2 For reviewers of a conformance claim

1. Cross-check the claim's `per_requirement` entries against the
   applicable rows in this index.
2. Any missing row is a gap in coverage.
3. Any row marked `"pass"` without evidence violates
   [test-suite.md](test-suite.md) §2.3.

### 17.3 For editors of future spec versions

1. New requirements MUST be numbered in their source document.
2. New requirements MUST appear in this index in the appropriate
   section, with a tier/class mapping.
3. Removing a requirement is a major version bump
   ([versioning.md](../versioning.md) R-2).

## 18. This index is not normative

This index consolidates requirements defined elsewhere. In case of
conflict between this index's summary and the requirement's full
text in the source document, the source document governs.

---

*End of REQUIREMENTS.md.*
