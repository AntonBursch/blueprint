# Canonical form

**Status:** Draft
**Version:** 0.1.0-draft
**Editor:** Anton Bursch

Part of the [Blueprint specification](overview.md), version 0.1.0-draft.

> This document defines the **canonical form** of a Blueprint document.
> Given a Blueprint, or a designated subset of one, canonical form
> produces a deterministic byte sequence that any conforming
> implementation can reproduce. That byte sequence is what signatures
> sign over and what hash-based identity is computed over. Without a
> canonical form, two semantically identical Blueprints could produce
> different signatures, and integrity guarantees would collapse.

## Conformance language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this
document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119)
and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when,
they appear in all capitals.

## Dependencies

- [overview.md](overview.md) — §1.3 fixes JSON as the carrier
- [vocabulary/mark.md](vocabulary/mark.md) — mark signatures sign over canonical form
- [vocabulary/signature.md](vocabulary/signature.md) — signature verification canonicalizes before verifying

## 1. What canonical form is

Canonical form is a function:

```
canonicalize(value) -> bytes
```

that takes a JSON value — the Blueprint document, a mark object, or a
compound declaration — and returns a byte sequence such that:

1. Two JSON values that are equal under the [RFC 8259](https://www.rfc-editor.org/rfc/rfc8259)
   JSON data model produce the same bytes.
2. Any conforming implementation, in any language, produces the same
   bytes from the same value.
3. The bytes are valid UTF-8 JSON, parseable by any conforming
   JSON parser.
4. The operation is deterministic, total, and cheap.

Canonical form is exclusively about the **document artifact and its
parts**. It is not a runtime wire encoding, not a snapshot format, and
not a schema normalization. Those concerns are defined elsewhere.

## 2. Algorithm

**Canonical form MUST be computed using [RFC 8785 — JSON Canonicalization Scheme (JCS)](https://www.rfc-editor.org/rfc/rfc8785).**

RFC 8785 fixes the following, which this specification adopts without
modification:

- Object members are sorted by the Unicode code-point values of their
  keys, ascending.
- Strings are serialized using the minimal-escape form defined by
  RFC 8259 §7, with the additional escape rules of RFC 8785 §3.2.2.3.
- Numbers are serialized per the ECMAScript `Number.prototype.toString`
  algorithm as incorporated by RFC 8785 §3.2.2.2, which produces a
  single shortest round-trippable representation for every finite
  IEEE-754 double-precision value.
- There is no insignificant whitespace — no spaces, no indentation,
  no trailing newline.
- The output is UTF-8 without a byte-order mark.

This specification does not weaken, extend, or reinterpret RFC 8785.
A value is canonicalized if and only if it is the exact byte sequence
produced by a conforming JCS implementation applied to the value.

**R-1.** Conforming engines and authoring tools MUST implement RFC 8785
canonicalization for the Blueprint values defined in §3.

**R-2.** A conforming implementation MUST NOT apply any transformation
to the input value prior to canonicalization except the signature-field
exclusion rule defined in §4. In particular, an implementation MUST NOT
normalize Unicode text, coerce numbers, or drop fields it does not
recognize.

## 3. Canonicalization targets

Canonical form is defined over three targets:

### 3.1 The whole Blueprint document

The entire top-level Blueprint JSON value. Used for:

- Computing a content-addressed document identity.
- Producing a whole-Blueprint signature (see
  [signature.md](vocabulary/signature.md)).
- Comparing two Blueprints for byte-identical content.

### 3.2 A specific mark

A single mark object as it appears in the `marks` array of a Blueprint.
Used for:

- Producing the signature bytes for a mark (see
  [mark.md](vocabulary/mark.md) and [signature.md](vocabulary/signature.md)).
- Deriving a content-addressed mark `id`.

### 3.3 A specific compound

A single compound declaration as it appears under `compounds`.
Reserved for future use (per-compound signing). Conforming
implementations MUST support canonicalization at this granularity but
are not required to consume the result in 0.1.0.

**R-3.** The three targets in §3.1, §3.2, and §3.3 MUST all use the
same algorithm (RFC 8785). Implementations MUST NOT provide
target-specific variants of canonicalization.

## 4. Signature-field exclusion

A canonical form that *included* the signature bytes over which the
signature was computed would be circular: the signature would depend on
its own value. Canonical form therefore excludes signature fields.

### 4.1 Rule for the whole document

When canonicalizing the whole Blueprint document (§3.1), every
`signature` field, at every level of the document, MUST be removed
before canonicalization. This includes:

- Signature objects inside the top-level `signatures` array.
- Any `signature` field nested inside a mark object in the `marks`
  array.
- Any `signature` field nested inside a compound or any of its
  descendants.

Removal is structural: the field key and its value are both absent from
the canonicalized value.

### 4.2 Rule for a single mark

When canonicalizing a mark (§3.2), the mark object's own `signature`
field MUST be removed before canonicalization. No other field is
removed. The mark's `id` field is retained.

### 4.3 Rule for a single compound

When canonicalizing a compound (§3.3), any `signature` field contained
within the compound or its descendants MUST be removed before
canonicalization. The compound's declaration — interface, body, marks
— is otherwise retained.

**R-4.** Implementations MUST apply the appropriate exclusion rule from
§4.1–§4.3 before invoking the RFC 8785 algorithm. Implementations MUST
NOT modify the signature fields in the source document; the exclusion
is an operation on the value being canonicalized, not on the document
in storage.

**R-5.** Removal MUST be a tree-structural operation: the exact field
key `signature` is removed wherever it appears as an object member.
Extension keywords that happen to contain the substring `signature`
are NOT removed.

## 5. Extensions and unknown keywords

Canonical form treats extension keywords as ordinary JSON values.
A field whose key begins with `x-` or any other extension-namespace
prefix is serialized with the same rules as any other field: its key
participates in object-member sorting, and its value is canonicalized
recursively.

**R-6.** Implementations MUST NOT omit unknown keywords from canonical
form. Unknown keywords participate in canonicalization exactly as known
keywords do.

**R-7.** Implementations MUST NOT treat the extension-namespace prefix
(`x-`, `ext.`) as semantically meaningful during canonicalization.

## 6. What canonical form does NOT cover

The following are out of scope for this document:

- **Runtime channel payloads.** Values flowing over channels at
  runtime are governed by channel wire encoding
  ([channel.md](vocabulary/channel.md) §7), not by canonical form.
- **Snapshots.** A snapshot captures running state, not the Blueprint
  document. Snapshot serialization rules are defined in
  [state.md](vocabulary/state.md) §6. A snapshot MAY be canonicalized
  by engines that choose to sign snapshots, but this is not required
  in 0.1.0.
- **In-memory parsed representations.** An engine's in-memory object
  graph for a loaded Blueprint is an implementation concern. Canonical
  form applies to the serialized artifact.

## 7. Examples

### 7.1 Member ordering

Input (two JSON representations of the same value):

```json
{ "title": "hello", "id": "blueprint:example" }
```

```json
{ "id": "blueprint:example", "title": "hello" }
```

Canonical form (identical for both inputs):

```
{"id":"blueprint:example","title":"hello"}
```

### 7.2 Number serialization

Input:

```json
{ "count": 1.0, "ratio": 0.1 }
```

Canonical form:

```
{"count":1,"ratio":0.1}
```

(Per RFC 8785 §3.2.2.2, `1.0` is canonicalized as `1` because that is
the shortest round-trippable form.)

### 7.3 Signature-field exclusion (whole document)

Input (abridged):

```json
{
  "blueprint": "0.1.0-draft",
  "id": "blueprint:example",
  "marks": [
    { "id": "mark:a", "action": "create", "author": { "kind": "human" }, "signature": { "algorithm": "Ed25519", "signature": "…" } }
  ],
  "signatures": [
    { "id": "sig:root", "mark": "blueprint", "algorithm": "Ed25519", "signature": "…" }
  ]
}
```

Canonical form is computed over a structurally identical value in which
both the `signature` field inside the mark and the entire `signatures`
array entries' `signature` fields have been removed. The `signatures`
array itself is retained (as an array of objects without their
`signature` field), so that its presence, ordering, and the other
metadata on each signature object are themselves covered by the
signature.

### 7.4 Mark canonicalization

Canonical form of a mark object is the mark with its own `signature`
field removed, then canonicalized per RFC 8785. Its `id`, `action`,
`author`, `timestamp`, `target`, `rationale`, and `provenance` fields
are all retained.

## 8. Open questions (deferred)

- **Canonical form of binary values.** JSON Schema 2020-12 permits
  base64-encoded binary via `contentEncoding`. Canonicalization
  operates on the JSON text; the encoded string is canonicalized as a
  string. A future revision may add normalization rules for binary
  content.
- **Length-prefixed canonical form.** Some use cases want a canonical
  form that prefixes its length for streaming verification. Out of
  scope for 0.1.0.
- **Compound-level signing.** §3.3 reserves the target but the
  signature document does not yet define compound signatures. Expected
  in a later version.

## 9. Normative requirements (summary)

1. Engines and authoring tools MUST implement RFC 8785 (R-1).
2. Implementations MUST NOT transform the input beyond signature
   exclusion (R-2).
3. All three canonicalization targets use the same algorithm (R-3).
4. Signature-field exclusion MUST precede canonicalization (R-4).
5. Exclusion is a structural removal of the exact key `signature` (R-5).
6. Unknown keywords participate in canonicalization (R-6).
7. Extension-namespace prefixes are not semantically special (R-7).
