# Blueprint JSON Schemas

Machine-readable schema definitions for Blueprint artifacts and their
components. These files are the canonical, executable form of the rules
described in the [vocabulary](../vocabulary/) documents.

Every validator, engine, and authoring tool MUST accept documents that
validate against `blueprint.schema.json` as well-formed. A Blueprint may
still fail deeper validity checks defined in the normative prose.

| File | Purpose | Status |
|---|---|---|
| blueprint.schema.json | Top-level Blueprint artifact | Not yet authored |
| mark.schema.json | Mark structure | Not yet authored |
| signature.schema.json | Signature structure | Not yet authored |

Schemas will be authored in [Phase 3](../ROADMAP.md#phase-3--formalization).
Each schema will be stamped with its `$id` and a stable URL so external
validators can fetch it.
