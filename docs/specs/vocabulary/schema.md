# Schema

**Status:** Draft
**Version:** 0.1.0-draft
**Editor:** Anton Bursch

Part of the [Blueprint specification](../overview.md), version 0.1.0-draft.

> This document defines how Blueprint uses **schemas** to describe the
> typed shape of every value that crosses an interface, flows on a
> channel, or is stored as state. Blueprint uses JSON Schema as its
> schema language. This document fixes the dialect, the role schemas
> play, the distinction between logical shape and physical
> representation, and the validation contract engines MUST honor.

## Conformance language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this
document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119)
and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when,
they appear in all capitals.

## Dependencies

- [compound.md](compound.md) — every interface member is declared with a schema
- [channel.md](channel.md) — channel payloads validate against schemas
- [state.md](state.md) — state fields are declared with schemas
- [../overview.md](../overview.md) — the surrounding specification

---

## 1. Role of schemas

A **schema** is the typed shape of data. Every place in Blueprint
where data crosses a boundary or persists over time, that data has a
declared schema:

- every interface member of a compound — parameter, input, output,
  exposed state — has a schema;
- every internal state field has a schema;
- every internal channel has a schema;
- the Blueprint document itself has a schema (the Blueprint JSON
  Schema, defined in Round 4).

Schemas are the vocabulary through which compounds communicate what
they need and what they produce, independently of any particular
engine, runtime, or programming language. A compound's interface is
nothing more than names paired with schemas.

Schemas also make Blueprints **agent-legible**: a reasoning system —
human or machine — can determine whether two compounds compose, what
a compound expects, and what kind of values it produces, by inspecting
schemas alone.

---

## 2. The schema language

