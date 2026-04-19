# Draft

**Status:** Draft
**Version:** 0.1.0-draft
**Editor:** Anton Bursch

Part of the [Blueprint specification](../overview.md), version 0.1.0-draft.

> This document defines **draft** as a formal lifecycle state of a
> Blueprint: work-in-progress, freely editable, not committed to a
> verifiable form. A Blueprint begins its life as a draft. It leaves
> the draft state only through an explicit, signed act of release.
> Once released, the Blueprint is immutable: further authoring happens
> on a new Blueprint, not on the released one. The term is both a verb
> (to *draft* a Blueprint) and an adjective (a *draft* Blueprint); both
> senses are normative here.

## Conformance language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this
document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119)
and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when,
they appear in all capitals.

## Dependencies

- [overview.md](../overview.md) — §4.7 introduces draft; §7.4 introduces release
- [mark.md](mark.md) — release is a mark action; draft transitions are marked
- [signature.md](signature.md) — release requires a signed release mark
- [canonical-form.md](../canonical-form.md) — release commits the canonical bytes

## 1. What draft is

A Blueprint is in **draft state** when any of the following holds:

1. Its top-level `status` field is absent.
2. Its top-level `status` field is the string `"draft"`.

Any other value of `status`, including `"released"`, removes the
Blueprint from draft state.

Draft state is not a separate file format. A draft Blueprint is the
same artifact as a released Blueprint; the only difference is which
rules this specification applies to it.

**R-1.** A conforming engine MUST determine draft status exclusively
from the `status` field. No heuristic (presence of signatures,
completeness of compounds, existence of release marks) may override
the field's value.

**R-2.** A Blueprint whose `status` field is neither absent, `"draft"`,
nor `"released"` is malformed. Engines MUST reject it with a
diagnostic identifying the invalid value.

## 2. Operations permitted in draft state

In draft state, a Blueprint MAY:

- Have compounds added, modified, or removed.
- Have marks appended, including marks of any action defined in
  [mark.md](mark.md) §4.
- Have signatures appended.
- Have its `metadata` field altered freely.
- Be loaded by authoring tools, sandbox engines, and development
  environments.

**R-3.** Authoring tools MAY freely edit a draft Blueprint. Each
substantive edit SHOULD be accompanied by a mark per
[mark.md](mark.md) §1.

**R-4.** Sandbox and development engines MAY execute a draft
Blueprint. Production engines SHOULD refuse to execute a draft
Blueprint and MUST clearly report draft status in any diagnostic when
doing so.

## 3. Operations prohibited in draft state

The only operation prohibited in draft state is the operation that
*leaves* draft state: committing the Blueprint as released. Release
has its own rules (§5). A draft Blueprint MUST NOT be treated as
released until the transition in §5 has occurred.

**R-5.** Engines MUST NOT honor release-gated features (immutability
guarantees, release-version identity, deployment-to-production
policies) for a Blueprint whose `status` is not `"released"`.

## 4. Marks in draft state

All mark actions defined in [mark.md](mark.md) §4 are valid in draft
state, including `release`. A `release` mark in a draft Blueprint
records an *intent* to release. The Blueprint does not actually become
released until §5 is satisfied.

**R-6.** A `release` mark in a Blueprint whose `status` is `"draft"`
or absent is advisory only. It MUST NOT be treated as effective. It
remains in the `marks` array and survives the transition when release
happens.

The mark history of a draft Blueprint is authoritative. Tools MUST
preserve it across edits (per [mark.md](mark.md) §6). Draft state does
not license removal, reordering, or squashing of marks.

## 5. Transition to released

A Blueprint transitions from draft state to released state by a single
operation:

1. Set the Blueprint's top-level `status` field to `"released"`.
2. Ensure the `marks` array contains at least one mark of action
   `release` (per [mark.md](mark.md) §4).
3. Ensure the `signatures` array contains at least one signature whose
   `mark` field equals the `id` of that release mark, and whose
   cryptographic verification succeeds per
   [signature.md](signature.md) §5 steps 1–4.

**R-7.** A Blueprint whose `status` is `"released"` MUST satisfy all
three conditions above. A Blueprint that sets `status` to `"released"`
without a valid signed release mark is malformed. Engines MUST reject
it with a diagnostic identifying which condition is unsatisfied.

**R-8.** The canonical bytes that a release signature signs over are
the canonical form of the Blueprint at the moment of release (see
[canonical-form.md](../canonical-form.md) §4.1). Those bytes are what
the release commits to. Any subsequent change to the document changes
its canonical bytes and therefore invalidates the release signature.

## 6. Released state and immutability

Once a Blueprint is released, the signed bytes are fixed.

**R-9.** A released Blueprint MUST NOT be modified in place. Engines
and authoring tools MUST refuse to write changes to a Blueprint whose
`status` is `"released"` and whose release signature is still valid.

**R-10.** Continued authoring on a released Blueprint is performed by
producing a **new Blueprint**: a fresh document with either a new `id`
or an incremented version in its `metadata`, copying the released
Blueprint's `marks` array as the starting history. The new Blueprint
enters draft state.

