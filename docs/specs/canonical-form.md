# Canonical form

**Status:** Draft — placeholder. Content not yet written.

Part of the [Blueprint specification](overview.md), version 0.1.0-draft.

## Purpose

Defines the canonical serialization of a Blueprint document: byte-for-byte
rules for key ordering, number formatting, string escaping, whitespace, and
Unicode normalization. Canonical form is the substrate over which hashes are
computed and signatures are produced. Without a canonical form, two semantically
identical Blueprints could produce different signatures.

## Planned sections

1. Scope of canonical form
2. Member ordering
3. Number formatting
4. String escaping and Unicode normalization
5. Whitespace
6. Handling of `null`, absent members, and optional fields
7. Hashing
8. Signing procedure
9. Normative requirements
10. Examples
11. Test vectors

---

*Scaffolding placeholder. See [ROADMAP.md](ROADMAP.md) for authoring order.*