Blueprint uses **[JSON Schema](https://json-schema.org/) draft 2020-12**
as its schema language.

Every schema in a Blueprint MUST be a valid JSON Schema 2020-12
document. Engines MUST validate schemas according to the 2020-12
specification. A schema that does not validate against the meta-schema
for 2020-12 is invalid; an engine MUST reject a Blueprint that
contains an invalid schema.

JSON Schema is used as-is, without profile or dialect modification.
This document adds no keywords, removes no keywords, and alters no
validation rules.

### 2.1 Why JSON Schema

The choice is not arbitrary. JSON Schema is:

- the prevailing standard for describing JSON data shapes;
- widely implemented in every major language;
- already used by OpenAPI, AsyncAPI, and adjacent specifications,
  so Blueprint interoperates with an existing ecosystem of validators,
  code generators, and documentation tools;
- rich enough to describe unions, recursive structures, references,
  formats, and constraints;
- extensible through `$defs`, `$ref`, and namespaced keywords
  (see §4).

### 2.2 Why not a custom schema language

Blueprint does not invent a schema language. A custom language would
duplicate existing work, fragment the ecosystem, and give authors and
agents one more artifact to learn. JSON Schema is the least-effort,
most-interoperable, best-documented choice.

---

## 3. Logical shape, not physical representation

A schema describes the **logical shape** of a value. It does not
prescribe any particular runtime representation.

A value **validates** against a schema when its logical structure
satisfies the schema's constraints, regardless of how the value is
currently represented — a JavaScript object, a Python dict, a Kotlin
data class, a Go struct, a binary buffer, a serialized byte stream on
a wire, or anything else. Representation is an engine concern;
validation is a specification concern.

This distinction is the reason the same Blueprint document can run on
engines written in different languages, targeting different runtimes,
with different performance characteristics, and still produce
equivalent observable behavior.

### 3.1 Canonical form is separate

A schema describes a value's shape. [canonical-form.md](canonical-form.md)
(forthcoming in this round) describes the canonical serialization used
for the Blueprint document itself and for signed content. Schemas do
not dictate canonical form, and canonical form does not modify
schemas. The two documents occupy orthogonal concerns.

### 3.2 Wire encoding is an engine concern

When values travel across process or host boundaries, they require a
wire encoding. The wire encoding MUST preserve validation: a value
that validates before encoding MUST validate after decoding. JSON is
the default wire encoding; engine pairs MAY negotiate more compact
encodings (MessagePack, CBOR, Protobuf, Avro, Arrow) if round-trip
validation is preserved. The choice of wire encoding is not part of
the schema.

See [channel.md](channel.md) for the runtime rules governing in-memory,
streaming, and cross-process channels.

---

## 4. Schema identity, reference, and reuse

Schemas may be declared **inline** at the place they are used, or
declared once and **referenced** by name.

### 4.1 Inline schemas

An inline schema is written as a JSON Schema object at the place of
use. For example, a compound's parameter declaration may contain an
inline schema directly:

```json
{
  "parameters": {
    "step": { "schema": { "type": "integer", "minimum": 0 } }
  }
}
```

Inline schemas MUST be complete JSON Schema documents. Inline schemas
MAY use JSON Schema's `$defs` and `$ref` facilities for internal
structure.

### 4.2 Top-level schemas

A Blueprint MAY declare a top-level `schemas` object holding schemas
keyed by name. Each named schema is addressable within the Blueprint
by reference. For example:

```json
{
  "schemas": {
    "Point": { "type": "object", "properties": { "x": { "type": "number" }, "y": { "type": "number" } }, "required": ["x", "y"] }
  },
  "compounds": {
    "Chart": {
      "interface": {
        "inputs": {
          "series": { "schema": { "type": "array", "items": { "$ref": "#/schemas/Point" } } }
        }
      }
    }
  }
}
```

The `$ref` value `#/schemas/Point` uses JSON Schema's native reference
resolution. Engines MUST resolve `$ref` values relative to the
Blueprint document's root, per JSON Schema 2020-12.

### 4.3 Imported schemas

A Blueprint MAY use schemas from an imported Blueprint. Schema
references into an imported Blueprint use the qualified form:

```json
{ "$ref": "imports:<alias>/schemas/<name>" }
```

where `<alias>` is the import alias declared at the top of the
Blueprint (see [compound.md](compound.md) §6.2) and `<name>` is the
schema's name in the imported Blueprint's top-level `schemas` object.
The exact syntax of the qualified reference is finalized by the
Blueprint JSON Schema in Round 4; the requirement here is that such
references MUST be resolvable at load time using only the importing
Blueprint's declarations and the imported artifact's content.

### 4.4 External schemas by URI

A `$ref` value MAY be an absolute URI pointing to a schema outside the
Blueprint. Engines MAY dereference such URIs. Engines MUST NOT require
network access at runtime: a Blueprint using external URI references
MUST be loadable offline when the referenced schemas are supplied
through an out-of-band mechanism the engine supports (cache, bundle,
sidecar file). The specifics of offline resolution are engine-defined.

This document does not prescribe a schema registry. A Blueprint
intended to run offline SHOULD prefer inline or top-level schemas over
external URI references.

---

## 5. Validation

Values validate against schemas at declared boundaries. An engine MUST
validate:

- every parameter value when a compound is instantiated;
- every message placed on or received from a channel whose validation
  is REQUIRED (see §5.1);
- every state field when it is initialized and, on change, if the
  engine's snapshot policy requires it.

A value that does not validate against its declared schema is invalid.
An engine MUST reject an invalid value and MUST report a diagnostic
identifying the offending field and the violated constraint.

### 5.1 When validation is REQUIRED vs. when it MAY be elided

Validation is REQUIRED at boundaries where an unchecked value would
escape the producer's type system:

- cross-process channels — REQUIRED;
- cross-scope channels whose producer cannot statically prove
  conformance — REQUIRED;
- compound parameter binding at instantiation — REQUIRED;
- any value entering a scope from an external source (connection
  input, adapter input, host-supplied input) — REQUIRED.

Validation MAY be elided where the engine can prove conformance from
the producer's static type system:

- internal channels whose producer emits a typed value that the
  engine's type system already guarantees satisfies the schema.

Even where validation MAY be elided at runtime, the schema itself
remains authoritative: a value whose schema-conformance cannot be
established (statically or dynamically) MUST be treated as invalid.

### 5.2 Diagnostics

When rejecting an invalid value, an engine MUST produce a diagnostic
that identifies:

- the schema that was violated (by document location, reference, or
  name);
- the value's location within its containing structure (by JSON
  Pointer where applicable);
- the violated constraint (minimally, the JSON Schema keyword that
  failed).

Tools SHOULD surface these diagnostics to authors in an
author-friendly form. The diagnostic format is not fully normatively
fixed in 0.1.0-draft.

---

## 6. Binary and streaming payloads

Not every value that flows on a channel is naturally a JSON object.
Audio buffers, video frames, tensors, images, and raw byte sequences
are values that pass in binary form at runtime. Their schemas still
describe their logical shape.

Binary and streaming schemas use JSON Schema's standard facilities:

- **`contentMediaType`** — the MIME type of the payload
  (e.g., `"audio/wav"`, `"image/png"`, `"application/vnd.tensor"`).
- **`contentEncoding`** — the encoding of the payload when serialized
  (e.g., `"base64"`, `"binary"`).
- **`contentSchema`** — a nested schema describing structure when the
  content is itself structured (JSON Schema 2020-12 extension).

Example:

```json
{
  "type": "string",
  "contentMediaType": "audio/wav",
  "contentEncoding": "binary"
}
```

Blueprint MAY register extension keywords under a namespaced prefix
(see §7) for format-specific metadata (for example, tensor shape and
dtype, video codec parameters, image dimensions). Such extensions are
documented separately and do not alter JSON Schema's validation
semantics.

