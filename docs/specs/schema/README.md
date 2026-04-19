# Blueprint JSON Schemas

Machine-readable schema definitions for Blueprint artifacts and their
components. These files are the canonical, executable form of the
structural rules described in the [vocabulary](../vocabulary/) documents.

Every validator, engine, and authoring tool MUST accept documents that
validate against `blueprint.schema.json` as well-formed. A Blueprint may
still fail deeper validity checks defined in the normative prose
(see [blueprint.md](../vocabulary/blueprint.md) §10).

## Scope

These schemas encode **well-formedness**: structural shape, required
fields, enumerated values, the extension-namespace passthrough rule.
They do not — and cannot — encode **validity**: semantic rules that
require resolving one part of a Blueprint against another, or
resolving it against a registry. The division is intentional and
mirrors `blueprint.md` §10.

Rules encoded:
- top-level field set and required members
- closed-object rule with `^(x-|ext\.)` extension passthrough
- mark `action` enum (six closed values)
- structured `author` with kind-dependent required sub-objects
- signature `algorithm` (three named algorithms plus extension namespace)
- signature `mark` references either `"blueprint"` or a mark `id`
- import declarations require alias, ref, and pin
- status enum (`draft` or `released`)

Rules not encoded (engine- or registry-level):
- the `entry` field names a composed compound with no parameters
- a signature's `mark` actually resolves to a mark in the same document
- same `id` with different canonical form across a registry
- imports resolve and match their pin
- mark `id` collisions within a Blueprint

## Files

| File | Purpose |
|---|---|
| [blueprint.schema.json](blueprint.schema.json) | Top-level Blueprint artifact |
| [mark.schema.json](mark.schema.json) | Mark structure and author shape |
| [signature.schema.json](signature.schema.json) | Signature structure |

## Dialect

All schemas declare `$schema` as JSON Schema draft 2020-12. No
custom keywords are introduced. No keyword semantics are modified.

## `$id` and resolution

Each schema declares a stable `$id` under
`https://specs.blueprint.build/0.1.0-draft/schema/`. The files are
self-contained: `blueprint.schema.json` references `mark.schema.json`
and `signature.schema.json` by relative `$ref`, so all three can be
validated offline from a single directory.

## Status

Authored in Round 4 (`v0.1.0-draft-round4`). Tracks the vocabulary as
of `v0.1.0-draft-round3`. The internal shape of a compound
declaration is deliberately unlocked in 0.1.0-draft; it is finalized
in Round 5 semantics.
