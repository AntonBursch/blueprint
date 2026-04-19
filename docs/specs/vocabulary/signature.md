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
4. Algorithms MUST be implementable symmetrically by human and machine operators
5. Key identity and key rotation
6. Machine signing keys are first-class; a signature from a machine is no weaker than one from a human
7. Signing targets (whole Blueprint, specific marks, specific compounds)
8. Verification procedure
9. Relationship to canonical form
10. Signature revocation (first-class)
11. Trust model and out-of-scope concerns
12. Normative requirements
13. Examples
14. Open questions

The normative content of this document must be faithful to the rationale
established in [rationale/co-authorship.md](../rationale/co-authorship.md)
and relies on the canonical serialization fixed in
[canonical-form.md](../canonical-form.md).

---

*Scaffolding placeholder. See [ROADMAP.md](../ROADMAP.md) for authoring order.*