The released artifact remains addressable, verifiable, and replayable
indefinitely by its `id`. It is the commitment of record.

## 7. Returning a released Blueprint to draft

A released Blueprint may be returned to draft state by revoking its
release, not by editing its `status` field in place.

**R-11.** To revert a released Blueprint to draft, an authoring tool
MUST produce a new Blueprint (per R-10) that:

- Has `status` absent or `"draft"`.
- Contains a mark of action `revoke` whose target is the release mark
  of the original Blueprint, per [mark.md](mark.md) §7 and
  [signature.md](signature.md) §9.3.
- Has a fresh `id` or incremented version.

The original released Blueprint itself is unchanged; it remains the
historical record of what was released. The new draft supersedes it
for future authoring.

## 8. Forking a released Blueprint

Forking is the same mechanism as R-10: produce a new Blueprint with a
new `id`, copy marks, enter draft state. A fork is not a special
operation in the specification; it is a particular use of the
general rule that continued authoring produces a new Blueprint.

**R-12.** A fork MUST carry a mark of action `create` whose
`rationale` or `provenance` identifies the Blueprint being forked.
Tools SHOULD include the source Blueprint's `id` in the mark so that
the fork's origin is discoverable.

## 9. Status transitions

The complete state machine:

```
(none | "draft")  --release-->  "released"
"released"        --supersede-->  (new Blueprint, "draft" by R-10/R-11)
```

The machine has no other transitions. Editing `status` directly from
`"released"` to `"draft"` on the same document is forbidden (violates
R-9).

## 10. Examples

### 10.1 A draft with no status field

```json
{
  "blueprint": "0.1.0-draft",
  "id": "urn:blueprint:example:dashboard",
  "compounds": { "root": { /* ... */ } },
  "entry": "root",
  "marks": [
    {
      "id": "sha256:aa11...",
      "target": "blueprint",
      "action": "create",
      "author": { "kind": "human", "id": "person:anton-bursch" },
      "timestamp": "2026-04-18T14:00:00-04:00"
    }
  ]
}
```

Status is absent, so this Blueprint is in draft state. Further marks
may be appended. Sandbox engines may execute it; production engines
will refuse.

### 10.2 A released Blueprint

```json
{
  "blueprint": "0.1.0-draft",
  "id": "urn:blueprint:example:dashboard",
  "status": "released",
  "compounds": { "root": { /* ... */ } },
  "entry": "root",
  "marks": [
    { "id": "sha256:aa11...", "action": "create", /* ... */ },
    { "id": "sha256:bb22...", "action": "approve", /* ... */ },
    { "id": "sha256:cc33...", "target": "blueprint", "action": "release",
      "author": { "kind": "human", "id": "person:anton-bursch" },
      "timestamp": "2026-04-19T11:05:00-04:00" }
  ],
  "signatures": [
    { "id": "sha256:dd44...", "mark": "sha256:cc33...",
      "algorithm": "Ed25519", "public_key": "…", "signature": "…" }
  ]
}
```

### 10.3 Superseding a released Blueprint

A second Blueprint with a new `id` (`urn:blueprint:example:dashboard/v2`
or `urn:blueprint:example:dashboard@2`, depending on registry
conventions) is authored. Its first mark identifies the predecessor.
Status is absent; the new Blueprint is a draft.

## 11. Normative requirements (summary)

1. Draft status is determined exclusively by the `status` field (R-1).
2. Invalid `status` values are rejected (R-2).
3. Authoring tools may freely edit drafts; edits produce marks (R-3).
4. Production engines refuse drafts; sandbox engines may execute them (R-4).
5. Release-gated features require `status: "released"` (R-5).
6. Release marks in drafts are advisory (R-6).
7. Release requires status, release mark, and valid release signature (R-7).
8. Release signs the canonical bytes at the moment of release (R-8).
9. Released Blueprints are immutable (R-9).
10. Continued authoring produces a new Blueprint (R-10).
11. Revert-to-draft is accomplished by a new Blueprint with a revoke
    mark (R-11).
12. Forks carry a `create` mark identifying the source (R-12).

## 12. Open questions (deferred)

- **Partial drafts and transport.** Whether an authoring tool may
  transmit a draft that is intentionally incomplete (e.g., for
  collaborative editing) and how engines distinguish intentional from
  unintentional incompleteness is out of scope for 0.1.0.
- **Numeric versioning for released Blueprints.** The specification
  does not yet fix where an application-level version (`1.2.3`) lives
  for released Blueprints. `metadata.version` is recommended but not
  normative.
- **Multi-release timelines.** Whether a single logical Blueprint can
  have multiple concurrent released versions (v1.5 and v2.0 both
  live) is a registry concern, not a Blueprint-document concern, and
  is deferred.
- **Amendment signatures.** A signature added to a released Blueprint
  after release (by a new approver) would require extending the
  signed-over rule. Deferred.
- **Status beyond draft/released.** Whether `status` values like
  `"deprecated"` or `"archived"` are registry concerns or Blueprint
  concerns is open.
