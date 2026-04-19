# Authorship (runtime)

**Status:** Draft
**Version:** 0.1.0-draft
**Editor:** Anton Bursch

Part of the [Blueprint specification](../overview.md), version 0.1.0-draft.

> This document fixes the runtime behavior of marks and signatures:
> when engines emit mark events, how signatures are verified during
> load, what observability marks project into at runtime, and how
> the authorship trace travels through snapshot and restore. The
> shape of marks and signatures themselves is fixed in
> [../vocabulary/mark.md](../vocabulary/mark.md) and
> [../vocabulary/signature.md](../vocabulary/signature.md); this
> document is about the engine's obligations at runtime.

## Conformance language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this
document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119)
and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when,
they appear in all capitals.

## Dependencies

- [../vocabulary/mark.md](../vocabulary/mark.md) — mark structure and vocabulary
- [../vocabulary/signature.md](../vocabulary/signature.md) — signature structure and verification procedure
- [../canonical-form.md](../canonical-form.md) — canonical bytes for signing
- [../vocabulary/draft.md](../vocabulary/draft.md) — release conditions
- [composition.md](composition.md) — load sequence (signature verification is step 5)
- [lifecycle.md](lifecycle.md) — snapshot/restore

## 1. Scope of this document

Marks and signatures are document-level constructs. They exist in the
Blueprint artifact, not in the runtime instance tree. This document
specifies the engine's obligations at three moments where document
authorship touches runtime:

- at **load**, when signatures are verified and draft/released status
  determined;
- at **run**, when engines surface authorship information through the
  observability surface;
- at **snapshot/restore**, when the authorship trace must be
  preserved across lifecycle transitions.

This document does not define a mechanism for producing new marks at
runtime. Marks record authorship events on the document; they are
produced by authoring tools, not by engines.

## 2. Load-time signature verification

Signature verification is step 5 of the load sequence
([composition.md](composition.md) §6).

### 2.1 Policy

An engine's signature-verification policy determines which
signatures are required, which are optional, and what counts as a
failure. Policy is engine-defined in 0.1.0-draft; this document
fixes the obligations every policy MUST respect.

**R-1.** An engine MUST define its signature-verification policy in
engine documentation. The policy MUST address at minimum:

- whether signature verification is enabled by default;
- whether a Blueprint with `status` of `"released"` requires all of
  the release conditions per
  [../vocabulary/draft.md](../vocabulary/draft.md) §5;
- what trust anchors (identity roots) the engine accepts;
- how revocations are consumed.

**R-2.** When signature verification is enabled, an engine MUST run
the verification procedure from
[../vocabulary/signature.md](../vocabulary/signature.md) §5 for
every signature in the Blueprint and every signature in resolved
imports.

**R-3.** When signature verification is disabled, an engine MUST
clearly report that it is disabled in any diagnostic concerning the
load. A silent skip is a conformance violation.

### 2.2 Draft and released

**R-4.** An engine enforcing release semantics MUST refuse to treat a
Blueprint as released unless all three conditions in
[../vocabulary/draft.md](../vocabulary/draft.md) §5 are satisfied:
`status` equals `"released"`, a release mark is present, and a valid
signature over the release mark exists.

**R-5.** An engine MAY run a Blueprint whose release conditions are
unsatisfied only if its policy permits draft execution. Production
engines SHOULD refuse.

### 2.3 Draft-spec execution

Independently of release status, the `blueprint` field's
specification version is governed by
[../vocabulary/blueprint.md](../vocabulary/blueprint.md) R-7: a
`-draft` suffix MUST NOT be deployed to production. This rule is
enforced at load, not at release verification.

## 3. Marks are not runtime objects

A mark is a static record in the `marks` array of the Blueprint
document. Instantiating the Blueprint does not create runtime mark
objects.

**R-6.** Engines MUST NOT add, modify, reorder, or remove marks on
a loaded Blueprint at runtime. The `marks` array is read-only from
the engine's perspective.

**R-7.** Engines MUST NOT synthesize marks from runtime events.
Mark creation is an authoring act, performed by tools that write
Blueprints, not by engines that execute them.

## 4. Authorship observability

An engine's observability surface makes runtime events inspectable.
Authorship observability is the subset that exposes document-level
authorship information at runtime.

### 4.1 What is observable

**R-8.** An engine's observability surface MUST make available, on
request:

- the full `marks` array of the loaded Blueprint;
- for any given compound or compound member, the marks whose
  `target` identifies it;
