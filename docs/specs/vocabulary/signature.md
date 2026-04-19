# Signature

**Status:** Draft — placeholder. Content not yet written.

Part of the [Blueprint specification](../overview.md), version 0.1.0-draft.

## Purpose

Defines signatures: cryptographic proofs of authorship and integrity over
a Blueprint artifact or a specific mark. Signatures are produced over the
canonical form of the signed element. Blueprint uses signatures to make
authorship claims verifiable and to make released Blueprints tamper-evident.

## Dependencies

- [mark](mark.md)
- [canonical-form](../canonical-form.md)

## Planned sections

1. What a signature is
2. Signature structure (algorithm, public key, signature bytes, signed-over)
3. Supported algorithms
4. Key identity and key rotation
5. Signing targets (whole Blueprint, specific marks, specific compounds)
6. Verification procedure
7. Relationship to canonical form
8. Signature revocation
9. Trust model and out-of-scope concerns
10. Normative requirements
11. Examples
12. Open questions

---

*Scaffolding placeholder. See [ROADMAP.md](../ROADMAP.md) for authoring order.*
