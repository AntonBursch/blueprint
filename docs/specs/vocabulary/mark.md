# Mark

**Status:** Draft
**Version:** 0.1.0-draft
**Editor:** Anton Bursch

Part of the [Blueprint specification](../overview.md), version 0.1.0-draft.

> This document defines the **mark**: a first-class authorship trace
> attached to an element of a Blueprint. Marks record *who* made a
> change, *what* change, *when*, and optionally *why*. Marks are
> additive: they are appended, never overwritten, and a retraction of a
> mark is itself a new mark. The term is taken from the printer's mark
> used in bookbinding to identify the shop responsible for a work.
> Marks are how Blueprint makes co-authorship between human operators
> and machine authors auditable without privileging either.

## Conformance language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this
document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119)
and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when,
they appear in all capitals.

## Dependencies

- [overview.md](../overview.md) — §4.8 introduces marks
- [rationale/co-authorship.md](../rationale/co-authorship.md) — the rationale for structural author kind
- [compound.md](compound.md) — marks target compounds and their members
- [canonical-form.md](../canonical-form.md) — mark signatures sign over canonical form
- [signature.md](signature.md) — mark signatures

## Relationship to overview §4.8

Overview §4.8 fixes the existence and top-level shape of marks. This
document is the normative source for the mark's full structure. In one
place this document supersedes the overview's shorthand:

- Overview §4.8 shows `author` as a single string identifier. This
  document specifies `author` as a **structured object** with a
  mandatory `kind` field and kind-specific sub-objects. The overview
  will be reconciled in a subsequent revision. The structured form is
  required by [rationale/co-authorship.md](../rationale/co-authorship.md)
  §4, which fixes author kind as structural rather than inferred: a
  logical description of an author must not be smuggled inside a
  free-form string.

## 1. What a mark is

A mark is a record, in the Blueprint document, of a single authorship
event on a single target. Every mark has an author, a target, a time,
and an action. A mark MAY carry a rationale, provenance, and a
signature that commits the authoring party to its content.

Marks are the primary evidence a reader has of how a Blueprint came to
be. A Blueprint with no marks is legal but opaque. A Blueprint with
marks is auditable.

**R-1.** A mark MUST reside in the top-level `marks` array of the
Blueprint document. Marks MUST NOT appear inline on the elements they
mark; they reference those elements by target (§5).

**R-2.** Marks MUST NOT be modified after they are written. A mark that
needs to be corrected or retracted is superseded by a new mark
(§6, §7).

## 2. Mark structure

A mark is a JSON object with the following members.

| Field | Requirement | Description |
| --- | --- | --- |
| `id` | MUST | Stable identifier for this mark (§2.1) |
| `target` | MUST | What this mark applies to (§5) |
| `author` | MUST | Who produced this mark (§3) |
| `timestamp` | MUST | Wall-clock time of the authoring event (§2.2) |
| `action` | MUST | One of the six action values (§4) |
| `rationale` | SHOULD | Why the change was made |
| `provenance` | MAY | Tool, session, or context that produced the change |
| `signature` | MAY | A signature committing this mark (§8) |

Any member whose key is not listed above and whose key does not begin
with an extension-namespace prefix (`x-`, `ext.`) is reserved. Engines
and authoring tools MUST NOT emit unknown non-extension fields.

### 2.1 `id`

`id` is a string that uniquely identifies the mark within its
Blueprint. Two forms are permitted:

1. A [UUID](https://www.rfc-editor.org/rfc/rfc4122) in canonical
   lowercase hexadecimal form.
2. A content-addressed form: the string `"sha256:"` followed by 64
   lowercase hexadecimal characters. The hash is computed over the
   canonical form ([canonical-form.md](../canonical-form.md)) of the
   mark with both its `id` and `signature` fields removed.

**R-3.** Engines MUST accept both `id` forms. Authoring tools
SHOULD emit the content-addressed form, because it makes marks
tamper-evident: modifying any signed-over field of a mark changes its
`id`.

### 2.2 `timestamp`

`timestamp` is an [RFC 3339](https://www.rfc-editor.org/rfc/rfc3339)
date-time string with explicit time-zone offset. The timestamp is
descriptive — it records the authoring tool's understanding of
wall-clock time. It is NOT used to order marks; ordering is governed
by array position (§6).

## 3. Author identity

The `author` field is a structured object. Its shape is fixed; it is
not a free-form string. The structural form is required because the
difference between human and machine authorship is not safely
inferable from a name, and because a logical claim about an author
must not be encoded as prose inside a string.

```json
"author": {
  "kind": "human",
  "id": "person:anton-bursch",
  "display": "Anton Bursch"
}
```

```json
"author": {
  "kind": "machine",
  "id": "machine:claude-sonnet-4.5",
  "display": "Claude Sonnet 4.5",
  "machine": {
    "model": "claude-sonnet-4.5",
    "version": "2026-03-15",
    "operator": {
      "kind": "human",
      "id": "person:anton-bursch",
      "display": "Anton Bursch"
    }
  }
}
```

```json
"author": {
  "kind": "tool",
  "id": "tool:blueprint-cli",
  "display": "Blueprint CLI",
  "tool": {
    "name": "blueprint-cli",
    "version": "0.9.2"
  }
}
```

### 3.1 Fields

| Field | Requirement | Description |
| --- | --- | --- |
| `kind` | MUST | One of `human`, `machine`, `tool` |
| `id` | MUST | Identity string (§3.3) |
| `display` | SHOULD | Human-readable name |
| `machine` | MUST when `kind` is `machine` | Machine author details (§3.4) |
| `tool` | MUST when `kind` is `tool` | Tool author details (§3.5) |

### 3.2 `kind`

`kind` is REQUIRED and MUST be exactly one of the three values
`human`, `machine`, `tool`. Authoring tools MUST NOT attempt to infer
`kind` from the `id` string. A reader determines authorship category
exclusively from this field.

**R-4.** Authoring tools MUST set `kind` explicitly for every mark
they emit.

**R-5.** Readers MUST treat `kind` as authoritative. A mark whose `id`
visually suggests a bot, service, or person does not change its
structural kind.

### 3.3 `id`

`id` is a URI-like opaque string identifying the author. This
specification does not prescribe an identity scheme; see
[signature.md](signature.md) §8 on `key_binding`. An identity scheme
MAY be reflected in the `id` prefix (`person:`, `machine:`, `did:`,
`mailto:`), but the `kind` field remains the authoritative statement
of author category.

### 3.4 Machine authors

When `kind` is `machine`, the `author.machine` sub-object is REQUIRED
and contains:

| Field | Requirement | Description |
| --- | --- | --- |
| `model` | MUST | Model identifier |
| `version` | MUST | Model version string |
| `operator` | SHOULD | The human or organization on whose behalf the machine acted |

`operator`, when present, is itself an author-shaped object with
`kind`, `id`, and optionally `display`. A machine with no operator
represents a machine acting autonomously; this is permitted and
structurally distinct from a machine acting under human direction.

### 3.5 Tool authors

When `kind` is `tool`, the `author.tool` sub-object is REQUIRED and
contains:

| Field | Requirement | Description |
| --- | --- | --- |
| `name` | MUST | Tool name |
| `version` | MUST | Tool version string |

A `tool` author represents mechanical transformations applied without
human or model judgment — a linter, a migration script, a code
generator. The distinction between `machine` and `tool` is structural:
`machine` implies non-deterministic reasoning; `tool` implies
deterministic transformation.

## 4. Actions

The `action` field fixes what the mark does. The 0.1.0-draft action
vocabulary is exactly six values:

| Action | Meaning |
| --- | --- |
| `create` | Introduced a new element |
| `modify` | Changed an existing element |
| `remove` | Removed an existing element |
| `approve` | Endorsed an element without changing it |
| `release` | Committed the Blueprint to released state |
| `revoke` | Negated an earlier mark (§7) |

**R-6.** Engines MUST recognize these six action values. Marks whose
action is none of these and whose action does not begin with an
extension-namespace prefix MUST be rejected.

**R-7.** Extensions MAY introduce additional actions under the `x-`
or `ext.` namespace. Extension actions are advisory: engines that do
not recognize them MUST preserve the mark in the Blueprint but MUST
treat the action as "unknown" for the purposes of workflow logic.

### 4.1 Overlay and `create`

Overlays (see [compound.md](compound.md) §6.3) are authored as new
compounds that reference a base. An overlay therefore produces a
`create` mark targeting the overlay compound itself; no distinct
`overlay` action exists in 0.1.0. The base compound's own marks are
preserved unchanged.

## 5. Targets

The `target` field names the element the mark applies to. It is a
string in one of three forms:

1. The literal string `"blueprint"` — the target is the whole document.
2. A compound identifier — a bare compound name from the `compounds`
   map, e.g. `"kpi-tile"`.
3. A [JSON Pointer](https://www.rfc-editor.org/rfc/rfc6901) rooted at
   the document, starting with `/` — e.g.
   `/compounds/kpi-tile/inputs/value` for a specific input on a
   compound.

**R-8.** The `target` string MUST resolve to an existing element in
the Blueprint at the moment the mark is written. Authoring tools MUST
NOT emit a mark whose target is absent.

**R-9.** When a `remove` mark is written, the target resolution
happens before removal: the target identifies what is being removed.

**R-10.** Engines MUST accept all three target forms. Engines MAY
offer tools for converting between forms for display but MUST preserve
the author's chosen form in the stored document.

## 6. Ordering and history

Marks form an ordered history. The order is the order of entries in
the `marks` array.

**R-11.** The order of entries in the `marks` array MUST be preserved
across reads, writes, and forks. Engines MUST NOT reorder marks on
load or save.

**R-12.** `timestamp` is descriptive only. Two marks with timestamps
out of order relative to array position are still ordered by array
position. Authoring tools SHOULD emit marks whose timestamps are
monotonic with array order, but this is not enforced; clock skew is a
real phenomenon.

A reader reconstructing the history of a Blueprint processes marks in
array order. An `approve` mark is only valid after the `create` mark
whose target it endorses. A `revoke` mark only takes effect from its
array position forward.

## 7. Revocation

Deletion of a mark is not permitted. A mark that needs to be retracted
is superseded by a new `revoke` mark.

A revocation mark has:

- `action`: `revoke`
- `target`: the `id` of the mark being revoked, in JSON Pointer form
  `/marks/<id>`
- `rationale`: SHOULD describe why

**R-13.** Tools presenting a Blueprint's history MUST present both the
revoked mark and its revocation. The revoked mark is not hidden. Its
effect is treated as non-authoritative from the revocation's array
position forward.

**R-14.** A revocation MAY itself be revoked (a second `revoke` mark
targeting the first revocation's `id`). The cycle is resolved by
array position: the latest applicable mark wins.

## 8. Relationship to signatures

A mark MAY carry a `signature` field. The signature commits the
authoring party to the mark's content. Its structure and algorithms
are defined in [signature.md](signature.md).

**R-15.** When a mark carries a `signature`, the signature bytes MUST
be computed over the canonical form of the mark object with its
`signature` field removed (see
[canonical-form.md](../canonical-form.md) §3.2 and §4.2).

**R-16.** The `signature.mark` field of the signature MUST equal the
`id` of the mark the signature covers.

A signed mark is the strongest authorship claim Blueprint offers: the
mark identifies the author structurally; the signature proves the
author's private key was present at authoring; canonical form makes
the signed bytes reproducible.

## 9. Examples

### 9.1 Human author creates a compound

```json
{
  "id": "sha256:a1b2c3...",
  "target": "kpi-tile",
  "author": {
    "kind": "human",
    "id": "person:anton-bursch",
    "display": "Anton Bursch"
  },
  "timestamp": "2026-04-19T10:15:30-04:00",
  "action": "create",
  "rationale": "Initial KPI tile compound."
}
```

### 9.2 Machine author modifies a compound under a human operator

```json
{
  "id": "sha256:d4e5f6...",
  "target": "/compounds/kpi-tile/inputs/value",
  "author": {
    "kind": "machine",
    "id": "machine:claude-sonnet-4.5",
    "machine": {
      "model": "claude-sonnet-4.5",
      "version": "2026-03-15",
      "operator": { "kind": "human", "id": "person:anton-bursch" }
    }
  },
  "timestamp": "2026-04-19T10:17:02-04:00",
  "action": "modify",
  "rationale": "Widened input schema to accept nullable numbers."
}
```

### 9.3 Tool author removes a dead channel

```json
{
  "id": "sha256:7890ab...",
  "target": "/compounds/pipeline/channels/legacy-tap",
  "author": {
    "kind": "tool",
    "id": "tool:blueprint-lint",
    "tool": { "name": "blueprint-lint", "version": "0.9.2" }
  },
  "timestamp": "2026-04-19T10:20:11-04:00",
  "action": "remove",
  "rationale": "Unreachable channel detected by reachability pass."
}
```

### 9.4 Approve

```json
{
  "id": "sha256:ffee11...",
  "target": "kpi-tile",
  "author": { "kind": "human", "id": "person:anton-bursch" },
  "timestamp": "2026-04-19T11:02:00-04:00",
  "action": "approve",
  "rationale": "Reviewed, matches proposal 3/20/2026."
}
```

### 9.5 Release

```json
{
  "id": "sha256:cafe22...",
  "target": "blueprint",
  "author": { "kind": "human", "id": "person:anton-bursch" },
  "timestamp": "2026-04-19T11:05:00-04:00",
  "action": "release",
  "rationale": "Approved for deployment."
}
```

### 9.6 Revocation

```json
{
  "id": "sha256:beef33...",
  "target": "/marks/sha256:d4e5f6...",
  "author": { "kind": "human", "id": "person:anton-bursch" },
  "timestamp": "2026-04-19T11:30:00-04:00",
  "action": "revoke",
  "rationale": "Schema change was incorrect; reverting authorship attribution."
}
```

## 10. Normative requirements (summary)

1. Marks live in the top-level `marks` array (R-1).
2. Marks are immutable once written (R-2).
3. Engines accept both UUID and `sha256:` mark ids (R-3).
4. Authoring tools set `kind` explicitly (R-4).
5. Readers treat `kind` as authoritative (R-5).
6. Engines recognize the six core actions (R-6).
7. Extension actions are preserved but treated as unknown (R-7).
8. Targets MUST resolve at write time (R-8).
9. `remove` resolves its target before removal (R-9).
10. Engines accept all three target forms (R-10).
11. Mark array order is preserved across reads and writes (R-11).
12. Timestamps are descriptive; array position orders history (R-12).
13. Tools show both the revoked mark and its revocation (R-13).
14. Revocations may be revoked; the latest applicable mark wins (R-14).
15. Mark signatures sign canonical form with `signature` removed (R-15).
16. `signature.mark` equals the covered mark's `id` (R-16).

## 11. Open questions (deferred)

- **Scoped mark histories.** Currently all marks live in the top-level
  `marks` array. Whether marks may live alongside a compound (for
  compound-local history) is deferred.
- **Cross-Blueprint mark references.** A mark in one Blueprint that
  references a mark in an imported Blueprint requires a URI-qualified
  target. Out of scope for 0.1.0.
- **Extension action discovery.** How engines advertise recognized
  extension actions and how Blueprints declare required actions is
  deferred to the extension contract document.
- **Mark compaction.** Long-lived Blueprints accumulate many marks.
  Whether a signed "snapshot mark" may archive prior history without
  deletion is an open question.
- **Organizational identity.** Organizations as first-class `author.kind`
  (rather than represented as humans or machines with organizational
  id) is deferred.