- the validity status of every signature (cryptographically valid,
  identity-bound, failed, or not verified), per
  [../vocabulary/signature.md](../vocabulary/signature.md) §5.

**R-9.** An engine MAY additionally project:

- author-kind summaries per scope (how many marks authored by
  humans, machines, tools);
- mark-event projections at scope begin and end, identifying the
  mark(s) that relate to the instantiated compound.

### 4.2 What is not observable

**R-10.** An engine MUST NOT expose the private keys, signing
infrastructure, or verification-trust-anchor secrets through the
observability surface. Public keys and key-binding metadata MAY be
projected; the secret halves MUST NOT be.

### 4.3 Relationship to channel-delivery diagnostics

Observability of authorship is independent of observability of
channel-delivery and lifecycle events. Implementations MAY use the
same underlying transport; the surfaces are specified independently.

## 5. Snapshot and restore

Authorship information is part of the loaded Blueprint's canonical
form, which is part of the snapshot
([lifecycle.md](lifecycle.md) §6.1).

**R-11.** A snapshot MUST preserve the loaded Blueprint's `marks`
and `signatures` exactly. A restored Blueprint MUST present the same
authorship trace as the snapshotted Blueprint.

**R-12.** A restored Blueprint MUST re-run the signature-verification
procedure per its engine's policy. A snapshot's previous verification
result is informative only; on restore, an engine MUST perform its
own verification (with its own trust anchors).

## 6. Signing at runtime

Runtime signing — the act of producing a new signature while a
Blueprint is live — is out of scope for 0.1.0-draft. Signatures are
produced by authoring tools before release, captured in the
document, and verified by engines at load.

**R-13.** Engines MUST NOT produce signatures over a loaded
Blueprint at runtime. Any signing operation belongs to an authoring
tool that writes a new Blueprint; engines execute, not author.

A future revision MAY define a runtime signing surface for engines
that act as authoring services; its design is out of scope here.

## 7. Revocation

Revocation is handled at two layers per
[../vocabulary/signature.md](../vocabulary/signature.md) §9:

- **Mark-level revocation.** A `revoke` mark supersedes a prior
  approval or release.
- **Key-level revocation.** A key's binding may be revoked by the
  identity system that issued it.

**R-14.** An engine's signature-verification policy MUST describe
how it obtains and consumes revocation information (mark-level from
the Blueprint itself, key-level from the identity scheme the engine
supports).

**R-15.** At load, an engine MUST refuse to treat a revoked release
mark as effective. A Blueprint whose only release mark has been
revoked by a later `revoke` mark MUST be loaded as a draft (the
`status` field is unchanged by revocation;
[../vocabulary/draft.md](../vocabulary/draft.md) §7 explains the
mechanism: revocation produces a new Blueprint).

### 7.1 Effect of revocation on running Blueprints

If a signature supporting a running Blueprint's release is revoked
after load, the running Blueprint does not spontaneously change
state. Engines MAY continue executing a running Blueprint whose
underlying release has been revoked; engines SHOULD surface a
diagnostic at the next observability-surface query.

**R-16.** Whether to halt a running Blueprint on late-arriving
revocation is an engine-policy decision. Engines MUST NOT mask the
revocation: if policy is to continue, the observability surface MUST
report the revoked status.

## 8. Normative requirements summary

- **R-1** Engines MUST document signature-verification policy.
- **R-2** When enabled, run verification on every signature.
- **R-3** Disabled verification MUST be reported.
- **R-4** Released status requires all three draft.md §5 conditions.
- **R-5** Production engines SHOULD refuse drafts.
- **R-6** Engines MUST NOT mutate the `marks` array at runtime.
- **R-7** Engines MUST NOT synthesize marks from runtime events.
- **R-8** Observability surface MUST expose marks and signature status.
- **R-9** Additional projections MAY be provided.
- **R-10** Secrets MUST NOT be observable.
- **R-11** Snapshots preserve marks and signatures exactly.
- **R-12** Restore re-runs signature verification.
- **R-13** Engines MUST NOT sign at runtime.
- **R-14** Policy MUST describe revocation handling.
- **R-15** Load MUST NOT treat revoked release marks as effective.
- **R-16** Late revocation MUST NOT be masked on the observability surface.

## 9. Open questions

- Whether a standard observability-surface protocol for authorship
  queries belongs in a later revision (for example, a read-only
  marks-and-signatures API).
- Whether late-revocation halt semantics (§7.1) should be normative
  in a later revision rather than engine-policy.
- Whether runtime signing (§6) is ever appropriate, or whether the
  authoring/engine split should remain permanent.

---

*End of authorship.md.*