Channels carrying binary payloads behave the same as any other channel
with respect to encapsulation, direction, scope, and delivery
semantics ([channel.md](channel.md)). Their payloads pass in their
native binary form at runtime; validation checks the payload's
conformance to the schema's logical shape.

---

## 7. Extensions

JSON Schema 2020-12 permits vocabulary extensions. Blueprint extensions
that add schema keywords MUST:

- declare the keyword under a namespaced prefix recognizable as an
  extension (the extension prefix convention is normatively fixed in
  [../extensions.md](../extensions.md), forthcoming Round 6);
- specify validation behavior explicitly (either as additional
  validation or as annotation-only);
- not alter the validation behavior of standard JSON Schema keywords.

An engine MUST treat an unknown extension keyword as an annotation
(no-op) unless it has explicitly opted in to the extension.

This mirrors OpenAPI's `x-*` convention and JSON Schema's own
extensibility model. It lets ecosystem authors carry tool-specific
metadata on schemas without invalidating those schemas for tools that
do not understand the extension.

---

## 8. Normative requirements

A conforming Blueprint document:

1. MUST declare every interface member, state field, and channel with
   a schema.
2. MUST provide only JSON Schema 2020-12 documents where schemas are
   required.
3. MUST make every `$ref` resolvable using the Blueprint's declarations,
   its declared imports, or out-of-band resources the engine supports.
4. MUST NOT alter the validation behavior of standard JSON Schema
   keywords through extensions.

A conforming engine:

5. MUST validate schemas against the JSON Schema 2020-12 meta-schema
   on load, and MUST reject Blueprints containing invalid schemas.
6. MUST validate parameter values at instantiation.
7. MUST validate values crossing REQUIRED validation boundaries (§5.1).
8. MUST produce diagnostics identifying the violated schema, the
   value location, and the failed constraint when rejecting an
   invalid value.
9. MUST resolve `$ref` references according to JSON Schema 2020-12.
10. MUST preserve validation across wire encoding: a value that
    validated before encoding MUST validate after decoding.
11. MUST treat unknown extension keywords as annotations unless the
    engine opts in to the extension.

A conforming authoring tool:

12. MUST NOT produce a Blueprint containing a schema that does not
    validate against the JSON Schema 2020-12 meta-schema.

---

## 9. Examples

### 9.1 A scalar parameter schema

```json
{ "type": "integer", "minimum": 1, "maximum": 3600 }
```

### 9.2 A structured channel schema

```json
{
  "type": "object",
  "properties": {
    "timestamp": { "type": "string", "format": "date-time" },
    "sensor_id": { "type": "string" },
    "reading":   { "type": "number" }
  },
  "required": ["timestamp", "sensor_id", "reading"],
  "additionalProperties": false
}
```

### 9.3 A reusable schema referenced from multiple compounds

```json
{
  "schemas": {
    "GeoPoint": {
      "type": "object",
      "properties": {
        "lat": { "type": "number", "minimum": -90, "maximum": 90 },
        "lng": { "type": "number", "minimum": -180, "maximum": 180 }
      },
      "required": ["lat", "lng"]
    }
  },
  "compounds": {
    "WeatherCard":  { "interface": { "parameters": { "location": { "schema": { "$ref": "#/schemas/GeoPoint" } } } } },
    "MapMarker":    { "interface": { "parameters": { "at":       { "schema": { "$ref": "#/schemas/GeoPoint" } } } } }
  }
}
```

### 9.4 A binary payload schema

```json
{
  "type": "string",
  "contentMediaType": "image/png",
  "contentEncoding": "binary"
}
```

At runtime this value is a PNG byte buffer. Its logical shape is
"an opaque binary value whose MIME type is `image/png`". An engine
delivering this value across a cross-process channel MUST preserve its
bytes.

---

## 10. Open questions

1. **Schema registry.** A public registry of reusable Blueprint schemas
   (weather observations, financial tickers, common user types) would
   accelerate authorship. Not in 0.1.0; likely an ecosystem concern.
2. **Named validation profiles.** A Blueprint MAY want to declare
   "validate strictly here; validate loosely there". Not in 0.1.0.
3. **Static type system integration.** Languages with expressive type
   systems (Kotlin, Rust, Scala, TypeScript) can generate types from
   schemas. The integration points are tool concerns, not
   specification concerns, but a standard mapping would help
   ecosystem cohesion. Deferred.
4. **Tensor and ML-native primitives.** A standard extension namespace
   for tensor shape, dtype, and framework metadata would help the
   cross-framework cases. Deferred to extensions (Round 6).

---

## 11. Document status

This document is version `0.1.0-draft` of the schema vocabulary
document. It is normative within the matching Blueprint specification
version.

---

*Authored by Anton Bursch. Apache License 2.0.*
