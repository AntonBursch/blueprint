# Blueprint (the artifact)

**Status:** Draft
**Version:** 0.1.0-draft
**Editor:** Anton Bursch

Part of the [Blueprint specification](../overview.md), version 0.1.0-draft.

> This document defines the **Blueprint** itself: the top-level
> artifact, the JSON document that describes an application. All other
> vocabulary terms in this specification — compound, scope, schema,
> channel, state, mark, signature, draft — describe pieces of a
> Blueprint or rules that govern those pieces. This document fixes the
> shape of the whole: the top-level fields, their requirements, the
> rules by which a document is recognized as a Blueprint, and the
> distinction between a well-formed Blueprint and a valid Blueprint.

## Conformance language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**,
**SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this
document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119)
and [RFC 8174](https://www.rfc-editor.org/rfc/rfc8174) when, and only when,
they appear in all capitals.

## Dependencies

- [overview.md](../overview.md) — §3 fixes the top-level shape; this document is its normative elaboration
- [compound.md](compound.md) — Blueprints contain compounds; one is the entry
- [scope.md](scope.md) — Blueprint loading establishes the root scope
- [schema.md](schema.md) — the `schemas` field holds top-level schemas
- [channel.md](channel.md) — channels live inside compounds; none at top level
- [state.md](state.md) — state lives inside compounds; none at top level
- [mark.md](mark.md) — the `marks` array
- [signature.md](signature.md) — the `signatures` array
- [draft.md](draft.md) — the `status` field and the lifecycle
- [canonical-form.md](../canonical-form.md) — identity and signing

## 1. What a Blueprint is

A Blueprint is a single JSON value — specifically, a JSON object —
that describes an application. The object is self-contained: given
its content and the content of its declared imports, a conforming
engine has everything it needs to load, validate, and execute the
application.

A Blueprint is distributed as a single artifact. It may be transmitted
as a `.json` file, as a payload in an HTTP response, as a blob in a
registry, or as a value in a database. It is not a bundle, directory,
or project; its state-of-completion is fully captured by the JSON
bytes that comprise it.

**R-1.** A Blueprint MUST be a JSON object. Scalars, arrays, and other
JSON values are not Blueprints.

**R-2.** The JSON text of a Blueprint MUST be valid UTF-8 and MUST
conform to [RFC 8259](https://www.rfc-editor.org/rfc/rfc8259).

## 2. Top-level fields

A Blueprint object has the following members. This section is the
normative source for their shape and requirements; [overview.md](../overview.md)
§3.1 provides the same table at survey level.

| Field | Requirement | Type | Description |
| --- | --- | --- | --- |
| `blueprint` | MUST | string | Specification version (§3) |
| `id` | MUST | URI string | Stable identifier (§4) |
| `title` | SHOULD | string | Human-readable title |
| `description` | MAY | string | Human-readable description |
| `status` | MAY | string | Lifecycle state — `"draft"` or `"released"` (§8) |
| `compounds` | MUST | object | Compounds defined by this Blueprint (§5) |
| `entry` | MUST | string | Name of the entry compound (§6) |
| `schemas` | MAY | object | Top-level schema definitions (§7.1) |
| `imports` | MAY | array | External Blueprints and registries (§7.2) |
| `marks` | MAY | array | Authorship trace (§8) |
| `signatures` | MAY | array | Cryptographic proofs (§8) |
| `metadata` | MAY | object | Unstructured informational fields (§9) |

**R-3.** A Blueprint object MUST NOT contain top-level members other
than those defined above or those whose keys begin with a registered
extension prefix (`x-`, `ext.`). Engines MUST reject documents with
unrecognized non-extension top-level fields.

**R-4.** A Blueprint MAY contain any combination of registered
extension top-level fields. Extension fields are advisory: engines
that do not recognize an extension MUST preserve the field
round-trip but MUST NOT attribute behavior to it.

## 3. The `blueprint` field

The `blueprint` field carries the version of the specification to
which this document conforms. Its value is a [Semantic Versioning
2.0.0](https://semver.org/) string. Documents targeting a draft
specification carry a `-draft` suffix (for example, `0.1.0-draft`).

**R-5.** The `blueprint` field MUST be the first field an engine
checks. Before processing any other part of the document, the engine
determines whether it supports the declared version.

**R-6.** An engine that does not support the declared version MUST
refuse to process the document and MUST report the version mismatch
with a diagnostic identifying both the declared version and the
versions the engine supports.

**R-7.** A Blueprint whose `blueprint` value carries a `-draft` suffix
MUST NOT be deployed to production. Production engines MUST refuse to
execute such Blueprints.

## 4. The `id` field

The `id` field is a URI that uniquely and stably identifies the
Blueprint. It is the name by which this Blueprint is referenced from
other Blueprints, registries, and deployments.

The `id` SHOULD be a [URN](https://www.rfc-editor.org/rfc/rfc8141)
under a namespace controlled by the Blueprint's author or publisher.
For example:

- `urn:blueprint:conedison:weather-dashboard`
- `urn:blueprint:acme:billing:invoice-approval`

Other URI forms are permitted when the context requires them (HTTPS
URLs for registry-hosted Blueprints, for example).

**R-8.** The `id` MUST be a syntactically valid URI per
[RFC 3986](https://www.rfc-editor.org/rfc/rfc3986).

**R-9.** Within any registry or system that holds Blueprints, two
Blueprints with the same `id` and different canonical forms (see
[canonical-form.md](../canonical-form.md) §3.1) constitute a conflict.
Registries MUST detect and report such conflicts. Engines loading a
Blueprint MUST refuse to proceed if two resolutions of the same `id`
yield different canonical bytes.

**R-10.** The `id` is independent of the `blueprint` version. Two
Blueprints targeting different specification versions MAY share an
`id` if and only if they are successive releases of the same logical
application. Registry policy determines how version is expressed;
§9.1 recommends `metadata.version` for this purpose.

## 5. The `compounds` field

The `compounds` field is an object whose keys are compound names and
whose values are compound declarations. The structure of a compound
declaration is defined in [compound.md](compound.md).

**R-11.** A Blueprint MUST declare at least one compound — the entry
compound (§6). `compounds` MUST NOT be empty.

**R-12.** Compound names MUST be unique within a Blueprint. Engines
MUST reject Blueprints with duplicate compound names.

**R-13.** Compound names SHOULD follow the identifier rules described
in [compound.md](compound.md) §7.1. Names containing characters that
conflict with JSON Pointer escaping
([RFC 6901](https://www.rfc-editor.org/rfc/rfc6901) §3) are permitted
but discouraged; authoring tools that emit such names MUST escape
them correctly when producing mark targets.

## 6. The `entry` field

The `entry` field is the name of the compound in `compounds` that
serves as the Blueprint's root of execution. It is the unique
composed compound the engine instantiates first at load time, whose
scope is the root scope ([scope.md](scope.md) §2).

**R-14.** The `entry` field MUST be the name of a composed compound
declared in `compounds`. Engines MUST reject Blueprints whose `entry`
names a nonexistent compound or a primitive compound.

**R-15.** The entry compound MUST declare no parameters
([compound.md](compound.md) §8). It MAY declare inputs and outputs
exposed at the Blueprint's boundary.

**R-16.** A Blueprint MUST have exactly one `entry`. Multi-entry
Blueprints are not expressible in 0.1.0.

## 7. Schemas and imports

### 7.1 The `schemas` field

The `schemas` field is an object whose keys are schema names and
whose values are JSON Schema 2020-12 declarations, per
[schema.md](schema.md). Top-level schemas are addressable throughout
the Blueprint by `#/schemas/<name>`.

**R-17.** Top-level schemas MUST conform to the rules in
[schema.md](schema.md) §4. Engines MUST validate `schemas` members at
load time.

### 7.2 The `imports` field

The `imports` field is an array of import declarations. Each
declaration gives an alias and a reference (another Blueprint `id`,
a registry URI, or a URL). Imported compounds are addressable as
`imports:<alias>/compounds/<name>` and imported schemas as
`imports:<alias>/schemas/<name>`.

**R-18.** Every entry in `imports` MUST declare a non-empty alias
unique within the Blueprint.

**R-19.** Engines MUST resolve all imports at load time. If any
import cannot be resolved, the load MUST fail with a diagnostic
identifying the failing import.

**R-20.** Imports MUST carry a version pin (per
[compound.md](compound.md) §6.2). Engines MUST fail the load if an
import resolves to a different canonical form than the one against
which the Blueprint was authored.

The detailed shape of import declarations (including resolution
rules, pin format, and registry protocols) is defined in
[compound.md](compound.md) §6.2 and is deliberately light-touch in
0.1.0. A later revision will elaborate; see §12.

## 8. Marks, signatures, status

Marks, signatures, and status together form the Blueprint's
**authorship and lifecycle metadata**. They do not affect runtime
behavior but they govern trust, attribution, and deployability.

### 8.1 The `marks` field

The `marks` field is an array of mark objects per
[mark.md](mark.md).

**R-21.** Every mark in the `marks` array MUST conform to
[mark.md](mark.md). The array's order is the authoritative order of
the Blueprint's history and MUST NOT be altered by engines on load
or save.

### 8.2 The `signatures` field

The `signatures` field is an array of signature objects per
[signature.md](signature.md).

**R-22.** Every signature in the `signatures` array MUST conform to
[signature.md](signature.md) and MUST reference either `"blueprint"`
or a mark `id` present in `marks`.

### 8.3 The `status` field

The `status` field carries the Blueprint's lifecycle state per
[draft.md](draft.md). Absent, it means `"draft"`.

**R-23.** The `status` field, when present, MUST be one of the string
values `"draft"` or `"released"`. No other value is defined in
0.1.0.

**R-24.** A Blueprint with `status: "released"` MUST satisfy all the
release conditions in [draft.md](draft.md) §5.

## 9. The `metadata` field

The `metadata` field is an object whose members carry information
that does not affect runtime behavior: human-readable provenance,
tooling hints, display preferences, version tags, licensing, and so
on.

**R-25.** Engines MUST NOT interpret `metadata` semantically during
load, execution, or verification. Reading `metadata` for display or
for tooling purposes is permitted.

**R-26.** Authoring tools MAY write any members into `metadata`
without registering extensions. Unrecognized metadata members do not
make a Blueprint non-conforming.

### 9.1 Recommended metadata members

| Key | Purpose |
| --- | --- |
| `version` | Application-level version (distinct from `blueprint`) |
| `license` | SPDX license identifier |
| `authors` | Array of display-friendly author strings |
| `repository` | URL of the source repository |
| `homepage` | URL of the project homepage |

These are recommended conventions. None is normative in 0.1.0.

## 10. Well-formedness vs validity

A document is **well-formed** when:

- It is a JSON object (R-1) encoded as valid UTF-8 JSON (R-2).
- It contains all required top-level fields (§2).
- Its top-level fields have the correct types.
- It contains no unrecognized non-extension top-level fields (R-3).

A Blueprint is **valid** when, additionally:

- Its `blueprint` version is supported by the processor (R-5, R-6).
- Its `id` is a valid URI (R-8).
- Its `compounds` contains the `entry` compound (R-14).
- Every compound in `compounds` is well-formed per
  [compound.md](compound.md).
- Every schema in `schemas` validates per JSON Schema 2020-12.
- All imports resolve and match their version pins (R-19, R-20).
- Every mark conforms to [mark.md](mark.md) and every signature to
  [signature.md](signature.md).
- If `status` is `"released"`, the release conditions in
  [draft.md](draft.md) §5 hold.

**R-27.** Engines MUST distinguish well-formedness failures from
validity failures in diagnostics. A well-formed but invalid Blueprint
is still parseable and its fields are still addressable by tools;
an ill-formed document is not.

**R-28.** Engines MUST NOT execute a Blueprint that is not valid in
the sense of §10. Authoring tools MAY operate on well-formed
Blueprints that are not yet valid, for the purpose of bringing them
to validity.

## 11. Examples

### 11.1 Minimal well-formed, valid draft

```json
{
  "blueprint": "0.1.0-draft",
  "id": "urn:blueprint:example:hello",
  "title": "Hello",
  "compounds": {
    "root": {
      "interface": {},
      "implementation": {
        "primitive": "text",
        "content": "Hello, world."
      }
    }
  },
  "entry": "root"
}
```

### 11.2 Minimal released

```json
{
  "blueprint": "0.1.0-draft",
  "id": "urn:blueprint:example:hello",
  "status": "released",
  "title": "Hello",
  "compounds": {
    "root": { /* ... */ }
  },
  "entry": "root",
  "marks": [
    { "id": "sha256:aa11...", "target": "root",
      "action": "create",
      "author": { "kind": "human", "id": "person:anton-bursch" },
      "timestamp": "2026-04-18T14:00:00-04:00" },
    { "id": "sha256:cc33...", "target": "blueprint",
      "action": "release",
      "author": { "kind": "human", "id": "person:anton-bursch" },
      "timestamp": "2026-04-19T11:05:00-04:00" }
  ],
  "signatures": [
    { "id": "sha256:dd44...", "mark": "sha256:cc33...",
      "algorithm": "Ed25519",
      "public_key": "MCowBQYDK2VwAyEA…",
      "signature": "Qm9rR7L…" }
  ],
  "metadata": {
    "version": "1.0.0",
    "license": "Apache-2.0"
  }
}
```

### 11.3 Imports and top-level schemas

```json
{
  "blueprint": "0.1.0-draft",
  "id": "urn:blueprint:example:dashboard",
  "compounds": { "root": { /* ... */ } },
  "entry": "root",
  "schemas": {
    "Reading": {
      "$schema": "https://json-schema.org/draft/2020-12/schema",
      "type": "object",
      "properties": { "value": { "type": "number" } },
      "required": ["value"]
    }
  },
  "imports": [
    {
      "alias": "weather",
      "ref": "urn:blueprint:noaa:weather-primitives",
      "pin": "sha256:9f2b1c…"
    }
  ]
}
```

## 12. Normative requirements (summary)

1. Blueprints are JSON objects (R-1).
2. Encoding is valid UTF-8 JSON per RFC 8259 (R-2).
3. Unrecognized non-extension top-level fields are rejected (R-3).
4. Extensions may carry top-level fields; engines preserve them (R-4).
5. `blueprint` is checked before any other field (R-5).
6. Unsupported versions are rejected with a clear diagnostic (R-6).
7. `-draft` versions MUST NOT be deployed to production (R-7).
8. `id` is a syntactically valid URI (R-8).
9. Same-`id` documents with different canonical forms are conflicts (R-9).
10. `id` is independent of specification version (R-10).
11. `compounds` is non-empty (R-11).
12. Compound names are unique (R-12).
13. Compound names follow identifier rules (R-13).
14. `entry` names a composed compound in `compounds` (R-14).
15. The entry compound has no parameters (R-15).
16. Exactly one `entry` (R-16).
17. Top-level schemas conform to schema.md (R-17).
18. Each import declares a unique alias (R-18).
19. All imports resolve at load time (R-19).
20. Imports carry and honor version pins (R-20).
21. Marks conform to mark.md; array order is preserved (R-21).
22. Signatures conform to signature.md; references resolve (R-22).
23. `status`, when present, is `"draft"` or `"released"` (R-23).
24. `"released"` requires the release conditions in draft.md (R-24).
25. Engines do not interpret `metadata` semantically (R-25).
26. Unknown metadata members do not affect conformance (R-26).
27. Diagnostics distinguish well-formed from valid (R-27).
28. Engines execute only valid Blueprints (R-28).

## 13. Open questions (deferred)

- **Multiple entries.** Applications with multiple root compounds
  (a web surface and a background daemon) are not expressible in
  0.1.0. §6 locks entry to one. Expected to lift in a later version.
- **Registry protocol.** How engines resolve `imports[].ref` when it
  is a registry URI (as opposed to a direct URL or a pinned content
  hash) is not defined here; it is deferred to a separate registry
  specification.
- **Status values.** Beyond `"draft"` and `"released"`, future values
  such as `"deprecated"` or `"archived"` may be added. §8.3 leaves
  room.
- **Cross-Blueprint signatures.** A signature in Blueprint A that
  covers a mark in Blueprint B would require a URI-qualified mark
  reference. Not defined in 0.1.0.
- **Compound identity across Blueprints.** Whether a compound
  published in a registry under a stable name is itself
  content-addressed (like marks) or named-addressed (like §5) is a
  registry concern. Deferred.
- **Explicit `$schema` for the Blueprint document.** The Blueprint's
  own JSON Schema is implied by the `blueprint` field. Whether
  documents may carry an explicit `$schema` pointing to the Blueprint
  JSON Schema is an authoring-tool convenience. Deferred.
